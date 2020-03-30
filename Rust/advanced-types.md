# 고급 타입

뉴타입의 유용성, 타입 별칭, `!` 타입, 동적 크기를 가진 타입

## 타입 안전성과 추상화를 위해 뉴타입 패턴을 사용한다.

뉴타입 패턴은 값의 단위를 정확하게 사용하는 데 유용하다.

`Millimeters`와 `Meters`는 둘 다 `u32`를 감싼 뉴타입으로 표현할 수 있다.

이를 통해 `Millimeters`를 파라미터로 받아야 하는 곳에 `Meters` 타입의 값이 들어오는 것을 막을 수 있다.

뉴타입으로 API 공개 수준을 조절할 수도 있고, 내부 구현사항을 숨길 수도 있다.

예를 들어, `HashMap<i32, String>`을 감싸는 `People` 타입을 제공할 수 있다.

`People`을 사용하는 코드는 공개 API만을 통해 상호작용하고 내부적으로 `i32`가 id로 사용된다는 것을 알 필요가 없다.

## 타입 별칭은 타입의 동의어를 만든다.

다음과 같이 `type` 키워드를 사용해서 타입 별칭을 선언할 수 있다.

```rust
type Kilometers = i32;
```

다만, 타입 별칭은 새로운 타입이 아니기 때문에 `Kilometers`는 `i32`와 동일하게 취급된다.

```rust
type Kilometers = i32;

let x: i32 = 5;
let y: Kilometers = 5;

println!("x + y = {}", x + y);
```

타입 별칭은 아래와 같이 긴 타입의 반복을 줄일 때 자주 사용된다.

```rust
Box<Fn() + Send + 'static>
```

아래와 같은 코드보다는

```rust
let f: Box<Fn() + Send + 'static> = Box::new(|| println!("hi"));

fn takes_long_type(f: Box<Fn() + Send + 'static>) {
    // --snip--
}

fn returns_long_type() -> Box<Fn() + Send + 'static> {
    // --snip--
}
```

아래와 같은 코드가 보기도 쉽고 관리하기도 편하다.

```rust
type Thunk = Box<Fn() + Send + 'static>;

let f: Thunk = Box::new(|| println!("hi"));

fn takes_long_type(f: Thunk) {
    // --snip--
}

fn returns_long_type() -> Thunk {
    // --snip--
}
```

표준 라이브러리 `std::io` 모듈을 보면 `Result<..., Error>` 타입이 참 많이 나온다.

```rust
use std::io::Error;
use std::fmt;

pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize, Error>;
    fn flush(&mut self) -> Result<(), Error>;

    fn write_all(&mut self, buf: &[u8]) -> Result<(), Error>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<(), Error>;
}
```

아래와 같이 타입 별칭을 선언하고

```rust
type Result<T> = Result<T, std::io::Error>;
```

`std::io` 모듈의 코드를 다음과 같이 줄일 수 있다.

```rust
pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize>;
    fn flush(&mut self) -> Result<()>;

    fn write_all(&mut self, buf: &[u8]) -> Result<()>;
    fn write_fmt(&mut self, fmt: Arguments) -> Result<()>;
}
```

`Result<T>`는 `Result<T, E>` 그 자체이기에 `Result<T, E>`의 메서드 뿐만아니라 `?` 같은 특수 문법도 사용할 수 있다.

## 결코 반환하지 않는 `!` 부정 타입

어떤 함수가 절대 값을 반환하지 않을 때, 반환 타입의 자리를 `!`가 대신한다.

아래 코드는 **함수 `bar`는 절대 값을 반환하지 않는다**라고 읽을 수 있다.

**발산 함수**라고도 부른다.

```rust
fn bar() -> ! {
    // --snip--
}
```

각 갈래값의 타입이 다른 아래 코드는 컴파일이 안돼도,

```rust
let guess = match guess.trim().parse() {
    Ok(_) => 5,
    Err(_) => "hello",
}
```

한 갈래가 `continue`로 끝나는 아래 코드는 컴파일이 된다.

```rust
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
```

`continue`가 `!` 값을 갖기 때문이다.

`!` 타입은 값을 반환하지 않으니 `match` 갈래값의 타입이 같아야 하는 제약이 적용되지 않는 것이다.

부정타입은 `panic!` 매크로에도 유용하게 쓰인다.

`Option<T>`에서 값을 생성하거나 패닉을 일으키는 `unwrap` 함수에도 부정 타입 이 쓰인다.

```rust
impl<T> Option<T> {
    pub fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!("called `Option::unwrap()` on a `None` value"),
        }
    }
}
```

`panic!`가 `!` 타입이므로 위의 `match`문은 유효하다.

마지막으로, `loop` 도 `!` 타입을 갖는다.

```rust
print!("forever ");

loop {
    print!("and ever ");
}
```

`break`가 없는 이상 루프는 끝나지 않으므로 `!` 타입을 갖는다.

## 동적 크기를 갖는 타입과 `Sized`

**DST(dynamically sized type)**는 런타임에서만 크기를 알 수 있는 값을 사용하는 코드를 작성할 수 있게 해준다.

`DST`는 **크기가 없는 타입(unsized type)**이라고도 불린다.

동적 크기 타입인 `str`을 세부적으로 알아본다.

문자열의 길이는 런타임에서만 알 수 있다.

```rust
// 작동 안되는 코드임
let s1: str = "Hello there!";
let s2: str = "How's it going?";
```

위의 코드를 보면 두 `str` 값은 타입이 같지만 다른 크기를 갖는다.

위의 코드를 작동시키기 위해서는 `str` 대신에 `&str`을 사용해야 한다.

문자열 그 자체가 아닌, 문자열 슬라이스의 시작 위치와 길이만 저장한다.

`&str`은 `str`의 주소와 길이를 갖는 두 개의 값을 나타낸다.

`str`의 크기를 몰라도 `&str`의 크기는 알 수 있으니 동적 크기를 갖는 타입을 사용할 수 있다.

트레잇 객체를 사용해서 서로 다른 타입의 값을 한번에 사용하려고 했을 때도, `&Trait`이나 `Box<Trait>` 같이 값을 포인터에 넣어서 사용했다.

러스트는 `Sized` 트레잇으로 `DST`를 다룬다.

`Sized` 트레잇은 어떤 타입의 크기를 컴파일 타임에 알 수 있는지 결정한다.

러스트는 암묵적으로 모든 제네릭 함수에게 `Sized` 트레잇 바운드를 추가한다.

다음과 같은 코드는

```rust
fn generic<T>(t: T) {
    // --snip--
}
```

다음과 같이 취급된다.

```rust
fn generic<T: Sized>(t: T) {
    // --snip--
}
```

`?Sized`는 `Sized`의 반대 개념이다.

`T`가 `Sized`일 수도 있고 아닐 수도 있음을 나타낸다.

아래와 같이 함수를 정의하면 컴파일 타입에 크기를 알 수 없는 타입을 사용할 수 있다.

```rust
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
```

크기를 알 수 없으므로 파라미터는 포인터를 사용해서 `T` 값을 다룬다.
