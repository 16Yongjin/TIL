# 러스트의 반복자

반복자 패턴은 시퀀스의 항목을 순회하고 순회 종료 시점을 결정하는 로직을 추상화한다.

## 반복자는 게으르다.

아래 코드는 벡터의 `iter` 메서드를 호출하지만, 그 자체로 의미있는 동작을 하지 않는다.

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();
```

아래처럼 반복자에 `for` 루프로 호출해야, 반복자의 요소가 사용된다.

```rust
for val in v1_iter {
    println!("Got: {}", val);
}
```

반복자를 지원하지 않는 언어에서는, 0부터 시작하는 인덱스 변수를 증가시키고, 변수로 벡터에 접근해서 값을 가져와야 한다.

이 과정에서 코드가 잘못 작성될 위험이 크다.

반복자는 시퀀스를 순회하는 로직을 대신 처리해줘서 인덱스 변수로 인해 코드가 엉망이 될 수 있는 상황을 방지해준다.

덤으로, 다양한 자료구조에 대해 동일한 로직도 사용할 수 있게 해준다.

## `Iterator` 트레잇과 `next` 메서드

모든 반복자는 아래의 `Iterator` 트레잇을 구현한다.

```rust
trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // 생락된 기타 메서드
}
```

`Item`로 시퀀스 항목의 타입을 알려주고 `next` 메서드를 구현하면 반복자가 완성된다.

`next` 메서드는 순회할 때마다 각 항목을 `Some`에 넣어서 반환하고, 반복자 종료 시 `None`을 반환한다.

## 반복자의 `next` 메서드는 직접 호출할 수 있다.

대신에 반복자는 가변으로 만들어져야 한다.

```rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];

    let mut v1_iter = v1.iter();

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```

`for` 사용 시에는 루프가 반복자의 소유권을 갖고 내부적으로 반복자를 가변으로 만들기 때문에 반복자가 가변일 필요 없다.

`next` 호출로 얻어온 값들은 벡터 안에 있는 값들의 불변 참조이다.

시퀀스 내부 항목의 소유권을 갖고 싶으면 `iter` 대신 `into_iter`을 사용하고, 가변 참조를 원한다면 `iter_mut`을 사용한다.

## 반복자를 소비하는 메서드

`Iterator`의 메서드의 `next`만 구현하면 사용할 수 있는 유용한 메서드가 많다.

`next`를 호출하는 메서드를 _소비하는 어댑터_ 라고 한다.

소비하는 어댑터가 호출되면 반복자를 다 써버리고 반복자는 빈 껍데기만 남는다.

그 중 하나가 반복자를 순회하면서 각 항목의 누적합계를 내는 `sum`메서드다

```rust
#[test]
fn iterator_sum() {
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    let total: i32 = v1_iter.sum();

    assert_eq!(total, 6);
}
```

## 다른 반복자를 생성하는 메서드들

반복자에서 다른 반복자로 변경하고 생성하는 메서드를 _반복자 어댑터_ 라고 한다.

단순한 반복자 어댑터 호출을 연결하면 복잡한 연산도 쉽게 수행할 수 있다.

모든 반복자는 게으르므로, 마지막에 소비하는 메서드를 호출해야 반복자 어댑터에서 결과를 얻을 수 있다.

아래 `map` 메서드는 벡터의 각 항목에 1을 증가한 새로운 반복자를 만든다.

```rust
let v1: Vec<i32> = vec![1, 2, 3];

v1.iter().map(|x| x + 1);
```

하지만, 위의 코드를 실행하면 다음과 같이 어댑터를 소비라는 경고가 발생한다.

```
unused `std::iter::Map` which must be used: iterator adaptors are lazy and do nothing unless consumed
```

`collect` 메서드를 호출하면 반복자를 소비할 수 있다.

```rust
let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
```

## 환경을 캡쳐하는 클로저 사용하기

`filter`는 불리언을 반환하는 클로저를 인자로 받는다.

클로저가 `true`를 반환하는 요소만 새로 생성되는 반복자에 포함된다.

아래 코드의 `shoes_in_my_size` 함수는 `Shoe` 구조체 인스턴스 벡터를 순회하면서, 캡처된 `shoe_size` 변수와 값이 같은 항목만 새로운 반복자에 넣는다.

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_my_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter()
        .filter(|s| s.size == shoe_size)
        .collect()
}

#[test]
fn filters_by_size() {
    let shoes = vec![
        Shoe { size: 10, style: String::from("sneaker") },
        Shoe { size: 13, style: String::from("sandal") },
        Shoe { size: 10, style: String::from("boot") },
    ];

    let in_my_size = shoes_in_my_size(shoes, 10);

    assert_eq!(
        in_my_size,
        vec![
            Shoe { size: 10, style: String::from("sneaker") },
            Shoe { size: 10, style: String::from("boot") },
        ]
    );
}
```

## `Iterator` 트레잇으로 자신만의 반복자 만들기

커스텀 타입에 `Iterator`를 구현해서 원하는 동작을 하는 반복자를 생성할 수 있다.

`next` 메서드만 구현하면 된다.

다음은 1에서 5까지 카운트하는 반복자를 만든다.

`Counter::new` 함수로 새 인스턴스가 `count`가 0으로 시작하도록 강제한다.

`Iterator`의 `next` 메서드는 `count`가 6보다 작으면, 현재값을 `Some`에 넣어서 반환하고, 아니면 `None`을 반환한다.

```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;

        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
```

아래 테스트는 새로운 `Counter` 인스턴스에 `next`를 호출하면서 반복자가 원하는 행위를 구현했음을 확인한다.

```rust

#[test]
fn calling_next_directly() {
    let mut counter = Counter::new();

    assert_eq!(counter.next(), Some(1));
    assert_eq!(counter.next(), Some(2));
    assert_eq!(counter.next(), Some(3));
    assert_eq!(counter.next(), Some(4));
    assert_eq!(counter.next(), Some(5));
    assert_eq!(counter.next(), None);
}
```

## 다른 `Iterator` 메서드 사용하기

`next`만 구현하면 공짜로 따라오는 메서드들이 있다.

아래와 같이 `zip`, `map`, `filter`, `sum` 등 다양한 메서드를 사용할 수 있다.

```rust
#[test]
fn using_other_iterator_trait_methods() {
    let sum: u32 = Counter::new().zip(Counter::new().skip(1))
                                 .map(|(a, b)| a * b)
                                 .filter(|x| x % 3 == 0)
                                 .sum();
    assert_eq!(18, sum);
}
```

## 반복자는 제로비용 추상화

반복자를 사용하면 고수준의 추상화를 할 수 있지만, 캄파일 시 성능은 저수준의 코드와 동일이다.

추상화를 해도 런타임 오버헤드가 없기때문에 반복자는 제로비용 추상화이다.
