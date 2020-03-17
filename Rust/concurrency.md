# 러스트의 동시성

## 동시성 프로그래밍

프로그램의 서로 다른 부분이 독립적으로 실행되는 것

## 병렬 프로그래밍

프로그램의 서로 다른 부분이 동시에 실행되는 것

## 동시성을 다룰 때의 트레이드 오프

Erlang은 메시지 패싱으로 동시성을 다루는 방법은 우아하지만, 스레드 내부 동작 방식을 이해하기 힘들다고 한다.

고수준 언어들은 추상화를 위해 저수준의 제어권을 포기한다.

러스트는 저수준 언어로서, 추상화 정도는 낮지만 뛰어난 성능을 낼 수 있다.

&nbsp;

# 스레드로 코드를 동시에 실행하기

## 스레드

프로그램 내에 동시에 실행되는 독립적인 부분을 실행하는 기능

한편, 프로그램을 스레드 여러 개로 나눠서 처리하는 것은 성능이 좋지만, 프로그램이 복잡해지는 문제가 있다.

## 스레드 사용 시 발생할 수 있는 문제

- 경쟁 조건 (Race Condition)
- 데드락 (Deadlock)
- 재현하기 힘들고 수정하기 힘든 버그

## 스레드 구현 방식

- 1:1 구조 : 운영체제의 스레드 하나가 언어의 스레드 하나에 대응됨
- 그린 스레드 M:N 구조 : 운영체제의 스레드와 언어 스레드의 개수가 다름

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

# 상태를 공유하는 동시성

메시지 패싱을 사용 시, 채널로 값을 보내는 순간 그 값을 사용할 수 없다.

공유 메모리 동시성 사용 시, 복수 소유권처럼 스레드들이 동시에 동일한 메모리에 접근할 수 있다.

## 뮤텍스를 사용해 한번에 하나의 스레드만 접근 허용하기

뮤텍스는 **Mut**ual **Ex**clusion, 상호 배제의 줄임말이다.

뮤텍스 내부 데이터 접근을 위해서 스레드는 락 요청으로 접근 신호를 보낸다.

## 뮤텍스 사용 시 주의점

- 데이터 사용 전에 락을 얻어야 한다.
- 데이터 사용 완료 후, 반드시 언락해야 한다.

러스트의 소유권 규칙 덕분에 락을 잘못 얻거나 언락을 못하는 일이 생기지 않는다.

## `Mutex<T>`의 API

아래 코드는 단일 스레드에서 뮤텍스를 사용한다.

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

`new` 연관함수로 뮤텍스를 생성하고 데이터에 접근하기 위해 `lock` 메서드로 락을 얻는다.

락을 얻는 동안 스레드는 블록된다.

락을 얻은 후, 반환되는 값을 가변 참조자처럼 다룰 수 있다.

`lock`은 `MutexGuard` 스마트 포인터를 반환한다.

이 스마트 포인터 내 구현된 `Deref`가 스코프를 벗어나면 자동으로 락을 해제한다.

덕분에 언락하는 것을 누락하고 다른 스레드가 뮤텍스를 사용하는 일을 신경쓰지 않아도 된다.

## 스레드 간 `Mutex<T>` 공유하기

아래 코드는 10개의 스레드가 각각 뮤텍스의 값을 1씩 증가시킨다.

```rust
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

스레드에 카운터를 이동시키고 `lock`을 호출해 락을 얻고, 뮤텍스 내부 값을 1 증가시키는 클로저를 `thread::spawn`로 생성한 스레드에 넘긴다.

`join()`을 호출해서 모든 스레드가 종료되길 기다린다.

하지만 위 코드는 클로저 내부로 이동된 `counter`를 사용하려 했다면서 컴파일 에러가 발생한다.

## 스레드 간 복수 소유권

복수의 소유권을 사용하면 각 스레드가 뮤텍스에 접근할 수 있게 할 수 있다.

`Rc<T>`는 복수를 소유권을 가능하게 하지만, 스레드에서 사용하기에 적합하지 않다.

참조 카운트가 증감이 잘못돼서 메모리 누수가 발생하거나 값 사용을 마치기 전에 값이 버려지는 오류가 발생할 수 있기 때문이다.

## `Arc<T>`로 아토믹(atomic) 참조 카운팅하기

`Mutex<T>`를 `Arc<T>`로 감싸면 여러 스레드 사이에서 뮤텍스의 소유권을 공유할 수 있다.

```rust
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

`Rc<T>`와 `Arc<T>`는 비슷한 API를 가지고 있지만, 내부적으로 참조 카운팅 방법이 다르다.

어떻게 다른지는 `std::sync::atomic` 문서에서 찾아봐야 알 수 있다.

## `RefCell<T>` - `Rc<T>`와 `Mutex<T>` - `Arc<T>` 간의 유사성

`Rc<T>`의 내용을 변경하기 위해 `RefCell<T>`을 사용한 것과 같은 방식으로, `Arc<T>` 내부의 값을 변경하기 위해 `Mutex<T>`를 이용한다.

# `Sync`와 `Send` 트레잇을 이용한 확장 가능한 동시성

러스트 언어 자체는 소수의 동시성 기능만 제공한다.

`std::marker` 트레잇인 `Sync`와 `Send`은 언어에 내장된 동시성 개념이다.

## `Send`를 사용하여 스레드 사이에 소유권 이전을 허용하기

`Send` 마커 트레잇은 `Send`가 구현된 타입의 소유권이 스레드 사이에서 이전될 수 있음을 나타낸다.

다만, `Rc<T>`는 `Send` 트레잇이 구현되어있지 않다.

`Rc<T>`는 여러 스레드 사이에서 참조 카운트 값이 어떻게 갱신될지 모르기 때문에 단일 스레드에서만 사용할 수 있다.

## `Sync`를 사용하여 여러 스레드의 접근을 허용하기

`Sync` 마커 트레잇은 `Sync`가 구현된 타입이 여러 스레드로부터 안전하게 참조 가능함을 나타낸다.

`&T`가 `Send`라면 `T`는 `Sync`다.

기초 타입들은 다 `Sync`다.

## `Send`와 `Sync`를 손수 구현하지 말자

`Send`와 `Sync` 트레잇들로 타입을 구성하면 자동적으로 그 타입은 `Send`이며 `Sync`이게 된다.

마크 트레잇의 특성상 구현할 메서드도 없다.

멀티 스레드를 사용하기 위해 가장 최신 크레이트를 가져다 쓰는 게 낫다.

# 정리

러스트 표준 라이브러리는 메세지 패싱을 위해 체널을 제공한다.

동시적 맥락에서 사용해도 안전한 `Mutex<T>`와 `Arc<T>` 같은 스마트 포인터 타입들도 제공한다.

타입 시스템과 빌림 검사기 덕분에 코드가 데이터 레이스나 모든 참조자가 유효함을 보장할 수 있다.

코드가 컴파일된다면, 스레드 안전하다.
