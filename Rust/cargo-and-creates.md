# 러스트 Cargo와 Crates.io

## 목차

- 릴리즈 프로필을 이용해 빌드 커스터마이징하기
- crates.io 에 라이브러리 배포하기
- 대규모 작업을 위한 작업공간 구성하기
- crates.io 에서 바이너리 설치하기
- 커스텀 명령어로 Cargo 확장하기

&nbsp;

# 릴리즈 프로필을 이용해 빌드 커스터마이징하기

릴리즈 프로필로 코드 컴파일 옵션을 제어할 수 있다.

각 프로필들은 서로 독립적으로 설정된다.

## Cargo의 두 가지 주요 프로필

### 1. `dev` 프로필

`cargo build` 실행 시 사용한다.

### 2. `release` 프로필

`cargo build --release` 실행 시 사용된다.

둘이 실행 시 출력도 다르다.

```
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
$ cargo build --release
    Finished release [optimized] target(s) in 0.0 secs
```

## 프로필의 기본 설정

_Cargo.toml_ 파일에 `[profile.*]`이 없으면 아래 설정이 기본으로 적용된다.

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` 옵션에 0 ~ 3 사이의 값을 설정해서 컴파일 최적화 정도를 정할 수 있다.

&nbsp;

# Crates.io에 크레이트 배포하기

`Crates.io`에 직접 만든 패키지를 배포해서 공유할 수 있다.

## 문서 주석 만들기

슬래시 세 개(`///`)를 사용해서 문서 주석을 작성할 수 있다.

문서 주석을 작성하고 `cargo doc`으로 HTML 문서를 생성할 수 있다.

생성된 문서는 `target/doc` 디렉터러에 저장된다.

`cargo doc --open`을 실행하면 작성한 문서와 모든 의존성의 문서까지 HTML으로 생성하고 웹 브라우저에 띄운다.

![rust-doc](https://rinthel.github.io/rust-lang-book-ko/img/trpl14-01.png)

<center>문서는 이렇게 생김</center>

## 자주 사용하는 구절

- **Examples**: 코드 사용 예시.

- **Panics**: 문서에 나온 기능이 패닉을 일으키는 시나리오, 패닉을 피하기 위한 방법을 알려줘야 한다.

- **Errors**: 함수가 `Result`를 반환할 경우, 발생할 수 있는 에러의 종류와 발생 조건, 처리 방법을 알려줘야 한다.

- **Safety**: 함수가 `unsafe`인 경우, 이 함수가 안전하지 않은 이유와 지켜야할 불변성을 알려줘야 한다.

## 문서 주석으로 테스트하기

`cargo test` 실행 시 예제로 포함한 코드가 테스트 된다.

## 모듈 전체 문서화

`//!`로 시작하는 문서 주석을 추가하면 크레이트나 모듈 전체를 대상으로 문서를 작성할 수 있다.

아래와 같이 적으면

```
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain
//! calculations more convenient.

/// Adds one to the number given.
// --snip--
```

아래처럼 문서 상단에 주석 내용이 표시된다.

![rust-doc-doc](https://rinthel.github.io/rust-lang-book-ko/img/trpl14-02.png)

## `pub use`를 이용해 공개 API를 사용자 편의적으로 구조화하기

개발할 때 만든 구조가 사용자에게 불편할 수 있다.

예를 들어, `use my_crate::UsefulType;` 대신 `my_crate::some_module::another_module::UsefulType;`을 사용하는 것은 끔찍하다.

`pub use`를 사용하면 내부 구조를 그대로 유지하면서, 공개 구조를 다시 구성할 수 있다.

아래의 `art` 라이브러리는 `kinds` 모듈에 `PrimaryColor`, `SecondaryColor` 열거형이 들어있고, `utils` 모듈에 `mix` 함수가 들어있다.

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        // --생략--
    }
}
```

다른 크레이트에서 이 라이브러리를 사용하기 위해서는 현재 정의된 구조대로 항목을 가져와야 한다.

```rust
extern crate art;

