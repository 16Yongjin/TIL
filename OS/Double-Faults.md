# 더블 폴트

CPU가 예외 처리함수 실행에 실패했을 때 발생하는 더블 폴트를 알아본다.

더블 폴트를 처리해야 시스템 리셋을 일으키는 치명적인 트리플 폴트를 피할 수 있다.

어떤 상황이든 트리플 폴트를 막기 위해, 더블 폴트를 잡는 인터럽트 스택 테이블을 분리된 커널 스택에 만든다.

## 더블 폴트란?

더블 폴트는 CPU가 예외 처리에 실패했을 때 발생한다. 예를 들면, 페이지 폴트가 발생했을 때, IDT에 등록된 페이지 폴트 처리함수가 없을 때 발생한다. 프로그래밍 언어에서 catch-all 블록과 비슷하다고 보면 된다.

더블 폴트는 일반적인 예외랑 비슷하다. 벡터 `8`번을 가지고 있고 IDT에 일반적인 처리함수를 정의할 수 있다. 더블 폴트 처리함수는 반드시 정의해야 한다. 없으면 트리플 폴트가 발생해서 하드웨어가 시스템을 리셋시킨다.

## 더블 폴트 일으키기

처리함수 정의가 안 된 예외를 일으켜서 더블 폴트를 발생시킬 수 있다.

```rust
// src/main.rs 안

#[no_mangle]
pub extern "C" fn _start() -> ! {
    println!("Hello World{}", "!");

    blog_os::init();

    // 페이지 폴트 발생
    unsafe {
        *(0xdeadbeef as *mut u64) = 42;
    };

    #[cfg(test)]
    test_main();

    println!("It did not crash!");
    loop {}
}
```

`unsafe`로 유효하지 않은 주소인 `0xdeadbeef`에 쓰기를 한다. 가상 주소가 페이지 테이블의 실제 주소에 매핑되지 않았기 때문에 페이지 폴드가 발생한다. 아직 IDT에 페이지 폴트 처리함수를 등록하지 않아서 더블 폴트가 발생한다.

커널을 실행하면 다음의 이유로 무한 부팅에 빠진다.

1. CPU가 `0xdeadbeef`에 쓰기를 시도해서 페이지 폴트가 발생한다.
2. CPU가 IDT에서 페이지 폴트 처리함수 찾기를 시도하지만, 실패해서 더블 폴트가 발생한다.
3. CPU가 IDT에서 더블 폴트 처리함수를 찾기를 시도하지만, 실패해서 트리플 폴트가 발생한다.
4. 트리플 폴트는 치명적이기에 QEMU는 대부분의 실제 하드웨어처럼 시스템 리셋을 한다.

트리플 폴트를 막으려면 페이지 폴트 처리함수나 더블 폴트 처리함수를 등록하면 된다. 모든 경우에서 트리플 폴트를 막는게 급선무이기에, 모든 처리되지 않은 예외 발생 시 실행되는 더블 폴트 처리함수를 먼저 정의한다.

## 더블 폴트 처리함수

더블 폴트는 일반적인 예외에 에러 코드가 추가되어 있다. 중단점 처리함수에 했던 것과 비슷하게 처리함수를 등록하면 된다.

```rust
// src/interrupts.rs 안

lazy_static! {
    static ref IDT: InterruptDescriptorTable = {
        let mut idt = InterruptDescriptorTable::new();
        idt.breakpoint.set_handler_fn(breakpoint_handler);
        idt.double_fault.set_handler_fn(double_fault_handler); // 새로 추가
        idt
    };
}

// 새로 추가
extern "x86-interrupt" fn double_fault_handler(
    stack_frame: &mut InterruptStackFrame, _error_code: u64) -> !
{
    panic!("EXCEPTION: DOUBLE FAULT\n{:#?}", stack_frame);
}
```

새로운 처리함수는 에러 메시지를 출력하고 예외 스택 프레임을 덤프한다. 더블 폴트 처리함수의 에러코드는 항상 0이다. 그래서 에러 코드를 출력할 필요가 없다. 중단점 처리함수 다른점이 있다면, 더블 폴트 처리함수는 *발산*한다. `x86_64` 아키텍처가 더블 폴트 예외에서 반환되는 것을 허용하지 않기 때문이다.

