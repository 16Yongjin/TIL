# 멀티 스레드 웹 서버 만들기

## 싱글 스레드 기반 웹서버 만들기

TCP를 이용한 바이트통신과 HTTP를 이용한 요청과 응답 주고받기

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

`handle_connection` 함수로 스트림을 인자로 받아 요청을 처리한다.

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

1. `TheadPool`은 채널을 생성하고 채널 송신자를 가진다.
2. `Worker`는 채널의 수신자를 가진다.
3. `execute` 메서드 호출 시 채널의 송신자로 작업을 전송한다.
4. 스레드 안에서 `Worker`에서 받은 작업을 실행한다.

`ThreadPool::new`에서 새 채널을 만든다.

```rust
let (sender, receiver) = mpsc::channel();
```

`sender`는 `ThreadPool` 구조체 필드 내에 저장한다.

`receiver`를 `Arc<Mutex<T>>`로 감싸서 각 `Worker` 구조체에 보내고 여러 스레드가 공유할 수 있게 한다.

`Arc` 덕분에 여러 스레드가 하나의 수신자를 공유할 수 있고,

`Mutex` 덕분에 한번에 하나의 `Worker`가 `receiver`에서 작업을 가져가도록 한다.

```rust
workers.push(Worker::new(id, Arc::clone(&receiver)));
```

## `execute` 메소드 구현

```rust
trait FnBox {
    fn call_box(self: Box<Self>);
}

impl<F: FnOnce()> FnBox for F {
    fn call_box(self: Box<F>) {
        (*self)()
    }
}

type Job = Box<FnBox + Send + 'static>;

pub fn execute<F>(&self, f: F)
        where
            F: FnOnce() + Send + 'static
    {
        let job = Box::new(f);

        self.sender.send(job).unwrap();
    }
```

클로저를 `Box<T>`에 담아서 보낸 뒤, 박스를 벗기고 호출해야 한다.

하지만, 러스트는 `Box<T>` 안의 값의 크기를 알 수 없으므로 박스 안의 값을 옮기는 것을 허용하지 않는다.

`Box<T>` 에 저장된 `Self` 값의 소유권을 다룰 수 있도록 `self: Box<Self>` 를 사용하는 `FnBox` 트레잇을 정의했다.

`call_box` 메서드는 `(*self)()`를 사용하여 클로저를`Box<T>` 밖으로 빼내고 호출한다.

```rust
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || {
            loop {
                let job = receiver.lock().unwrap().recv().unwrap();

                println!("Worker {} got a job; executing.", id);

                job.call_box();
            }
        });

        Worker {
            id,
            thread,
        }
    }
}
```

# 우아한 종료와 정리

스레드가 종료되기 전에 처리하던 요청을 마저 처리하도록 `ThreadPool`에 `Drop` 트레잇을 구현한다.

또한, 스레드에 새로운 요청을 그만 받고 종료하라는 메시지를 받게 한다.

## `ThreadPool` 에 `Drop` 트레잇 구현하기

```rust
struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}


impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

`Worker`를 빌리기만 해서 `thread`의 소유권을 빼낼 수가 없기 때문에

`Worker`의 `thread::JoinHandle<()>`을 `Option`으로 감싼다.

`Option`의 `take` 메서드로 `Some` 내부 값은 빼내고, 옵션을 `None` 값으로 바꿀 수 있다.

## 스레드가 작업 리스닝을 중지하도록 신호하기

```rust
enum Message {
    NewJob(Job),
    Terminate,
}
```

`Worker`가 종료 신호도 받을 수 있도록 `Job` 대신 `Message`를 사용한다.

`Worker::new`의 일부분을 다음과 같이 수정한다.

```rust
let thread = thread::spawn(move ||{
    loop {
        let message = receiver.lock().unwrap().recv().unwrap();

        match message {
            Message::NewJob(job) => {
                println!("Worker {} got a job; executing.", id);

                job.call_box();
            },
            Message::Terminate => {
                println!("Worker {} was told to terminate.", id);

                break;
            },
        }
    }
});
```

`execute` 메서드로 `Message`를 보내면

`Worker`는 `Message`를 받아서 `NewJob`인 경우 작업을 실행하고,

`Terminate`인 경우 작업 신호에 대기하고 있던 루프에서 벗어나서 스레드를 종료한다.

`Drop` 트레잇 구현은 다음과 같이 수정한다.

```rust
impl Drop for ThreadPool {
    fn drop(&mut self) {
        println!("Sending terminate message to all workers.");

        for _ in &mut self.workers {
            self.sender.send(Message::Terminate).unwrap();
        }

        println!("Shutting down all workers.");

        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

워커들을 두 번 순회해서 한번은 `Terminate` 메시지를 보내고, 한번은 스레드에 `join`을 호출한다.

메시지 전송과 `join` 호출을 한번에 하면, 일부 스레드가 일하느라 메시지를 못 받는 경우, 다른 스레드가 이 메시지를 채간다.

메시지를 못받은 스레드는 영원히 종료되지 않고 교착상태에 빠진다.
