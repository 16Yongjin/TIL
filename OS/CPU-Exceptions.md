# CPU 예외 처리

CPU 예외는 잘못된 메모리 접근이나 0으로 나누기 같은 에러 상황에서 발생한다.

예외처리를 위해 인터럽트 디스크립터 테이블(IDT)에 처리함수들을 준비해놓는다.

커널이 중단점 예외를 처리하고 다시 일반적인 실행 상태로 돌아오게 만들어본다.

## 개요

현재 실행 중인 명령어에 문제가 있으면 예외 신호가 발생한다. (ex, 0으로 나누기) 예외 발생 시, CPU는 하고있는 작업을 멈추고 예외에 맞는 처리함수를 즉시 호출한다.

x86에는 CPU 예외가 20개 정도 있는데, 다음에 나오는 것들이 가장 중요하다.

- 페이지 폴트: 메모리에 잘못 접근하면 발생한다. 현재 명령어가 매핑되지 않은 페이지를 읽거나 읽기 전용 페이지를 읽으려고 할 때 발생한다.

- 유효하지 않은 Opcode: 잘못된 명령어 실행 시 발생한다. 새 SSE 명령어를 지원되지 않는 오래된 CPU에 사용했을 때 발생한다.

- 일반 보호 장애(General Protection Fault): 다양한 원인으로 발생한다. 유저 레벨에서 권한 없는 명령어를 실행하거나 설정 레지스터의 보호 필드에 쓰기를 하려고 하면 발생한다.

- 더블 폴트: 예외 처리함수 실행 중에 예외 발생 시 CPU는 더플 폴트 예외를 발생시킨다. 예외를 처리할 함수가 없을 때 발생한다.

- 트리플 폴트: 더플 폴트 처리함수 실행 중에 예외 발생 시 치명적인 트리플 폴트 예외가 발생한다. 트리플 폴트는 잡을 수 없어서 대부분의 CPU는 리셋이나 운영체제 리부팅으로 예외에 대응한다.

## 인터럽트 디스크립터 테이블

각 요소는 다음의 16 bit 구조를 갖는다.

| 타입 | 이름                | 설명                                                 |
| ---- | ------------------- | ---------------------------------------------------- |
| u16  | 함수 포인터 [0:15]  | 처리함수를 가리키는 포인터의 낮은 비트               |
| u16  | GDT 선택자          | 글로벌 디스크립터 테이블에 있는 코드 세그먼트 선택자 |
| u16  | 옵션                | (아래에서 설명함)                                    |
| u16  | 함수 포인터 [16:31] | 처리함수를 가리키는 포인터의 중간 비트               |
| u32  | 함수 포인터 [32:63] | 처리함수를 가리키는 포인터의 나머지 비트             |
| u32  | 예약됨              |                                                      |

옵션 필드는 아래와 같은 포맷이다.

| 비트  | 이름                               | 설명                                                                                                |
| ----- | ---------------------------------- | --------------------------------------------------------------------------------------------------- |
| 0-2   | 인터럽트 스택 테이블 인덱스        | 0: 스택을 전환하지 말기, 1-7: 처리함수 호출 시 인터럽트 스택 테이블에 있는 n번째 스택으로 전환하기. |
| 3-7   | 예약됨                             |                                                                                                     |
| 8     | 0: 인터럽트 게이트, 1: 트랩 게이트 | 이게 0이면 처리함수 호출 시 인터럽트가 취소된 거임                                                  |
| 9-11  | 반드시 1임                         |                                                                                                     |
| 12    | 반드시 0임                         |                                                                                                     |
| 13‑14 | Descriptor Privilege Level (DPL)   | 이 처리함수 호출에 필요한 최소 권한 수준.                                                           |
| 15    | Present                            |                                                                                                     |

모든 예외는 IDT 인덱스가 정해져 있다.

유효하지 않은 Opcode 예외는 6번 테이블 인덱스를, 페이지 폴트 예외는 14번 테이블 인덱스를 가진다.

이렇게 해서, 하드워어가 예외 발생 시 해당하는 IDT 시작점을 불러올 수 있다.

### 예외 발생 시 CPU의 대략적인 동작

