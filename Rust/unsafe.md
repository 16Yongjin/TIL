# 러스트 고급 기능

- Unsafe Rust: 러스트의 손을 떠나 수동으로 안전을 보장하기
- 고급 라이프타임
- 고급 트레잇: 연관 타입, 기본 타입 파리미터, 완전 정규화 문법, 슈퍼 트레잇, ..
- 고급 타입: 신종 타입 패턴, 타입 별칭, `never` 타입, 동적 크기 조절 타입
- 고급 함수 및 클로저: 함수 포인터와 클로저 반환하기

# 안전하지 않은 러스트

컴파일 타임에 강제되는 메모리 안전성을 해제하고 코드를 작성할 수 있다.

## 안전하지 않은 슈퍼파워

안전하지 않은 코드를 작성하기 위해 `unsafe` 키워드를 사용한다.

안전하지 않은 러스트는 아래 4가지 행동이 가능하다.

- 로우 포인터 역참조하기
- 안전하지 않은 함수나 메서드 호출하기
- 가변 정적 변수 접근과 수정하기
- 안전하지 않은 트레잇 구현하기

`unsafe` 내의 코드가 무저건 위험한 것은 아니다.

`unsafe`는 그 안의 코드가 올바른 방법으로 메모리에 접근하겠다는 것을 명시하는 것이다.

안전하지 않은 코드를 안전한 API로 추상화할 수도 있다.

## 로우 포인터 역참조하기

로우 포인터는 참조자 처럼 불변이나 가변 타입을 가진다.

불변은 `*const T`로 가변은 `*mut T`라고 쓴다.

여기서 애스터리스크 `*`는 역참조가 아니라 타입명의 일부다.

로우 포인터의 *불변*은 포인터가 역참조된 후에 직접 대입될 수 없음을 의미한다.

## 로우 포인터의 성질

- 로우 포인터는 빌림 규칙을 무시할 수 있어서 포인터가 불변과 가변 성질을 다 갖거나 다수의 가변 포인터가 존재할 수 있다.
- 로우 포인터가 가리키는 메모리가 유효한지 알 수 없다.
- 널이 될 수 있다.
- 메모리 정리가 자동으로 되지 않는다.

안정성을 포기하고 성능을 향상하거나 다른 언어나 하드웨어와 상호작용을 할 수 있다.

아래 코드는 참조자에서 불변, 가변 로우 포인터를 만든다.

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;
```

위 코드는 유효한 참조자에서 로우 포인터를 만들어서, 로우 포인터가 유효함을 알 수 있다.

반면, 다음과 같이 임의의 위치를 가리켜서 유효한지 알 수 없는 로우 포인터를 만들 수 있다.

```rust
let address = 0x012345usize;
let r = address as *const i32;
```

안전한 코드 내에서는 로우 포인터 생성은 가능해도, 로우 포인터를 역참조해서 해당 포인터가 가리키는 데이터는 읽을 수 없다.

다음 코드는 `unsafe` 블록 내에서 로우 포인터의 값을 사용한다.

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;

unsafe {
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}
```

위에 처럼 동일한 메모리를 가리키는 로우 포인터를 여러개 만들 수 있기 때문에, 데이터 레이스를 유의해야 한다.

## 안전하지 않은 함수나 메소드 호출하기

안전하지 않은 함수는 앞에 `unsafe`가 붙어 있다.

이 함수들은 다음과 같이 항상 `unsafe` 블록 내에서 호출되어야 한다.

```rust
unsafe fn dangerous() {}

unsafe {
    dangerous();
}
```

함수 본문는 `unsafe` 블록이므로, 본문 내에서 안전하지 않은 함수 사용 시 `unsafe` 블록을 추가할 필요 없다.

## 안전하지 않은 코드 상에 안전한 추상화 생성하기

함수가 `unsafe` 코드를 가지고 있어도 안전할 수 있다.

슬라이스를 쪼개서 둘로 만드는 `split_at_mut` 메서드는 다음과 같이 사용한다.

```rust
let mut v = vec![1, 2, 3, 4, 5, 6];

let r = &mut v[..];

let (a, b) = r.split_at_mut(3);

assert_eq!(a, &mut [1, 2, 3]);
assert_eq!(b, &mut [4, 5, 6]);
```

