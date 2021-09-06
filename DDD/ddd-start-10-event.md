# [DDD START!] 10장 이벤트

## 시스템 간 강결합의 문제

- 바운디드 컨텍스트 간 강결합은 많은 문제를 일으킨다.
- 주문 취소 기능에서 주문 도메인이 환불 기능까지 수행하는 경우
  - 환불은 외부 결제 시스템을 사용한다.
    - 외부 시스템 오작동 시 트랜잭션 처리 방식이 애매하다.
    - 외부 시스템의 응답 속도가 느려지면 내부 시스템의 속도에도 영향을 미친다.
  - 주문 로직과 결제 로직이 섞인다.
    - 환불 기능 변경 시 주문 도메인도 변경해야 한다.
  - 새로운 기능을 추가하기 힘들어 진다.
    - 주문 취소 후 고객에 통지해야 하는 요구사항 발생 시
    - 통지하는 외부 서비스에 트랜잭션 처리 방식과 성능이 영향받는다.
- 이벤트를 사용하면 두 시스템 간 결합을 낮출 수 있다.
- 추가로 비동기를 사용해 시스템의 성능도 높일 수 있다.

## 이벤트란

- 웹 브라우저에서 버튼이 눌리면 `click` 이벤트가 발생하고 스크롤을 하면 `scroll` 이벤트가 발생한다.
- 이벤트 발생은 상태 변경을 의미한다.
- 이벤트 발생 시 그 이벤트에 반응해서 원하는 기능을 실행해야 한다.

## 이벤트의 구성요소

![event components](https://user-images.githubusercontent.com/22253556/132224924-cb6fe245-7f3a-4d08-8f70-1e41fe320a05.png)

- 도메인 모델에 이벤트 도입을 위해서 다음 4개의 구성요소가 필요하다.
  1. 이벤트
  2. 이벤트 생성 주체
  3. 이벤트 디스패처
  4. 이벤트 핸들러

### 이벤트 생성 주체

- 도메인 객체(ex, 엔티티, 밸류, 도메인 서비스)에서 로직 실행 후 상태 변경 시 이벤트를 발생시킨다.

### 이벤트 핸들러

- 이벤트에 반응한다.
- 이벤트에 담긴 데이터를 이용해서 원하는 기능을 실행한다.
- ex) `주문 취소됨 이벤트`는 주문자에게 SMS로 주문 취소를 통지할 수 있다.

### 이벤트 디스패처

- 이벤트 생성 주체와 이벤트 핸들러를 연결한다.
- 이벤트를 받아 해당 이벤트를 처리할 수 있는 핸들러에 전파한다.

## 이벤트 구성요소

1. 이벤트 종류: 클래스 이름으로 이벤트 타입을 표현한다.
   - 과제 시제를 사용한다. ex) `OrderChangedEvent`
2. 이벤트 발생 시간
3. 추가 데이터: 주문 번호 등

## 이벤트 처리

1. 이벤트 클래스를 생성해서 이벤트를 생성한다.
   - 이벤트에 필요한 최소한의 데이터만 담는다.
2. `Events.raise()` 같은 디스패처에 이벤트를 전달한다.
3. 이벤트 핸들러는 이벤트에 담긴 데이터를 읽어서 기능을 수행한다.
   - 필요한 경우 DB에서 데이터를 가져온다.

## 이벤트의 용도

### 1. 트리거

- 도메인의 상태를 변경한 후 후처리를 할 때 사용한다.
- ex) 주문 취소 후 환불 처리 기능 트리거하기
- ex) 영화 애매 후 SMS 발송하기

### 2. 시스템 간 동기화

- ex) 배송지 변경 후 외부 배송 서비스에 변경된 배송지 정보 전송하기

## 이벤트의 장점

- 서로 다른 도메인 로직이 섞이는 것을 방지한다.
- 기능 확장이 용이하다.
  - 기능을 구현하고 관련 이벤트를 처리하는 핸들러를 디스패처에 등록하면 된다.
  - 이벤트를 발생시키는 로직을 수정할 필요가 없다.

## 이벤트, 핸들러, 디스패처 구현