1. 명령어 포인터와 RFLAGS 레지스터 등을 스택에 넣는다.
2. IDT에서 해당하는 시작점을 읽는다 (예, 페이지 폴트 발생 시 14번째 시작점 읽음)
3. 시작점이 있는지 확인하고 없으면 더블 폴트를 일으킨다.
4. 시작점이 인터럽트 게이트라면 하드웨어 인터럽트를 끈다.
5. 지정한 GDT 선택자를 CS 세그먼트로 불러온다.
6. 지정된 처리함수로 점프한다.

## IDT 타입

직접 IDT 타입을 만들지 않고 `x86_64` 크레이트의 `InterruptDescriptorTable` 구조체를 사용한다.

```rust
#[repr(C)]
pub struct InterruptDescriptorTable {
    pub divide_by_zero: Entry<HandlerFunc>,
    pub debug: Entry<HandlerFunc>,
    pub non_maskable_interrupt: Entry<HandlerFunc>,
    pub breakpoint: Entry<HandlerFunc>,
    pub overflow: Entry<HandlerFunc>,
    pub bound_range_exceeded: Entry<HandlerFunc>,
    pub invalid_opcode: Entry<HandlerFunc>,
    pub device_not_available: Entry<HandlerFunc>,
    pub double_fault: Entry<HandlerFuncWithErrCode>,
    pub invalid_tss: Entry<HandlerFuncWithErrCode>,
    pub segment_not_present: Entry<HandlerFuncWithErrCode>,
    pub stack_segment_fault: Entry<HandlerFuncWithErrCode>,
    pub general_protection_fault: Entry<HandlerFuncWithErrCode>,
    pub page_fault: Entry<PageFaultHandlerFunc>,
    pub x87_floating_point: Entry<HandlerFunc>,
    pub alignment_check: Entry<HandlerFuncWithErrCode>,
    pub machine_check: Entry<HandlerFunc>,
    pub simd_floating_point: Entry<HandlerFunc>,
    pub virtualization: Entry<HandlerFunc>,
    pub security_exception: Entry<HandlerFuncWithErrCode>,
    // some fields omitted
}
```

각 필드는 `idt::Entry<F>` 타입으로 IDT 시작점을 나타낸다.

여기서 타입 파라미터 `F`는 필요한 처리함수 타입을 정의한다.

시작점들이 `HandlerFunc`나 `HandlerFuncWithErrCode`를 필요로 하고 페이지 플트만 `PageFaultHandlerFunc`를 필요로 한다.

`HandlerFunc`는 다음과 같이 생겼다.

```rust
type HandlerFunc = extern "x86-interrupt" fn(_: &mut InterruptStackFrame);
```

`HandlerFunc`는 `extern "x86-interrupt" fn`의 타입 별칭이다.

`extern` 키워드는 `foreign calling convention`을 따르는 함수를 정의할 때 사용되며, C 코드와 사용되기도 한다.

# Interrupt Calling Convention

예외 처리는 함수 호출과 동작이 비슷하다. CPU가 함수의 첫 번째 명령어로 점프한 뒤 이를 실행한다. 그 다음에 CPU가 복귀 주소로 점프하고 부모 함수의 실행을 이어간다.

차이점은 함수 호출은 컴파일러가 삽입한 `call` 명령어를 통해서만 실행되고, 예외는 명령어 종류에 상관없이 발생할 수 있다. 이런 차이가 어떤 결과를 만드는지 함수 호출을 자세히 알아봄으로써 이해할 수 있다.

호출 규약(Calling conventions)은 함수 호출의 작동방식을 지정하는데, 함수 파라미터가 어디에 위치하는지(레지스터나 스택) 결과값이 어떻게 반환되는지 등을 지정한다.

x86_64 리눅스에서는 다음과 같은 규칙이 C 함수에 적용된다.

- 처음 정수 인자 6개는 레지스터 `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`에 넘겨진다.
- 추가 인자는 스택으로 넘겨진다.
- 결과값은 `rax`와 `rdx`에 반환된다.

러스트는 C ABI를 따르지 않으므로 위 규칙은 `extern "C" fn`으로 선언된 함수에만 적용된다. (사실 러스트는 ABI가 아직 없다.)

## 보존 레지스터와 스크래치 레지스터

호출 규약으로 레지스터는 보존과 스크래치 레지스터로 나뉘게 된다.

보존 레지스터는 함수 호출에도 값이 변하지 않는다. 호출되는 함수(callee)는 반환하기 전에 레지스터 값을 원상복구할 수 있을 때만 덮어쓸 수 있다. 그래서 이 레지스터들은 "callee-saved"라고 부른다. 일반적으로 함수의 시작부에서 이 레지스터를 스택에 저장하고 반환 전에 복구한다.

