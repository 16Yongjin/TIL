# 러스트와 객체 지향 프로그래밍

## 객체 지향 언어의 특성

객체 지향 언어는

- 객체
- 캡슐화
- 상속

을 지원한다.

## GOF의 OOP 정의

> 객체-지향 프로그램은 객체로 구성된다. 객체는 데이터 및 이 데이터를 활용하는 프로시저를 묶는다. 이 프로시저들은 보통 메소드 혹은 연산 (operation) 으로 불린다.

이 정의에 따라 러스트도 객체 지향적이라고 할 수 있다.

구조체와 열거형으로 데이터를 표현하고 `impl` 블럭으로 메서드를 제공한다.

## 캡슐화

객체 사용자가 그 객체의 세부 구현에 접근하는 것을 막는 것이다

사용자가 코드 내부 데이터나 동작을 변경하는 것을 막고 공개된 API만 사용하게 하면, 사용자 코드는 그대로 두면서 객체 내부를 변경할 수 있다.

## 러스트의 캡슐화

모듈, 타입, 함수, 메서드는 기본적으로 비공개이고 `pub` 키워드로 공개 여부를 결정한다.

아래 코드는 벡터의 값의 평균값을 캐시하는 구조체이다.

```rust
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}
```

구조체 자체는 공개적으로 사용할 수 있지만, 구조체 안의 항목들은 비공개이다.

```rust
impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            },
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
```

사용자는 공개 메서드인 `add`, `remove`, `average`를 통해서만 객체의 인스턴스를 수정할 수 있다.

나중에 벡터를 해시맵으로 리팩터링한다고 해도, 이 객체를 사용하는 코드는 변경할 필요가 없다.

## 상속

한 객체가 다른 객체의 데이터와 동작을 재정의할 필요없이 상속해서 가져다 쓸 수 있는 매커니즘이다.

러스트의 구조체는 필드와 메서드 구현을 상속받을 수 없다.

대신에 다른 방법으로 상속의 장점을 취한다.

## 상속의 두 가지 장점

1. 코드를 재사용할 수 있다.

러스트는 기본 트레잇 메서드로 코드를 공유할 수 있다.

2. 자식 타입을 부모 타입처럼 사용할 수 있다.(다형성)

제약이 있는 제네릭 타입인 트레잇 바운드로 매개변수형 다형성을 구현한다.

## 언어 설계에서 상속의 인기가 하락하는 이유

1. 하위 클래스가 부모 클래스의 모든 특성을 상속받아 필요보다 많은 코드를 공유하게 된다.

2. 단일 상속만 허용하는 언어들은 프로그램 디자인의 유연성을 떨어뜨린다.

러스트에서는 상속 대신 트레잇 객체를 사용한다.

# 트레잇 객체를 사용하여 다른 타입 간의 값을 허용하기

## gui 라이브러리 만들기

`gui` 라이브러리는 `Button`, `TextField` 같은 요소를 `draw` 메서드 호출로 화면에 그린다.

## 상속 기능이 있는 경우

상속이 있는 언어는 `draw` 메서드를 가진 `Component` 클래스를 만든다.

요소들이 이를 상속받아 `draw`메서드를 오버라이딩해서 각각의 고유 동작을 정의한다.

라이브러리는 모든 요소를 `Component`로 다루고 `draw` 메서드를 호출하기만 한다.

## 공통 동작을 위한 트레잇 정의

러스트에서 `gui` 라이브러리를 구현하기 위해 `draw` 메서드를 갖는 `Draw` 트레잇을 정의한다.

```rust
pub trait Draw {
    fn draw(&self);
}
```

`components` 벡터를 가진 `Screen` 구조체를 정의한다.

`Box<Draw>`가 트레잇 객체이다.

```rust
pub struct Screen {
    pub components: Vec<Box<Draw>>,
}
```

`Screen` 구조체의 `run` 메서드를 정의한다.

`run`은 각 `component`에 `draw` 메서드를 호출한다.

