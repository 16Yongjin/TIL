# 러스트의 트레잇

트레잇은 러스트 컴파일러에게 특정 타입이 다른 타입과 공유하는 기능 있음을 알려준다.

제네릭 타입이 특정 트레잇을 구현해야 한다고 명시해서 트레잇 바운드를 만들 수 있다.

## 트레잇 정의하기

한 타입의 **동작**은 해당 타입에서 호출 가능한 메서드로 이루어져 있다.

타입이 달라도 동일한 메서드를 호출할 수 있으면, 타입들이 동일한 동작을 공유하는 것이다.

트레잇은 같은 목적을 가진 동작을 정의하기 위해 메서드 시그니처를 묶는 것이다.

&nbsp;

`NewsArticle` 구조체는 뉴스를 가지고 있고 `Tweet` 구조체는 140글자와 트윗 정보를 나타내는 메타데이터를 가지고 있다.

각 인스턴스에 `summary` 메서드를 호출하면 요약된 내용이 나오게 하고 싶다.

이러한 개념을 `Summarizable` 트레잇으로 나타낸다.

```rust
pub trait Summarizable {
    fn summary(&self) -> String;
}
```

`summary` 메서드 시그니처를 선언만 하고 몸체는 정의하지 않았다.

컴파일러는 `Summarizable` 트레잇을 갖는 모든 타입이 `summary` 메서드를 갖고 있도록 강제할 수 있다.

## 특정 타입에 대해 트레잇 구현하기

아래는 `NewsArticle` 구조체와 `Tweet` 구조체의 `Summariable` 트레잇 구현이다.

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summarizable for NewsArticle {
    fn summary(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summarizable for Tweet {
    fn summary(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

`impl` 뒤에 구현할 트레잇 이름을 넣고, `for`과 트레잇을 구현하려는 타입의 이름을 쓴다.

트레잇을 구현하면 `NewsArticle`와 `Tweet` 인스턴스에서 `summary` 메서드를 호출할 수 있다.

```rust
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summary());
```

## 트레잇 구현 시 제한사항

트레잇이나 타입이 내 크레이트 안에서 작성된 경우만 해당 타입의 트레잇을 정의할 수 있다.

외부 타입에 대한 외부 트레잇 구현은 불가능하다.

예를 들어, `Vec`에 대한 `Display` 트레잇은 구현 불가능하다.

둘 다 외부인 표준 라이브러리에 정의되어 있기 때문이다.

`Tweet`(내부) 타입의 `Display`(외부) 트레잇 구현은 가능하다.

`Vec`(외부)에 대한 `Summarizable`(내부) 구현도 가능하다.

두 개의 다른 크레이트에서 동일한 타입에 동일한 트레잇을 구현해서 충돌을 일으키는 일을 방지하기 위해 러스트는 고아 규칙을 강제한다.

## 기본 구현

모든 타입의 트레잇 구현에 커스텀 동작을 정의하지 않아도 사용할 수 있는 기본 동작을 정의할 수 있다.

```rust
pub trait Summarizable {
    fn summary(&self) -> String {
        String::from("(Read more...)")
    }
}
```

`Summarizable` 트레잇 구현체들은 `summary` 메서드를 그대로 사용하거나 오버라이드할 수 있다.

&nbsp;

기본 구현으로 동일한 트레잇 내의 다른 메서드를 호출할 수 있다.

호출하는 다른 메서드의 기본 구현이 없어도 된다.

아래 코드에서 `summary` 메서드는 `author_summary`를 호출한다.

```rust
pub trait Summarizable {
    fn author_summary(&self) -> String;

    fn summary(&self) -> String {
        format!("(Read more from {}...)", self.author_summary())
    }
}
```

`Summarizable` 구현체에서 `author_summary` 만 구현해주면 `summary` 메서드를 사용할 수 있다.

다만, 오버라이딩된 구현에서 기본 구현을 호출하는 것은 불가능하다.

## 트레잇 바운드

제너럭 타입 파라미터를 사용하는 트레잇을 사용할 수 있다.

특정한 트레잇(동작)을 가진 제네릭 타입만 입력 받을 수 있도록 한다.

&nbsp;

아래는 `item`의 `summary` 메서드를 호출하는 `notify` 함수이다.

`T`에 대한 트레잇 바운드를 사용해서, `item`이 `Summarizable` 트레잇을 구현한 타입임을 확실히 해야 컴파일 에러가 발생하지 않는다.

```rust
pub fn notify<T: Summarizable>(item: T) {
    println!("Breaking news! {}", item.summary());
}
```

`<T: Summarizable>` 덕분에 `summary` 메서드가 있는 `NewsArticle`, `Tweet` 의 인스턴스를 파라미터로 넘길 수 있다.

`Summarizable`를 구현하지 않는 `i32`나 `String`을 파라미터로 넘기면 컴파일이 안 될 것이다.

## `+`로 트레잇 바운드 체인

`+`를 사용해 제네릭 타입에 여러 개의 트레잇 바운드를 설정할 수 있다.

타입 `T`가 `summary` 뿐만 아니라 형식화된 출력이 가능하길 원하면
`T: Summarizable + Display`와 같이 트레잇 바운드를 설정하면 된다.

## `where` 절로 트레잇 바운드 옮겨 적기

`<>` 사이에 트레잇 바운드 정보를 너무 많이 적으면 코드 읽기 어려워 진다.

아래와 같은 코드 대신

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 { ... }
```

`where` 절을 사용해서 트레잇 바운드를 뒤로 옮길 수 있다.

```rust
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{ ... }
```

## 트레잇 바운드를 사용하여 `largest` 함수 고치기

`largest` 함수는 이렇게 생겼었고 타입 `T`에 `std::cmp::PartialOrd` 트레잇 바운드가 없어서 컴파일에 실패한다.

```rust
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

```

아래와 같이 `T`에 `PartialOrd` 트레잇 바운드를 추가한다.

(`PartialOrd`는 Prelude에 있다.)

```rust
fn largest<T: PartialOrd>(list: &[T]) -> T {
```

그러면 `cannot move out of type [T], a non-copy array`라는 컴파일 에러를 얻게 된다.

고정된 크기를 가지고 있어서 스택에 저장되는 `i32`와 `char`는 `Copy` 트레잇을 이미 가지고 있지만

`list` 파라미터에 `Copy` 트레잇을 구현하지 않은 타입이 들어올 수도 있다.

아래의 `T: PartialOrd + Copy`같이 `Copy` 트레잇 바운드를 설정하면 구현이 완료된다.

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}
```

`Copy` 대신 `Clone`을 사용할 수도 있지만 그러면 `clone` 함수를 사용해서 많은 양의 데이터가 들어온 경우 동작이 느려진다.

반환 타입을 `T` 대신 `&T`로 바꾸고 함수 본체도 참조자를 반환하도록 하면 `Copy`나 `Clone` 트레잇을 사용하지 않고 힙 할당도 필요가 없다.

## 결론