반대로, 스크래치 레지스터는 호출된 함수가 제한 없이 덮어쓸 수 있다. 호출자가 함수 호출 사이에 스크래치 레지스터의 값을 보존하려면, 함수 호출 전에 값을 백업하고 복구해야 한다.(스택에 값을 집어넣기) 그래서 스크래치 레지스터는 "caller-saved"라고 부룬다.

x86_64의 C 호출 규약은 다음의 보존 레지스터와 스크래치 레지스터를 지정해놓았다.

| 보존 레지스터                                   | 스크래치 레지스터                                           |
| ----------------------------------------------- | ----------------------------------------------------------- |
| `rbp`, `rbx`, `rsp`, `r12`, `r13`, `r14`, `r15` | `rax`, `rcx`, `rdx`, `rsi`, `rdi`, `r8`, `r9`, `r10`, `r11` |
| callee-saved                                    | caller-saved                                                |

컴파일러는 이런 규칙에 따라 코드를 생성한다. 예를 들어, 대부분의 함수는 `rbp`를 스택에 백업하는 `push rbp` 명령어로 시작한다.

## 모든 레지스터를 보존하기

함수 호출과 다르게, 예외는 모든 명령어에서 발생할 수 있다. 대부분의 경우, 컴파일 타임에서는 생성된 코드가 예외를 일으키는지 알 수 없다. 예를 들어, 컴파일러는 어떤 코드가 스택 오버플로나 페이지 폴트를 일으킬 지 알 수 없다.

예외가 언제 발생할 지 모르니 사전에 레지스터를 백업할 수도 없다.
즉, caller-saved 레지스터에 의존하는 호출 규약을 예외 처리에 활용할 수 없다. 대신에, 모든 레지스터를 저장하는 `x86-interrupt` 호출 규약을 사용한다. 이 규약은 함수 반환 시 모든 레지스터의 값이 원래 값으로 복구되도록 한다.

함수 시작 시 스택에 모든 레지스터가 저장되는게 아니라, 컴파일러가 함수에 의해 덮어쓰기된 레지스터만 백업한다. 이 방식으로, 사용하는 레지스터가 많지 않은 짧은 함수는 코드를 효율적으로 생성할 수 있다.

## 인터럽트 스택 프레임

`call` 명령어를 사용하는 일반적인 함수 호출의 경우, CPU는 타깃 함수로 점프하기 전에 복귀 주소를 스택에 넣는다. `ret` 명령어를 사용하는 함수 반환 시, CPU는 복귀 주소를 꺼내서 그 주소로 점프한다.

일반적인 함수 호출의 스택 프레임은 다음과 같이 생겼다.

<center><img alt="interrupt stack frame" src="https://user-images.githubusercontent.com/22253556/79334173-c2310c80-7f5a-11ea-904d-6e30939641b6.png" /></center>

한편, 예외와 인터럽트를 처리할 때는 복귀 주소를 스택에 넣는 것만으로는 부족하다. 인터럽트 처리함수는 다른 문맥(스택 포인터, CPU 플래그 등)에서 작동하기 때문이다. 대신에, CPU는 인터럽트 발생 시 다음 절차를 따른다.

1. **스택 포인터 정렬하기**: 어떤 명령어든 인터럽트가 발생할 수 있으므로, 스택 포인터도 어떤 값이든 가질 수 있다. 그런데, CPU 명령어 중에는 CPU가 인터럽트 직후에 정렬을 수행할 수 있도록 스택 포인터가 16 바이트 바운더리에 정렬되기를 요구하는 명령어가 있다.(ex SSE 명령어)

2. **스택 교체** (몇몇 경우에만): CPU 권한 레벨 변경 시 스택 교체가 발생한다. (예를 들어, 유저 모드 프로그램에서 CPU 예외가 발생한 경우) *인터럽트 스택 테이블*을 사용하는 특정 인터럽트를 위해 스택 교체를 설정할 수 있다.

3. **오래된 스택 포인터를 스택에 넣기**: CPU는 인터럽트 발생 시(정렬 전) 스택 포인터(`rsp`)와 스택 세그먼트(`ss`)를 스택에 넣는다. 이 덕분에 인터럽트 처리함수에서 복귀할 때 원래 스택 포인터를 복구할 수 있다.

