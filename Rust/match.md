## match 흐름 제어 연산자

패턴으로 값에 따라 코드를 수행할 수 있다.

패턴에 리터럴 값, 변수명, 와일드카드 등을 사용할 수 있다.

`match` 덕분에 컴파일러는 열거형의 모든 경우가 다뤄지는지 검사할 수 있다.

## `Option<T>`를 이용하는 매칭

`Option<T>`는 아래와 같이 생겼다.

```rust
enum Option<T> {
    Some(T),
    None,
}
```

아래는 `Option<i32>`를 파라미터로 받아서 값이 있으면 1을 더하는 `plus_one` 함수다.

값이 없으면 계산이 계속 `None` 값으로 진행된다.

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

## 모든 가능성을 다루는 `match`

아래와 같이 위의 `plus_one` 함수에 `None` 케이스를 다루지 않으면

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
    }
}
```

컴파일러는 `` non-exhaustive patterns: `None` not covered ``에러를 낸다.

## `_` 플레이스홀더

가능한 값 모두를 나열하고 싶지 않다면 `_` 패턴을 사용한다.

`u8`은 0에서 255까지의 값을 갖는다.

1, 3, 5, 7 값만 사용하고 나머지는 신경 쓰고 싶지 않다면

`_` 패턴으로 0, 2, 4, 6, 8, 9, ..., 255까지의 값을 대신할 수 있다.

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```

## 한 가지 경우만 고려하기

한 가지 경우만 고려하기에 `match`는 보일러 플레이트가 많다.

그럴 경우 `if let`을 사용한다.
