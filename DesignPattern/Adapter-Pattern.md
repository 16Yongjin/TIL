# 어댑터 패턴

![real-adapter](https://cdn.shopify.com/s/files/1/0004/5750/6867/products/Screen_Shot_2018-02-15_at_1.22.06_AM_grande_0fd4ebb1-b609-4a63-81ef-7c0b5896e011_1024x1024.png?v=1550683159)

어댑터 패턴은 코드 변경 없이 한 인터페이스를 다른 인터페이스에서 사용할 수 있게 한다.

## 클래스 다이어그램

어댑터에서 타겟 인터페이스를 구현한다.

모든 요청은 어댑터에 위임된다.

클라이언트 -> request() -> 어댑터 - specificRequest() -> 어댑티.

![adapter](https://user-images.githubusercontent.com/22253556/71764765-61dc0900-2f2f-11ea-80de-1bdba5098b56.png)

## 클래스 어댑터 패턴

어댑터 패턴에는 객체 어댑터, 클래스 어댑터 두 종류가 있다.

위 다이어그램에도 나온 객체 어댑터 패턴은 객체 구성을 사용하는 반면,

클래스 어댑터 패턴은 다중 상속을 이용하기에 다중 상속 기능이 없는 언어에서는 사용할 수 없다.

## 구현

닭꼬치 노점상에서 오랜만에 장사가 잘 되고 있는데 닭이 떨어졌다.

마침 비둘기 떼가 옆을 지나간다.

### `Chicken` 인터페이스

```typescript
interface Chicken {
  cluck(): void;
  fly(): void;
}
```

### `Chicken`을 구현하는 하림치킨

```typescript
class HarimChicken implements Chicken {
  cluck() {
    console.log("꼬끼오");
  }

  fly() {
    console.log("푸드덕");
  }
}
```

### `Pigeon` 인터페이스

```typescript
interface Pigeon {
  coocoo(): void;
  fly(): void;
}
```

### 마침 옆을 지나가던 비둘기(`Columbidae`)

```typescript
class Columbidae implements Pigeon {
  coocoo() {
    console.log("구구");
  }

  fly() {
    console.log("살쪄서 조금 낢");
  }
}
```

### 비둘기를 닭으로 바꾸는 어댑터

타겟 인터페이스에 들어있는 메서드를 모두 구현한다.

> 그래서 어댑터 구현 시, 타겟 인터페이스의 크기와 구조에 따라 코딩해야 할 분량이 결정된다.

```typescript
class PigeonAdapter implements Chicken {
  pigeon: Pigeon;

  constructor(pigeon: Pigeon) {
    this.pigeon = pigeon;
  }

  cluck() {
    this.pigeon.coocoo();
  }

  fly() {
    this.pigeon.fly();
  }
}
```

## 테스트

```typescript
function testChicken(chicken: Chicken) {
  chicken.cluck();
  chicken.fly();
}

const chicken: Chicken = new HarimChicken();
const columbidae: Pigeon = new Columbidae();
const pigeonAdapter = new PigeonAdapter(columbidae);

console.log("진짜 치킨");
testChicken(chicken);

console.log("\n가짜 치킨");
testChicken(pigeonAdapter);
```

### 결과

비둘기를 닭처럼 사용할 수 있다.

```
진짜 치킨
꼬끼오
푸드덕

가짜 치킨
구구
살쪄서 조금 낢
```

## 데코레이터 패턴과의 차이점

어댑터 패턴은 객체를 감싸서 인터페이스를 바꾸기 위한 용도로 사용된다.

반면, 데코레이터 패턴은 객체를 감싸서 **새로운 행동(책임)을 추가**하기 위한 용도로 사용된다.