4. **`RFLAGS` 레지스터를 스택에 넣고 갱신하기**: `RFLAGS` 레지스터는 다양한 컨트롤 비트와 상태 비트를 담고 있다. 인터럽트 시작 시, CPU는 몇몇 비트를 바꾸고 옛날 값은 스택에 넣는다.

5. **명령어 포인터를 스택에 넣기**: 인터럽트 처리함수로 점프하기 전에 CPU는 명령어 포인터(`rip`)와 코드 세그먼트(`cs`)를 스택에 넣는다. 일반 함수 호출 시 복귀 주소를 스택에 넣는 것과 대비된다.

6. **에러 코드 스택에 넣기**(몇몇 예외에만): 페이지 폴트 같은 특정 예외의 경우, CPU는 예외의 원인을 설명하는 에러 코드를 스택에 넣는다.

7. **인터럽트 처리함수 실행**: CPU가 인터럽트 처리함수의 주소와 세그먼트 디스크립트를 해당하는 IDT 필드에서 읽어온다. 그런 다음, 값들을 `rip`와 `cs` 레지스터에 불러와서 처리함수를 실행한다.

인터럽트 스택 프레임은 다음과 같이 생겼다.

<center><img alt="interrupt stack frame" src="https://user-images.githubusercontent.com/22253556/79336421-aa5b8780-7f5e-11ea-9dd9-8e25f6e465d6.png" /></center>

`x86_64` 크레이트의 인터럽트 스택 프레임은 `InterruptStackFrame` 구조체로 표현된다. 이 구조체는 `&mut`으로 인터럽트 처리함수로 전달되고 예외 발생 원인에 대한 추가 정보를 얻기 위해 사용된다. 에러 코드를 사용하는 예외가 몇 개 없으므로 이 구조체에 에러 코드 필드는 없다. 에러 코드가 필요한 예외는 `error_code` 인자를 가지고 있는 `HandlerFuncWithErrCode` 함수 타입을 따로 사용한다.

## Behind the Scenes

`x86-interrupt` 호출 규약은 예외 처리 과정의 지저분한 세부사항을 가려주는 추상화를 제공한다. 그럼에도, 내부에서 일이 어떻게 일어나는 지 알면 유용하다. `x86-interrupt` 호츌 규약이 해결해주는 것을 대략적으로 설명해본다.

- **인자들을 되찾아오기**: 대부분의 호출 규약에서는 인자가 레지스터로 전달되지만, 예외 처리시에는 불가능하다. 레지스터의 값을 스택에서 복구하기 전에 레지스터 덮어쓰기를 하면 안 되기 때문이다. 대신에 `x86-interrupt` 호출 규약은 스택의 특정 위치에 인자들이 놓여있다는 것을 알고있다.

- **`iretq`를 사용하는 복귀**: 인터럽트 스택 프레임이 일반 함수 호출의 스택 프레임과 완전히 다르므로, 일반적인 `ret` 명령어를 사용해서 처리함수로부터 복귀를 할 수 없다. 대신에 `iretq` 명령어를 사용해야 한다.

- **에러 코드 처리**: 몇몇 예외에 사용하기 위해 스택에 넣어지는 에러 코드는 처리를 복잡하게 만든다. 에러 코드는 스택 정렬을 바꾸고 복귀 전에 스택에서 제거돼야 한다. `x86-interrupt` 호출 규약이 이 복잡한 과정을 떠맡아준다. 그런데, 호출 규약은 어떤 예외에 어떤 처리함수를 써야하는지 몰라서, 함수 인자의 개수에서 정보를 추론한다. 그래서 각 예외 처리에 알맞은 함수를 고르는 것은 프로그래머의 몫이다. `x86_64` 크레이트에 정의된 `InterruptDescriptorTable` 타입을 사용하면 올바른 함수 타입 사용을 보장할 수 있다.

- **스택 정렬하기**: 16 byte 스택 정렬이 필요한 명령어들이 있다.(특히 SSE 명령어) 예외 발생 시 CPU가 정렬을 해주지만, 몇몇 예외들이 스택에 에러 코드를 넣어서 정렬을 망가뜨리는 경우가 있다. 그런 경우는 `x86-interrupt` 호출 규약이 스택을 재정렬해준다.

## 구현하기

인터럽트 모듈을 담을 `src/interrupts.rs` 파일을 만든다.

