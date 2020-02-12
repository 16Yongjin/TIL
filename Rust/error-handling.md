# 러스트의 에러처리

복구 가능한 에러를 위한 `Result<T, E>`와 타입과 복구 불가능한 에러를 위한 `panic!` 매크로를 사용한 에러처리

## 복구 가능한 에러

재시도 가능

`Result<T, E>`로 다룸

## 복구 불가능한 에러

인덱스 에러 같은 버그

`panic!`으로 다룸

## `panic!`을 사용하는 복구 불가능한 에러

`panic!` 매크로가 실행되면 실패를 메시지 출력하고, 스택을 되감고 청소 후 종료된다.

## 스택 되감지 않기

*Cargo.toml*에 아래를 추가하면 `panic!` 발생 시 데이터 제거 없이 프로그램이 끝난다.

프로그램이 사용하던 메모리는 운영체제가 청소한다.

```toml
[profile.release]
panic = 'abort'
```

## `panic!` 백트레이스 사용하기

백트레이스란 어떤 지점에 도달하기까지 호출해온 모든 함수의 리스트를 말한다.

아래 코드는 벡터에 유효하지 않은 인덱스를 접근해서 `panic!`이 발생한다.

```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

에러 메시지는 `libcollections/vec.rs`에서 `panic!`이 발생했다고 알려준다.

```
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27 secs
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is
100', /stable-dist-rustc/build/src/libcollections/vec.rs:1362
note: Run with `RUST_BACKTRACE=1` for a backtrace.
error: Process didn't exit successfully: `target/debug/panic` (exit code: 101)
```

`RUST_BACKTRACE` 환경 변수를 설정하고 코드를 실행하면 아래와 같은 출력이 나온다.

```
$ RUST_BACKTRACE=1 cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 100', /stable-dist-rustc/build/src/libcollections/vec.rs:1392
stack backtrace:

..생략..

  11:     0x560ed90e727a - panic::main::h5d6b77c20526bc35
                        at /home/you/projects/panic/src/main.rs:4

..생략..
```

백트레이스의 11번 라인에서 문제가 일어난 부분을 알아낼 수 있고, 그 부분을 수정 코드를 고칠 수 있다.

&nbsp;

## `Result`를 사용하는 복구 가능한 에러

`Result` 열거형은 다음과 같이 `Ok`와 `Err`라는 두 개의 변형을 갖고 있다.

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

아래 코드의 `f`는 타입이 `File::open`의 반환 타입인 `Result<std::fs::File, std::io::Error>` 이다.

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
}
```

`File::open` 호출이 성공했을 때 파일 핸들을, 실패했을 때 에러 정보를 반환한다는 것을 알 수 있다.

## `match`로 `Result` 타입 처리하기

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("There was a problem opening the file: {:?}", error)
        },
    };
}
```

결과가 `Ok`일 시에는 `file`을 반환하고 `Err`인 경우 `panic!` 매크로를 실행한다.

`match` 사용 시 `Result`의 변형 타입들은 Prelude로 가져와져서 `Ok`와 `Err` 앞에 `Result::`를 붙이지 않아도 된다.

## 서로 다른 에러에 대해 매칭하기

`File::open`이 파일이 없어서 실패한 경우, 새로운 파일을 만들어서 핸들을 반환한다.

권한 문제 등 다른 문제로 실패한 경우는 `panic!`을 일으킨다.

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(ref error) if error.kind() == ErrorKind::NotFound => {
            match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => {
                    panic!(
                        "Tried to create file but there was a problem: {:?}",
                        e
                    )
                },
            }
        },
        Err(error) => {
            panic!(
                "There was a problem opening the file: {:?}",
                error
            )
        },
    };
}
```

`Err` variant내의 `io::Error` 구조체 값이 들어있다.

이 구조체에 `kind` 메서드는 `io::ErrorKind` 열거형을 반환한다.

`io::ErrorKind` 열거형을 통해 `io` 연산 에러들을 구분할 수 있다.

위 코드의 `ErrorKind::NotFound`는 파일이 아직 존재하지 않음을 나타낸다.

조건문 `if error.kind() == ErrorKind::NotFound`는 `match`문을 좀더 정제해주는 **매치 가드**이다.

## `unwrap`과 `expect`로 에러를 짧게 처리하기

`match`는 잘 동작하지만 너무 장황해서 코드를 파악하기 힘들 수 있다.

## `unwrap` 사용하기

`unwrap`은 `Ok` 시 `Ok` 내부의 값을 반환하고, `Err`라면 `panic!`을 일으킨다.

## `expect` 사용하기

`expect`는 `unwrap`과 비슷한데, `panic!`의 에러 메시지를 선택할 수 있게 한다.

`expect`으로 에러 메시지를 잘 써서 패닉의 원인을 추적하기 쉽게 할 수 있다.

`expect`는 아래와 같이 사용한다.

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

## 에러 전파하기

에러를 직접 처리하는 대신 반환해서 에러 처리를 사용자에게 맡긴다.

