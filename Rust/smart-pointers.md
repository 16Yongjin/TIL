# 러스트의 스마트 포인터

`Box<T>`로 힙 데이터를 참조하고, `Rc<T>`로 다중 소유권을 갖고, `RefCell<T>`로 불변 값을 변경하고, `Weak<T>`로 약한 참조를 가질 수 있다.

## 포인터

메모리 주소 값을 담고 있는 변수

## 참조자

러스트에서 가장 흔한 포인터

`&` 심볼로 나타내고 값을 빌리는 역할을 한다.

## 스마트 포인터

포인터처럼 작동하지만 추가적인 기능과 메타데이터를 가지고 있는 데이터 구조

## 참조자와 스마트 포인터의 차이점

참조자는 데이터를 빌리기만 하는 포인터다.

반면, 스마트 포인터는 자신이 가리키고 있는 데이터를 *소유*한다.

자주 쓰는 `String`과 `Vec<T>`도 메모리를 소유하므로 스마트 포인터에 속한다.

## 스마트 포인터 구현

스마트 포인터는 `Deref`와 `Drop` 트레잇을 구현한 구조체다.

`Deref` 트레잇은 스마트 포인터가 참조자처럼 작동하게 한다.

`Drop` 트레잇은 스마티 포인터가 스코프 밖으로 벗어났을 때 실행되는 코드를 커스터마이징할 수 있게 한다.

# 힙에 있는 데이터를 가리키고, 크기를 알 수 있는 `Box<T>`

박스로 데이터를 스택에 말고 힙에 저장할 수 있다.

스택에는 힙에 있는 데이터를 가리키는 포인터만 남는다.

## 박스를 쓰는 상황

- 타입의 크기를 컴파일 타임에 알 수 없지만, 타입의 정확한 사이즈를 알아야 하는 상황일 때

- 거대한 데이터의 소유권을 옮기는 경우, 데이터가 복사되는 일이 없도록 하고 싶을 때

- 구체적인 타입은 모르고 특정 트레잇을 구현한 타입이라는 점만 알고 싶을 때 (트레잇 객체)

## `Box<T>`로 힙에 데이터를 저장하기

스택에 저장되는 `i32` 값을 박스를 사용해서 힙에 저장할 수 있다.

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```

`b`를 스택에 저장된 데이터처럼 사용할 수 있다.

또한, `main`을 벗어나면 `b`는 할당해제된다.

## 재귀 타입 정의하기

러스트는 컴파일 타임에 타입이 차지하는 공간을 정확히 알아야 컴파일이 된다.

재귀 타입의 값은 내부에 자신과 동일한 타입의 다른 값 가질 수 있다.

값의 내포는 무한할 수 있으므로, 러스트는 재귀 타입이 차지하는 공간을 알 수 없다.

하지만, 박스는 크기를 알 수 있으므로, 재귀 타입 정의에 박스를 넣으면 재귀 타입을 정의할 수 있다.

## Cons List 재귀 타입

대표적인 재귀 타입 Cons List.

다른 함수형 언어처럼 아래와 같이 cons list를 정의해볼 수 있다.

```rust
enum List {
    Cons(i32, List),
    Nil,
}
```

위의 코드가 가능하다면, List에 1, 2, 3을 저정하려면 아래와 같이 코드를 작성해야 한다.

```rust
use List::{Cons, Nil};

fn main() {
    let list = Cons(1, Cons(2, Cons(3, Nil)));
}
```

하지만, 어림도 없다.

러스트는 아래 에러를 뿜으며, `List` 타입이 무한한 크기를 갖는다고 말한다.

```
error[E0072]: recursive type `List` has infinite size
 --> src/main.rs:1:1
  |
