# 러스트의 모듈

## 라이브러리 크레이트란

다른 사람들이 프로젝트의 디펜던시로 추가할 수 있는 크레이트 ex) `rand` 크레이트

## 라이브러리 크레이트 생성하기

`--lib` 옵션을 붙인 명령어로 라이브러리 크레이트를 만든다.

```
cargo new --lib communicator
```

생성된 크레이트는 `main.rs`가 없어서 `cargo run` 커맨드로 코드를 실행할 수는 없고 `cargo build`로 코드 컴파일만 가능하다.

## `mod`로 모듈 정의하기

`mod` 키워드로 모듈을 네임스페이스를 정의한다.

```rust
mod network {
    fn connect() {
    }
}
```

모듈 내 함수 사용 시 `network::connect()`와 같이 사용한다.

## 모듈 안에 모듈 넣기

두 가지 방법이 있다.

1. 상위 모듈의 네임스페이스 안에 직접 하위 모듈을 넣기
2. 하위 모듈을 파일로 분리하고 `mod 모듈명` 선언으로 상위 모듈로 불러오기

## 1. 직접 넣기

상위 모듈의 네임스페이스 안에 직접 하위 모듈을 넣는다.

```rust
mod network {
    fn connect() {}

    mod client {
        fn connect() {}
    }
}
```

`network` 모듈에 `network::connect`와 `network::client::connect` 함수가 생겼다.

## 2. 선언으로 불러오기

하위 모듈을 파일로 분리하고 `mod 모듈명` 선언으로 상위 모듈로 불러온다.

`src/client.rs`을 만들고 아래 코드를 작성하면 `client` 모듈이 생성된다.

```rust
fn connect() {}
```

그리고, `src/lib.rs`에서 `mod client;` 선언으로 `client` 모듈을 가져온다.

```rust
mod client;

mod network {
    fn connect() {}
}
```

## 디렉터리로 모듈 선언하기

디렉터리의 이름으로 모듈을 정의한다.

`lib/network` 디렉터리를 만들고 `mod.rs` 파일을 만들면 `mod.rs`의 내용으로 `network` 모듈이 생성된다.

`lib/network` 디렉터리에 `server.rs` 파일을 만들고

`mod.rs`에서 `mod server;`으로 가져오면 `network::server` 모듈이 생긴다.

```
├── src
│   ├── client.rs
│   ├── lib.rs
│   └── network
│       ├── mod.rs
│       └── server.rs
```

위의 구조로 `client`, `network`, `network::server` 모듈이 생겼다.

`mod.rs` 파일이 Node.js의 `index.js`처럼 디렉터리의 기본 임포트 파일처럼 작동한다.

&nbsp;

## `pub`으로 모듈 노출시키기

사용할 함수 앞에 `pub` 키워드를 붙이면

컴파일 시 나오는 `function is never used` 경고를 없애며

라이브러리 사용 시 나오는 `` module `모듈명` is private `` 에러를 없앨 수 있다.

ex)

```rust
pub fn connect() {}
```

&nbsp;

## `use`로 네임스페이스 사용하기

아래 모듈의 `nested_modules()` 함수를 사용하려면

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}
```

함수 이름을 참조하기 위해 긴 네임스페이스들을 입력해야 한다.

```rust
fn main() {
    a::series::of::nested_modules();
}
```

`use` 키워드를 사용해서 `nested_modules()` 함수를 루트 스코프로 가져올 수 있다.

```rust
use a::series::of::nested_modules;

fn main() {
    nested_modules();
}
```

## 여러 개의 항목 가져오기

중괄호와 쉼표로 열거형 내부의 variant들을 선택적으로 가져올 수 있다.

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::{Red, Yellow};

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = TrafficLight::Green;
}
```

## 모든 항목 가져오기

`*` 문법으로 네임스페이스 내의 모든 항목을 가져올 수 있다.

```rust
use TrafficLight::*;
```

## `super`로 부모 모듈 접근하기

`src/lib.rs`에 있는 테스트를 위한 `tests` 모듈도 하나의 모듈이다.

아래와 같이 `client::connect();`로 내부의 다른 모듈을 참조하면 선언되지 않은 모듈 에러가 난다.

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        client::connect();
    }
}
```

`::`로 시작해서 루트부터 모듈 경로 시작하거나

`::client::connect();`

`super`를 사용해서 현재 모듈에서 한 모듈 위부터 모듈 경로를 시작할 수 있다.

`super::client::connect();`

둘 중 한 가지 방법으로 `tests` 모듈에서 같은 라이브러리 내 모듈을 참조할 수 있다.

테스트할 함수마다 `super`를 붙이기 귀찮으면 `use` 키워드를 사용하면 된다.

```rust
#[cfg(test)]
mod tests {
    use super::client;

    #[test]
    fn it_works() {
        client::connect();
    }
}
```