커널을 시작해보면 더플 폴트 처리함수가 실행된 것을 볼 수 있다:

![double-fault-caught](https://user-images.githubusercontent.com/22253556/79637507-a94d7480-816f-11ea-9bed-54fc38ea9572.png)

다음의 일이 발생했다.

1. CPU가 `0xdeadbeef`에 쓰기를 시도해서 페이지 폴드가 발생했다.
2. 전과 같이, CPU가 IDT에서 페이지 폴트 처러함수를 찾아보지만, 실패해서 더블 폴트를 일으킨다.
3. CPU가 (지금은 존재하는) 더블 폴트 처리함수로 점프한다.

CPU가 더블 폴트 처리함수를 실행시킬 수 있기에 트리플 폴트는 더 이상 발생하지 않는다.

대부분의 더블 폴트를 잡을 수 있지만, 현재 방식으로 더블 폴트를 잡을 수 없는 경우가 있다.

## 더블 폴트 발생 원인

특수한 경우를 살펴보기 전에, 더블 폴트의 원인을 정확히 알아야 한다.
위에서 다음과 같이 애매한 정의를 내렸다:

> 더블 폴트는 CPU가 예외 처리함수 실행에 실패했을 때 발생하는 특수한 예외다.

"실행에 실패했을 때"가 정확히 어떤 의미일까? 처리함수가 없을 때? 처리함수가 스왑됐을 때? 또한, 처리함수 자체가 예외를 일으키면 어떤일이 발생할까?

예를 들어, 다음과 같은 일이 발생했을 때:

1. 중단점 예외가 발생했지만, 해당하는 처리함수가 스왑됐을 때?
2. 페이지 폴트가 발생했지만, 페이지 폴트 처리함수가 스왑됐을 때?
3. 0으로 나누기 예외 처리함수가 중단점 예외를 일으켰지만, 중단점 처리함수가 스왑됐을 때?
4. 커널이 스택 오버플로를 발생시키고 가드 페이지에 접근했을 때?

다행히도, AMD64 매뉴올에 정확한 정의가 나와있다. 이에 따르면, "더블 폴트 예외는 두 번째 예외가 앞선(첫 번째) 예외 처리 중에 발생했을 때 일어날 수 있다". 여기서 "일어날 수 있다"라는 말은 특정한 예외의 조합에서만 더블 폴트가 일어난다는 뜻이다. 그 조합은 다음과 같다.

<table>
<thead><tr><th>첫 번째 예외</th><th>두 번째 예외</th></tr></thead><tbody>
<tr><td><a href="https://wiki.osdev.org/Exceptions#Divide-by-zero_Error">0으로 나누기</a>,<br><a href="https://wiki.osdev.org/Exceptions#Invalid_TSS">유효하지 않은 TSS</a>,<br><a href="https://wiki.osdev.org/Exceptions#Segment_Not_Present">세그먼트가 없음</a>,<br><a href="https://wiki.osdev.org/Exceptions#Stack-Segment_Fault">스택 세그먼트 폴트</a>,<br><a href="https://wiki.osdev.org/Exceptions#General_Protection_Fault">일반 보호 폴트</a></td><td><a href="https://wiki.osdev.org/Exceptions#Invalid_TSS">Invalid TSS</a>,<br><a href="https://wiki.osdev.org/Exceptions#Segment_Not_Present">세그먼트가 없음</a>,<br><a href="https://wiki.osdev.org/Exceptions#Stack-Segment_Fault">스택 세그먼트 폴트</a>,<br><a href="https://wiki.osdev.org/Exceptions#General_Protection_Fault">일반 보호 폴트</a></td></tr>
<tr><td><a href="https://wiki.osdev.org/Exceptions#Page_Fault">페이지 폴트</a></td><td><a href="https://wiki.osdev.org/Exceptions#Page_Fault">페이지 폴트</a>,<br><a href="https://wiki.osdev.org/Exceptions#Invalid_TSS">유효하지 않은 TSS</a>,<br><a href="https://wiki.osdev.org/Exceptions#Segment_Not_Present">세그먼트가 없음</a>,<br><a href="https://wiki.osdev.org/Exceptions#Stack-Segment_Fault">스택 세그먼트 폴트</a>,<br><a href="https://wiki.osdev.org/Exceptions#General_Protection_Fault">일반 보호 폴트</a></td></tr>
</tbody>
</table>

예를 들어, 페이지 폴트 뒤에 발생하는 0으로 나누기 예외는 괜찮지만(페이지 폴트 처리함수가 실행됨), 일반 보호 폴트 뒤에 발생하는 0으로 나누기 예외는 더블 폴트를 일으킨다.

위 테이블을 참고하면 위에서 했던 질문 중 세 가지를 대답할 수 있다.

1. 중단점 에외 발생했는데 해당 처리함수가 스왑된 경우, 페이지 폴트가 발생해서 페이지 폴트 처리함수가 실행된다.
2. 페이지 폴트가 발생했는데 페이지 폴트 처리함수가 스왑된 경우, 더블 폴트가 발생하고 더블 폴트 처리함수가 실행된다.
3. 0으로 나누기 처리함수가 중단점 예외를 발생시키는 경우, CPU는 중단점 처리함수 실행을 시도한다. 중단점 처리함수가 스왑된 경우, 페이지 폴트가 발생해서 페이지 폴트 처리함수가 실행된다.

## 커널 스택 오버플로

네 번째 질문은 "커널이 스택 오버플로를 발생시키고 가드 페이지에 접근했을 때 어떤 일이 발생할까?"이다.

가드 페이지는 스택 맨 아래에서 스택 오버플로를 막아주는 특수한 메모리 페이지이다. 이 페이지는 물리적인 구역이 아니여서 접근 시 조용하게 다른 메모리를 오염시키는 대신, 페이지 폴트를 발생시킨다. 부트로더가 커널 스택에 가드 페이지를 추가해줘서, 스택 오버플로 시 페이지 폴트가 발생한다.

페이지 폴트 발생 시 CPU는 IDT에서 페이지 폴트 처리함수를 찾아서 인터럽트 스택 프레임을 스택에 넣는다. 그러나 현재 스택 포인터가 여전히 존재하지 않는 가드 페이지를 가리킨다. 그래서 두 번째 페이지 폴트가 발생해서 더블 폴트도 발생한다.

그래서 CPU가 더블 폴트 처리함수를 호출하면 CPU는 예외 스택 프레임을 스택에 넣으려고 한다. 스택 포인터가 여전히 가드 포인터를 가리키고 있어서 세 번째 페이지 폴트가 발생해서, 트리폴 폴트와 시스템 재부팅이 일어난다. 이와 같은 경우에 현재까지 작성한 더블 폴트 처리함수는 트리플 폴트를 막을 수 없다.

무한재귀를 하는 함수를 실행해서 직접 위와 같은 상황을 재현할 수 있다:

```rust
// src/main.rs 안

#[no_mangle]
pub extern "C" fn _start() -> ! {
    println!("Hello World{}", "!");

    blog_os::init();

    fn stack_overflow() {
        stack_overflow(); // 각 재귀마다 복귀 주소가 스택에 쌓인다.
    }

    // 스택 오버플로를 발생시킨다.
    stack_overflow();

    […] // test_main(), println(…), and loop {}
}
```

QEMU에 이 코드를 실행시키면, 시스템이 무한 부팅에 빠지는 것을 볼 수 있다.

어떻게 하면 이 문제를 피할 수 있을끼? 예외 스택 프레임이 스택으로 들어가는 일은 CPU 자체에서 일어나는 일이기에 막을 수 없다. 그래서 더블 폴트 예외가 일어나도 스택이 항상 유효하도록 조치를 취해야 한다. 다행히도 x86_64 아키텍처에 해결책이 마련되어있다.

## 스택 바꾸기

x86_64 아키택처는 예외 발생 시 미리 정해둔 정상적인 스택으로의 변경이 가능하다. 이 변경은 하드웨어 수준에서 발생하기에, CPU가 예외 스택 프레임을 스택에 넣기 전에 실행된다.

변경 작동방식은 인터럽트 스택 테이블(Interrupt Stack Table, IST)로 구현되어있다. IST는 정상적인 스택을 가리키는 포인터 7개가 들어있는 테이블이다. 러스트 의사코드를 작성해보면 다음과 같다.

```rust
struct InterruptStackTable {
    stack_pointers: [Option<StackPointer>; 7],
}
```

각각의 예외 처리함수에 사용할 스택 프레임을 IDT 시작점의 `stack_pointers` 필드를 통해 IST에서 고를 수 있다. 예를 들어, 더플 폴트 처리 시에 IST에 있는 첫 번째 스택을 골라 사용할 수 있다. 그러면 CPU가 더블 폴트가 발생할 떄마다 자동으로 그 스택으로 변경한다. 이 스택은 뭔가가 넣어지기 전에 생성되기에, 트리플 폴트를 막아준다.

## IST와 TSS

인터럽트 스택 테이블(IST)는 작업 상태 세그먼트(TSS)라는 오래된 레거시 구조의 일부분이다. TSS는 32비트 모드의 작업에 대한 여러가지 정보(ex, 프로세서 레지스터 상태)를 담고 있고 하드웨어 컨택스트 스위칭에도 사용되곤 했다. 그런데 하드웨어 컨택스트 스위칭이 64비트 모드에서 더이상 지원되지 않아서 TSS의 형식은 완전히 변경됐다.

x86_64의 TSS는 작업 관련된 정보를 더 이상 담지 않는다. 대신에, 두 개의 스택 테이블(그 중하나는 IST)을 담는다. 32비트와 64비트 TSS사이의 유일한 공통점은 *I/O 포트 권한 비트맵*이다.

64비트 TSS는 다음의 형식을 갖는다.

| 필트                 | 타입     |
| -------------------- | -------- |
| (예약됨)             | u32      |
| 권한 스택 테이블     | [u64; 3] |
| (예약됨)             | u64      |
| 인터럽트 스택 테이블 | [u64; 7] |
| (예약됨)             | u64      |
| (예약됨)             | u16      |
| I/O 맵 기준 주소     | u16      |

CPU는 권한 수준 변경 시 권한 스택 테이블을 사용한다. 예를 들어, CPU가 유저 모드(권한 수준 3)에 있을 때 예외가 발생한 경우 CPU는 보통 예외 처리함수를 실행하기 전에 커널 모드(권한 수준 0)로 변경한다. 이 때, CPU는 권한 스택 테이블의 0번째 스택으로 변경한다.(0이 목표 권한 수준이기 때문이다). 커널엔 유저 모드 프로그램이 아직 없으므로 이 테이블을 잠시 신경끄도록 한다.

## TSS 생성하기

분리된 더블 폴트 스택을 가진 TSS을 인터럽트 스택 테이블에 새로 만든다. 그럴려면 TSS 구조체가 필요한데, 다행히도 `x86_64` 크레이트에 `TaskStateSegment`가 들어있다.

`gbt` 모듈을 새로 만들고 이 안에 TSS를 생성한다.

```rust
// src/lib.rs 내부

pub mod gdt;

// src/gdt.rs 내부

use x86_64::VirtAddr;
use x86_64::structures::tss::TaskStateSegment;
use lazy_static::lazy_static;

pub const DOUBLE_FAULT_IST_INDEX: u16 = 0;

lazy_static! {
    static ref TSS: TaskStateSegment = {
        let mut tss = TaskStateSegment::new();
        tss.interrupt_stack_table[DOUBLE_FAULT_IST_INDEX as usize] = {
            const STACK_SIZE: usize = 4096;
            static mut STACK: [u8; STACK_SIZE] = [0; STACK_SIZE];

            let stack_start = VirtAddr::from_ptr(unsafe { &STACK });
            let stack_end = stack_start + STACK_SIZE;
            stack_end
        };
        tss
    };
}
```

0번째 IST 시작점을 더블 폴트 스택으로 설정한다. 그런 다음 더블 폴트 스택의 최상위 주소를 0번째 시작점에 쓴다. x86에서는 스택이 아래로(높은 주소에서 낮은 주소로) 커지기 때문에 최상위 주소에 썼다.