1 | enum List {
  | ^^^^^^^^^ recursive type has infinite size
2 |     Cons(i32, List),
  |               ----- recursive without indirection
  |
  = help: insert indirection (e.g., a `Box`, `Rc`, or `&`) at some point to
  make `List` representable
```

러스트는 타입이 차지하는 공간을 파악할 때 변형타입 중 가장 크기가 큰 타입을 살펴본다.

`List`의 크기를 파악할 때, 내부의 변형타입을 무한으로 살펴보게 된다.

![infinite-cons](https://rinthel.github.io/rust-lang-book-ko/img/trpl15-01.svg)

컴파일러 에러 아래쪽의 도움말을 보면, 값을 간접적으로 나타내기 위해 다른 데이터 구조를 사용하라고 한다.

`List` 타입을 아래와 같이 바꾸면 컴파일이 된다.

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
}
```

`Cons` 변형타입이 `i32`와 박스의 포인터 데이터를 저장할 공간만 요구한다.

![finite-list](https://rinthel.github.io/rust-lang-book-ko/img/trpl15-02.svg)

박스 덕분에 무한하고 재귀적인 연결을 끊어서 `List` 값 저장에 필요한 크기를 알아낼 수 있다.

&nbsp;

# `Deref` 트레잇으로 스마트 포인터를 일반 참조자처럼 다루기

`Deref` 트레잇으로 역참조 연산자(`*`)의 동작을 커스터마이징할 수 있다.

그러면, 참조자를 사용하는 코드에도 스마트 포인터를 사용할 수 있다.

## `*`와 함께 포인터를 따라가서 값을 얻기

아래 코드는 `i32` 값의 참조자를 생성하고, 역참조 연산자 `*`로 참조자를 따라가서 값을 얻는다.

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

`y` 앞에 `*` 연산자를 빼면 `` can't compare `{integer}` with `&{integer}` ``, 숫자와 숫자에 대한 참조자를 비교했다는 에러가 난다.

## `Box<T>`를 참조자처럼 사용하기

위의 코드를 참조자 대신 `Box<T>`를 사용해서 작성할 수 있다.

역참조 연산자도 똑같이 작동한다.

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

## 커스텀 스마트 포인터 정의하기

`Box<T>` 타입은 하나의 요소를 가진 튜플 구조체로 정의되므로 `MyBox<T>` 타입도 동일하게 정의한다.

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

이 상태에서는 `MyBox<T>` 타입에 역참조 연산을 할 수 없다.

## `Deref` 트레잇을 구현하여 임의의 타입을 참조자처럼 다루기

`Deref` 트레잇은 `self`를 빌려서 내부 데이터의 참조자를 반환하는 `deref` 메서드를 구현하도록 요구한다.

```rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}
```

`type Target = T;` 문법은 트레잇이 사용할 연관 타입을 정의한다.

`Deref` 트레잇을 구현했으므로, `MyBox<T>` 값에 `*`을 호출하는 아래 코드는 오류 없이 컴파일된다.

```rust
fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

`deref` 메서드는 컴파일러에게 어떻게 역참조하는지 알려준다.

위의 `*y` 코드는 실제로 `*(y.deref())` 코드로 실행된다.

## 암묵적 역참조 강제

함수나 메서드에 들어온 인자값의 타입이 요구되는 타입과 다른 경우, 그 값에 `deref`를 호출해서 인자 타입을 일치시킨다.

역참조 강제 덕분에 메소드 호출 시, 명시적으로 참조, 역참조를 나태내기 위해 `&`나 `*`를 적을 필요가 없다.

아래 함수는 스트링 슬라이스를 파라미터로 갖는다.

```rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}
```

`hello("Rust");`와 같이 스트링 슬라이스를 인자로 해서 함수를 호출할 수도 있고

아래와 같이 `MyBox<String>` 타입의 값에 역참조 강제를 적용해서 `hello` 함수를 호출할 수 있다.

```rust
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```

`MyBox<String>`에서 `Deref` 트레잇을 구현한 덕분에, `deref`를 호출해서 `&MyBox<String>`을 `&String`으로 바꿀 수 있다.

역참조 강제 기능이 없었다면, 아래와 같이 복잡한 코드를 작성해야 됐을 것이다.

```rust
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&(*m)[..]);
}
```

`(*m)`로 `MyBox<String>`을 `String`로 역참조하고, `&`와 `[..]`로 `String`의 스트링 슬라이스를 얻는다.

## 가변 참조자를 위한 `DerefMut`

`Deref` 트레잇은 불변 참조자에 대한 `*` 연산자를 오버라이딩하기 위해 사용했다.

가변 참조자에 대한 `*`을 오버라이딩하려면 `DerefMut` 트레잇을 사용해야 한다.

## 러스트가 역참조 강제를 수행하는 조건

- `T: Deref<Target=U>`일때 `&T`에서 `&U`로

- `T: DerefMut<Target=U>`일때 `&mut T`에서 `&mut U`로

- `T: Deref<Target=U>`일때 `&mut T`에서 `&U`로

첫 번째 경우, `&T`를 가지고 있는데 `T`가 `U`에 대한 `Deref`를 구현했다면 `&U`를 얻을 수 있다는 뜻이다.

`hello` 함수에서 `&MyBox<String>`를 가지고 있는데 `MyBox<String>`이 `String`에 대한 `Deref`를 구현했으므로 `&String`을 얻을 수 있었다.

세 번째의 경우, 가변 참조자를 불변 참조자로 강제할 수 있다.

빌림 규칙에 따르면, 가변 참조자는 해당 데이터의 유일한 참조자이다.

이 가변 참조자를 불변 참조자로 바꾸는 것은 빌림 규칙을 위반하지 않는다.

반대로, 불변 참조자를 가변 참조자로 바꾸는 경우는, 불변 참조자가 한 개만 있다는 것을 보장할 수 없어서 불가능하다.

&nbsp;

# 메모리 정리 코드를 실행하는 `Drop` 트레잇

값이 스코프 밖으로 벗어날 때 일어나는 일을 커스터마이징 할 수 있다.

모든 타입에 `Drop` 트레잇 구현이 가능하다.

파일이나 네트워크 연결 같은 자원 해제할 때도 사용 가능하다.

보통은 스마트 포인터 구현을 위해 사용된다.

ex) `Box<T>`에서 박스가 가리키는 힙 공간을 할당 해제

&nbsp;

다른 언어에서 스마트 포인트 사용 시, 메모리나 자원 해제 코드를 직접 호출해야 한다.

러스트에서는 그 코드를 자동으로 삽입해주기 때문에, 자원 누수를 걱정하지 않아도 된다.

&nbsp;

아래 코드는 인스턴스가 스코프 밖으로 벗어났을 때 `Dropping CustomSmartPointer!`를 출력하는 구조체를 사용한다.

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("`{}` 데이터를 가진 CustomSmartPointer이 버려짐!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("my stuff") };
    let d = CustomSmartPointer { data: String::from("other stuff") };
    println!("CustomSmartPointers 생성됨.");
}
```

