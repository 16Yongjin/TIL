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

값을 무시할 경우, `_` 패턴을 사용한다.

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
