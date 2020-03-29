# 고급 트레잇

연관 타입, 기본 제네릭 타입 파라미터, 완전 정규화 문법, 슈퍼 트레잇, 뉴타입 패턴

## 연관 타입

연관 타입은 타입 플레이스홀더와 트레잇을 연결한다.

트레잇 메서드 정의에 플레이스홀더 타입을 사용할 수 있다.

`Iterator` 트레잇을 정의할 때 사용했던 `Item`이 연관 타입이다.

```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```

`Iterator` 구현 시 `Item`의 구체적인 타입이 명시될 것이고

`next` 메서드는 `Item`이 나타내는 구체 타입의 옵션값을 반환할 것이다.

## 연관 타입 vs. 제네릭

연관 타입과 제네릭은 둘 다 타입을 특정하지 않아도 함수를 정의할 수 있게 도와준다.

`Counter`의 `Iterator` 구현 시 아래와 같이 코드를 작성한다.

```rust
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
```

제네릭을 사용 시 다음과 같이 `Iterator`를 정의할 수 있다.

```rust
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```

제네릭을 이용하면 각 구현마다 타입을 명시해야 한다.

제네릭 타입 `T`에 들어갈 타입이 100개라면 100개의 `Iterator` 구현이 필요할 것이다.

`Iterator<String> for Counter`, `Iterator<Point> for Counter`, ..

연관 타입을 이용하면 `Item`의 타입을 한 번만 명시할 수 있다.

`impl Iterator for Counter`은 한 번만 쓸 수 있기 때문이다.

## 기본 제네릭 타입 파라미터와 연산자 오버로딩

제네릭 타입에 들어갈 기본 타입을 명시할 수 있다.

`<PlaceholderType=ConcreteType>` 이런 식이다.

이 기능을 연산자 오버로딩에 사용하면 유용하다

러스트에서는 `std::ops`에 있는 연산자를 구현해서 연산자 오버로딩을 할 수 있다.

아래 코드는 두 개의 `Point`를 더하기 위해 `+` 연산자를 오버로딩한다.

```rust
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
               Point { x: 3, y: 3 });
}
```

연관 타입 `Output`으로 `add` 메서드에서 반환되는 타입을 결정한다.

`Add` 트레잇을 살펴보면 기본 제네릭 타입(`<RHS=Self>`)을 확인할 수 있다.

```rust
trait Add<RHS=Self> {
    type Output;

    fn add(self, rhs: RHS) -> Self::Output;
}
```

제네릭 타입 파라미터 `RHS`(우변, right hand side 줄임말)은 `add` 메서드의 `rhs` 파라미터 타입을 정의한다.

`RHS`의 기본 타입은 `Self`다.

동일한 타입을 더하고 싶을 때는 트레잇에 제네릭 타입 명시를 생략해도 된다.

다음의 코드는 서로 다른 타입인 `Millimeters`와 `Meters`를 더하기 위해 `Add` 트레잇을 구현한다.

```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

`impl Add<Meters>`를 명시해서 `RHS` 타입 파라미터에 기본값 `Self` 대신 `Meters`를 지정한다.

서로 다른 타입인 `Millimeters`와 `Meters`를 더할 수 있게 됐다.

### 기본 타입 파라미터를 사용하는 경우

- 기본 코드를 망가뜨리지 않고 타입을 확장하기 위해
- 특수한 상황의 유저를 위해 커스터마이징을 허용하고 싶을 때

## 모호성 방지를 위한 완전 정규화 (fully qualified) 문법: 동일한 이름의 메소드 호출하기

러스트에서는 서로 다른 트레잇들이 이름이 같은 메서드를 가질 수 있고, 같은 타입에 대한 구현도 가능하다.

특정 타입이 가진 메서드와 이름이 같은 트레잇 메서드도 구현할 수 있다.

메서드 이름이 같을 때, 어떤 것을 사용할지 러스트에게 알려줘야한다.

다음의 코드는 `fly` 메서드를 가진 `Pilot`과 `Wizard` 트레잇을 정의한다.

두 트레잇은 이미 `fly` 메서드를 가진 `Human` 타입에 대한 구현도 했다.

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("안녕하십니까. 비행을 맡고 있는 기장입니다.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("위로!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*팔을 격하게 흔드는 중*");
    }
}
```

다음과 같이 `Human` 인스턴스의 `fly` 메서드를 호출한다.

```rust
fn main() {
    let person = Human;
    person.fly();
}
```

컴파일러 해당 타입에 기본적으로 구현된 메서드를 호출하므로 `*팔을 격하게 흔드는 중*`이 출력된다.

`Pilot`나 `Wizard` 트레잇의 `fly` 메소드를 호출하기 위해서는 어떤 `fly` 메서드를 호출하는지 다음과 같이 명시해야 한다.

```rust
fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
```

트레잇 메서드에 `self` 파라미터 덕분에 어떤 트레잇의 메서드를 호출할지 결정할 수 있다.

