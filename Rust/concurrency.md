# 러스트의 동시성

## 동시성 프로그래밍

프로그램의 서로 다른 부분이 독립적으로 실행되는 것

## 병렬 프로그래밍

프로그램의 서로 다른 부분이 동시에 실행되는 것

## 동시성을 다룰 때의 트레이드 오프

Erlang은 메시지 패싱으로 동시성을 다루는 방법은 우아하지만 스레드 내부 동작 방식을 이해하기 힘들다고 한다.

고수준 언어들은 추상화를 위해 저수준의 제어권을 포기한다.

러스트는 저수준 언어로서, 추상화 정도는 낮지만 뛰어난 성능을 낼 수 있다.

&nbsp;

# 스레드로 코드를 동시에 실행하기

## 스레드

프로그램 내에 동시에 실행되는 독립적인 부분을 실행하는 기능

한편, 프로그램을 스레드 여러 개로 나눠서 처리하는 것은 성능이 좋지만 프로그램이 복잡해지는 문제가 있다.

## 스레드 사용 시 발생할 수 있는 문제

- 경쟁 조건 (Race Condition)
- 데드락 (Deadlock)
- 재현하기 힘들고 수정하기 힘든 버그

## 스레드 구현 방식

- 1:1 구조 : 운영체제의 스레드 하나가 언어의 스레드 하나에 대응됨
- 그린 스레드 M:N 구조 : 운영체제의 스레드와 언어의 스레드의 개수가 다름

그린 스레드는 스레드를 관리하기 위한 런타임 바이너리가 크기 때문에, 러스트 표준 라이브러리는 1:1 스레드 구현만 제공한다.

## `spawn`으로 스레드 생성하기

`thread::spawn` 함수에 스레드에서 실행할 코드를 클로저에 담아서 넘긴다.

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the 생성된 스레드!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the 메인 스레드!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

위에 코드를 실행하면 스레드에 실행되던 코드가 다음과 같이 중간에 끊긴다.

```
hi number 1 from the 메인 스레드!
hi number 1 from the 생성된 스레드!
hi number 2 from the 메인 스레드!
hi number 2 from the 생성된 스레드!
hi number 3 from the 메인 스레드!
hi number 3 from the 생성된 스레드!
hi number 4 from the 메인 스레드!
hi number 4 from the 생성된 스레드!
hi number 5 from the 생성된 스레드!
```

## `join` 핸들을 사용하여 모든 스레드들이 끝날 때까지 기다리기

메인 스레드가 종료되면 생성된 스레드도 작동을 멈춘다.

`thread::spawn` 함수는 `JoinHandle`을 반환한다.

`JoinHandle`의 `join` 메서드를 호출하면 생성된 스레드가 끝날 때까지 기다린다.

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the 생성된 스레드!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the 메인 스레드!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

메인 스레드는 생성된 스레드가 종료될 때까지 기다린다.

```rust
hi number 1 from the 메인 스레드!
hi number 2 from the 메인 스레드!
hi number 1 from the 생성된 스레드!
hi number 3 from the 메인 스레드!
hi number 2 from the 생성된 스레드!
hi number 4 from the 메인 스레드!
hi number 3 from the 생성된 스레드!
hi number 4 from the 생성된 스레드!
hi number 5 from the 생성된 스레드!
hi number 6 from the 생성된 스레드!
hi number 7 from the 생성된 스레드!
hi number 8 from the 생성된 스레드!
hi number 9 from the 생성된 스레드!
```

`handle.join().unwrap();`을 `for i in 1..5 {` 위에 놓으면, 생성된 스레드에서 1에서 10까지 출력이 끝난 후 나머지 1에서 5가 출력된다.

`join`의 호출 위치에 따라 스레드 실행 방식이 달라진다.

## 스레드에 `move` 클로저 사용하기

`move` 클로저로 스레드 사이에 데이터를 주고 받을 수 있다.

`move` 키워드로 새로운 스레드로 데이터의 소유권을 넘겨야, 다른 스레드에서 해당 데이터가 변경되거나 제거되는 일을 방지할 수 있다.

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

&nbsp;