### 이벤트 클래스

- 이벤트 이름을 과제 시제로 사용해야 한다.
  - ex) `OrderCanceledEvent` or `OrderCanceled`
- 이벤트를 처리하는데 필요한 최소한의 데이터를 포함해야 한다.
- 이벤트의 공통 프로퍼티가 존재한다면 상위 클래스를 만들 수 있다.

  - 모든 이벤트가 발생 시간을 갖도록 할 수 있다.

    ```ts
    class Event {
      #timestamp: number

      constructor() {
        this.#timestamp = Date.now()
      }

      get timestamp() {
        return this.#timestamp
      }
    }
    ```

  - 모든 이벤트 클래스는 `Event` 클래스를 상속받아 구현한다.

### 이벤트 핸들러

#### 자바 버전

```java
package com.myshop.common.event;

import net.jodah.typetools.TypeResolver;

public interface EventHandler<T> {
  void handle(T event);

  default boolean canHandle(Object event) {
    Class<?>[] typeArgs = TypeResolver.resolveRawArguments(EventHandler.class, this.getClass());
    return typeArgs[0].isAssignableFrom(event.getClass())
  }
}

```

#### 타입스크립트 버전

- 타입 정보는 컴파일 후 사라지므로, 이벤트 클래스를 생성자의 인자로 넘긴다.

```ts
abstract class EventHandler<T> {
  event: T

  constructor(event: T) {
    this.event = event
  }

  handle(event: T): void

  canHandle(event: object) {
    return event instanceof this.event
  }
}
```

- `handle` 메서드를 구현해서 이벤트를 처리한다.

```ts
const handler = new (class extends EventHandler<PasswordChangedEvent> {
  handle(event: PasswordChangedEvent) {
    /** 이벤트 처리 */
  }
})(PasswordChangedEvent)

const result = handler.canHandle(new PasswordChangedEvent(id, pwd))
```

### 디스패쳐인 `Events`

- `Events.handle()`로 핸들러를 등록하고 도메인 기능을 실행한다.
- `Events.raise()` 메서드로 이벤트를 발생시킨다.

```ts
class Events {
  static #handlers: List<EventHandler<unknown>> = []

  raise(event: object) {
    this.#handlers
      .filter((handler) => handler.canHandle(event))
      .forEach((handler) => handler.handle(event))
  }

  handle(handler: EventHandler<unknown>) {
    this.#handlers.push(handler)
  }

  reset() {
    this.#handlers = []
  }
}
```

- 아래처럼 직접 이벤트 핸들러를 등록할 수 있다.

```ts
@Transactional
async cancel(orderNo: OrderNo) {
  // 이벤트 핸들러 등록
  Events.handle((e: OrderCanceledEvent) => refundService.refund(e.orderNumber))
  const order = await findOrder(orderNo)
  order.cancel() // 여기서 OrderCanceledEvent 이벤트 발생

  Events.reset() // 핸들러 무한 등록으로 메모리 부족 방지
}
```

- 스프링에서는 AOP를 사용해서 자동으로 `Events.reset()`이 호출되게 할 수 있다.

## 비동기 이벤트 처리

- 회원 가입 후 이메일이 조금 늦게 도착해도 괜찮다.
  - 이메일을 못 받아도 재전송 가능하다.
- 주문을 취소하자마자 바로 결제를 취소하지 않아도 된다.
  - 몇 분 또는 며칠 안에만 결제가 취소되면 된다.
- `A하면 B하라`라는 요구사항은 `A하면 최대 언제 까지 B하라`인 경우가 많다.
  - 여기서 `A하면`를 이벤트로 볼 수 있다.
- 이러한 요구사항은 비동기로 처리 가능하다.

### 이벤트를 비동기로 처리하는 방법은 다양하다.

1. 로컬 핸들러를 비동기로 실행하기
2. 메시지 큐를 사용하기
3. 이벤트 저장소와 이벤트 포워더 사용하기
4. 이벤트 저장소와 이벤트 제공 API 사용하기

### 로컬 핸들러를 비동기로 실행하기