`Drop`은 Prelude에 이미 있어서 가져올 필요 없다.

실행 시 결과는 다음과 같다.

```
CustomSmartPointers 생성됨.
`other stuff` 데이터를 가진 CustomSmartPointer이 버려짐!
`my stuff` 데이터를 가진 CustomSmartPointer이 버려짐!
```

스코프 밖으로 벗어났을 때 `drop`이 호출되는 것을 알 수 있다.

추가로, 변수는 생성의 역순으로 버려진다.

## `std::mem::drop`을 이용하여 값을 일찍 버리기

수동으로 `Drop` 트레잇의 `drop` 메서드를 직접 호출할 수 없다. (중복 해제 방지)

또한, `drop`이 자동 실행되는 것을 막을 수도 없다. (메모리 누수 방지)

대신에, `std::mem::drop`를 호출해서, 스코프 밖으로 벗어나기 전에 값을 강제로 버릴 수 있다.

아래 코드는 `drop` 메서드 호출해서 `explicit destructor calls not allowed` 에러 발생한다.

```rust
fn main() {
    let c = CustomSmartPointer { data: String::from("some data") };
    println!("CustomSmartPointer created.");
    c.drop();
    println!("CustomSmartPointer dropped before the end of main.");
}
```

값이 일찍 메모리에서 정리되게 하려면, 아래와 같이 `std::mem::drop` 함수를 사용한다.

`std::mem::drop`는 Prelude에 이미 포함되어 있어서 `drop`으로 호출한다.

```rust
fn main() {
    let c = CustomSmartPointer { data: String::from("some data") };
    println!("CustomSmartPointer 생성됨.");
    drop(c);
    println!("main이 끝나기 전에 CustomSmartPointer이 버려짐.");
}
```

결과는 다음과 같이 나온다.

```
CustomSmartPointer 생성됨.
`some data` 데이터를 가진 CustomSmartPointer이 버려짐!
main이 끝나기 전에 CustomSmartPointer이 버려짐.
```

`drop` 함수로 버려진 값을 사용하려고 하면, `borrow of moved value` 에러가 난다.

빌림 시스템의 옮기기를 이용해서 할당 해제된 값이 사용되는 것을 막았다.👍

