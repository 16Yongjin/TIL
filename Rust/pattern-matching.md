# 러스트의 패턴매칭

패턴은 복잡한 타입 구조에서 값을 비교하기 위한 문법이다.

## 패턴 조합 요소

- 리터럴 값
- 분해한 배열, 열거형, 구조체, 튜플
- 와일드카드
- 임시값 (플레이스홀더)

## 패턴이 사용될 수 있는 모든 곳

`match` 키워드는 다음과 같은 구조를 가진다.

```rust
match 값 {
    패턴 => 표현,
    패턴 => 표현,
}
```

대응되는 값은 모든 경우를 빠짐없이 표현해야 한다.

값을 무시하고 싶으면 `_` 패턴을 사용

## `if let` 조건 표현

`match`를 사용하려면 값이 가질 수 있는 모든 경우에 대응해야 한다.

한 가지 경우만 다루고 싶다면, `if let` 표현으로 더 짧은 코드를 작성할 수 있다.

`if let`, `else if`, `else if let` 표현을 섞어서 사용할 수 있다.

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {}, as the background", color);
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

위 코드는 `Using purple as the background color`를 출력한다.

`if let Ok(age) = age`에서 추출한 `Ok` 안의 `age` 변수는 조건문 아래의 새로운 스코프에서 사용할 수 있다.

그래서 `if let Ok(age) = age && age > 30` 이렇게 못 쓴다.

## `while let` 조건 루프

```rust
let mut stack = Vec::new();

stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
    println!("{}", top);
}
```

위 코드는 `vector` 비어서 `None`을 반환하기 전까지 스택의 내용을 출력한다.

## `for` 루프

`for x in y`에서 `for` 바로 다음에 나오는 키워드가 패턴이다.

```rust
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```

`enumerate` 메서드는 반복자를 값과 인덱스로 구성된 튜플로 순회할 수 있게 한다.

## `let` 구문

```rust
let (x, y, z) = (1, 2, 3);
```

위 코드는 튜플을 분해해서 각각을 변수에 대입한다.

```rust
let (x, y) = (1, 2, 3);
```

위와 같은 코드는 타입이 다르면 패턴도 다르기에 컴파일 에러난다.

`_`나 `..`를 추가해서 값을 무시할 수도 있다.

## 함수 매개변수

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

매개변수도 패턴매칭으로 분해해서 사용할 수 있다

# 반증 가능성(Refutability)

*반증 가능성*은 패턴이 매칭에 실패할지의 여부를 뜻한다.

그에 반대인 *반증 불가 패턴*은 모든 값에 대응하는 패턴이다.

`let x = 5;`는 `x`가 모든 값에 대응하기에 실패할 수 없어서 반증 불가 패턴이다.

`if let Some(x) = a_value;`는 `a_value`가 `None`인 경우 실패하기에 반증 가능 패턴이다.

`let` 구문, `for` 루프는 반증 불가한 패턴만 허용한다.

`if let`과 `while let` 표현은 반증 가능 패턴만 허용한다.

이 둘의 구분은 컴파일러가 도와준다.

## 반증 불가 패턴이 필요한 곳에서 반증 가능 패턴을 쓰는 경우

```rust
let Some(x) = some_option_value;
```

반증 불가 패턴만 허용하는 `let`에 반증 가능 패턴을 사용했다.

`None`에 대응하지 않아서 컴파일 에러가 난다.

```rust
if let Some(x) = some_option_value {
    println!("{}", x);
}
```

`let` 대신 반증 가능 패턴을 허용하는 `if let`을 사용하면 해결된다.

## 반증 가능 패턴이 필요한 곳에서 반증 불가 패턴을 쓰는 경우

```rust
if let x = 5 {
    println!("{}", x);
};
```

`if let`은 반증 가능 패턴만 허용하지만, `x = 5`는 틀릴 수 없는, 반증 불가 패턴이기에 컴파일러 에러가 발생한다.

# 패턴 문법의 모든 것

모든 패턴 문법 나열

## 리터럴 매칭

```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

`x`가 1이므로 `one`을 출력한다.

## 명명 변수 매칭

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {:?}", y),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {:?}", x, y);
}
```

