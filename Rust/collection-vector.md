## 러스트의 벡터

같은 타입의 값을 메모리 상에 이웃하도록 저장한다.

## 새 백터 만들기

비어있는 벡터를 생성한다.

값이 아무것도 들어있지 않아 타입을 명시해야 한다.

```rust
let v: Vec<i32> = Vec::new();
```

## `vec!` 매크로로 벡터 만들기

초기값을 제공해서 타입을 명시할 필요 없다.

```rust
let v = vec![1, 2, 3];
```

## 벡터 갱신하기

`mut` 키워드로 변수를 가변으로 만들어야 값을 추가할 수 있다.

추가되는 데이터에서 타입을 추론하기 때문에 `Vec<i32>` 명시할 필요 없다.

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```

## 벡터의 요소 읽기

인덱스로 `T` 값을 얻거나 `get(인덱스)` 메서드로 `Option<T>` 값을 얻을 수 있다.

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
let third: Option<&i32> = v.get(2);
```

벡터의 크기를 벗어나는 인덱스에 직접 접근하면 `panic!`을 일으킨다.

같은 인덱스를 `get()` 메서드로 접근하면 `None`이 반환된다.

```rust
let v = vec![1, 2, 3, 4, 5];

let does_not_exist = &v[100]; // panic!
let does_not_exist = v.get(100); // None
```

## 벡터에도 참조자 규칙이 그대로 적용됨

아래 코드는 불변 참조자를 만든 뒤 가변 참조자를 만들려고 해서 에러가 난다.

```rust
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);
```

벡터에 새 요소 추가 시, 저장 공간 부족으로 새로운 메모리 공간에 벡터 요소들을 옮기는 일이 발생할 수 있다.

이 경우 `first`는 할당 해제된 메모리를 가리키게 되는데, 빌림 규칙으로 이런 상황을 막을 수 있다.

## 벡터 요소 반복처리

불변 참조자를 얻어서 반복하기

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}
```

가변 참조자를 얻어서 각 요소 변경하기

값을 변경하기 전에 역참조 연산자(`*`)를 사용해서 참조에서 값을 얻어야 한다.

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

## 열거형으로 여러 타입 저장하기

열거형 내에 다양한 값이 있어도 모두 같은 타입으로 다뤄진다.

벡터 내에 다양한 타입의 값을 넣고 싶으면 열거형을 사용하면 된다.

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

각 요소를 저장하기 위해 필요한 힙 메모리 크기를 알기 위해 컴파일 타임에 벡터에 저장할 타입을 알아야 한다.

저장할 타입이 런타임에 정해져서 모른다면 트레잇 객체를 사용한다.
