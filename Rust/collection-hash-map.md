## 러스트의 해쉬맵

키 타입 `K`에 값 타입 `V`를 매핑한 해쉬맵 타입 `HashMap<K, V>`

JS의 오브젝트, 맵, 파이썬의 딕셔너리와 같다.

벡터처럼 인덱스 사용하기 보단 임의 타입의 키로 데이터를 찾을 때 유용하다.

## `new`로 해쉬맵 생성하기

해쉬맵의 `new` 함수로 생성하고 `insert`로 요소를 추가한다.

해쉬맵이 다른 컬렉션보다 잘 사용되는 편이 아니라 prelue에 불러져 있지 않아서, 표준 라이브러리에서 가져와야 한다. (해쉬맵 생성 매크로도 없다.)

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

## 벡터의 `collect()` 메서드로 해쉬맵 생성하기

키, 값으로 된 튜플 벡터에 `collect()` 함수를 호출해서 해쉬맵을 생성한다.

```rust
use std::collections::HashMap;

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

`HashMap<_, _>`로 타입 명시를 해서 벡터에 담긴 타입으로 해쉬에 담길 타입을 추론하게 한다.

## 해쉬맵과 소유권

`Copy` 트레잇을 구현 타입(ex, `i32`)은 값들이 해쉬맵으로 복사된다.

`String` 등은 해쉬맵으로 소유권이 옮겨진다.

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name과 field_value은 이 지점부터 유효하지 않다.
```

해쉬맵에 참조자를 넣으면 소유권은 이동하지 않지만 참조하는 값들이 해쉬맵이 유효할 때까지 유효해야 한다.

## `get` 메서드로 해쉬맵 내의 값 접근하기

`get` 메서드에 키를 제공해서 값을 얻어온다.

얻어온 값은 `Option<&V>` 타입을 가지고 있다.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

## `for` 루프로 키/값 쌍에 반복작업하기

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

## 해쉬맵 값 덮어쓰기

이미 있는 키에 다른 값을 삽입하면 값이 새 값으로 대신된다.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10); // 블루가 10이었다가
scores.insert(String::from("Blue"), 25); // 25가 됨

println!("{:?}", scores); // {"Blue": 25} 출력
```

## `entry` 메서드로 키에 할당된 값이 없을 경우에만 삽입하기

`entry` 메서드는 열거형 `Entry`를 반환하면서 해당 키 존재 여부를 나타낸다.

`Entry`의 `or_insert or_insert` 메서드는 키가 존재하면 해당 값을 반환하고

아니면 새 값을 삽입 후 수정된 `Entry` 값을 반환한다.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10); // "Blue" 키는 이미 존재

scores.entry(String::from("Yellow")).or_insert(50); // 없어서 잘 들감
scores.entry(String::from("Blue")).or_insert(50); // 이미 있어서 안 들감

println!("{:?}", scores); // {"Yellow": 50, "Blue": 10}
```

## 예전 값을 기초로 값을 갱신하기

아래 코드는 `text` 내의 각 단어의 빈도수를 센다.

처음 본 단어면 0을, 아니면 해당 빈도수에 1을 더한 값을 삽입한다.

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map); // {"world": 2, "hello": 1, "wonderful": 1}
```

## 해쉬 함수

`HashMap`의 기본 해쉬 함수는 보안을 위해 성능을 절충했다.

원한다면, 해쉬 함수를 `BuildHasher` 트레잇을 구현한 다른 함수로 변경할 수 있다.