`match` 문 스코프 속 `Some(y)`의 `y`는 밖에 `y`를 덮어쓴다.

`Matched, y = 5`를 출력한다.

## 다중 패턴

`or`을 뜻하는 `|` 구문을 이용하면 패턴을 여러개 매치할 수 있다.

```rust
let x = 1;

match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

`one or two`를 출력한다.

## `...`로 값의 범위 매칭하기

숫자와 `char` 값에만 사용할 수 있다.

아래는 숫자 값의 범위를 매칭하는 예시다.

```rust
let x = 5;

match x {
    1 ... 5 => println!("one through five"),
    _ => println!("something else"),
}
```

`|` 이용하면 `1 | 2 | 3 | 4 | 5`로 작성해야 하는 것을 `1 ... 5`로 간편하게 작성할 수 있다.

아래는 `char` 값의 범위를 매칭한다.

```rust
let x = 'c';

match x {
    'a' ... 'j' => println!("early ASCII letter"),
    'k' ... 'z' => println!("late ASCII letter"),
    _ => println!("something else"),
}
```

## 값을 해체하여 분리하기

구조체, 열거형, 튜플, 참조자 등을 해체(destructuring)할 수 있다.

## 구조체 해체하기

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7};

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
}
```

`p` 변수의 필드 `x`, `y`의 값을 각각 변수 `a`와 `b`에 대입한다.

패턴이 구조체 필드명과 일치하면 약칭 구문을 사용할 수 있다.

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

필드 `x`, `y`의 값을 각각 같은 이름의 변수 `x`와 `y`에 대입한다.

## 구조체 패턴 일부에 리터럴 값을 이용해서 매치하기

```rust
fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
}
```

각 갈래는 `y`가 0인 경우, `x`가 0인 경우, 나머지 경우를 매치한다.

## 열거형 해체

각 패턴은 내장된 데이터 정의가 일치해야 열거형을 해체할 수 있다.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.")
        },
        Message::Move { x, y } => {
            println!(
                "Move in the x direction {} and in the y direction {}",
                x,
                y
            );
        }
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
    }
}
```

## 참조자 해체

참조자를 가진 값은 패턴 내에서 `&`를 사용해 값을 해체한다.

```rust
let points = vec![
    Point { x: 0, y: 0 },
    Point { x: 1, y: 5 },
    Point { x: 10, y: -3 },
];

let sum_of_squares: i32 = points
    .iter()
    .map(|&Point { x, y }| x * x + y * y)
    .sum();
```

`iter` 메서드가 반환하는 값들은 실제 값이 아닌 참조자다.

위 코드에서 `&`를 빼면 타입 불일치 에러가 발생한다.

## 구조체와 튜플 해체

아래 같이 튜플과 구조체가 섞인 복잡한 패턴 해체도 가능하다.

```rust
let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });
```

## 패턴 내에서 값 무시하기

`_는`match`의 마지막 갈래에서 아무것도 안 하는 나머지 값을 매치할 때 유용하다.

값의 나머지를 무시하기 위헤 `..`도 사용할 수 있다.

## `_`를 이용해 전체 값 무시하기

아래 코드는 매개변수를 `_`로 무시한다.

```rust
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}

fn main() {
    foo(3, 4);
}
```

특정 타입의 시그니처가 필요하지만, 함수 본문에서 매개변수가 필요없는 경우 유용하다.

## 중첩된 `_`를 이용해 값의 일부 무시하기

```rust
let mut setting_value = Some(5);
let new_setting_value = Some(10);

match (setting_value, new_setting_value) {
    (Some(_), Some(_)) => {
        println!("Can't overwrite an existing customized value");
    }
    _ => {
        setting_value = new_setting_value;
    }
}

println!("setting is {:?}", setting_value);
```

`setting_value`와 `new_setting_value`가 둘 다 `Some` 타입인지만 확인하고 내부 값은 무시한다.

```rust
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, _, third, _, fifth) => {
        println!("Some numbers: {}, {}, {}", first, third, fifth)
    },
}
```

언더스코어를 여러번 사용할 수 있다.

두 번째 네 번째 요소만 무시한다.

