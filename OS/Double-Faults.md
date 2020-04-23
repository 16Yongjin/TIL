# 더블 폴트

CPU가 예외 처리 함수 실행에 실패했을 때 발생하는 더블 폴트를 알아본다.

더블 폴트를 처리해야 시스템 리셋을 일으키는 치명적인 트리플 폴트를 피할 수 있다.

어떤 상황이든 트리플 폴트를 막기 위해, 더블 폴트를 잡는 인터럽트 스택 테이블을 분리된 커널 스택에 만든다.

## 더블 폴트란?

더블 폴트는 CPU가 예외 처리에 실패했을 때 발생한다. 예를 들면, 페이지 폴트가 발생했을 때, IDT에 등록된 페이지 폴트 처리 함수가 없을 때 발생한다. 프로그래밍 언어에서 catch-all 블록과 비슷하다고 보면 된다.

더블 폴트는 일반적인 예외랑 비슷하다. 벡터 `8`번을 가지고 있고 IDT에 일반적인 처리 함수를 정의할 수 있다. 더블 폴트 처리 함수는 반드시 정의해야 한다. 없으면 트리플 폴트가 발생해서 하드웨어가 시스템을 리셋시킨다.

### 더블 폴트 일으키기

처리 함수 정의가 안 된 예외를 일으켜서 더블 폴트를 발생시킬 수 있다.

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

`unsafe`로 유효하지 않은 주소인 `0xdeadbeef`에 쓰기를 한다. 가상 주소가 페이지 테이블의 실제 주소에 매핑되지 않았기 때문에 페이지 폴드가 발생한다. 아직 IDT에 페이지 폴트 처리 함수를 등록하지 않아서 더블 폴트가 발생한다.

커널을 실행하면 다음의 이유로 무한 부팅에 빠진다.

1. CPU가 `0xdeadbeef`에 쓰기를 시도해서 페이지 폴트가 발생한다.
2. CPU가 IDT에서 페이지 폴트 처리 함수 찾기를 시도하지만, 실패해서 더블 폴트가 발생한다.
3. CPU가 IDT에서 더블 폴트 처리 함수를 찾기를 시도하지만, 실패해서 트리플 폴트가 발생한다.
4. 트리플 폴트는 치명적이기에 QEMU는 대부분의 실제 하드웨어처럼 시스템 리셋을 한다.

트리플 폴트를 막으려면 페이지 폴트 처리 함수나 더블 폴트 처리 함수를 등록하면 된다. 모든 경우에서 트리플 폴트를 막는게 급선무이기에, 모든 처리되지 않은 예외 발생 시 실행되는 더블 폴트 처리 함수를 먼저 정의한다.

## 더블 폴트 처리 함수

더블 폴트는 일반적인 예외에 에러 코드가 추가되어 있다. 중단점 처리 함수에 했던 것과 비슷하게 처리 함수를 등록하면 된다.

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

새로운 처리 함수는 에러 메시지를 출력하고 예외 스택 프레임을 덤프 한다. 더블 폴트 처리 함수의 에러코드는 항상 0이다. 그래서 에러 코드를 출력할 필요가 없다. 중단점 처리 함수 다른 점이 있다면, 더블 폴트 처리 함수는 *발산*한다. `x86_64` 아키텍처가 더블 폴트 예외에서 반환되는 것을 허용하지 않기 때문이다.

커널을 시작해보면 더플 폴트 처리 함수가 실행된 것을 볼 수 있다:

![double-fault-caught](https://user-images.githubusercontent.com/22253556/79637507-a94d7480-816f-11ea-9bed-54fc38ea9572.png)

다음의 일이 발생했다.

1. CPU가 `0xdeadbeef`에 쓰기를 시도해서 페이지 폴드가 발생했다.
2. 전과 같이, CPU가 IDT에서 페이지 폴트 처리 함수를 찾아보지만, 실패해서 더블 폴트를 일으킨다.
3. CPU가 (지금은 존재하는) 더블 폴트 처리 함수로 점프한다.

CPU가 더블 폴트 처리 함수를 실행시킬 수 있기에 트리플 폴트는 더 이상 발생하지 않는다.

대부분의 더블 폴트를 잡을 수 있지만, 현재 방식으로 더블 폴트를 잡을 수 없는 경우가 있다.

## 더블 폴트 발생 원인

특수한 경우를 살펴보기 전에, 더블 폴트의 원인을 정확히 알아야 한다.
위에서 다음과 같이 애매한 정의를 내렸다:

> 더블 폴트는 CPU가 예외 처리 함수 실행에 실패했을 때 발생하는 특수한 예외다.

"실행에 실패했을 때"가 정확히 어떤 의미일까? 처리 함수가 없을 때? 처리 함수가 스왑 됐을 때? 또한, 처리 함수 자체가 예외를 일으키면 어떤일이 발생할까?

예를 들어, 다음과 같은 일이 발생했을 때:

1. 중단점 예외가 발생했지만, 해당하는 처리 함수가 스왑됐을 때?
2. 페이지 폴트가 발생했지만, 페이지 폴트 처리 함수가 스왑 됐을 때?
3. 0으로 나누기 예외 처리 함수가 중단점 예외를 일으켰지만, 중단점 처리 함수가 스왑됐을 때?
4. 커널이 스택 오버플로를 발생시키고 가드 페이지에 접근했을 때?

다행히도, AMD64 매뉴얼에 정확한 정의가 나와있다. 이에 따르면, "더블 폴트 예외는 두 번째 예외가 앞선(첫 번째) 예외 처리 중에 발생했을 때 일어날 수 있다". 여기서 "일어날 수 있다"라는 말은 특정한 예외의 조합에서만 더블 폴트가 일어난다는 뜻이다. 그 조합은 다음과 같다.

<table>
<thead><tr><th>첫 번째 예외</th><th>두 번째 예외</th></tr></thead><tbody>
<tr><td><a href="https://wiki.osdev.org/Exceptions#Divide-by-zero_Error">0으로 나누기</a>,<br><a href="https://wiki.osdev.org/Exceptions#Invalid_TSS">유효하지 않은 TSS</a>,<br><a href="https://wiki.osdev.org/Exceptions#Segment_Not_Present">세그먼트가 없음</a>,<br><a href="https://wiki.osdev.org/Exceptions#Stack-Segment_Fault">스택 세그먼트 폴트</a>,<br><a href="https://wiki.osdev.org/Exceptions#General_Protection_Fault">일반 보호 폴트</a></td><td><a href="https://wiki.osdev.org/Exceptions#Invalid_TSS">Invalid TSS</a>,<br><a href="https://wiki.osdev.org/Exceptions#Segment_Not_Present">세그먼트가 없음</a>,<br><a href="https://wiki.osdev.org/Exceptions#Stack-Segment_Fault">스택 세그먼트 폴트</a>,<br><a href="https://wiki.osdev.org/Exceptions#General_Protection_Fault">일반 보호 폴트</a></td></tr>
<tr><td><a href="https://wiki.osdev.org/Exceptions#Page_Fault">페이지 폴트</a></td><td><a href="https://wiki.osdev.org/Exceptions#Page_Fault">페이지 폴트</a>,<br><a href="https://wiki.osdev.org/Exceptions#Invalid_TSS">유효하지 않은 TSS</a>,<br><a href="https://wiki.osdev.org/Exceptions#Segment_Not_Present">세그먼트가 없음</a>,<br><a href="https://wiki.osdev.org/Exceptions#Stack-Segment_Fault">스택 세그먼트 폴트</a>,<br><a href="https://wiki.osdev.org/Exceptions#General_Protection_Fault">일반 보호 폴트</a></td></tr>
</tbody>
</table>

예를 들어, 페이지 폴트 뒤에 발생하는 0으로 나누기 예외는 괜찮지만(페이지 폴트 처리 함수가 실행됨), 일반 보호 폴트 뒤에 발생하는 0으로 나누기 예외는 더블 폴트를 일으킨다.

위 테이블을 참고하면 위에서 했던 질문 중 세 가지를 대답할 수 있다.

1. 중단점 예외 발생했는데 해당 처리 함수가 스왑된 경우, 페이지 폴트가 발생해서 페이지 폴트 처리 함수가 실행된다.
2. 페이지 폴트가 발생했는데 페이지 폴트 처리 함수가 스왑 된 경우, 더블 폴트가 발생하고 더블 폴트 처리 함수가 실행된다.
3. 0으로 나누기 처리 함수가 중단점 예외를 발생시키는 경우, CPU는 중단점 처리 함수 실행을 시도한다. 중단점 처리 함수가 스왑된 경우, 페이지 폴트가 발생해서 페이지 폴트 처리 함수가 실행된다.

### 커널 스택 오버플로

네 번째 질문은 "커널이 스택 오버플로를 발생시키고 가드 페이지에 접근했을 때 어떤 일이 발생할까?"이다.

가드 페이지는 스택 맨 아래에서 스택 오버플로를 막아주는 특수한 메모리 페이지이다. 이 페이지는 물리적인 구역이 아니여서 접근 시 조용하게 다른 메모리를 오염시키는 대신, 페이지 폴트를 발생시킨다. 부트로더가 커널 스택에 가드 페이지를 추가해줘서, 스택 오버플로 시 페이지 폴트가 발생한다.

페이지 폴트 발생 시 CPU는 IDT에서 페이지 폴트 처리 함수를 찾아서 인터럽트 스택 프레임을 스택에 넣는다. 그러나 현재 스택 포인터가 여전히 존재하지 않는 가드 페이지를 가리킨다. 그래서 두 번째 페이지 폴트가 발생해서 더블 폴트도 발생한다.

그래서 CPU가 더블 폴트 처리 함수를 호출하면 CPU는 예외 스택 프레임을 스택에 넣으려고 한다. 스택 포인터가 여전히 가드 포인터를 가리키고 있어서 세 번째 페이지 폴트가 발생해서, 트리폴 폴트와 시스템 재부팅이 일어난다. 이와 같은 경우에 현재까지 작성한 더블 폴트 처리 함수는 트리플 폴트를 막을 수 없다.

무한 재귀를 하는 함수를 실행해서 직접 위와 같은 상황을 재현할 수 있다:

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

어떻게 하면 이 문제를 피할 수 있을까? 예외 스택 프레임이 스택으로 들어가는 일은 CPU 자체에서 일어나는 일이기에 막을 수 없다. 그래서 더블 폴트 예외가 일어나도 스택이 항상 유효하도록 조치를 취해야 한다. 다행히도 x86_64 아키텍처에 해결책이 마련되어있다.

## 스택 바꾸기

x86_64 아키택처는 예외 발생 시 미리 정해둔 정상적인 스택으로의 변경이 가능하다. 이 변경은 하드웨어 수준에서 발생하기에, CPU가 예외 스택 프레임을 스택에 넣기 전에 실행된다.

변경 작동방식은 인터럽트 스택 테이블(Interrupt Stack Table, IST)로 구현되어있다. IST는 정상적인 스택을 가리키는 포인터 7개가 들어있는 테이블이다. 러스트 의사 코드를 작성해보면 다음과 같다.

```rust
struct InterruptStackTable {
    stack_pointers: [Option<StackPointer>; 7],
}
```

각각의 예외 처리 함수에 사용할 스택 프레임을 IDT 시작점의 `stack_pointers` 필드를 통해 IST에서 고를 수 있다. 예를 들어, 더플 폴트 처리 시에 IST에 있는 첫 번째 스택을 골라 사용할 수 있다. 그러면 CPU가 더블 폴트가 발생할 때마다 자동으로 그 스택으로 변경한다. 이 스택은 뭔가가 넣어지기 전에 생성되기에, 트리플 폴트를 막아준다.

### IST와 TSS

인터럽트 스택 테이블(IST)은 작업 상태 세그먼트(TSS)라는 오래된 레거시 구조의 일부분이다. TSS는 32비트 모드의 작업에 대한 여러 가지 정보(ex, 프로세서 레지스터 상태)를 담고 있고 하드웨어 컨택스트 스위칭에도 사용되곤 했다. 그런데 하드웨어 컨택스트 스위칭이 64비트 모드에서 더 이상 지원되지 않아서 TSS의 형식은 완전히 변경됐다.

x86_64의 TSS는 작업 관련된 정보를 더 이상 담지 않는다. 대신에, 두 개의 스택 테이블(그중 하나는 IST)을 담는다. 32비트와 64비트 TSS사이의 유일한 공통점은 *I/O 포트 권한 비트맵*이다.

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

CPU는 권한 수준 변경 시 권한 스택 테이블을 사용한다. 예를 들어, CPU가 유저 모드(권한 수준 3)에 있을 때 예외가 발생한 경우 CPU는 보통 예외 처리 함수를 실행하기 전에 커널 모드(권한 수준 0)로 변경한다. 이때, CPU는 권한 스택 테이블의 0번째 스택으로 변경한다.(0이 목표 권한 수준이기 때문이다). 커널엔 유저 모드 프로그램이 아직 없으므로 이 테이블을 잠시 신경 끄도록 한다.

### TSS 생성하기

분리된 더블 폴트 스택을 가진 TSS을 인터럽트 스택 테이블에 새로 만든다. 그러려면 TSS 구조체가 필요한데, 다행히도 `x86_64` 크레이트에 `TaskStateSegment`가 들어있다.

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

아직 메모리 관리 기능을 구현하지 않았으므로, 새 스택에 메모리를 적절하게 할당할 방법이 없다. 대신에, `static mut` 배열을 스택을 저장하는데 당분간 사용한다. 컴파일러가 가변 정적 요소에 접근할 때 데이터 레이스가 없을 거라고 확신할 수 없어서 `unsafe`를 사용했다. 불변 `static` 대신 `static mut`을 써서, 부트로더가 이를 읽기 전용 페이지에 매핑하는 것을 막았다. 나중에 적절한 스택 할당 방법을 사용해서 `unsafe`를 없앨 것이다.

이 더블 폴트 스택은 스택 오버플로를 막아줄 가드 페이지가 없다. 스택 오버플로 발생 시 스택 아래의 메모리가 오염될 수 있으므로, 더블 폴트 처리 시 스택을 많이 사용하는 작업을 할 수 없다.

#### TSS 불러오기

새로 만든 TSS를 CPU가 사용하게 해야 한다. 그런데 TSS가 세그먼테이션 시스템을 사용하므로 작업이 조금 번거롭다. 테이블을 직접 로딩하는 대신에, 글로벌 디스크립터 테이블(GDT)에 새 세그먼트 디스크립터를 추가해야 한다. 그런 다음 각각의 GDT 인덱스를 사용해 `ltr` 명령어로 TSS를 불러올 수 있다. (새 모듈 이름이 `gdt`인 이유다.)

## 글로벌 디스크립터 테이블

글로벌 디스크립터 테이블은 페이징이 표준이 되기 전에 메모리 세그먼테이션에 사용되던 것이 남아 있는 것이다. 커널/유저 모드 설정이나 TSS 불러오기 기능을 위해 64 모드에서는 아직 쓰인다.

GDT는 프로그램의 세그먼트를 담고 있는 구조체다. 옛날 아키텍처에서 프로그램을 각각 분리시키기 위해 사용됐었다. 무료로 볼 수 있는 [“Three Easy Pieces” 책](http://pages.cs.wisc.edu/%7Eremzi/OSTEP/)에서 GDT에 대해 더 자세히 알 수 있다. 세그먼테이션이 64비트에서 사용되지 않지만, GDT는 커널 공간과 유저 공간 사이를 전환하거나 TSS 구조체를 불러오기 위해 아직도 사용된다.

### GDT 만들기

정적 `GDT`를 만들어서 `TSS` 세그먼트를 갖게 한다.

```rust
// src/gdt.rs 안

use x86_64::structures::gdt::{GlobalDescriptorTable, Descriptor};

lazy_static! {
    static ref GDT: GlobalDescriptorTable = {
        let mut gdt = GlobalDescriptorTable::new();
        gdt.add_entry(Descriptor::kernel_code_segment());
        gdt.add_entry(Descriptor::tss_segment(&TSS));
        gdt
    };
}
```

코드 세그먼트와 TSS 세그먼트를 가진 GDT를 만들었다.

#### GDT 불러오기

`GDT`를 불러오는 `gdt::init` 함수를 만들고 이를 `init` 함수에서 호출한다.

```rust
// src/gdt.rs 안

pub fn init() {
    GDT.load();
}

// src/lib.rs 안

pub fn init() {
    gdt::init();
    interrupts::init_idt();
}
```

GDT를 불러왔지만, 부트 루프에서 아직 스택 오버플로가 발생한다.

### 마지막 단계

세그먼트와 TSS 레지스터가 옛날 GDT의 값을 계속 가지고 있기 때문에, GDT 세그먼트가 활성화되지 않아서 문제가 생겼다. 더블 폴트 IDT 시작점이 새로운 스택을 사용하도록 변경한다.

요약하면 다음에 나오는 것 해야한 다.

1. **코드 세그먼트 레지스터 다시 불러오기**: GDT를 바꿨으므로 `cs` 즉 코드 세그먼트 레지스터를 다시 불러와야 한다.
2. **TSS 불러오기**: TSS 선택자를 가진 GDT를 불러오긴 했지만, CPU가 바로 그 TSS를 사용하도록 해야 한다.
3. **IDT 시작점 업데이트하기**: TSS가 로딩되면, CPU가 유효한 인터럽트 스택 테이블(IST)에 접근할 수 있다. 그런 다음 더블 폴트 IDT 시작점을 수정해서 CPU가 새로운 더블 폴트 스택에 접근하도록 해야 한다.

1, 2번은 `gdt::init` 함수에 있는 `code_selector`와 `tss_selectro` 변수에 접근하면 된다. `Selector` 구조체를 생성하고 이를 정적으로 만든다.

```rust
// src/gdt.rs 안

use x86_64::structures::gdt::SegmentSelector;

lazy_static! {
    static ref GDT: (GlobalDescriptorTable, Selectors) = {
        let mut gdt = GlobalDescriptorTable::new();
        let code_selector = gdt.add_entry(Descriptor::kernel_code_segment());
        let tss_selector = gdt.add_entry(Descriptor::tss_segment(&TSS));
        (gdt, Selectors { code_selector, tss_selector })
    };
}

struct Selectors {
    code_selector: SegmentSelector,
    tss_selector: SegmentSelector,
}
```

이제 선택자들을 사용해 `cs` 세그먼트 레지스터와 `TSS`를 다시 불러올 수 있다.

```rust
// src/gdt.rs 안

pub fn init() {
    use x86_64::instructions::segmentation::set_cs;
    use x86_64::instructions::tables::load_tss;

    GDT.0.load();
    unsafe {
        set_cs(GDT.1.code_selector);
        load_tss(GDT.1.tss_selector);
    }
}
```

`set_cs`로 코드 세그먼트 레지스터를, `load_tss`로 TSS를 다시 불러왔다. 각 함수가 `unsafe`로 표시됐으므로 `unsafe` 블록에서 함수를 호출한다. 유효하지 않은 선택자를 불러올 시 메모리 안정성이 저해될 수 있기 때문이다.

유효한 TSS와 인터럽트 스택 테이블을 불러왔기에 IDT 내부의 더블 폴트 처리 함수에 사용할 스택 인덱스를 설정할 수 있다.

```rust
// src/interrupts.rs 안

use crate::gdt;

lazy_static! {
    static ref IDT: InterruptDescriptorTable = {
        let mut idt = InterruptDescriptorTable::new();
        idt.breakpoint.set_handler_fn(breakpoint_handler);
        unsafe {
            idt.double_fault.set_handler_fn(double_fault_handler)
                .set_stack_index(gdt::DOUBLE_FAULT_IST_INDEX); // new
        }

        idt
    };
}
```

`set_stack_index` 메서드는 호출하는 사람이 유효하고 다른 예외에서 사용되지 않은 스택 인덱스를 사용하도록 `unsafe` 표시가 되어있다.

다 됐다! 이제 CPU는 더블 폴트가 발생할 때마다 현재 스택을 더블 폴트 스택으로 변경할 수 있다. 그러므로 커널 스택 오버플로를 포함한 모든 더블 폴트를 잡을 수 있게 됐다.

![image](https://user-images.githubusercontent.com/22253556/80093941-5a08a900-85a0-11ea-93fe-0ea2b2381903.png)

커널을 실행해보면 트리플 폴트가 발생하지 않는다.

## 스택 오버플로 테스트

`gdt` 모듈을 테스트하고 더블 폴트 처리 함수가 스택 오버플로 발생 시 잘 호출되는지 확인하기 위한 통합 테스트를 추가한다. 테스트 함수에서 더블 폴트를 일으켜서 더블 폴트 처리 함수가 호출되는지 확인해본다.

다음과 같이 테스트 코드 구조를 잡는다.

```rust
// in tests/stack_overflow.rs

#![no_std]
#![no_main]

use core::panic::PanicInfo;

#[no_mangle]
pub extern "C" fn _start() -> ! {
    unimplemented!();
}

#[panic_handler]
fn panic(info: &PanicInfo) -> ! {
    blog_os::test_panic_handler(info)
}
```

`panic_handler` 테스트에서 처럼, 테스트 도구(test harness)를 사용하지 않고 테스트를 실행할 것이다. 왜냐하면 더블 폴트 발생 후에는 실행을 계속할 수 없기 때문이다. 그래서 테스트를 1개 이상으로 실행시킬 수가 없다. `Cargo.toml`에 다음 코드를 추가해서 테스트 도구를 끈다.

```toml
# Cargo.toml 안

[[test]]
name = "stack_overflow"
harness = false
```

`cargo xtest --test stack_overflow`를 실행하면 컴파일에는 성공하지만 `unimplemented` 매크로 때문에 패닉이 일어나 테스트는 실패한다.

### `_start` 구현하기

다음과 같이 `_start` 함수를 구현한다.

```rust
// tests/stack_overflow.rs 안

use blog_os::serial_print;

#[no_mangle]
pub extern "C" fn _start() -> ! {
    serial_print!("stack_overflow... ");

    blog_os::gdt::init();
    init_test_idt();

    // trigger a stack overflow
    stack_overflow();

    panic!("Execution continued after stack overflow");
}

#[allow(unconditional_recursion)]
fn stack_overflow() {
    stack_overflow(); // for each recursion, the return address is pushed
}
```

`gdt::init` 함수로 새로운 GDT를 초기화한다. 작성한 `interrupts::init_idt` 함수 대신 나중에 설명할 `init_test_idt` 함수를 호출한다. 패닉 하는 대신 `exit_qemu(QemuExitCode::Success)`를 실행할 커스텀 더블 폴트 처리 함수를 등록하기 위해서다.

`stack_overflow` 함수에는 `allow(unconditional_recursion)` 속성을 추가해서 함수가 무한히 재귀한다는 경고를 껐다.

### IDT 테스트하기

커스텀 더블 폴트 처리 함수를 가진 IDT를 만들어서 테스트에 사용해야 한다. 구현은 다음과 같다.

```rust
// tests/stack_overflow.rs 안

use lazy_static::lazy_static;
use x86_64::structures::idt::InterruptDescriptorTable;

lazy_static! {
    static ref TEST_IDT: InterruptDescriptorTable = {
        let mut idt = InterruptDescriptorTable::new();
        unsafe {
            idt.double_fault
                .set_handler_fn(test_double_fault_handler)
                .set_stack_index(blog_os::gdt::DOUBLE_FAULT_IST_INDEX);
        }

        idt
    };
}

pub fn init_test_idt() {
    TEST_IDT.load();
}
```

구현을 보면 `interrupts.rs`의 일반적인 IDT와 매우 비슷하다. 일반적인 IDT처럼 IST에 더블 폴트 처리 함수를 등록하고 스택 인덱스를 설정해서 별도의 스택으로 전환이 가능하게 했다. `init_test_idt` 함수는 IDT를 `load` 메서드로 CPU에 불러온다.

### 더플 폴트 처리 함수

마지막으로 남은 더블 폴트 함수는 다음과 같이 구현한다.

```rust
// tests/stack_overflow.rs 안

use blog_os::{exit_qemu, QemuExitCode, serial_println};
use x86_64::structures::idt::InterruptStackFrame;

extern "x86-interrupt" fn test_double_fault_handler(
    _stack_frame: &mut InterruptStackFrame,
    _error_code: u64,
) -> ! {
    serial_println!("[ok]");
    exit_qemu(QemuExitCode::Success);
    loop {}
}
```

더블 폴트 처리 함수 실행 시, 테스트 통과를 나타내는 성공 종료 코드와 함께 QEMU를 종료한다. 통합 테스트는 완전히 분리된 실행 파일이므로, 테스트 파일 맨 위에 `#![feature(abi_x86_interrupt)]` 속성을 추가해야 한다.

이제 `cargo xtest --test stack_overflow`로 (모든 테스트를 실행하려면 `cargo xtest`로) 테스트를 실행할 수 있다. 실행 시 `stack_overflow... [ok]`가 콘솔에 출력된다. `set_stack_index`을 주석 처리하면 테스트는 실패할 것이다.

## 요약

이번 장에서 더블 폴트의 정의와 발생 조건을 알아봤다. 에러 메시지를 출력하는 기본적인 더블 폴트 처리 함수를 추가하고 이를 테스트하기 위한 통합 테스트를 추가했다.