&nbsp;

# 참조 카운팅 스마트 포인터 `Rc<T>`

하나의 값을 여러 참조자가 소유해야 할 때 사용된다.

예를 들어, 그래프 자료 구조에서, 여러 에지가 동일한 노드를 가리킬 수 있다.

노드는 가리키는 에지가 없어질 때까지 메모리에서 정리되면 안 된다.

&nbsp;

`Rc<T>`는 Reference Counting의 약자이다.

참조자의 개수로 참조 여부를 파악한다.

값의 참조자 수가 0이라면, 그 값은 메모리 할당 해제가 가능하다.

값이 여러 부분에서 사용될 때, 사용이 끝나는 부분을 안다면, 그 부분을 데이터의 소유자로 만들면 된다.

하지만, 어디서 사용이 끝날 지 모를 때는 `Rc<T>` 타입을 사용한다.

> ⚠ `Rc<T>`는 오직 단일 스레드 상에서만 사용 가능하다.

## `Rc<T>`를 사용하여 데이터 공유하기

아래 그림과 같이 `a`의 소유권을 공유하는 `b`, `c` 두 리스트를 만들어 본다.

![cons-list-1](https://rinthel.github.io/rust-lang-book-ko/img/trpl15-03.svg)

`Box<T>`를 사용한 `List`로는 위와 같이 만들 수 없다.

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let a = Cons(5,
        Box::new(Cons(10,
            Box::new(Nil))));
    let b = Cons(3, Box::new(a));
    let c = Cons(4, Box::new(a));
}
```

`b` 만들 때 쓴 `a`를, `c` 만들 때도 쓰려고 해서 `` use of moved value: `a` `` 에러가 발생한다.

`List`에서 `Box<T>`를 `Rc<T>`로 변경한다.

`b`를 만들 때 `a`의 소유권을 얻는 대신 `a`를 가지고 있는 `Rc<List>`를 클론한다.

참조자의 개수가 1에서 2가 되고, `a`와 `b`가 값을 공유한다.

`c`를 만들 때도 마찬가지다.

`Rc::clone` 호출할 때마다 참조 카운트가 증가되고, 참조자의 수가 0이 아닌 이상 메모리가 정리되지 않는다.

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```

`Rc<T>`는 `use std::rc::Rc;`로 가져온다.

`Rc::clone(&a)` 대신 `a.clone()`를 호출할 수도 있지만, 러스트에서는 관례적으로 `Rc::clone`을 사용한다.

`Rc::clone`는 깊은 복사를 하는 대신 참조 카운트만 증가시켜서 속도가 빠르다.

## `Rc<T>`의 클론 생성은 참조 카운트를 증가시킨다.

참조 카운트는 `Rc::strong_count` 함수로 얻을 수 있다.

`weak_count`도 있어서 `count`가 아닌 `strong_count`로 호출한다.

```rust
fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("a 생성 후 카운트 = {}", Rc::strong_count(&a));
    let b = Cons(3, Rc::clone(&a));
    println!("b 생성 후 카운트 = {}", Rc::strong_count(&a));
    {
        let c = Cons(4, Rc::clone(&a));
        println!("c 생성 후 카운트 = {}", Rc::strong_count(&a));
    }
    println!("c가 스코프를 벗어난 후 카운트 = {}", Rc::strong_count(&a));
}
```

위의 코드는 다음을 출력한다.

```
a 생성 후 카운트 = 1
b 생성 후 카운트 = 2
c 생성 후 카운트 = 3
c가 스코프를 벗어난 후 카운트 = 2
```

카운트가 1에서 시작해서 `clone`을 호출할 때마다 카운트가 1씩 증가한다.

스코프를 벗어나면 자동으로 카운트가 1씩 감소한다.

`main`이 끝나면, 카운트가 0이 돼서, `Rc<List>`는 완전히 메모리에서 정리된다.

&nbsp;

# `RefCell<T>`와 내부 가변성 패턴

## 내부 가변성

불변값을 가변으로 빌려서 데이터를 변형할 수 있게 해주는 기능

데이터 구조 내에서 `unsafe` 코드를 사용해서 러스트의 빌림 규칙을 깨뜨릴 수 있다.

## `Box<T>`, `Rc<T>`, `RefCell<T>` 비교

### 데이터 소유권 개수 :

