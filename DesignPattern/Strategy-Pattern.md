# 스트래티지 패턴(Strategy Pattern)

객체의 행동을 자유롭게 바꿔 끼울 수 있는 패턴

상속(Inheritance)보단 조립(Composition)을 활용하라는 객체지향 원칙을 따른다.

## 오리는 날고 싶다.

quack(), swim() 메서드를 가진 "오리" 클래스가 있다.

![duck](https://user-images.githubusercontent.com/22253556/71516935-100fff00-28ef-11ea-987b-4c24ff6d8164.png)

오리들이 날아다닐 수 있도록 하고 싶다.

## 상속 사용하기

오리 클래스에 fly() 메서드를 추가했다.

![duck2](https://user-images.githubusercontent.com/22253556/71516946-228a3880-28ef-11ea-9a3b-da8e6a42b181.png)

그랬더니 날면 안 되는 고무오리까지 날아다니게 됐다.

![flying0rubber-duck](https://www.globaltimes.cn/Portals/0/attachment/2011/ec83121c-c227-49c1-8772-1856ca1ad10f.jpeg)

상속으로 코드를 재사용할 수 있을 거 같았지만 별 도움이 안 된다.

날면 안 되는 오리에 날지 못하도록 fly() 메서드를 오버라이드 해보려니

오리마다 fly() 메서드가 아무것도 안 하도록 오버라이드를 해야 돼서 코드 칠 게 너무 많다.

## 인터페이스 사용하기

fly() 메서드를 가진 Flayable 인터페이스와, quack() 메서드를 가진 Quackable 인터페이스를 만들었다.

![duck(3)](https://user-images.githubusercontent.com/22253556/71516964-37ff6280-28ef-11ea-9c14-6de717a2b0a9.png)

인터페이스를 사용해서 자식 오리 클래스들이 자신들이 하는 행동만 구현해서 사용할 수 있도록 했다.

오리 종류 100가지나 된다면, 모든 오리 서브 클래스의 메서드를 오버라이드 하기란 쉽지 않다.

## 디자인 원칙 1

**애플리케이션에서 달리지는 부분을 찾아내고, 달라지지 않는 부분으로부터 분리시킨다.**

다시 말하면,

바뀌는 부분은 따로 뽑아서 캡슐화시킨다. 그렇게 하면 나중에 바뀌지 않는 부분에는 영향을 미치지 않은 채로 그 부분만 고치거나 확장할 수 있다.

## 바뀌는 부분과 그렇지 않은 부분 분리하기

fly()와 quack()은 오리 클래스에서 오리마다 달라진다.

두 메서드를 분리해서 각 행동을 나타내는 새로운 집합을 만든다.

## 디자인 원칙 2

**구현이 아닌 인터페이스에 맞춰서 프로그래밍한다.**

## 오리의 행동

FlyBehavior와 QuackBehavior라는 두 인터페이스와 구체적인 행동을 구현하는 클래스들을 만든다.

![duck-Page-2](https://user-images.githubusercontent.com/22253556/71516978-4baac900-28ef-11ea-8b04-61c5602fdaa0.png)

이런 식으로 디자인하면 다른 타입의 객체에서도 나는 행동과 꽥꽥거리는 행동을 재사용할 수 있다.

또한, 기존의 오리 클래스를 건드리지 않고도 새로운 행동을 추가할 수 있다.

## 오리 행동 통합하기

오리의 행동을 오리 클래스에서 구현하지 않고 다른 클래스에 위임한다.

![duck-Page-3(1)](https://user-images.githubusercontent.com/22253556/71516994-60875c80-28ef-11ea-9170-60f81672732a.png)

이런 식으로

```typescript
class 오리 {
  // 모든 오리는 QuackBehavior 인터페이스를
  // 구현하는 것에 대한 레퍼런스를 가진다.
  quackBehavior: QuackBehavior;

  performQuack(): void {
    // 꽥꽥거리는 행동을 직접 처리하는 대신,
    // quackBehavior로 참조되는 객체에 그 행동을 위임한다.
    quackBehavior.quack();
  }
}
```

## 행동을 동적으로 지정하기

오리 클래스에 아래 메서드를 추가하고 필요에 따라 호출해서 행동을 바꾸면 된다.

```typescript
setFlyBehavior(fb: FlyBehavior) {
  this.flyBehavior = fb;
}

setQuackBehavior(qb: QuackBehavior) {
  this.quackBehavior = qb;
}
```

## 구현

```typescript
abstract class 오리 {
  // 행동 인터페이스에 대한 레퍼런스
  flyBehavior: FlyBehavior;
  quackBehavior: QuackBehavior;

  constructor() {}

  abstract display(): void;

  // 클래스에 행동을 위임
  performFly(): void {
    this.flyBehavior.fly();
  }

  performQuack(): void {
    this.quackBehavior.quack();
  }

  setFlyBehavior(fb: FlyBehavior) {
    this.flyBehavior = fb;
  }

  setQuackBehavior(qb: QuackBehavior) {
    this.quackBehavior = qb;
  }

  swim(): void {
    console.log("모든 오리는 물에 뜬다.");
  }
}

// 나는 행동
interface FlyBehavior {
  fly(): void;
}

class FlyWithWings implements FlyBehavior {
  fly(): void {
    console.log("날고 있어요");
  }
}

class FlyNoWay implements FlyBehavior {
  fly(): void {
    console.log("못 날아요");
  }
}

// 꽥꽥거리는 행동
interface QuackBehavior {
  quack(): void;
}

class Quack implements QuackBehavior {
  quack(): void {
    console.log("꽥");
  }
}

class MuteQuack implements QuackBehavior {
  quack(): void {
    console.log("(조용..)");
  }
}

class Squeak implements QuackBehavior {
  quack(): void {
    console.log("삑");
  }
}

class 청둥오리 extends 오리 {
  constructor() {
    super();
    this.flyBehavior = new FlyWithWings();
    this.quackBehavior = new Quack();
  }

  display(): void {
    console.log("진짜 청둥오리");
  }
}

const 청둥 = new 청둥오리();

청둥.performFly(); // 날고 있어요
청둥.performQuack(); // 꽥
```

## 생각거리

- 행동 메서드를 호출할 때 실행에 필요한 의존성(변수, 다른 메서드)은 어떻게 구하는가?

## 참고자료

- Head First Design Patterns
