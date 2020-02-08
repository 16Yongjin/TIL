# 열거형

하나의 타입이 가질 수 있는 값을 열거한다.

## 열거형 정의하기

IP 주소 버전을 나타내는 `IpAddrKind`

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

## 열거형 값

`IpAddrKind`의 인스턴스는 다음과 같이 만든다.

```rust
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

## 열거형에 직접 데이터 붙이기

열거형의 각 변종(variant)에 데이터를 넣어서 구조체 사용을 대신할 수 있다.

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```

## 열거형과 구조체가 비슷한 점

아래와 같이 열거형을 정의하는 것은

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

아래와 같은 구조체들을 정의하는 것과 같다.

```rust
struct QuitMessage; // 유닛 구조체
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // 튜플 구조체
struct ChangeColorMessage(i32, i32, i32); // 튜플 구조체
```

다만 이와 같이 여러 개의 구조체를 정의하면, 함수를 정의할 때 한 가지 인수로 받기 어렵다.

연관된 구조체들은 열거형으로 묶어주는게 좋다.

## 열거형 메서드

구조체처럼 열거형에도 메서드를 정의할 수 있다.

열거형 값을 가져오기 위해 `self`를 사용하는 것도 같다.

```rust
impl Message {
    fn call(&self) {
        // 메소드 내용은 여기 정의할 수 있습니다.
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

## Option 열거형

러스트에는 `null`이 없다.

대신, 값이 존재하거나 부재하는 것을 표현하기 위해 `Option<T>` 열거형을 사용한다.

```rust
enum Option<T> {
    Some(T),
    None,
}
```
