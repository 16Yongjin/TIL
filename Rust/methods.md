# 러스트의 메서드와 연관함수

구조체 내에 정의되는 함수

첫번째 파라미터는 언제나 `self`이다. (파이썬 같다.)

`self`는 메서드가 호출되는 구조체의 인스턴스를 가리킨다.

## 메서드 정의하기

사각형을 나타내는 `Rectangle` 구조체가 있다.

```rust
struct Rectangle {
    length: u32,
    width: u32,
}
```

이 구조체의 넓이를 구하는 `area` 메서드를 만든다.

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.length * self.width
    }
}
```

`impl` 키워드를 사용해서 메서드를 정의한다.

`&self` 파라미터로 구조체 인스턴스 소유권을 빌려서 사용한다.

> 메서드가 `self`의 타입을 가진 덕분에 참조 및 역참조가 자동으로 동작한다. 덕분에 C++ 처럼 `->` 연산자를 사용할 필요 없다.

## 연관함수

`impl` 블록 내에 `self` 파라미터를 갖지 않는 함수도 정의할 수 있다.

함수가 동작할 때 구조체 인스턴스가 필요없지만 해당 구조체와 관련이 있어서 연관함수라고 불린다.

연관함수는 `String::from` 같이 새 구조체 인스턴스를 반환하는 생성자로 많이 쓴다.

정사각형 `Rectangle`을 만들 때 아래와 같이 연관함수를 만들어두면 편하다.

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { length: size, width: size }
    }
}
```

사용은 `let sq = Rectangle::square(3);` 같이 구조체::함수 형태이다.
