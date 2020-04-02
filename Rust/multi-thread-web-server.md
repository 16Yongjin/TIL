# 멀티 스레드 웹 서버 만들기

## 싱글 스레드 기반 웹서버 만들기

TCP를 이용한 바이트통신과 HTTP를 이용한 요청과 응답 주고 받기

```rust
use std::fs::File;
use std::io::prelude::*;
use std::net::TcpListener;
use std::net::TcpStream;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();
        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];
    stream.read(&mut buffer).unwrap();

    let get = b"GET / HTTP/1.1\r\n";

    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND\r\n\r\n", "404.html")
    };

    let mut file = File::open(filename).unwrap();
    let mut contents = String::new();
    file.read_to_string(&mut contents).unwrap();

    let response = format!("{}{}", status_line, contents);

    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

## TCP 연결 처리

표준 라이브러리 `std::net` 모듈을 사용한다.

`TcpListener::bind`로 TCP 연결을 수신한다.

`TcpListener`의 `incoming` 메서드로 커넥션 스트림을 받는다.

## 요청 데이터 읽기

`handle_connection`함수로 스트림을 인자로 받아 요청을 처리한다.

`TcpStream`은 내부에서 반환되는 데이터를 추적하므로 `stream` 매개변수를 가변으로 받는다.

`std::io::prelude`에서 스트림 읽고 쓰는 특성을 가져온다.

버퍼를 생성하고 `stream.read`에 전달해서 요청 바이트를 읽는다.

## 응답 작성하기

HTTP 응답은 다음과 같은 양식을 갖는다.

```
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

`response`의 `as_bytes`를 호출해서 문자열 데이터를 바이트 배열(`&[u8]`)로 변환한다.

이 바이트 배열을 `stream`의 `write` 메서드에 넣고 `flush` 메서드로 버퍼를 비워서 커넥션으로 전송한다.

## 요청을 확인하고 선택적으로 응답하기

`b"GET / HTTP/1.1\r\n";`로 `/` 경로로 요청이 왔는지 확인한다.

버퍼에서 원시 바이트를 읽어들였으므로 비교도 바이트로 한다.

# 싱글 스레드 서버를 멀티 스레드 서버로 바꾸기

싱글 스레드 기반 서버는 한 요청이 끝나야 다음 요청을 처리할 수 있어서 많은 요청이 들어오면 처리가 늦어진다.

## 스레드 풀로 처리량 높이기

대기나 준비 상태에 있는 스레드들을 모아둔 게 스레드 풀이다.

요청은 큐로 저장되고, 스레드 하나당 큐에서 꺼낸 하나의 요청을 처리를 한다.

처리가 완료되면 스레드는 풀로 다시 돌아와 대기 상태로 돌아간다.

다음 코드는 모든 요청에 스레드 하나씩 생성한다.

```rust
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        thread::spawn(|| {
            handle_connection(stream);
        });
    }
}
```

스레드가 무한히 생성될 수 있으므로, 시스템 부하를 막을 방법이 필요하다.

## 스레드 수를 제어하기 위한 인터페이스

`thread::spawn` 대신 `ThreadPool`이라는 인터페이스를 만들려고 한다.

```rust
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }
}
```

## 컴파일러 주도 개발로 `ThreadPool` 라이브러리 만들기

`cargo check` 명령에서 나오는 컴파일러 에러로 개발을 진행할 수 있다.

`cargo check` 시 `ThreadPool` 모듈이 없다고 에러가 난다.

**src/lib.rs** 에 다음 코드를 작성한다.

```rust
pub struct ThreadPool;
```

**main.rs** 파일은 **src/bin/main.rs**로 옮기고 다음 가져오기 코드를 추가한다.

```rust
extern crate hello;
use hello::ThreadPool;
```

`cargo check` 시 `new` 함수가 없다고 에러가 난다.

`new` 함수를 정의하고, 스레드 개수는 항상 양수이므로 인자로 `usize` 타입을 받는다.

```rust
impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        ThreadPool
    }
}
```

`cargo check` 시 `execute` 함수가 없다고 에러가 난다.

표준 라이브러리의 `thread::spawn` 구현체를 참고한다.

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T + Send + 'static,
        T: Send + 'static
```

요청 처리는 한번만 하므로 `FnOnce`를 클로저의 트레잇으로 사용됐다.

스레드 간 클로저가 전달되므로 `Send` 트레잇을, 스레드의 제거 시점을 알 수 없으므로 `'static` 라이프사이클을 사용됐다.

`execute`는 다음과 같이 정의한다.

```rust
pub fn execute<F>(&self, f: F)
        where
            F: FnOnce() + Send + 'static
    {

    }
```

## `Worker` 구조체 사용

`thread:::spawn`는 클로저를 받아 생성되자마자 실행된다.

스레드 실행 타이밍을 조절하는 `Worker` 구조체를 만든다.

```rust
struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}
```

`Worker` 구조체는 다른 워커와 구별하기 위한 `id`와 빈 클로저를 가진 스레드를 담는 `thread` 필드로 구성되어 있다.

`ThreadPool` 구조체는 `Worker` 벡터를 가진다.

```rust
pub struct ThreadPool {
    workers: Vec<Worker>,
}
```

## 채널을 통해 스레드에 요청 보내기

## `execute` 메소드 구현