# 메시지 패싱으로 스레드 간에 데이터 전송하기

메시지 패싱은 안전한 동시성을 보장하는 방법 중 하나이다.

## Go의 동시성 슬로건

메모리를 공유하는 것으로 통신하지 마세요. 대신, 통신해서 메모리를 공유하세요;

## 채널

러스트는 메시지 패싱을 통한 동시성을 위해 *채널*을 제공한다.

채널은 메시지를 보내는 송신자와 메시지를 받는 수신자로 나눠져 있다.

송신자나 수신지가 버려지면 채널도 닫힌다.

## 채널을 생성해서 송, 수신자 만들기

`mpsc::channel` 함수로 채널을 새로 만든다.

`mpsc`는 복수 생성자, 단일 소비자인 multiple producer, single consumer의 줄임말이다.

여러 하천이 큰 강 하나로 합쳐지는 모양과 같다.

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
}
```

`tx`는 송신자, `rx`는 수신자를 나태낸다.

## 메시지 보내기

```rust
let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let val = String::from("hi");
    tx.send(val).unwrap();
});
```

위 코드는 `move` 클로저로 `tx`를 스레드로 이동 시킨다.

데이터를 보내는 `rx`의 `send` 메서드는 `Result<T, E>` 타입을 반환한다.

수신자가 제거돼서 데이터를 보낼 곳이 없으면 에러를 반환한다.

## 메시지 받기

```rust
let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let val = String::from("hi");
    tx.send(val).unwrap();
});

let received = rx.recv().unwrap();
println!("Got: {}", received);
```

메시지 수신은 `recv`와 `try_recv` 메서드를 사용한다.

`recv`는 메시지를 받을 때까지 스레드를 블록하고 결과를 `Result<T, E>`로 반환한다.

송신자가 닫히면 에러를 반환한다.

`try_recv`는 블록하지 않고 즉시 `Result<T, E>`를 반환한다.

전달된 메시지가 있으면 `Ok`를, 없으면 `Err`를 반환한다.

`try_recv`를 루프에 넣고 메시지를 기다리면서 다른 작업을 할 수 있다.

## 채널과 소유권 전달

송신자로 데이터를 보내면 소유권도 같이 보낸다.

`send` 메서드 이후 `val`을 사용하는 아래 코드는 `use of moved value` 에러가 나면서 컴파일에 실패한다.

```rust
let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let val = String::from("hi");
    tx.send(val).unwrap();
    println!("val is {}", val);
});

let received = rx.recv().unwrap();
println!("Got: {}", received);
```

## 여러 개의 값을 보내고 수신자가 기다리는지 보기

아래 코드는 여러 개의 데이터를 전송하고 1초 씩 멈춘다.

```rust
let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let vals = vec![
        String::from("hi"),
        String::from("from"),
        String::from("the"),
        String::from("thread"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

for received in rx {
    println!("Got: {}", received);
}

```

메시지 수신 시 `recv` 메서드를 호출하지 않고 `rx`를 반복자처럼 사용하고 있다.

값이 수신될 때마다 `for` 내부가 실행되고, 채널이 닫히면 반복이 종료된다.

## 송신자를 복제하여 생성자 여러 개 만들기

송신자를 복제하면 동일한 수신자로 데이터를 보내는 여러 개의 송신자를 만들 수 있다.

`mpsc::Sender::clone` 메서드로 송신자를 복제하고 같은 수신자에 데이터를 보내는 두 스레드를 만든다.

```rust
let (tx, rx) = mpsc::channel();

let tx1 = mpsc::Sender::clone(&tx);
thread::spawn(move || {
    let vals = vec![
        String::from("hi"),
        String::from("from"),
        String::from("the"),
        String::from("thread"),
    ];

    for val in vals {
        tx1.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

thread::spawn(move || {
    let vals = vec![
        String::from("more"),
        String::from("messages"),
        String::from("for"),
        String::from("you"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

for received in rx {
    println!("Got: {}", received);
}
```

코드 실행 시, 다음과 같이 각 송신자에서 보낸 데이터가 번갈아 가면서 출력된다.

```
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```
