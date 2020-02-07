# 러스트의 구조체

연관된 여러 값을 묶어서 의미있는 데이터 단위 정의한다.

## 구조체 만들기

`struct` 키워드와 필드, 타입을 입력하면 구조체를 만들 수 있다.

아래는 사용자 계정 정보를 저장하는 구조체

```rust
struct User {
  username: String,
  email: String,
  sign_in_count: u64,
  active: bool,
}
```

## 구조체 사용하기

각 필드 값을 입력한 인스턴스를 만들어서 사용한다.

```rust
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

## 필드 초기화 축약

변수명과 구조체의 필드명이 같으면 축약해서 필드를 초기화할 수 있다.

아래 코드에서 매개변수와 구조체 필드명이 `email`과 `username`로 같아서 축약 초기화를 했다.

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

ES6의 객체 리터럴 축약 표기와 같다.

## 기존 구조체 인스턴스 재활용해서 새 구조체 인스턴스 만들기

`user2`는 `email`과 `username` 필드에는 새 값을 할당하고

나머지 필드는 `user1`에서 재사용했다.

```rust
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1
};
```

ES6의 객체 전개와 같다.

## 필드 이름이 없는 튜플 구조체

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

`black`과 `origin`은 구조체 내 타입이 모두 동일해도 서로 다른 타입이다.

## 필드가 없는 유사 유닛 구조체

유닛 타입인 `()`와 비슷하다.

필드가 없다.

특정한 타입의 트레잇(Trait)을 구현해야 되는데 타입 자체에 데이터를 저장하지 않고 싶을 때 유용하다.

## 구조체 출력하기

```rust
struct Rectangle {
    length: u32,
    width: u32,
}
```

이 구조체를 출력하려고 할 때

```rust
fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };

    println!("rect1 is {}", rect1);
}
```

`println!`에 `{}`만 사용하면 `Rectangle`의 기본 포맷터가 없다고 에러가 나고

`Debug`라는 출력포맷을 사용하겠다는 뜻을 나타내는 `{:?}`를 사용하라고 한다.

```rust
println!("rect1 is {:?}", rect1);
```

위와 같이 `{:?}`을 사용하면 에러가 나면서
`#[derive(Debug)]`어노테이션을 구조체 정의 앞에 붙이라고 한다.

```rust
#[derive(Debug)]
struct Rectangle {
    length: u32,
    width: u32,
}
```

어노테이션을 붙이면 출력이 잘 된다.

```
rect1 is Rectangle { length: 50, width: 30 }
```

`{:?}` 대신에 `{:#?}`를 사용하면 아래와 같이 보기 좋게 구조체를 출력해줘서 큰 구조체를 디버깅할 때 도움이 된다.

```
rect1 is Rectangle {
    length: 50,
    width: 30
}
```
