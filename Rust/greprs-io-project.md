# IO 프로젝트, 커맨드 라인 프로그램 만들기 노트

## 인자 읽어들이기

`std::env::args` 함수를 호출한다.

`collect()` 함수로 반복자를 벡터로 만든다.

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    println!("{:?}", args);
}
```

인자 벡터의 첫 번째 값은 바이너리 이름이 들어있다.

## 파일 읽기

`std::fs::File`와 `std::io::prelude::*` 라이브러리가 필요하다.

`File::open` 함수에 파일이름을 전달해서 파일 핸들을 얻는다.

파일 핸들에 `read_to_string` 메서드에 가변 참조 문자열을 넘겨서 파일의 내용을 읽어들인다.

## 단일 책임 원칙

`main` 함수에 많은 기능을 넣으면 안 된다.

프로그램의 로직은 `lib.rs`에 모두 작성한다.

### `main` 함수에 남아도 되는 핵심 기능

- 인자 값 파싱 로직 호출
- 환경 설정
- `lib.rs`의 `run` 함수 호출
- `run` 함수의 에러 처리

## 강박적인 기본타입 사용 피하기

아무런 의미도 담지 못 하는 튜플보단 구조체를 만들어서 데이터에 확실한 의미를 부여하는 것이 유지보수하기 쉽고 코드의 목적도 명확해진다.

아래 코드 보단,

```rust
let (query, filename) = parse_config(&args);
```

아래처럼 설정값을 담은 구조체를 사용하는 것이 좋다.

`new` 메서드로 구조체 생성과 입력 인자 검증을 한다.

```rust
struct Config {
    query: String,
    filename: String,
}

impl Config {
    fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let filename = args[2].clone();

        Ok(Config { query, filename })
    }
}
```

위 코드에서 `let query = args[1].clone();`와 같이 문자열의 `.clone` 메서드를 사용한다.

`args`와 그 안에 들어 있는 값은 참조자로 빌린 값이라서 그냥 가져다 반환하면 안 된다.

값을 새로 복사해서 문자열의 소유권을 가진 다음에 반환해야 한다.

## 테스트 주도 개발 기법의 단계

1. 실패할 테스트 작성 후, 의도대로 실패하는지 실행해본다.
2. 테스트를 통과할 수 있는 코드를 작성한다.
3. 코드를 리팩터링을 하고 여전히 테스트에 통과하는지 확인한다.
4. 1단계부터 반복.

## 환경변수 가져오기

`std::env::var` 함수를 사용해서 환경변수를 가져온다.

아래 코드는 `CASE_INSENSITIVE` 환경 변수 존재 여부를 확인한다.

```rust
use std::env;

fn main() {
  let case_sensitive = env::var("CASE_INSENSITIVE").is_err();
}
```

## 표준에러 메시지 출력하기

`println!`으로 출력한 에러 메시지는 표준출력으로 전달된다.

표준에러를 사용하면 프로그램에서 나온 에러 매시지가 화면에 표시만 되고, 다른 프로그램에 스트림되는 오류를 막을 수 있다.

`eprintln!` 매크로로 쉽게 에러출력을 할 수 있다.
