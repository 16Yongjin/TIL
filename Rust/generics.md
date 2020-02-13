# 러스트의 제네릭

## 제네릭이란

컨셉의 복제를 위한 도구

구체 타입의 추상화

ex) `Option<T>`, `Vec<T>`, `HashMap<K, V>`, `Result<T, E>`

## 함수를 추출해서 중복 없애기

아래 프로그램은 리스트에서 가장 큰 숫자를 찾아낸다.

```rust
fn main() {
    let numbers = vec![34, 50, 25, 100, 65];

    let mut largest = numbers[0];

    for number in numbers {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
}
```

서로 다른 숫자 리스트에서도 가장 큰 숫자를 찾을려면 코드를 복사해야 한다.

```rust
fn main() {
    let numbers = vec![34, 50, 25, 100, 65];

    let mut largest = numbers[0];

    for number in numbers {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);

    let numbers = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let mut largest = numbers[0];

    for number in numbers {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
}
```

가장 큰 숫자를 찾는 코드가 중복되어 있다.

중복되는 코드를 `largest`라는 이름의 함수로 추출할 수 있다.

```rust
fn largest(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}
```

## 함수 정의할 때 제네릭 데이터 타입을 이용하기

위의 `largest` 함수는 `i32` 슬라이스에만 사용할 수 있다.

가장 큰 `char`을 찾을려면 함수를 또 만들어야 한다.

```rust
fn largest_i32(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> char {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}
```

함수 `largest_i32`와 `largest_char`는 똑같은 함수 본체를 가지고 있다.

제네릭 파라미터를 사용해서 두 함수를 하나로 만들어서 중복을 피할 수 있다.

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

fn main() {
    let numbers = vec![34, 50, 25, 100, 65];

    let result = largest(&numbers);
    println!("The largest number is {}", result);

    let chars = vec!['y', 'm', 'a', 'q'];

    let result = largest(&chars);
    println!("The largest char is {}", result);
}
```

근데 위 코드를 컴파일하면 아래와 같은 에러가 나온다.

```
error[E0369]: binary operation `>` cannot be applied to type `T`
  |
5 |         if item > largest {
  |            ^^^^
  |
note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

입력될 수 있는 모든 `T` 타입에 대해 동작하지 않을 거라는 에러다.

`T`가 `std::cmp::PartialOrd` 트레잇(trait)을 구현해서, 비교 연산이 가능하다는 것을 컴파일러에게 알려줘야 에러가 사라진다.

## 구조체 정의할 때 제네릭 데이터 타입 사용하기

다음은 `x`와 `y` 좌표값을 가질 수 있는 `Point` 구조체 정의다.

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

`x`와 `y`가 동일한 타입을 가질 것임을 보장한다.

`x`와 `y`에 다른 타입을 대입하면 컴파일 에러난다.

`x`와 `y`가 서로 다른 타입을 가지게 하려면 `T`와 `U` 두 가지 제네릭 타입을 사용하면 된다.

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

## 열거형 정의할 때 제네릭 데이터 탸입 사용하기

값이 있을 수도 있고 없을 수도 있음을 나타내는 `Option<T>`

아래 정의 하나면 중복없이 "옵션 값"이라는 추상적인 개념을 어떤 타입에도 사용할 수 있다.

```rust
enum Option<T> {
    Some(T),
    None,
}
```

성공 또는 실패 상태를 나타내는 `Result`

성공했을 때의 타입 `T`의 값을 `Ok`가 가지고 있고, 실패 시 타입 `E`의 값을 `Err`가 가지고 있는다.

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

## 메서드 정의할 때 제네릭 데이터 타입 사용하기

`impl` 뒤에 `T`를 정의해야 `Point<T>` 메서드 구현 시 `T`를 사용할 수 있다.

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

## 제네릭을 이용한 코드의 성능

제네릭을 써도 런타임에 느려지지 않는다.

러스트는 컴파일 시에 _단형성화(monomorphization)_ 를 실행한다.

제네릭 코드를 실제로 사용할 구체 타입으로 바꾼다는 말이다.
