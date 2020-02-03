# 러스트의 슬라이스

컬렉션의 일부 연속 요소를 참조하는데 사용한다.

## 스트링 슬라이스

슬라이스는 `[start..end]` 문법을 사용하고 `start`로 시작해 `end`는 포함하지 않는 연속 범위를 나타낸다.

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

## 0으로 시작하면 `start` 생략 가능

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

## 끝까지 포함하면 `end` 생략 가능

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

## 전체 스트링의 슬라이스를 만든다면 양쪽 값 생략 가능

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

## 스트링 슬라이스를 나타내는 타입

`&str`

## 스트링 리터럴 == 슬라이스

아래 `s`의 타입은 `&str`이다.

```rust
let s = "Hello, world!";
```

## 스트링 슬라이스를 파라미터로 사용하기

아래 함수 시그내쳐를

```rust
fn first_word(s: &String) -> &str {
```

아래와 같이 작성할 수 있다.

```rust
fn first_word(s: &str) -> &str {
```

스트링 슬라이스를 넘긴다면 그대로 넘기고
`String`을 넘긴다면 `String`의 전체 슬라이스를 넘긴다.

## 배열의 슬라이스

아래 슬라이스를 `&[i32]` 타입을 갖는다.

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];
```