use art::kinds::PrimaryColor;
use art::utils::mix;

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}
```

아래와 같이 `pub use`로 내부 항목을 밖으로 빼면,

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub use kinds::PrimaryColor;
pub use kinds::SecondaryColor;
pub use utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

사용자는 아래와 같이 편하게 라이브러리를 사용할 수 있다.

```rust
extern crate art;

use art::PrimaryColor;
use art::mix;

fn main() {
    // --생략--
}
```

API 문서에서도 필요한 항목을 더 쉽게 찾을 수 있다.

![rust-pub-use](https://rinthel.github.io/rust-lang-book-ko/img/trpl14-04.png)

## Cartes.io 계정 설정하기

1. crates.io에서 깃헙 계정으로 로그인
2. 계정 설정 페이지에서 API 키 발급
3. `cargo login <api key>` 명령어 실행

## 크레이트에 Metadata 추가하기

패키지 이름, 설명, 라이센스 메타데이터를 추가해야 crates.io에 패키지를 배포할 수 있다.

예시:

```toml
[package]
name = "guessing_game"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"
```

## Crates.io에 배포하기

한 번 crates.io 저장소에 올라간 코드는 수정할 수 없다.

프로젝트가 의존하고 있는 crates.io에 등록된 크레이트가 삭제되거나 변경돼서 빌드가 실패하는 일이 없도록 하기 위함이다.

`cargo publish` 명령어로 Crates.io에 패키지를 배포한다.

## `cargo yank`를 이용해 Crates.io 에서 버전 제거하기

특정 크레이트 버전에 문제가 생긴 경우, 새 프로젝트에서 해당 버전을 의존성으로 사용할 수 없게 막는다.

버전을 끌어내려도 기존의 프로젝트는 그 버전에 의존할 수 있고, 해당 버전을 다운받을 수도 있다.

`1.0.1` 버전 크레이트를 끌어내리기:

```
$ cargo yank --vers 1.0.1
```

`--undo` 옵션으로 끌어내리기 취소하고 해당 버전을 의존성 갖는 것을 허용하기:

```
$ cargo yank --vers 1.0.1 --undo
```

`yank` 실행으로 코드가 삭제되지 않으므로 비밀 정보를 실수로 업로드 했다면, 그 비밀 정보를 재설정해야 한다.

&nbsp;

# Cargo 작업공간(Workspace)

거대한 라이브러리 크레이트 여래의 라이브러리 크레이트로 분리할 수 있다.

## 작업공간 생성

작업공간은 동일한 _Cargo.lock_ 과 출력 디렉토리를 공유하는 패키지들의 집합이다.

일반적으로 바이너리 크레이트 하나와 라이브러리 크레이트 여러 개를 작업공간에 넣는다.

작업공간으로 사용할 폴더를 만들고 `Cargo.toml` 파일을 생성한다.

`Cargo.toml` 파일에는 `[package]` 같은 메타데이터는 없고 `[workspace]`로 시작하는 구절을 갖는다.

Cargo.toml의 내용:

```toml
[workspace]

members = [
    "adder",
    "add-one",
]
```

작업공간 디렉터리에 크레이트들을 만든다.

```
$ cargo new --bin adder
$ cargo new --lib add-one
```

디렉터리 구조는 아래처럼 된다.

```
├── Cargo.lock
├── Cargo.toml
├── add-one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

바이너리 크레이트에서 라이브러리 크레이트를 의존하게 만든다.

```toml
[dependencies]

add-one = { path = "../add-one" }
```

바이너리 크레이트를 실행하기 위해 `cargo run -p <패키지 이름>` 명령을 실행한다.

## 작업공간의 외부 크레이트에 의존성 갖기

외부 크레이트가 필요한 크레이트마다 `Cargo.toml`에 의존성을 추가해준다.

```toml
[dependencies]

rand = "0.3.14"
```

여러 곳에서 사용되는 외부 크레이트는 작업공간 내에서 재활용되기에 공간이 절약된다.

## 작업공간 내 크레이트 배포하기

각 크레이트 디렉터리로 가서 `cargo publish` 명렁을 실행해야 한다.

모든 크레이트를 한번에 배포할 방법은 없다.