## 사용하지 않는 변수 무시하기

러스트는 변수를 생성하고 사용하지 않으면 경고한다.

프로그램을 작성하는 중 변수를 미리 만들어 놓은 경우, 이런 경고가 귀찮을 수 있다.

```rust
fn main() {
    let _x = 5;
    let y = 10;
}
```

미사용 변수 앞에 언더스코어를 붙이면 경고를 끌 수 있다.

## 언더스코어로 시작하는 변수는 값이 바인드된다.

```rust
let s = Some(String::from("Hello!"));

if let Some(_s) = s {
    println!("found a string");
}

println!("{:?}", s);
```

그냥 `_`와는 다르게 `_s`는 값이 바인드되어 `s`의 소유권을 가져간다.

`println!("{:?}", s);`에서 이미 옮겨진 값을 사용하려고 해서 에러가 발생한다.

## `..`를 이용해 값의 나머지 부분 무시하기

```rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}
```

구조체의 필드 `x`만 사용하고 나머지 필드는 무시한다.

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {}, {}", first, last);
        },
    }
}
```

튜플의 첫 번째와 마지막 값만 매치하고 나머지는 모두 무시한다.

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (.., second, ..) => {
            println!("Some numbers: {}", second)
        },
    }
}
```

위와 같이 `..`를 두 곳에서 사용하는 경우 `second`를 특정할 수 없어서 에러가 발생한다.

## `ref` 와 `ref mut` 를 이용해 패턴 내에서 참조자 생성하기

`ref`로 참조자를 만들어 값의 소유권이 패턴 내의 변수로 이동하는 것을 막을 수 있다.

```rust
let robot_name = Some(String::from("Bors"));

match robot_name {
    Some(name) => println!("Found a name: {}", name),
    None => (),
}

println!("robot_name is: {:?}", robot_name);
```

위 코드는 `robot_name`의 소유권이 `match` 스코프 내 `name`로 이동했다.

이미 이동한 `robot_name`을 출력하려고 해서 컴파일 에러가 발생한다.

단순히 `&`를 사용해서 `Some(name)`을 `Some(&name)` 바꾼다고 해결이 되지 않는다.

패턴 내 `&`는 참조자를 생성하는게 아니라 이미 존재하는 참조자를 값으로 매치한다.

대신에 아래와 같이 변수 `name` 앞에 `ref` 키워드를 붙이면 패턴에 참조자를 생성할 수 있다.

```rust
let robot_name = Some(String::from("Bors"));

match robot_name {
    Some(ref name) => println!("Found a name: {}", name),
    None => (),
}

println!("robot_name is: {:?}", robot_name);
```

가변 참조자를 생성하려면 `ref mut`을 사용한다.

```rust
let mut robot_name = Some(String::from("Bors"));

match robot_name {
    Some(ref mut name) => *name = String::from("Another name"),
    None => (),
}

println!("robot_name is: {:?}", robot_name);
```

가변 참조자인 `name`의 값을 변경하기 위해서는 `*` 연산자로 역참조해야 한다.

## 매치 가드를 이용한 추가 조건

매치 가드는 `match` 갈래 뒤에 붙은 `if` 문이다.

다음 코드는 `Some` 타입이면서 값이 5 미만인 패턴을 매치한다.

```rust
let num = Some(4);

match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
```

`|` 다중 패턴 뒤에도 매치 가드를 사용할 수 있다.

```rust
let x = 4;
let y = false;

match x {
    4 | 5 | 6 if y => println!("yes"),
    _ => println!("no"),
}
```

첫 번째 갈래는 `4 | 5 | 6` 패턴 전체에 해당되면서 `y`가 `true`인 경우를 뜻한다.

4 또는 5이거나, 6이면서 `y`가 `true`인 경우가 아니다.

## `@`(at 연산자) 바인딩

`@`는 값이 패턴과 매치되는 동시에 해당 값을 가진 변수를 생성한다.

아래 코드는 `Message::Hello`의 `id` 필드가 `3..7` 범위내 있다면 그 값을 `id_variable`에 바인드한다.

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3...7 } => {
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10...12 } => {
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
}
```
