# CPU 예외

CPU 예외는 잘못된 메모리 접근이나 0으로 나누기 같은 에러 상황에서 발생한다.

예외처리를 위해 인터럽트 디스크립터 테이블(IDT)에 처리 함수들을 준비해놓는다.

커널이 중단점 예외를 처리하고 다시 일반적인 실행 상태로 돌아오게 만들어본다.

## 개요

현재 실행 중인 명령어에 문제가 있으면 예외 신호가 발생한다. (ex, 0으로 나누기)

예외 발생 시, CPU는 하고 있는 작업을 멈추고 예외에 맞는 처리 함수를 즉시 호출한다.

x86에는 CPU 예외가 20개 정도 있는데, 다음에 나오는 것들이 가장 중요하다.

- 페이지 폴트: 메모리에 잘못 접근하면 발생한다. 현재 명령어가 매핑되지 않은 페이지를 읽거나 읽기 전용 페이지를 읽을려고 할 때 발생한다.

- 유효하지 않은 Opcode: 잘못된 명령어 실행 시 발생한다. 새 SSE 명령어를 지원되지 않는 오래된 CPU에 사용했을 때 발생한다.

- 일반 보호 장애(General Protection Fault): 다양한 원인으로 발생한다. 유저 레벨에서 권한 없는 명령어를 실행하거나 설정 레지스터의 보호 필드에 쓰기를 하려고 하면 발생한다.

- 더블 폴트: 예외 처리 함수 실행 중에 예외 발생 시 CPU는 더플 폴트 예외를 발생시킨다. 예외를 처리할 함수가 없을 때 발생한다.

- 트리플 폴트: 더플 폴트 처리 함수 실행 중에 예외 발생 시 치명적인 트리플 폴트 예외가 발생한다. 트리플 폴트는 잡을 수 없어서 대부분의 CPU는 리셋이나 운영체제 리부팅으로 예외에 대응한다.

## 인터럽트 디스크립터 테이블

각 요소는 다음의 16 bit 구조를 갖는다.

| 타입 | 이름                | 설명                                                 |
| ---- | ------------------- | ---------------------------------------------------- |
| u16  | 함수 포인터 [0:15]  | 처리 함수를 가리키는 포인터의 낮은 비트              |
| u16  | GDT 선택자          | 글로벌 디스크립터 테이블에 있는 코드 세그먼트 선택자 |
| u16  | 옵션                | (아래에서 설명함)                                    |
| u16  | 함수 포인터 [16:31] | 처리 함수를 가리키는 포인터의 중간 비트              |
| u32  | 함수 포인터 [32:63] | 처리 함수를 가리키는 포인터의 나머지 비트            |
| u32  | 예약됨              |                                                      |

옵션 필드는 아래와 같은 포맷이다.

| 비트  | 이름                               | 설명                                                                                                 |
| ----- | ---------------------------------- | ---------------------------------------------------------------------------------------------------- |
| 0-2   | 인터럽트 스택 테이블 인덱스        | 0: 스택을 전환하지 말기, 1-7: 처리 함수 호출 시 인터럽트 스택 테이블에 있는 n번째 스택으로 전환하기. |
| 3-7   | 예약됨                             |                                                                                                      |
| 8     | 0: 인터럽트 게이트, 1: 트랩 게이트 | 이게 0이면 처리 함수 호출 시 인터럽트가 취소된 거임                                                  |
| 9-11  | 반드시 1임                         |                                                                                                      |
| 12    | 반드시 0임                         |                                                                                                      |
| 13‑14 | Descriptor Privilege Level (DPL)   | 이 처리 함수 호출에 필요한 최소 권한 수준.                                                           |
| 15    | Present                            |                                                                                                      |

모든 예외는 IDT 인덱스가 정해져 있다.

유효하지 않은 Opcode 예외는 6번 테이블 인덱스를, 페이지 폴트 예외는 14번 테이블 인덱스를 가진다.

이렇게 해서, 하드워어가 예외 발생 시 해당하는 IDT 시작점을 불러올 수 있다.

### 예외 발생 시 CPU의 대략적인 동작

1. 명령어 포인터와 RFLAGS 레지스터 등을 스택에 넣는다.
2. IDT에서 해당하는 시작점을 읽는다 (예, 페이지 폴트 발생 시 14번째 시작점 읽음)
3. 시작점이 있는지 확인하고 없으면 더블 폴트를 일으킨다.
4. 시작점이 인터럽트 게이트라면 하드웨어 인터럽트를 끈다.
5. 지정한 GDT 선택자를 CS 세그먼트로 불러온다.
6. 지정된 처리 함수로 점프한다.

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

여기서 타입 파라미터 `F`는 필요한 처리 함수 타입을 정의한다.

시작점들이 `HandlerFunc`나 `HandlerFuncWithErrCode`를 필요로 하고 페이지 플트만 `PageFaultHandlerFunc`를 필요로 한다.

`HandlerFunc`는 다음과 같이 생겼다.

```rust
type HandlerFunc = extern "x86-interrupt" fn(_: &mut InterruptStackFrame);
```

`HandlerFunc`는 `extern "x86-interrupt" fn`의 타입 별칭이다.

`extern` 키워드는 `foreign calling convention`을 따르는 함수를 정의할 때 사용되며, C 코드와 사용되기도 한다.

# Interrupt Calling Convention

예외 처리는 함수 호출 동작과 비슷하다.

CPU가 함수의 첫 번째 명령어로 점프한 뒤 이를 실행한다.

그 다음에 CPU가 반환 주소로 점프하고 부모 함수의 실행을 이어간다.
