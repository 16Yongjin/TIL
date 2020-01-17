# 스테이트 패턴

내부 상태를 나타내는 객체가 컨텍스트 객체의 행동을 결정한다.

유한 상태 기계를 객체의 조합으로 표현할 수 있다.

## 클래스 다이어그램

`State` 인터페이스는 각 상태의 공통 인터페이스를 정의한다.

`Context` 클래스에 여러 상태가 들어있고 `request()` 메서드가 호출되면 상태 객체에서 작업이 실행된다.

현재의 상태 객체가 알아서 작업을 실행해주므로 `Context` 클래스는 내부 상태가 어떤지 신경쓰지 않아도 된다.

![state-diagram](https://user-images.githubusercontent.com/22253556/72606014-2c88df80-3961-11ea-9451-9def02ea1a15.png)

## 스트래티지 패턴과의 관계

위의 다이어그램은 스트래티지 패턴의 다이어그램과 똑같다.

둘 다 객체 조합을 이용한다.

스트래티지 패턴은 행동을 변경하기 힘든 상속을 대체하기 위함이고

스테이트 패턴은 코드를 복잡하게 만드는 거대한 조건문을 대체하기 위해 사용된다.

## 단순한 구현

현대인의 삶의 형태를 모델링한다.

![personState](https://user-images.githubusercontent.com/22253556/72610095-dff5d200-3969-11ea-9832-60899e9716dd.png)

현대인의 상태를 나타내는 `State`

```typescript
enum State {
  Broke,
  Hungry,
  Sleepy
}
```

돈없으면 일하고, 일하면 배고프고, 배고프면 밥먹고, 졸리면 자고, 일어나면 배고프다.

```typescript
class Person {
  state: State = State.Broke;

  work() {
    switch (this.state) {
      case State.Broke:
        this.state = State.Hungry;
        return console.log("일을 했더니 배가 고프다.");
      case State.Hungry:
        return console.log("배고파서 일하기 싫음");
      case State.Sleepy:
        return console.log("졸려서 일하기 싫음");
    }
  }

  eat() {
    switch (this.state) {
      case State.Broke:
        return console.log("돈 걱정에 밥이 안들어감");
      case State.Hungry:
        this.state = State.Sleepy;
        return console.log("배부르니까 졸리다.");
      case State.Sleepy:
        return console.log("졸려서 밥 먹기 귀찮음");
    }
  }

  eatOut() {
    switch (this.state) {
      case State.Broke:
        return console.log("빕 사먹을 돈 없음");
      case State.Hungry:
        this.state = State.Broke;
        return console.log("밖에서 밥먹으니까 돈이 없다.");
      case State.Sleepy:
        return console.log("졸려서 밥 먹기 귀찮음");
    }
  }

  sleep() {
    switch (this.state) {
      case State.Broke:
        return console.log("돈 걱정에 잠이 안 온다.");
      case State.Hungry:
        return console.log("배고파서 잠이 안 온다.");
      case State.Sleepy:
        this.state = State.Hungry;
        return console.log("자고 일어났더니 배고프다.");
    }
  }
}
```

## 위와 같이 구현하면 안 좋은 점

새로운 상태를 추가하고 싶으면 모든 메서드를 고쳐야 한다.

그리고 코드도 복잡해서 버그 발생 확률이 높다.

## 스테이트 패턴으로 재구현하기

상태들의 공통 인터페이스인 `PersonState`이다.

```typescript
interface PersonState {
  work(): void;
  eat(): void;
  eatOut(): void;
  sleep(): void;
}
```

각 상태를 나타내는 클래스를 구현한다.

돈이 없는 상태를 나타내는 `BrokeState` 클래스

```typescript
class BrokeState implements State {
  person: Person;

  constructor(person: Person) {
    this.person = person;
  }

  work() {
    this.person.setState(this.person.hungryState);
    console.log("일을 했더니 배가 고프다.");
  }

  eat() {
    console.log("돈 걱정에 밥이 안들어감");
  }

  eatOut() {
    console.log("빕 사먹을 돈 없음");
  }

  sleep() {
    console.log("돈 걱정에 잠이 안 온다.");
  }
}
```

배고픈 상태를 나타내는 `HungryState` 클래스

```typescript
class HungryState implements State {
  person: Person;

  constructor(person: Person) {
    this.person = person;
  }

  work() {
    console.log("배고파서 일하기 싫음");
  }

  eat() {
    this.person.setState(this.person.sleepyState);
    console.log("배부르니까 졸리다.");
  }

  eatOut() {
    this.person.setState(this.person.brokeState);
    console.log("밖에서 밥먹으니까 돈이 없다.");
  }

  sleep() {
    console.log("배고파서 잠이 안 온다.");
  }
}
```

졸린 상태를 나타내는 `SleepyState` 클래스

```typescript
class SleepyState implements State {
  person: Person;

  constructor(person: Person) {
    this.person = person;
  }

  work() {
    console.log("졸려서 일하기 싫음");
  }

  eat() {
    console.log("졸려서 밥 먹기 귀찮음");
  }

  eatOut() {
    console.log("졸려서 밥 먹기 귀찮음");
  }

  sleep() {
    this.person.setState(this.person.hungryState);
    console.log("자고 일어났더니 배고프다.");
  }
}
```

컨텍스트를 나타내는 `Person` 클래스다.

```typescript
class Person {
  brokeState: State = new BrokeState(this);
  hungryState: State = new HungryState(this);
  sleepyState: State = new SleepyState(this);

  state = this.brokeState;

  setState(state: State) {
    this.state = state;
  }

  work() {
    this.state.work();
  }

  eat() {
    this.state.eat();
  }

  eatOut() {
    this.state.eatOut();
  }

  sleep() {
    this.state.sleep();
  }
}
```

각 상태 객체에 행동을 위임해서 코드가 길어졌지만 컨텍스트 클래스는 간결해졌고

새로운 상태를 추가하려면 상태 객체만 추가하면 되므로 확장에 용이하다.

## 테스트

```typescript
const person = new Person();

person.work();
person.eat();
person.sleep();
person.eatOut();
```

### 결과

```
일을 했더니 배가 고프다.
배부르니까 졸리다.
자고 일어났더니 배고프다.
밖에서 밥먹으니까 돈이 없다.
```
