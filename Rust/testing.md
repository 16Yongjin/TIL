## 러스트의 테스트

프로그램이 기대하는 기능을 하는지 검증한다.

## 테스트 함수의 동작

1. 필요한 데이터나 상태를 설정하기
2. 테스트하고 싶은 코드를 실행하기
3. 결과가 예상대로인지 단언하기(assert)

## 러스트의 테스트 함수

`#[test]` 어노테이션이 위에 달려있는 함수

`cargo test` 커맨드를 실행하면 `test` 속성이 달려있는 함수를 실행하고 테스트 성공 여부를 보고한다.

ex)

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

위의 테스트를 실행하면 아래와 같이 나온다.

```
$ cargo test

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

테스트가 1 중 1개가 통과했기에 `1 passed; 0 failed;`,

`#[ignore]` 어노테이션으로 무시한 테스트가 없기 때문에 `0 ignored;`,

벤치마크 테스트를 하지 않아서 `0 measured;` (나이틀리 러스트에서만 가능)

라고 나왔다.

## `assert!` 매크로로 결과 확인하기

`assert!` 매크로의 인자로 `true`가 들어가면 아무일도 일어나지 않고

`false`가 들어가면 `panic!` 매크로가 호출돼서 테스트를 실패하게 한다.

## assert_eq!와 assert_ne!를 이용한 Equality 테스트

`aseert_eq!`는 입력되는 두 인자가 같은지 확인하고

`assert_ne!`는 입력 인자가 다른지 확인한다.

다른 테스트 프레임워크에서는 단언 함수의 파리미터를 `expected`와 `actual`로 불러서 인자를 넣는 순서가 중요한데

`aseert_eq!` 매크로는 인자를 `left`와 `right`로 불러서 인자 넣는 순서가 중요하지 않다.

&nbsp;

`assert_eq!`와 `assert_ne!` 매크로는 각각 ==과 != 연산자를 이용한다.

비교되는 값들이 `PartialEq`와 `Debug` 트레잇을 구현해야 한다.

두 트레잇이 없으면 `#[derive(PartialEq, Debug)]` 어노테이션을 추가하면 된다.

## 커스텀 실패 메시지 추가하기

`assert!`, `assert_eq!`, `assert_ne!` 매크로는 필요한 인자 이후의 인자들은 `format!` 매크로에 넘겨진다.

아래 `assert!` 매크로는 테스트 실패 첫 번쨰 인자 뒤에 있는 인자들을 출력한다.

```rust
fn greeting_contains_name() {
    let result = greeting("Carol");
    assert!(
        result.contains("Carol"),
        "Greeting did not contain name, value was `{}`", result
    );
}
```

## `should_panic`을 이용한 패닉에 대한 체크

테스트 함수에 `#[should_panic]` 어노테이션을 추가한다.

패닉이 안 일어나면 테스트에 실패한다.

```rust
#[cfg(test)]
mod tests {

    #[test]
    #[should_panic]
    fn greater_than_100() {
        panic!("Panic!!!")
    }
}
```

단순 패닉 발생뿐만 아니라, 패닉 메시지까지 테스트 해야 한다면

`should_panic` 속성에 `expected` 파라미터를 추가한다.

```rust
#[cfg(test)]
mod tests {

    #[test]
    #[should_panic(expected = "Panic!!!")]
    fn greater_than_100() {
        panic!("Panic!!!")
    }
}
```

&nbsp;

# 테스트 실행 방식 제어하기

`cargo test` 바로 뒤에 나열된 옵션은 `cargo test`에 입력되고

`--` 구분자 뒤에 나열된 옵션은 테스트 바이너리에 입력된다.

## 테스트를 병렬 혹은 연속으로 실행하기

테스트 실행 시, 스레드를 이용해서 병렬로 수행된다.

테스트를 빨리 끝내서 피드백을 빨리 얻기 위함이다.

테스트 간 공유하는 환경이나 자원이 있다면, 테스트 실행 시 예상하지 못한 결과가 나올 수 있다.

예를 들어, 같은 이름의 텍스트 파일을 읽는 테스트와 쓰는 테스트가 있다면 각 테스트가 서로 간섭을 일으켜서 테스트가 실패할 수 있다.

테스트를 한 번에 하나씩만 실행하게 해서 문제를 해결할 수 있다.

테스트 바이너리에 `--test-threads` 옵션을 줘서 사용하려는 스레드 개수를 설정할 수 있다.

다음 명령어는 스레드 개수를 1개로 지정해서, 테스트 프로그램이 병렬이 아닌 연속적으로 실행되게 한다.

```
$ cargo test -- --test-threads=1
```

## 함수 결과 보여주기

테스트 실행 시 발생하는 표준 출력은 테스트 실패 시에만 볼 수 있다.

아래 `prints_and_returns_10` 함수는 입력 값을 출력하고 10을 반환한다.

```rust
fn prints_and_returns_10(a: i32) -> i32 {
    println!("I got the value {}", a);
    10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn this_test_will_pass() {
        let value = prints_and_returns_10(4);
        assert_eq!(10, value);
    }

    #[test]
    fn this_test_will_fail() {
        let value = prints_and_returns_10(8);
        assert_eq!(5, value);
    }
}
```