`Rc<T>`는 여러 개, `Box<T>`와 `RefCell<T>`는 하나의 데이터의 소유권을 가질 수 있다.

### 빌림 규칙 허용 :

`Box<T>`는 컴파일 타임에 확인한 불변과 가변 빌림을 허용한다.

`Rc<T>`는 컴파일 타임에 확인한 불변 규칙만 허용한다.

`RefCell<T>`는 런타임에 확인한 불변과 가변 빌림을 허용한다.

### 내부 값 변경 여부 :

`RefCell<T>`만 런타임 빌림 검사를 해서, 불변값이라도 내부 값 변경이 가능하다.

## Mock 객체를 만들어서 테스트할 때 유용한 `RefCell<T>`

목 객체는 테스트 중 일어난 일을 기록해서, 기능이 정확하게 작동했는지 확인한다.

편한 테스트를 위해 불변으로 빌린 값을 변경해야 하지만, 함수 시그내처를 바꿀 수 없을 때, `RefCell<T>`를 사용하면 된다.

`RefCell::new()`로 값을 만들고, 값을 `borrow_mut()` 메서드로 가변으로 빌리거나, `borrow` 메서드로 불변으로 빌린다.

여러 개의 불변이나 한 개의 가변 빌림을 가지고 있는지 여부는 런타임에 확인한다.

아래와 같이 `borrow_mut()` 메서드로 값을 두 번 가변으로 빌리면 패닉이 일어난다. (`already borrowed: BorrowMutError` 에러)

```rust
impl Messenger for MockMessenger {
    fn send(&self, message: &str) {
        let mut one_borrow = self.sent_messages.borrow_mut();
        let mut two_borrow = self.sent_messages.borrow_mut();

        one_borrow.push(String::from(message));
        two_borrow.push(String::from(message));
    }
}
```

## `Rc<T>`와 `RefCell<T>`의 조합으로 가변 데이터의 복수 소유자 만들기

Cons List 정의에 `RefCell<T>`을 추가해서 리스트 내부의 값을 변경할 수 있다.

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}
```

`Rc<RefCell<i32>>` 인스턴스인 `value`를 만들고 이를 기반으로, `Cons` 변형타입 `a`와 리스트 `b`, `c`를 만든다.

```rust
use List::{Cons, Nil};
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

`value`에 `borrow_mut`를 호출해서 `RefMut<T>` 스마트 포인터를 얻어낸다.

여기에 역참자 연산자를 사용해서 내부 값을 변경한다.

코드를 실행하면 리스트가 5가 아닌 15를 가지고 있는 것을 알 수 있다.

```
a after = Cons(RefCell { value: 15 }, Nil)
b after = Cons(RefCell { value: 6 }, Cons(RefCell { value: 15 }, Nil))
c after = Cons(RefCell { value: 10 }, Cons(RefCell { value: 15 }, Nil))
```

&nbsp;

# 메모리 릭을 발생시키는 순환 참조

`Rc<T>`와 `RefCell<T>`로 순환 참조를 만들면 메모리 릭을 허용할 수 있다.

참조자들이 서로를 참조하면, 순환 고리 속에서 참조 카운트가 0이 될 일이 없기 때문에, 값이 버려지지 않는다.

## 순환 참조 만들기

`Cons` 속 `List`를 번경할 수 있는 Cons List

```rust
use std::rc::Rc;
use std::cell::RefCell;
use List::{Cons, Nil};

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match *self {
            Cons(_, ref item) => Some(item),
            Nil => None,
        }
    }
}
```

아래 같은 순환 참조를 만든다.