하지만, 트레잇의 연관 함수는 `self` 파라미터가 없다.

이 경우, **완전 정규화 문법**을 사용해야 어떤 함수를 호출할 지 결정할 수 있다.

아래 코드는 연관 함수를 가진 트레잇(`Animal`)과, 이 트레잇을 구현하면서 동시에 이름이 같은 연관 함수를 가지고 있는 타입(`Dog`)이 있다.

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("태식이")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("강아지")
    }
}

fn main() {
    println!("아기 개는 {}라고 부른다.", Dog::baby_name());
}
```

`Dog`의 `baby_name`은 모든 강아지 이름을 태식이로 짓고 싶은 동물 보호소를 위한 코드이다.

`Animal`은 모든 동물이 가진 특성을 설명하고 `baby_name`은 아기 개는 강아지라고 불리는 것을 나타내기 위한 코드이다.

위 코드는 `아기 개는 태식이라고 부른다.`가 출력된다.

`아기 개는 강아지라고 부른다.`라고 출력되게 하기 위해서 다음의 코드를 시도해 볼 수 있다.

```rust
fn main() {
    println!("A baby dog is called a {}", Animal::baby_name());
}
```

위의 코드에서 러스트는 어떤 구현체의 `baby_name` 함수인지 알 수 없다.

모호성을 방지하고 러스트에게 `Dog`에 대한 `Animal` 구현체를 사용하고 싶다고 알려주기 위해서는 다음과 같이 완전 정규화 문법을 사용해야 한다.

```rust
fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

### 완전 정규화 문법 정의

```
<Type as Trait>::function(메서드일_경우_사용할_리시버, 다음_인자, ...);
```

연관 함수는 리시버가 없고 인자들의 리스트만 입력하면 된다.

함수나 메서드를 호출하는 모든 곳에서 완전 정규화 문법을 사용할 수 있지만, 러스트는 알아낼 수 있는 부분에서는 생략할 수 있다.

## 슈퍼트레잇(supertrait)으로 어떤 트레잇 내에서 다른 트레잇의 기능 요구하기

트레잇 구현자에게 다른 트레잇의 종속성 강제할 수 있다.

값을 `*`로 감싸서 출력하는 `outline_print` 메서드를 가진 `OutlinePrint` 트레잇을 구현해본다.

`Display`를 구현해서 `(x, y)`를 출력하는 `Point` 구조체가 있을 때,

`Point` 인스턴스에 `outline_print` 메서드 호출 시 다음과 같이 출력돼야 한다.

```
**********
*        *
* (1, 3) *
*        *
**********
```

`OutlinePrint` 구현 시 `Display`의 기능이 필요하다는 것을 명시해야 한다.

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

위와 같이, 트레잇을 정의할 때 `OutlinePrint: Display`라고 명시한다.

트레잇에 트레잇 바운드를 추가하는 것 같다.

`self`가 `Display` 트레잇을 구현하도록 강제됐기 때문에, `self`의 `to_string` 메서드를 호출할 수 있다.

아래와 같이 `Display`를 구현하지 않은 타입이 `OutlinePrint`를 구현하면 `` the trait bound `Point: std::fmt::Display` is not satisfied `` 에러가 발생한다.

```rust
struct Point {
    x: i32,
    y: i32,
}

impl OutlinePrint for Point {}
```

아래와 같이 `Point`에 `Display`를 구현해서 `OutlinePrint`의 요구사항을 만족시키면 에러를 없앨 수 있다.

```rust
use std::fmt;

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

## 외부 타입에 대해 외부 트레잇을 구현하기 위한 뉴타입 패턴

트레잇을 구현하려면 타입이나 트레잇 둘 중 하나가 내 크레이트 안에 있어야 하는 고아 규칙을 따라야 한다.

고아 규칙을 우회하려면 뉴타입 패턴(newtype pattern)을 사용하면 된다.

뉴타입 패턴은 트레잇을 구현하려는 타입을 튜플로 감싸서 내 크레이트 내에 있게한다.

`Vec` 타입에 `Display` 트레잇을 구현하려고 하면, 둘 다 내 크레이트 외부에 정의되어 있어서 고아 규칙에 걸려 구현을 할 수가 없다.

`Vec`을 감싸는 `Wrapper` 구조체를 만들고, `Wrapper`에 `Display`를 구현하면 `Vec` 값을 이용할 수 있다.

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

`self.0`으로 `Vec`에 접근한다.

## 뉴타입 패턴의 단점

`Wrapper`가 새로운 타입이므로, 내부 값의 메서드를 사용할 수 없다.

`Wrapper`가 `Vec`처럼 사용되려면, `Wrapper` 상에 `Vec` 메서드를 모두 직접 구현하고 이를 `self.0`에 위임해야 한다.

또는, `Wrapper`에 `Deref` 트레잇을 구현해서 내부 타입이 가진 메서드를 `Wrapper`에서 사용할 수 있게 한다.
