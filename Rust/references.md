# 러스트 참조자와 빌림

소유권을 넘기지 않고 참조자를 통해 함수에 인자를 넘길 수 있다.

## 참조자 규칙

둘 중 하나만 가질 수 있다.

1. 하나의 가변 참조자
2. 여러 개의 불변 참조자

## 불변 참조자 문법

엠퍼센드(&) 기호로 참조자를 설정할 수 있다.

`calculate_length` 함수가 `s`가 문자열 참조자라는 것을 나타낸다.

```rust
fn calculate_length(s: &String) -> usize {
    s.len()
}
```

## 불변으로 빌린 참조자는 변경할 수 없다.

아래 코드는 불변 참조자를 변경하려고 해서 `` cannot borrow immutable borrowed content `*some_string` as mutable ``라는 에러가 발생한다.

```rust
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

## 가변 참조자 문법

위의 코드에서 `s`를 `mut`로 바꾸고 함수 인자의 타입 앞에도 `&mut`를 붙이면 가변 참조자를 만들 수 있다.

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

## 가변 참조자는 하나만 만들 수 있다.

아래 코드는 가변 참조자를 두 개나 만들려고 해서 `` cannot borrow `s` as mutable more than once at a time ``라는 컴파일 에러가 난다.

```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;
```

이 제한사항 덕분에 데이터 레이스를 방지할 수 있다.

## 데이터 레이스 조건

1. 두 개 이상의 포인터가 동시에 같은 데이터에 접근한다.
2. 그 중 적어도 하나의 포인터가 데이터를 수정한다
3. 데이터 접근 동기화 메커니즘이 없다.

러스트는 데이터 레이스가 발생할 수 있는 코드를 컴파일 시키기 않아서 문제가 발생하지 않는다.

## 불변 참조자와 가변 참조자를 동시에 만들 수 없다.

아래 코드는 불변 참조자를 2개 만들고(여기까지는 괜찮음), 가변 참조자까지 만들어서 `` cannot borrow `s` as mutable because it is also borrowed as immutable ``라는 에러가 난다.

```rust
let mut s = String::from("hello");

let r1 = &s; // 문제 없음
let r2 = &s; // 문제 없음
let r3 = &mut s; // 큰 문제
```

## 댕글링 참조자(Dangling References)

댕글링 포인터란 이미 해제된 메모리를 가리키는 포인터다.

러스트의 라이프타임 특성이 댕글링 참조자가 생기는 것을 막아주는데 이는 나중에 다룬다.

아래 `dangle` 함수에서 `s`는 함수 스코프를 벗어나면 없어지는데 `s`의 참조자를 반환하려고 한다.

```rust
fn dangle() -> &String {

    let s = String::from("hello");

    &s
}
```

코드를 실행하면 `this function's return type contains a borrowed value, but there is no value for it to be borrowed from.` (해석: 이 함수의 반환 타입은 빌린 값을 포함하고 있는데, 빌려온 실제 값은 없습니다.)라는 에러를 낸다.