`src/lib.rs`에는 모듈 선언을 추가한다.

```rust
pub mod interrupts;
```

`InterruptDescriptorTable`를 생성하는 `init_idt` 함수를 선언한다.

```rust
use x86_64::structures::idt::InterruptDescriptorTable;

pub fn init_idt() {
    let mut idt = InterruptDescriptorTable::new();
}
```

다음으로 중단점(breakpoint) 예외 처리함수를 만든다.
중단점 예외는 중단점 명령어인 `int3`가 실행되면 임시로 프로그램을 멈춘다. 예외 처리를 테스트하기 매우 적합하다.

중단점 예외는 디버거에서 많이 사용한다. 사용자가 중단점을 설정하면 디버거는 해당 명령어 부분을 `int3` 명령어로 바꿔치기하고 프로그램이 그 줄에 다다르면 CPU가 중단점 예외를 발생시킨다. 사용자가 프로그램 실행을 계속하고 싶을 때, 디버거는 `int3` 명령어를 원래 명령어로 되돌린다.

중단점 명령어가 실행되면 메시지를 출력만 하고 프로그램은 그대로 실행되게 하는 `breakpoint_handler` 함수를 만들고 IDT에 등록한다.

```rust
use x86_64::structures::idt::{InterruptDescriptorTable, InterruptStackFrame};
use crate::println;

pub fn init_idt() {
    let mut idt = InterruptDescriptorTable::new();
    idt.breakpoint.set_handler_fn(breakpoint_handler);
}

extern "x86-interrupt" fn breakpoint_handler(
    stack_frame: &mut InterruptStackFrame)
{
    println!("EXCEPTION: BREAKPOINT\n{:#?}", stack_frame);
}
```

컴파일러의 `x86-interrupt ABI is experimental and subject to change` 에러는 `lib.rs`에 `#![feature(abi_x86_interrupt)]`를 추가해서 끈다.

## IDT 불러오기

CPU가 인터럽트 디스크립터 테이블을 사용하도록 `lidt` 명령어로 테이블을 불러와야 한다.
`InterruptDescriptorTable` 구조체의 `load` 메서드를 사용하면 된다.

```rust
pub fn init_idt() {
    let mut idt = InterruptDescriptorTable::new();
    idt.breakpoint.set_handler_fn(breakpoint_handler);
    idt.load();
}
```

컴파일 시 `` `idt` does not live long enough ``에러가 발생한다. `load` 메서드는 `&'static self`를 필요로 하는데 다른 IDT를 불러오기 전까지 CPU는 항상 같은 IDT 테이블에 접근하기 때문이다.

`idt`에 `'static` 라이프타임을 설정하는 방법은 여러가지 있다.

1. `Box`를 사용해 IDT를 힙에 넣기: 커널은 아직 힙이 없어서 X
2. `static`으로 선언하기: `static`은 불변이기에 중단점 시작 변경이 불가능해서 X
3. `static mut` 사용하기: 데이터 레이스 발생 가능, 접근할 때마다 `unsafe` 블록 사용필요해서 X
4. `lazy_static!` 메크로 사용하기 O

```rust
use lazy_static::lazy_static;

lazy_static! {
    static ref IDT: InterruptDescriptorTable = {
        let mut idt = InterruptDescriptorTable::new();
        idt.breakpoint.set_handler_fn(breakpoint_handler);
        idt
    };
}

pub fn init_idt() {
    IDT.load();
}
```

## 실행하기

`lib.rs`에 IDT를 초기화하는 `init` 함수를 만든다.

```rust
pub fn init() {
    interrupts::init_idt();
}
```

`main.rs`의 `_start` 함수에 `init` 함수를 호출하고 중단점 예외를 던진다.

```rust
#[no_mangle]
pub extern "C" fn _start() -> ! {
    println!("Hello World{}", "!");

    blog_os::init(); // new

    // invoke a breakpoint exception
    x86_64::instructions::interrupts::int3(); // new

    // as before
    #[cfg(test)]
    test_main();

    println!("It did not crash!");
    loop {}
}
```

![image](https://user-images.githubusercontent.com/22253556/79566839-2d5f1800-80ee-11ea-8543-58b694584846.png)

CPU가 중단점 예외 처리함수를 실행한 덕분에 예외 메시지를 출력한 다음, 다시 `_start` 함수로 돌아와서 `It did not crash!`를 출력했다.