- 이벤트 핸들러를 별도의 스레드로 실행한다.
  - 노드는 `setImmediate` 사용
- 자바의 경우 핸들러를 실행하는 람다식을 `ExecutorService`에 전달한다.
- 이때, 이벤트 핸들러의 실행은 트랜잭션 범위에 묶이지 않는다.
- 한 트랙잭션으로 실행해야 하는 이벤트 핸들러는 반드시 동기적으로 처리해야 한다.

### 메시징 시스템을 이용한 비동기 구현

- RabbitMQ를 사용한다.
- 메시지 큐에 이벤트를 전달하고, 메시지 리스너는 알맞은 핸들러로 이벤트를 처리한다.
- 도메인 기능와 이벤트 저장을 한 트랜잭션으로 묶으려면 글로벌 트랜잭션을 사용해야 한다.
  - 글로벌 트랜잭션을 사용하면 시스템 성능이 떨어지는 단점이 있다.
- 0.11.0 버전 이상의 Kafka를 사용할 수도 있다.

### 이벤트 저장소를 이용한 비동기 처리

- 이벤트를 일단 DB에 저장하고 별도 프로그램으로 이벤트 핸들러에 전달한다.

- 포워더가 주기적으로 이벤트 저장소에서 이벤트를 가져와 이벤트 핸들러를 실행한다.

- API 방식으로 이벤트를 외부에 제공할 수 있다.
  - 포워더는 이벤트를 외부에 전달하지만
  - API 방식은 외부 핸들러가 API 서버에서 이벤트 목록을 가져온다.
  - 이벤트를 어디까지 처리했는지 추적하는 역할이 달라진다.

## 이벤트 저장소 구현

### `EventEntry` 구현

- 이벤트 저장소에 보관할 데이터
- `id`: 식별자
- `type`: 이벤트 타입
- `contentType`: 직렬화된 데이터 형식
- `payload`: 이벤트 데이터
- `timestamp`: 이벤트 발생 시간

### `EventStore` 구현

- 이벤트를 저장하고 가져올 수 있는 인터페이스
  - 이벤트는 과거에 발생한 사건이므로 변경이 불가능하다.
  - 이벤트 추가와 조회 기능만 제공하고 수정 기능은 제공하지 않는다.

### `Rest API` 구현

- `offset`과 `limit` 요청 패러미터를 사용해서 `EventStore`의 결과를 JSON으로 반환한다.

### 포워더 구현

일정주기로 `EventStore`에서 이벤트를 읽어와 이벤트 핸들러에 전달한다.

- 마지막으로 전달한 이벤트와 오프셋을 기억하고, 다음 조회 시 마지막으로 처리한 오프셋부터 이벤트를 가져온다.

## 이벤트 적용 시 유의사항

### 1. 이벤트 소스를 `EventEntry`에 추가할지

- 이벤트 발생 주체를 추가할지를 결정해야 한다.
- 특정 주체가 발생한 이벤트만 조회 같은 기능을 구현할 때 사용한다.

### 2. 포워더 전송 실패 시 재전송 횟수

- 특정 이벤트 전송 실패시, 실패한 이벤트 부터 다시 읽는다.
- 계속 전송 실패 시, 다른 이벤트 전송이 막히게 된다.
- 이벤트 재전송 획수 제한이 필요하다.
- ex) 이벤트 전송 3회 실패 시, 해당 이벤트 생략하기

### 3. 이벤트 손실

- 이벤트 저장소 방식은 이벤트 발생과 저장이 하나의 트랜잭션으로 처리된다.
- 하지만 로컬 핸들러 비동기 처리 시 이벤트 실패가 이벤트 유실로 이어진다.

### 4. 이벤트 순서

- 메시징 시스템에 따라 이벤트 발생 순서와 전달 순서가 다를 수 있다.

### 5. 이벤트 재처리

- 왔던 이벤트가 또 왔을 때 어떻게 할지 결정해야 한다.
- 마지막 이벤트 순번을 기록했다가 또 같은 이벤트가 오면 해당 이벤트를 무시할 수 있다.
- 이벤트 처리를 멱등적으로 하는 방법도 있다.