![circular-ref-list](https://rinthel.github.io/rust-lang-book-ko/img/trpl15-04.svg)

리스트 `a`를 만들고, `a`를 가리키는 리스트 `b`를 만든다.

다시 `a`가 `b`를 가리키게 해서 순환 참조를 만든다.

```rust
fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a의 초기 참조 카운트 = {}", Rc::strong_count(&a));
    println!("a의 다음 항목 = {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("b 생성 후 a의 참조 카운트 = {}", Rc::strong_count(&a));
    println!("b의 초기 참조 카운트 = {}", Rc::strong_count(&b));
    println!("b의 다음 항목 = {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("a 변경 후 b의 참조 카운트 = {}", Rc::strong_count(&b));
    println!("a 변경 후 a의 참조 카운트 = {}", Rc::strong_count(&a));

    // 다음 줄 주석 해제 시, 순환 참조 고리를 볼 수 있음
    // 스택 오버플로가 일어날 것임
    // println!("a의 다음 항목 = {:?}", a.tail());
}
```

`a`와 `b`의 참조 카운트가 2이다.

스코프를 벗어나서 참조 카운트가 1로 줄어도, 카운트가 0이 아니기 때문에 메모리 릭이 발생한다.

```
a의 초기 참조 카운트 = 1
a의 다음 항목 = Some(RefCell { value: Nil })
b 생성 후 a의 참조 카운트 = 2
b의 초기 참조 카운트 = 1
b의 다음 항목 = Some(RefCell { value: Cons(5, RefCell { value: Nil }) })
a 변경 후 b의 참조 카운트 = 2
a 변경 후 a의 참조 카운트 = 2
```

순환 참조를 피하기 위해, 어떤 참조자는 소유권을 갖고 다른 참조자는 못 갖게 할 수 있다.

## 참조 순환을 방지하기 위해 `Rc<T>`를 `Weak<T>`로 바꾸기

`Rc<T>`를 `Rc::downgrade`에 넣고 호출하면 `Rc<T>` 내의 값을 가리키는 약한 참조인 `Weak<T>` 타입의 스마트 포인터를 얻을 수 있다.

`Rc::downgrade`는 `strong_count` 대신 `weak_count`를 1 증가 시킨다.

`weak_count`은 0이 아니여도 `Rc<T>` 인스턴스가 제거될 수 있다.

## `Weak<T>`가 참조하고 있는 값 사용하기

`Weak<T>`가 참조하고 있는 값이 이미 버려져있을 수 있다.

`Weak<T>`가 참조하고 있는 값 사용하기 위해 `Weak<T>`의 `upgrade` 메소드를 호출하면 `Option<Rc<T>>` 반환된다.

결과가 `Some`이라면 값이 아직 존재하고 `None`이라면 값이 버려졌다는 뜻이다.

## 트리 데이터 구조 만들기

자식 노드를 가지고 있는 트리를 만든다.

자식을 소유하기 위해 소유권을 공유하는 `Rc<Node>` 타입을 사용하고, 자식 노드를 수정하기 위해 `RefCell` 타입을 사용했다.

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}
```

아래와 같이 `leaf`와 이를 자식으로 갖는 `branch` 노드를 만들 수 있다.

```rust
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        children: RefCell::new(vec![]),
    });

    let branch = Rc::new(Node {
        value: 5,
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });
}
```

`branch`에서 `leaf`로 접근할 수 있지만, `leaf`에서 `branch`로 접근할 수 없다.

## 자식으로부터 부모로 가는 참조자 추가하기

노드 구조체에 `parent` 필드를 추가해서 자식 노드에서 부모 노드로 가는 연관성을 추가한다.

부모 노드가 버려지면 자식 노드도 버려져야 한다.

하지만, 자식 노드가 버려져도 부모 노드는 그대로 있어야 한다.

이를 위해 `parent`의 타입은 `Rc<T>` 대신 `Weak<T>`를 사용해야 한다.

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}
```

노드가 부모 노드를 참조할 수 있지만, 부모를 소유하지 않는다.

```rust
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
}
```

`*leaf.parent.borrow_mut() = Rc::downgrade(&branch);`로 `leaf` 노드의 부모 노드 `branch`를 설정한다.

순혼 참조가 발생하지 않아서, 노드를 콘솔에 출력해도 무한 출력이 발생하지 않는다.

```
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

&nbsp;

# 정리

`Box<T>` 타입은 크기를 알 수 있고, 힙에 할당한 데이터를 가리킨다.

`Rc<T>` 타입은 힙에 있는 데이터에 대한 참조자의 개수를 추적하여 그 데이터가 소유자를 여러 개 갖을 수 있도록 한다.

`RefCell<T>`은 불변 타입의 값을 변경할 수 있고, 컴파일 타임 대신 런타임에 빌림 규칙을 적용한다.

`Weak<T>` 타입으로 약한 참조를 만들어서 순환 참조와 그로 인한 메모리 릭을 방지할 수 있다.

`Deref` 트레잇으로 역참조 연산자의 동작을, `Drop` 트레잇으로 스코프 이탈 시 동작을 커스터마이징할 수 있다.
