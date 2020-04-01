# 멀티 스레드 웹 서버 만들기

## 싱글스레드 기반 웹서버 만들기

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
