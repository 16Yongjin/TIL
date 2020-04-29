# 하드웨어 인터럽트

이번엔 프로그램 가능한 인터럽트 컨트롤러를 만들어서 CPU에 알맞은 하드웨어 인터럽트를 보낸다. 다른 예외처럼 하드웨어 인터럽트를 처리하기 위해서는 인터럽트 디스크립터 테이블에 시작점을 추가해야한다.
주기적인 타이머 인터럽트를 받는 법과 키보드에서 입력 받는 법을 배운다.

## 개요

인터럽트로 연결된 하드웨어 장치에서 일어난 일을 CPU에 알릴 수 있다. 그래서 커널이 키보드에서 새 입력이 왔는지 주기적으로 확인(폴링 방식)하지 않아도 키보드가 커널에 키입력을 알릴 수 있다. 커널은 인터럽트가 일어났을 때만 작동하면 되므로 매우 효율적이다. 또한, 커널이 즉시 반응을 할 수 있어서 응답시간도 빠르다.

모든 하드웨어 장치를 CPU에 직접 연결할 수는 없다. 대신에 별도의 *인터럽트 컨트롤러*가 모든 장치가 보낸 인터럽트를 모아서 CPU에 알린다.

```
                           ____________             _____
      Timer ------------> |            |           |     |
      Keyboard ---------> | Interrupt  |---------> | CPU |
      Other Hardware ---> | Controller |           |_____|
      Etc. -------------> |____________|
```

인터럽트 컨트롤러는 대부분 프로그램이 가능하므로 인터럽트마다 다른 우선순위를 부여할 수 있다. 예를 들면, 타이머 인터럽트에 키보드 인터럽트보다 높은 우선 순위를 부여해서 시간을 정확하게 셀 수 있도록 한다.

하드웨어 인터럽트는 예외와는 다르게 비동기적으로 발생한다. 즉, 하드웨어 인터럽트는 현재 실행 중인 코드와 독립적으로 발생하고 아무 때나 발생할 수 있다. 이에 따라 커널에 일종의 동시성과 그에 따른 잠재적인 버그가 생긴다. 가변 전역 상태를 금지하는 러스트의 엄격한 소유권 모델의 도움을 받을 수 있지만, 그래도 데드락이 여전히 발생할 수 있다.

## 8259 PIC

인텔 8259는 1976년에 출시된 프로그램 가능한 인터럽트 컨트롤러다. 새로 나온 APIC에 대체됐어도 하위 호환성을 이유로 현재 시스템에서 인터페이스를 지원한다.
APIC보다 설정하기 쉬운 8259 PIC로 인터럽트를 설명하고 나중에 APIC로 변경한다.

8259는 인터럽트 라인 8개와 CPU와 통신하는 라인 여러 개가 있다. 옛날에 쓰던 시스템은 8259 PIC를 두 개 장착해서 주 PIC는 CPU에, 다른 부 PIC는 주 PIC에 연결해서 썼다.

```
                     ____________                          ____________
Real Time Clock --> |            |   Timer -------------> |            |
ACPI -------------> |            |   Keyboard-----------> |            |      _____
Available --------> | Secondary  |----------------------> | Primary    |     |     |
Available --------> | Interrupt  |   Serial Port 2 -----> | Interrupt  |---> | CPU |
Mouse ------------> | Controller |   Serial Port 1 -----> | Controller |     |_____|
Co-Processor -----> |            |   Parallel Port 2/3 -> |            |
Primary ATA ------> |            |   Floppy disk -------> |            |
Secondary ATA ----> |____________|   Parallel Port 1----> |____________|

```

위 그림은 인터럽트 라인을 할당하는 일반적인 방식을 보여준다. 라인 15개 중 대부분이 고정 매핑되어 있다. 예를 들어, 부 PIC의 4번 라인은 마우스에 할당되어 있다.

각 컨트롤러는 "명령" 포트와 "데이터" 포트로 구성된 I/O 포트 2개로 설정이 가능하다. 주 컨트롤러는 `0x20`(명령)과 `0x21`(데이터) 포트를 사용하고 부 컨트롤러는 `0xa0`(명령)과 `0xa1`(데이터) 포트를 사용한다.

