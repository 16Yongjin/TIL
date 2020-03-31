# 고급 함수와 클로저

함수 포인터로 함수를 인자로 넘기기, 클로저를 반환하는 방법을 다룬다.

## 함수 포인터

일반 함수도 함수 인자에 넘길 수 있다.

클로저를 새로 정의하긴 싫고 기존에 정의해둔 함수를 사용하고 싶을 때 유용하다.

함수가 함수를 인자로 받으려면 `fn` 타입을 가진 함수 포인터를 사용한다.

`Fn` 클로저 트레잇과 다르다.

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!("The answer is: {}", answer);
}
```

`do_twice` 함수는 `i32`를 받아서 `i32`를 반환하는 `fn`을 인자 `f`로 받는다.

함수 포인터는 클로저 트레잇 세 가지(`Fn`, `FnMut`, `FnOnce`) 모두를 구현하므로, 클로저를 인자로 받는 함수에도 함수 포인터를 넘길 수 있다.

> 그래서 클로저와 함수 모두를 인자로 받을 수 있게 함수를 작성하면 좋다.

인자로 `fn`만 허용하고 클로저는 거부해야 하는 경우가 있다.

ex) 클로저가 없는 C 함수를 사용하고 싶을 때

인라인 클로저나 이름 있는 함수 둘 중 아무거나 써도 되는 경우도 있다.

숫자 벡터를 스트링 벡터로 변환하는 다음 코드는 `map`의 인자로 클로저도 사용할 수 있고,

```rust
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> = list_of_numbers
    .iter()
    .map(|i| i.to_string())
    .collect();
```

다음과 같이 함수 이름을 사용할 수도 있다.

```rust
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> = list_of_numbers
    .iter()
    .map(ToString::to_string)
    .collect();
```

`to_string` 이름을 가진 함수가 여러 개 있기에, 완전 정규화 문법을 이용해서 `ToString::to_string`를 인자로 넘겼다.

두 사용예시 모두 컴파일 시 같은 코드가 되므로 보기 좋은 스타일을 골라서 코드를 작성하면 된다.

# 클로저 반환하기

클로저는 트레잇으로 표현되므로 클로저를 직접 반환할 수 없다.

트레잇 반환 시에는 그 트레잇을 구현한 구체 타입을 사용한다.

클로저는 반환할 수 있는 구체 타입이 없어서 반환할 수 없는 것이다.

다음과 같이 클로저를 직접 반환하려는 코드는 컴파일 시 실패하고 에러를 뿜어낸다.

```rust
fn returns_closure() -> Fn(i32) -> i32 {
    |x| x + 1
}
```

`` the trait bound `std::ops::Fn(i32) -> i32 + 'static:std::marker::Sized` is not satisfied ``

`Sized` 트레잇 관련 에러인데, 러스트가 클로저를 저장하는데 필요한 공간을 알 수 없어서 생긴 문제로 해석할 수 있다.

다음과 같이 `Box` 트레잇 객체를 사용해서 클로저를 반환할 수 있다.

```rust
fn returns_closure() -> Box<Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```