위 테스트 실행 시

테스트에 성공하는 `this_test_will_pass` 함수에서 발생한 표준 출력은 무시되고

테스트에 실패하는 `this_test_will_fail` 함수에서 발생한 표준 출력만 테스트 정리 출력에 나타난다.

`--nocapture` 옵션을 주면, 성공한 테스트에서 나오는 표준 출력도 볼 수 있다.

```
cargo test -- --nocapture
```

## 이름으로 테스트의 일부분만 실행하기

`cargo test`에 실행하고 싶은 테스트 함수 이름을 넘겨서 일부 테스트만 실행할 수 있다.

ex)

```
cargo test test_name
```

실행이 안 된 테스트는 `filtered`로 표시된다.

## 여러 개의 테스트를 실행시키기 위한 필터링

테스트 함수 이름 중 `add_two_and_two`와 `add_three_and_two`와 같이 `add`로 시작하는 테스트를 실행하려면 `cargo test add` 커맨드를 실행한다.

## 테스트 무시하기

실행 시간이 긴 테스트는 `#[ignore]` 어노테이션을 추가해서 무시할 수 있다.

`cargo test -- --ignored` 커맨드를 실행하면 무시된 테스트만 실행할 수 있다.

&nbsp;

# 테스트 조직화

테스트는 크게 단위(unit) 테스트와 통합(integration) 테스트로 나눈다.

단위 테스트는 작고, 한 번에 하나의 모듈만 분리해서 테스트하고, 비공개 인터페이스를 테스트한다.

통합 테스트는 라이브러리 외부에서 공개 인터페이스를 이용해 여러 개의 모듈을 테스트한다.

## 단위 테스트

관례적으로 각 파일마다 테스트 함수를 담고 있는 `tests` 모듈을 만들고, 이 모듈에 `cfg(test)` 어노테이션을 붙인다.

## 테스트 모듈과 `#[cfg(test)]`

`#[cfg(test)]` 어노테이션은 `cargo build` 실행 시에는 무시하고 `cargo test` 실행 시에만 컴파일하고 실행하라고 러스트에게 지시한다.

통합 테스트는 다른 디렉터리에 있어서 `#[cfg(test)]` 어노테이션이 필요없지만

단위 테스트는 해당 코드와 같이 있기에 컴파일 시 결과물 크기를 줄이기 위해 `#[cfg(test)]`가 필요하다.

## 비공개 함수 테스트하기

```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

`internal_adder` 함수는 `pub` 키워드가 없는 비공개 모듈이다.

`tests` 모듈은 같은 파일 내에 있어서 비공개 함수도 일반 함수처럼 가져와서 테스트한다.

## 통합 테스트

라이브러리의 공개 API 부분에 속하는 함수들만 호출해서 테스트한다.

라이브러리의 수 많은 요소들이 함께 올바르게 동작하는지 확인한다.

## _tests_ 디렉토리

프로젝트 디렉토리의 최상위, _src_ 옆에 _tests_ 디렉터리를 만든다.

_tests_ 디렉터리 안에 테스트 파일을 넣어두면 Cargo가 각 파일을 개별 크레이트처럼 컴파일해서 테스트를 실행한다.

_tests/integration_test.rs_ 파일을 만들고 아래 코드를 집어넣는다.

```rust
extern crate adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

tests 디렉터리 내 각 테스트가 모두 개별적인 크레이트이기에, `extern crate adder`으로 라이브러리를 가져왔다.

`#[cfg(test)]` 어노테이션도 필요 없다.

`cargo test`에 `--test test_name` 옵션을 줘서 특정 통합 테스트 함수를 실행 시킬 수 있다.

ex)

```
cargo test --test integration_test
```

## 통합 테스트 내의 서브모듈

통합 테스트 내에서 헬퍼 함수를 공유함과 동시에 테스트 출력에 나오지 않게 하고 싶다

러스트는 `tests/common/mod.rs` 파일로 만든 `common` 모듈을 테스트 파일로 취급하지 않는다.

`tests/common/mod.rs` 만든 후, 아래와 같이 `mod common;`으로 헬퍼 모듈을 불러와 사용할 수 있다.

```rust
extern crate adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```

## 바이너리 크레이트를 위한 통합 테스트

프로젝트가 `src/lib.rs` 파이링 없고 `src/main.rs` 파일만 가지고 있는 바이너리 프로젝트라면,

`tests` 디렉터리 내에 통합 테스트를 만들어도 `src/main.rs`에 정의된 함수를 가져오기 위해 `extern crate`를 사용할 수 없다.

보통 `src/lib.rs`에 중요한 기능들이 정의되어 있고 이는 `extern crate`으로 불러 와서 테스트할 수 있다.

라이브러리 내의 중요 기능들이 잘 작동한다면, `src/main.rs` 내에 있는 소량의 코드들은 테스트할 필요가 없다.

## 마무리

타입 시스템과 소유권 규칙이 몇 가지 버그를 방지해줘도

테스트는 논리 버그를 잡아서 코드가 기대한 동작을 수행한다고 검증할 수 있다.