러스트를 안전하게만 사용하면 이 메서드를 구현할 수 없다.

`split_at_mut`를 `i32` 타입만 지원하는 함수로 구현해본다.

```rust
fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();

    assert!(mid <= len);

    (&mut slice[..mid],
     &mut slice[mid..])
}
```

컴파일 시 `` cannot borrow `*slice` as mutable more than once at a time `` 에러가 발생한다.

슬라이스의 서로 다른 부분을 빌리고 있지만, 러스트는 같은 슬라이스를 두 번 빌리고 있다고 생각한다.

`unsafe` 블록과 함수, 로우 포인터를 사용해서 함수를 동작하게 만들 수 있다.

```rust
use std::slice;

fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (slice::from_raw_parts_mut(ptr, mid),
         slice::from_raw_parts_mut(ptr.offset(mid as isize), len - mid))
    }
}
```

변수 `ptr`은 `*mut i32` 타입 로우 포인터를 갖는다.

`slice::from_raw_parts_mut` 함수는 `ptr`로 시작하는 `mid` 길이의 슬라이스를 생성한다.

`assert!(mid <= len);`로 `unsafe` 블록 내의 모든 로우 포인터들이 슬라이스 안의 데이터를 가리킴을 보장한다.

`split_at_mut` 함수는 안전하기에 `unsafe` 표시를 하지 않아도 된다.

## `extern` 함수로 외부 코드 호출하기

`extern` 키워드로 Foreign Function Interface를 생성하고 사용할 수 있다.

다음 코드는 C 표준 라이브러리의 `abs` 함수를 가져올 수 있도록 설정한다.

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

`"C"` 부분은 해당 외부 함수의 ABI(application binary interface)를 정의해서 어셈블리 수준에서 어떻게 함수를 호출할 지 정한다.

`"C"` ABI는 C 언어의 ABI를 준수한다.

## [참고] 다른 언어에서 러스트 함수 호출하기

```rust
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

`#[no_mangle]` 어노테이션으로 컴파일 최적화 과정에서 함수의 이름이 변경되는 것을 방지한다.

`extern` 키워드 뒤에 ABI를 명시하고 `fn` 키워드 뒤에 함수 시그내처를 작성한다.

다른 언어에 함수를 노출하기 위한 `extern` 사용 시 `unsafe`가 필요 없다.

## 가변 정적 변수 접근하고 수정하기

러스트에서 전역 변수를 사용할 수는 있다.

하지만, 전역 변수는 소유권 문제를 일으킬 뿐만 아니라, 다수의 스레드에서 동일한 전역 변수 접근 시 데이터 레이스 문제가 발생할 수 있다.

러스트는 전역 변수를 정적 변수라고 부른다.

```rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
```

## 상수와 다른 정적 변수의 특징

- 변수 이름은 반드시 대문자 스네이크 케이스이다.
- 타입이 반드시 명시돼야 한다.
- 정적 변수의 참조자는 `'static` 라이프타임을 갖는다.
- 값이 메모리 내의 고정된 주소값을 갖고 접근하는 값의 주소가 항상 동일하다. (상수는 사용될 때마다 복사됨)
- 가변으로 선언할 수 있다.

다음 코드는 가변 정적 변수를 선언 후 접근하고 수정한다.

```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

`static mut` 키워드로 가변 정적 변수를 선언한다.

`unsafe` 블록 내에서만 가변 정적 변수를 읽거나 쓸 수 있다.

멀티 스레드 환경에서는 가변 정적 변수보다 스레드 세이프한 스마트 포인터를 사용하는 편이 낫다.

## 안전하지 않은 트레잇 구현하기

`trait` 키워드 앞에 `unsafe`를 붙여서 안전하지 않은 트레잇을 만든다.

```rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}
```

내부 타입들이 모두 `Sync`와 `Send`하다면 자동으로 `Sync`와 `Send` 트레잇이 구현된다.

`Sync`와 `Send`하지 않은 타입에 `Sync`와 `Send` 표시를 하려면 `unsafe`를 사용해야 한다.