```rust
impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

위는 트레잇 바운드와 제네릭 타입 파라미터를 사용하는 구조체 정의와 다르게 동작한다.

제네릭 타입 파라미터는 한 개의 구체 타입만 넣을 수 있지만, 트레잇 객체는 런타임에 여러 구체 타입을 넣어 사용 가능하다.

```rust
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

impl<T> Screen<T>
    where T: Draw {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

위의 구현은 `T`가 전부 `Button`이거나 `TextField`인 컴포넌트 리스트를 갖게 한다.

제네릭은 컴파일 타임에 구체 타입 사용을 위해 단형성화(monomorphize)된다.

## 트레잇 구현하기

`Draw` 트레잇을 구현하는 `Button` 구조체를 만든다.

```rust
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // code to actually draw a button
    }
}
```

사용자가 `SelectBox` 구조체를 구현한다면 다음과 같을 것이다.

```rust
extern crate gui;
use gui::Draw;

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // code to actually draw a select box
    }
}
```

`Screen` 인스턴스를 만들기 위해 `main` 함수를 구현한다.

`SelectBox`와 `Button`가 트레잇 객체가 되도록 하기 위해 각각을 `Box<T>` 안에 넣는다.

```rust
use gui::{Screen, Button};

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No")
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```

`Screen`은 `Draw` 트레잇의 `draw` 메서드가 구현된 구조체 타입 모두에 작동한다.

이 특성은 **덕 타이핑**이다.

## 트레잇 객체는 동적 디스패치를 실행한다

제네릭 타입은 컴파일 시 모두 구체타입으로 변경되는 정적 디스패치가 실행된다.

러스트는 트레잇 객체를 사용하는 코드의 타입과 메서드를 컴파일 타임에 알 수 없다.

대신에 트레잇 객체 내 포인터로 어떤 메서드가 호출될 지 런타임에 알아낸다.

또한, 동적 디스패치 사용 시 유연한 코드를 작성할 수 있지만, 메서드 인라인화를 할 수 없어서 몇가지 최적화를 놓친다.

## 트레잇 객체 사용 시 객체 안전성이 필요하다.

트레잇의 모든 메서드가 다음 속성을 가진다면 해당 트레잇은 객체-안전하다.

- 반환 타입이 `Self`가 아니다.
- 제네릭 타입 매개변수가 없다.

트레잇 객체 사용 시, 트레잇에 구현된 구체 타입을 알 수 없다.

트레잇의 타입의 별칭인 `Self` 타입도 알 방법이 없다.

제네릭 타입 파라미터도 마찬가지로 트레잇 객체 사용 시 구체 타입을 알 수 없어서 제네릭 타입을 특성할 수 없다.

객체 안전하지 않은 트레잇의 예로 `Clone` 트레잇이 있다.

```rust
pub trait Clone {
    fn clone(&self) -> Self;
}
```

`Screen` 구조체의 `Draw` 트레잇을 `Clone` 트레잇으로 대체한다면 컴파이 에러가 난다.

```rust
pub struct Screen {
    pub components: Vec<Box<Clone>>,
}
```

반환 타입이 `Self`인 `Clone`은 객체 안전하지 않기 때문이다.

# 객체 지향 디자인 패턴 구현하기

상태 패턴(State Pattern)을 구현한다.

상태 패턴은 내부 상태에 따라 객체의 동작이 변경된다.

각 상태 객체는 다른 상태로 이전을 담당한다.

## 블로그 게시물 올리기 작업 흐름

1. 빈 초안으로 시작
2. 초안 완료 시 게시물 검토
3. 게시물 승인 시 게시
4. 오직 게시된 블로그 게시물만 내용을 반환할 수 있음

구현된 `blog` API는 아래와 같이 사용된다.

```rust
extern crate blog;
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());

    post.request_review();
    assert_eq!("", post.content());

    post.approve();
    assert_eq!("I ate a salad for lunch today", post.content());
}
```

사용자는 `Post` 타입으로만 상호작용을 하지만, 내부 상태는 초안, 리뷰 대기 중, 게시됨 중 하나의 상태값을 가진다.

## `Post`를 정의하고 초안 상태의 인스턴스 생성하기

```rust
pub struct Post {
    state: Option<Box<State>>,
    content: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }
}

trait State {}

struct Draft {}

impl State for Draft {}
```

`Post` 구조체, `Post` 인스턴스를 만드는 `new` 함수, `State` 트레잇과 `Draft` 구조체를 정의한다.

`Post` 생성 시 내부 상태는 항상 `Draft`로 시작하게 된다.

## 게시물 콘텐츠에 글 저장하기

```rust
impl Post {
    // --snip--
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
```

`add_text` 메서드는 `Post` 인스턴스를 변경하기에 가변 참조자 `self`를 필요로 한다.

## 초안 게시물의 내용이 비어있음을 보장하기

임시로 빈 스트링 슬라이스를 반환하는 `content` 메서드를 구현한다.

```rust
impl Post {
    // --snip--
    pub fn content(&self) -> &str {
        ""
    }
}
```

## 게시물 리뷰 요청으로 내부 상태 변경하기

리뷰 요청을 하는 `request_review` 메서드로 `Draft` 상태를 `PendingReview` 상태로 변경한다.

```rust
impl Post {
    // --snip--
    pub fn request_review(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.request_review())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<State>;
}

struct Draft {}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<State> {
        Box::new(PendingReview {})
    }
}

struct PendingReview {}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<State> {
        self
    }
}
```

`State` 트레잇의 `request_review` 메서드는 `self:Box<Self>`를 인자로 받는다.

이 문법은 해당 타입을 보유한 `Box` 상에서만 메서드 호출이 허용됨을 뜻한다.

`Box<Self>`의 소유권을 가져가서 `Post` 이전 상태를 무효화하고 새 상태로 변화시킨다.

`Option` 타입은 `state`를 `None` 값으로 설정해서 이전 상태를 사용할 수 없게 한다.

## `content`의 동작을 변경하는 `approve` 메소드 추가하기

```rust
impl Post {
    // --snip--
    pub fn approve(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.approve())
        }
    }
}


trait State {
    fn request_review(self: Box<Self>) -> Box<State>;
    fn approve(self: Box<Self>) -> Box<State>;
}

struct Draft {}

impl State for Draft {
    // --snip--
    fn approve(self: Box<Self>) -> Box<State> {
        self
    }
}

struct PendingReview {}

impl State for PendingReview {
    // --snip--
    fn approve(self: Box<Self>) -> Box<State> {
        Box::new(Published {})
    }
}

struct Published {}

impl State for Published {
    fn request_review(self: Box<Self>) -> Box<State> {
        self
    }

    fn approve(self: Box<Self>) -> Box<State> {
        self
    }
}
```

`PendingReview` 상태일 때만 `approve` 메서드 호출 시 `Published` 상태로 이전된다.

`Published` 상태일 때 `content` 필드의 값을 반환하게 한다.

```rust
impl Post {
    // --snip--
    pub fn content(&self) -> &str {
        self.state.as_ref().unwrap().content(&self)
    }
    // --snip--
}
```

`state`는 `Option<Box<State>>`이므로 `as_ref`를 호출하면 `Option<&Box<State>>`가 반환된다.

`state`는 항상 `Some` 값이므로 `unwrap`으로 `&Box<State>`를 가져온다.

`&Box<State>`에 `content` 호출 시 역참조 강제로 인해 `content` 메서드는 `State` 트레잇을 구현하는 타입에서 호출된다.

```rust
trait State {
    // --snip--
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        ""
    }
}

// --snip--
struct Published {}

impl State for Published {
    // --snip--
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        &post.content
    }
}
```

`content` 메서드 호출 시 `Draft`와 `PendingReview` 상태인 경우 기본 구현이 실행된다.

`Published` 상태인 경우 `post.content`의 값을 반환한다.

## 상태 패턴의 기회비용

상태 패턴 대신 `match`를 사용한다면, 여러 겹의 `match` 문을 작성해야 될 것이다.

상태 패턴을 사용하면 기능을 추가하기 쉽다.

하지만, 단점도 존재한다.

상태 간 전환으로 인해, 상태들이 서로 묶이게 된다.

또한, 로직도 중복된다.

## 상태와 동작을 타입처럼 인코딩하기