### 구현

기본 설정 상태의 PIC는 PIC에 0-15사이의 인터럽트 벡터 번호를 보낸다. 이 번호들은 이미 CPU 예외에 사용되고 있으므로(ex, 8번은 더블 폴트) 기본 설정 상태의 PIC는 사용할 수 없다. 번호가 겹치는 문제를 해결하기 위해 PIC 인터럽트를 다른 번호로 옮겨야 한다. 예외만 피하면 아무 번호대나 사용해도 괜찮지만, 일반적으로 32-47번을 사용한다. 이 번호들이 예외를 담은 32개의 숫자 다음에 처음으로 나오는 사용되지 않는 번호이기 때문이다.

PIC의 커맨드와 데이터 포트에 특수값을 작성해서 설정을 할 수 있다. 다행히도 `pic8259_simple` 크레이트를 이용하면 초기화 순서를 직접 작성하지 않아도 된다.
크레이트가 어떻게 작동하는지는 [이 소스 코드](https://docs.rs/crate/pic8259_simple/0.1.1/source/src/lib.rs)를 확인하면 된다. 길지 않고 문서화가 잘 돼있다.

다음과 같이 크레이트를 추가한다.

```toml
# in Cargo.toml

[dependencies]
pic8259_simple = "0.1.1"
```

이 크레이트는 `ChainedPics` 구조체를 주요 추상화로 제공한다. `ChainedPics` 구조체는 위에서 봤던 주/부 PIC의 레이아웃을 표현하고 다음과 같이 사용된다.

```rust
// src/interrupts.rs 내부

use pic8259_simple::ChainedPics;
use spin;

pub const PIC_1_OFFSET: u8 = 32;
pub const PIC_2_OFFSET: u8 = PIC_1_OFFSET + 8;

pub static PICS: spin::Mutex<ChainedPics> =
    spin::Mutex::new(unsafe { ChainedPics::new(PIC_1_OFFSET, PIC_2_OFFSET) });
```

pic의 오프셋을 설정해서 32-47번을 가지도록 했다. `ChainedPics` 구조체를 `Mutex`로 감쌌기에 `lock` 메서드를 활용하면 안전하게 가변 접근을 할 수 있다. `ChainedPics::new` 함수는 인자로 잘못된 오프셋이 들어올 수 있으므로 안전하지 않다.

이제 `init` 함수에서 8259 PIC를 초기화한다.

```rust
// src/lib.rs 내부

pub fn init() {
    gdt::init();
    interrupts::init_idt();
    unsafe { interrupts::PICS.lock().initialize() }; // 새로 추가
}
```

`initialize`로 초기화를 하는데 `ChainedPics::new` 함수처럼 PIC를 잘못 설정하면 알 수 없는 동작을 할 수 있으므로 `initialize` 메서드는 안전하지 않다.

`cargo xrun`을 실행하면 "It did not crash" 메시지가 나온다.

## 인터럽트 허용하기

CPU 설정에서 인터럽트를 허용하지 않아서 아무 일도 일어나지 않는다. 즉, CPU는 인터럽트 컨트롤러를 전혀 듣고 있지 않아서 인터럽트가 CPU로 못 가고 있다. 다음과 같이 설정을 변경한다.

```rust
// src/lib.rs 내부

pub fn init() {
    gdt::init();
    interrupts::init_idt();
    unsafe { interrupts::PICS.lock().initialize() };
    x86_64::instructions::interrupts::enable();     // 새로 추가
}
```

`x86_64` 크레이트의 `interrupts::enable` 함수는 `sti` 명령어(set interrupts)를 실행해서 외부 인터럽트를 허용한다. `cargo xrun`을 다시 실행해보면 이제는 더블 폴트가 발생한다.

하드웨어 타이머(정확히는 Intel 8253)가 기본적으로 켜져 있기 때문에 더블 폴트가 발생했다. 인터럽트를 허용하자마자 타이머 인터럽트를 받기 시작했다. 타이머 인터럽트를 다룰 처리 함수를 아직 정의하지 않아서 더블 폴트 처리 함수가 실행됐다.

## 타이머 인터럽트 처리하기

위의 그림에서 봤던 것처럼, 타이머는 주 PIC의 0번 선을 사용한다. 그래서 CPU에 32번 인터럽트(0 + 오프셋 32)로 전달된다. 이 숫자를 하드코딩된 인덱스 32 대신에 `InterruptIndex` 열거형에 담는다.

```rust
// src/interrupts.rs 내부

#[derive(Debug, Clone, Copy)]
#[repr(u8)]
pub enum InterruptIndex {
    Timer = PIC_1_OFFSET,
}

impl InterruptIndex {
    fn as_u8(self) -> u8 {
        self as u8
    }

    fn as_usize(self) -> usize {
        usize::from(self.as_u8())
    }
}
```

`InterruptIndex` 열거형은 C의 열거형처럼 각 변형에 직접 인덱스를 특정할 수 있다. `repr(u8)` 속성이 각 변형이 `u8`로 표현되게 한다. 나중에 다른 인터럽트를 담을 변형을 추가할 것이다.

이제 타이머 인터럽트 처러 함수를 추가한다.

```rust
// src/interrupts.rs 내부

use crate::print;

lazy_static! {
    static ref IDT: InterruptDescriptorTable = {
        let mut idt = InterruptDescriptorTable::new();
        idt.breakpoint.set_handler_fn(breakpoint_handler);
        […]
        idt[InterruptIndex::Timer.as_usize()]
            .set_handler_fn(timer_interrupt_handler); // new

        idt
    };
}

extern "x86-interrupt" fn timer_interrupt_handler(
    _stack_frame: &mut InterruptStackFrame)
{
    print!(".");
}
```

`timer_interrupt_handler` 함수는 다른 예외 처리 함수와 시그내처가 같다. CPU가 예외와 외부 인터럽트에 동일하게 반응하기 떄문이다(유일한 차이점은 일부 예외가 에러 코드를 스택에 넣는 것이다). `InterruptDescriptorTable` 구조체는 `IndexMut` 트레잇을 구현해서 배열 인덱스 접근 문법으로 각 시작점에 접근할 수 있다.

정의한 타이머 인터럽트 처리 함수는 화면에 점을 찍는다. 타이머 인터럽트가 주기적으로 발행하므로, 각 타이머 틱마다 점이 생겨야 되는데 아래처럼 점이 하나만 출력됐다.

![single-tick](https://user-images.githubusercontent.com/22253556/80371531-031b1080-88cd-11ea-9d11-5bf576478830.png)

### 인터럽트 종료

PIC는 인터럽트 처리 함수가 명시적인 인터럽트 종료(end of interrupt, EOI) 신호를 보낼거라고 기대하기 때문에 점이 한 개만 찍혔다. EOI 신호는 PIC에게 인터럽트가 처리됐고 시스템이 다음 인터럽트를 받을 준비가 됐다고 말해준다. PIC는 아직 첫 번째 타이머 인터럽트를 처리하는 중이라 생각해서, 다음 인터럽트를 보내기 전에 EOI 신호를 가만히 기다린다.

`PICS` 구조체를 사용해서 EOI를 보낸다.

```rust
// src/interrupts.rs 내부

extern "x86-interrupt" fn timer_interrupt_handler(
    _stack_frame: &mut InterruptStackFrame)
{
    print!(".");

    unsafe {
        PICS.lock()
            .notify_end_of_interrupt(InterruptIndex::Timer.as_u8());
    }
}
```

`notify_end_of_interrupt` 메서드는 주/부 PIC 중 어떤 PIC가 인터럽트를 보낸 지 알아낸 후에 `command`와 `data` 포트로 각 컨트롤러에 EOI 신호를 보낸다. 만약 부 PIC가 인터럽트를 보냈으면 부 PIC가 주 PIC에 연결되있으므로 양 PIC 모두 알아야 한다.

인터럽트 벡터 숫자를 정확히 사용해야 한다. 잘못하면 보내지 않은 인터럽트를 실수로 지워버리거나 시스템이 중지될 수 있다. 이런 이유로 `notify_end_of_interrupt`는 안전하지 않은 함수이다.

이제 `cargo xrun`을 실행하면 주기적으로 화면에 점이 찍힌다.

![image](https://user-images.githubusercontent.com/22253556/80484940-80a95400-8993-11ea-8ee2-48675cefa721.png)

### 타이머 설정하기

여기서 사용한 하드웨어 타이머는 프로그램 가능한 간격 타이머(Progammable Interval Timer, PIT)이다. 이름에서 알 수 있듯이, 인터럽트 사이 간격을 설정할 수 있다. 잠시 뒤에 APIC 타이머를 사용할 예정이라 자세한 건 다루지 않지만 OSDev 위키의 [PIT 설정법](https://wiki.osdev.org/Programmable_Interval_Timer)에 자세한 설명을 찾아볼 수 있다.

## 데드락

현재 커널은 타이머 인터럽트가 비동기적으로 일어나는 일종의 동시성을 가지게 됐다. 그래서 타이머 인터럽트가 언제라도 `_start` 함수를 방해할 수 있다. 다행히도 러스트의 소유권 시스템이 컴파일 타임에 동시성과 관련된 많은 버그를 막아준다. 하지만 데드락은 예외다. 데드락은 스레드가 절대 풀릴 일 없는 락을 얻으려고 할 때 발생하고 이 경우 스레드는 무한정 정지상태에 놓인다.

이미 커널에서 데드락을 일으킨 적이 있다. `println` 매크로는 `vga_buffer::_print`를 호출하면서 스핀락을 사용하는 전역 `WRITER`의 락을 건다.

```rust
// src/vga_buffer.rs 내부

[…]

#[doc(hidden)]
pub fn _print(args: fmt::Arguments) {
    use core::fmt::Write;
    WRITER.lock().write_fmt(args).unwrap();
}
```

위 코드는 `WRITER`에 락을 걸고 거기에 `write_fmt`를 호출한다. 그리고 함수가 끝나면 암시적으로 락을 푼다. `WRITER`가 락 상태일 때 인터럽트가 발생해서 인터럽트 처리 함수가 무언가를 출력하는 경우를 생각해본다:

| Timestep | \_start                      | interrupt_handler                                    |
| -------- | ---------------------------- | ---------------------------------------------------- |
| 0        | `println!` 호출              |                                                      |
| 1        | `print`가 `WRITER`에 락 시킴 |                                                      |
| 2        |                              | 인터럽트가 발생하고, 처리 함수가 작동하기 시작함     |
| 3        |                              | `println!` 실행                                      |
| 4        |                              | `print`가 이미 락이 걸린 `WRITER`의 락을 얻으려고 함 |
| 5        |                              | `print`가 이미 락이 걸린 `WRITER`의 락을 얻으려고 함 |
| …        |                              | …                                                    |
| 불가능   | `WRITER`의 락이 풀림         |                                                      |

`WRITER`가 락이 된 상태이므로 인터럽트 처리 함수는 `WRITER`의 락이 풀릴 때까지 기다린다. 그러나, `_start` 함수는 인터럽트 처리 함수가 반환된 후에 실행을 계속하기에 락이 절대 풀리지 않는다. 그래서 전체 시스템이 중지된다.

### 데드락 발생시키기

`_start` 함수 끝에 있는 루프에서 출력하면 커널이 데드락에 걸리게 할 수 있다.

```rust
// src/main.rs 내부

#[no_mangle]
pub extern "C" fn _start() -> ! {
    […]
    loop {
        use blog_os::print;
        print!("-");        // new
    }
}
```

커널을 실행시키면 다음과 같이 나온다.

![image](https://user-images.githubusercontent.com/22253556/80589559-6172fb00-8a55-11ea-8ea4-b08ce6111387.png)

타이머 인터럽트가 처음 발생하기 전까지만 밑줄이 출력된다. 그런다음 타이머 인터럽트 처리 함수가 점을 출력하려고해서 시스템이 멈춘다. 위 그림에서 점이 찍혀있지 않은 이유다.

타이머 인터럽트가 비동기적으로 일어나므로 밑줄이 출력되는 개수는 실행할 때마다 다르다. 이러한 비결정적 특성이 동시성 관련 버그를 잡기 힘들게 한다.

### 데드락 방지하기

데드락을 피하기 위해 `Mutex`가 락 상태인 경우 인터럽트를 끌 수 있다.

```rust
// src/vga_buffer.rs 내부

/// 전역 `WRITER` 인스턴스를 사용해서
/// VGA 텍스트 버퍼에 입력받은 포맷된 문자열을 출력한다.
#[doc(hidden)]
pub fn _print(args: fmt::Arguments) {
    use core::fmt::Write;
    use x86_64::instructions::interrupts;   // 추가

    interrupts::without_interrupts(|| {     // 추가
        WRITER.lock().write_fmt(args).unwrap();
    });
}
```

`without_interrupts` 함수는 클로저를 인자로 받아서 인터럽트가 없는 환경에서 실행한다. `without_interrupts` 함수로 `Mutex`가 락 상태일 때 인터럽트 발생을 막았다. 커널을 실행해보면 멈추는 일이 없이 계속해서 동작한다. (루프 안의 출력이 너무 빨라서 점은 볼 수 없다. 루프에 `for _ in 0..10000 {}`를 넣어서 출력 속도를 늦추면 된다.)

시리얼 출력 함수에도 똑같이 데드락 방지 코드를 추가한다.

```rust
// src/serial.rs 내부

#[doc(hidden)]
pub fn _print(args: ::core::fmt::Arguments) {
    use core::fmt::Write;
    use x86_64::instructions::interrupts;       // 추가

    interrupts::without_interrupts(|| {         // 추가
        SERIAL1
            .lock()
            .write_fmt(args)
            .expect("Printing to serial failed");
    });
}
```

인터럽트를 끈다고 모든 문제가 해결되지는 않는다. 인터럽트 레이턴시, 즉 시스템이 인터럽트에 반응하는 시간이 늘어나는 문제가 생긴다. 그래서 매우 짧은 시간 동안만 인터럽트를 꺼야한다.

## 경쟁 조건 방지하기

`cargo xtest`를 실행하면 `test_println_output` 테스트가 실패한다.

```
> cargo xtest --lib
[…]
Running 4 tests
test_breakpoint_exception...[ok]
test_println... [ok]
test_println_many... [ok]
test_println_output... [failed]

Error: panicked at 'assertion failed: `(left == right)`
  left: `'.'`,
 right: `'S'`', src/vga_buffer.rs:205:9
```

테스트와 타이머 사이에 경쟁 조건이 발생해서 생긴 문제이다. 테스트는 다음과 같다.

```rust
// src/vga_buffer.rs 내부

#[test_case]
fn test_println_output() {
    serial_print!("test_println_output... ");

    let s = "Some test string that fits on a single line";
    println!("{}", s);
    for (i, c) in s.chars().enumerate() {
        let screen_char = WRITER.lock().buffer.chars[BUFFER_HEIGHT - 2][i].read();
        assert_eq!(char::from(screen_char.ascii_character), c);
    }

    serial_println!("[ok]");
}
```

테스트는 VGA 버퍼에 출력한 뒤 `buffer_chars`를 직접 반복해서 출력을 확인한다.
`println`과 화면 출력 문자를 읽는 사이에 타이머 인터럽트 처리 함수가 실행돼서 경쟁 조건이 발생했다. 한편, 이 경쟁 조건이 그렇게 위험하지 않아서 컴파일 타임에 제거되지 않았다. 이와 관련된 내용은 [러스트노미콘](https://doc.rust-lang.org/nomicon/races.html)에 나와있다.

타이머 처리 함수가 중간에 `.`을 못 찍도록 테스트가 진행되는 동안 `WRITER`를 락 상태에 놓아서 경쟁 상태를 방지할 수 있다. 수정된 테스트는 다음과 같다.

```rust
// src/vga_buffer.rs 내부

#[test_case]
fn test_println_output() {
    use core::fmt::Write;
    use x86_64::instructions::interrupts;

    serial_print!("test_println_output... ");

    let s = "Some test string that fits on a single line";
    interrupts::without_interrupts(|| {
        let mut writer = WRITER.lock();
        writeln!(writer, "\n{}", s).expect("writeln failed");
        for (i, c) in s.chars().enumerate() {
            let screen_char = writer.buffer.chars[BUFFER_HEIGHT - 2][i].read();
            assert_eq!(char::from(screen_char.ascii_character), c);
        }
    });

    serial_println!("[ok]");
}
```

다음의 변화를 줬다.

- 명시적으로 `lock()` 메서드를 호출해서 테스트가 완료될 때까지 `WRITER`에 락이 걸리게 했다. `println` 대신에 `writeln` 매크로를 사용해서 락을 통해 얻은 `writer`로 출력이 가능하도록 했다.
- 다른 데드락을 방지하기 위해, 테스트가 진행되는 동안 인터럽트를 껐다. 이렇게 안하면 `WRITER`에 락이 걸린 동안 테스트가 방해된다.
- 테스트 실행 전에 타이머 인터럽트 처리함수가 실행될 수 있으므로, 문자열 `s`를 출력하기 전에 줄바꿈 `\n`을 추가했다. 이를 통해 타이머 처리 함수가 현재 줄에 `.`을 추가해서 테스트가 실패하는 일을 막았다.

위의 변경사항을 반영하고 `cargo xtest`를 실행하면 테스트가 결정적으로 성공한다.

전에 발생했던 경쟁 조건은 크게 위험하지 않아서 고작 테스트만 실패시켰다. 다른 경쟁 조건 같았으면 비결정적 특성때문에 디버깅하기 매우 어려웠을 것이다. 다행히도 러스트는 정의되지 않은 동작(시스템 충돌이나 메모리 손상)을 일으킬 수 있는 심각한 경쟁 조건은 막아준다.

## `hlt` 명령어

지금까지 `_start`와 `panic` 함수 끝에 빈 루프를 사용해서 CPU가 영원히 작동하게 했다. 그러나, 이런 루프는 아무 일도 안 하는데 CPU를 최고속도로 돌게해서 비효율적이다. 커널을 실행했을 떄 작업 관리자를 보면 QEMU 프로세스가 전체 시간동안 CPU를 거의 100% 사용한다.

`hlt` 명령어는 다음 명령어 전까지 CPU를 멈춰서 에너지 소모가 적은 슬립 상태로 들어갈 수 있게 한다. `hlt` 명령어를 사용해서 에너지 효율적인 무한 루프를 만든다:

```rust
// src/lib.rs 내부

pub fn hlt_loop() -> ! {
    loop {
        x86_64::instructions::hlt();
    }
}
```

`instructions::hlt` 함수는 `hlt` 어셈블리 명령어를 [얇게 감싼다](https://github.com/rust-osdev/x86_64/blob/5e8e218381c5205f5777cb50da3ecac5d7e3b1ab/src/instructions/mod.rs#L16-L22). `hlt` 함수는 메모리 안정성을 해칠 방법이 없기에 안전하다.

`_start`와 `_panic`에 무한 루프 대신에 `hlt_loop`를 사용한다.

```rust
// src/main.rs 내부

#[no_mangle]
pub extern "C" fn _start() -> ! {
    […]

    println!("It did not crash!");
    blog_os::hlt_loop();            // new
}

#[cfg(not(test))]
#[panic_handler]
fn panic(info: &PanicInfo) -> ! {
    println!("{}", info);
    blog_os::hlt_loop();            // new
}
```

`lib.rs`도 변경한다.

```rust
// src/lib.rs 내부

/// Entry point for `cargo xtest`
#[cfg(test)]
#[no_mangle]
pub extern "C" fn _start() -> ! {
    init();
    test_main();
    hlt_loop();         // new
}

pub fn test_panic_handler(info: &PanicInfo) -> ! {
    serial_println!("[failed]\n");
    serial_println!("Error: {}\n", info);
    exit_qemu(QemuExitCode::Failed);
    hlt_loop();         // new
}
```

이제 커널을 실행해보면, CPU 사용량이 줄어든 것을 볼 수 있다.