아래는 파일에서 사용자 이름을 읽는 함수이다.

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
  let f = File::open("hello.txt");

  let mut f = match f {
    Ok(file) => file,
    Err(e) => return Err(e), // 에러를 값처럼 반환!!
  };

  let mut s = String::new();

  match f.read_to_string(&mut s) {
    Ok(_) => Ok(s),
    Err(e) => Err(e),
  } // 함수의 마지막이라 return을 명시하지 않음
}
```

## `?`로 에러 전파 짧게 하기

아래 코드는 위 코드의 `read_username_from_file` 함수를 물음표(`?`) 연산자로 다시 구현한다.

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io:Error> {
  let mut f = File::open("hello.txt")?;
  let mut s = String::new();
  f.read_to_string(&mut s)?;
  Ok(s)
}
```

`Result` 값 뒤의 `?`는 `Result`가 `Ok`라면 값이 얻어지고 `Err`라면 `return` 키워드로 에러 값을 함수 사용자에게 넘긴다.

에러 타입이 `From` 트레잇의 `from` 함수를 구현하면 물음표 연산자를 사용할 수 있다.

## `?`로 메서드 체이닝하기

`File::open("hello.txt")?`의 결과에 `read_to_string`을 호출할 수 있다.

위의 코드와 동일하면서 작성하기 더 편하다.

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?; // WOW

    Ok(s)
}`
```

## `?`는 `Result`를 반환하는 함수에서만 사용가능

`?`는 내부적으로 `return Err(e)`를 실행하기 때문에 반환 타입은 이와 호환 가능한 `Result`이어야 한다.

&nbsp;

## `panic!` 호출 VS `Result` 반환

함수가 실패할 지 모르겠으면 일단 `Result` 반환

## 예제, 프로토타입, 테스트에서는 `panic!`

**예제** 속 강건한 에러 처리 코드는 예제를 불필요하게 복잡하게 만든다.

`unwrap`을 써서 에러 처리가 필요한 부분을 보여줄 수 있음

**프로토타입**을 만들 때는 에러 처리 방침이 결정되기 전에 일단 `unwrap`과 `expect` 써서 프로토타이핑을 하면 편하다.

**테스트** 내에서 실패 발생 시 전체 테스트를 실패시켜서 어떻게 테스트가 실패했는지 보여줄 수 있음

## 컴파일러보다 개발자가 더 많은 정보를 가지고 있을 때도 `panic!`

아래 코드는 하드코딩된 IP주소를 파싱해서 `IpAddr` 인스턴스를 만든다.

```rust
use std::net::IpAddr;

let home = "127.0.0.1".parse::<IpAddr>().unwrap();
```

`127.0.0.1`가 유효한 IP라는 것을 컴파일러가 확인해 줄 수 없기 때문에 `upwrap`으로 실패의 책임을 개발자에게 돌린다.

IP 주소가 하드코딩된 게 아니라 사용자에게 입력을 받아야 하는 거라면 당연히 `Result`를 강건하게 처리해야 한다.

## 함수의 계약조건을 위반 했을 때도 `panic!`

호출자가 요구사항을 만족했을 때만 함수의 행동이 보장된다.

이 계약을 위반 했다면 패닉에 빠지게 해서 호출자가 코드를 고치도록 해야 한다.

## 그럼 `Result`는 언제?

일단 `Result`를 기본적으로 사용하고 위에 나온 경우에 `panic!`을 사용한다.

추가로, 어쩔 수 없이 나쁜 상태에 도달하는 경우(기형적 입력이 들어온 파서나 속도제한 상태의 HTTP)는 `Result`를 반환해서 실패가 예상가능한 것임을 알려준다.

## 유효성을 보장하는 커스텀 타입

1에서 100 사이의 숫자를 입력받아야 할 때, 아래의 `if`문을 유효성 검사를 할 수 있다.

```rust
if guess < 1 || guess > 100 {
    println!("The secret number will be between 1 and 100.");
    continue;
}
```

하지만, 코드의 모든 곳에서 위와 같이 검사를 반복하는 것 지루하다.

커스텀 타입을 만들어서, 인스턴스 생성 시 유효성을 확인하도록 할 수 있다.

아래 코드는 1에서 100 사이의 값만 받을 수 있는 `Guess` 타입이다.

```rust
pub struct Guess {
    value: u32,
}

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }

    pub fn value(&self) -> u32 {
        self.value
    }
}
```

`new` 연관 함수로 유효성을 테스트한 뒤 `Guess`를 반환한다.

입력 값이 테스트에 통과하지 못하면 `panic!`을 호출해서 프로그래머에게 버그를 고칠 것을 알려준다.

`Guess::new`가 패닉을 일으킬 수도 있는 조건은 API 문서에 명시해야 한다.

테스트를 통과한 `value` 항목이 다른 곳에서 비정상값으로 변경되는 일을 막기 위해 `value()` 게터 메서드를 만들어서 `value` 항목을 비공개로 만든다.
