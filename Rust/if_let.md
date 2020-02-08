# `if let`을 사용한 간결한 흐름제어

하나의 패턴만 매칭하고 나머지는 무시하기

## `match`를 사용한 경우

`Option<u8>`에서 값이 3인 경우에만 코드를 실행하고 싶다면 아래와 같이 코드를 작성한다.

```rust
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}
```

한 가지 경우만 사용하지만 `match` 표현식을 만족하기 위해 너무 많은 보일러 플레이트를 작성하고 있다.

## `if let` 사용하기

`if let`을 사용하면 타이핑을 줄일 수 있다.

```rust
if let Some(3) = some_u8_value {
    println!("three");
}
```

다만 `match`가 강제했던 빠짐없는 검사를 잃게 된다.
