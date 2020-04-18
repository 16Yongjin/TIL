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

    // trigger a page fault
    unsafe {
        *(0xdeadbeef as *mut u64) = 42;
    };

    // as before
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

대부분의 더블 폴트를 잡을 수 있지만, 현재 방식으로 잡지 못하는 더블 폴트가 있다.
