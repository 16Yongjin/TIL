# 데코레이터 패턴

데코레이터 패턴에서는 객체에 추가적인 요건을 동적으로 추가한다. 데코레이터는 서브클래스를 만들어서 기능을 유연하게 확장할 수 있는 방법을 제공한다.

## 클래스 다이어그램

![decorator-pattern](https://user-images.githubusercontent.com/22253556/71577990-9ae43a00-2b39-11ea-97d6-c74788c70cd8.png)

각 데코레이터 안에는 구성요소에 대한 레퍼런스가 들어있는 인스턴스 변수가 있다.

데코레이터가 동적으로 구성요소에 새로운 상태나 행동을 추가할 수 있다.

## 특징

- 상속을 통한 확장보다 유연하다.
- 기존 코드를 수정하지 않고도 행동을 확장할 수 있다.
- 구성요소를 감싸는 데코레이터의 개수에는 제한이 없다.
- 다만, 자잘한 객체들이 너무 많이 추가되어, 그런 경우 코드가 복잡해질 수 있다. (이 경우, 팩토리와 빌더 패턴으로 해결할 수 있다.)

## 구현

첨가물을 추가하기 쉽고 자동으로 첨가물 가격이 추가되는 음료 클래스를 만들려한다.

### 음료를 나타내는 추상 클래스

```typescript
abstract class Beverage {
  description = "없음";

  getDescription(): string {
    return this.description;
  }

  abstract cost(): number;
}
```

### 첨가물을 나타내는 추상 클래스

```typescript
abstract class CondimentDecorator extends Beverage {
  abstract getDescription(): string;
}
```

### 음료 코드 구현

아메리카노와 하우스 블랜드 커피를 구현한다.

```typescript
class Americano extends Beverage {
  constructor() {
    super();
    this.description = "아메리카노";
  }

  cost(): number {
    return 3800;
  }
}

class HouseBlend extends Beverage {
  constructor() {
    super();
    this.description = "하우스 블렌드 커피";
  }

  cost(): number {
    return 4600;
  }
}
```

### 첨가물 코드 구현

모카와 휘핑크림, 얼음을 구현한다.

```typescript
// 모카
class Mocha extends CondimentDecorator {
  beverage: Beverage;

  constructor(beverage: Beverage) {
    super();
    this.beverage = beverage;
  }

  getDescription(): string {
    return `모카 ${this.beverage.getDescription()}`;
  }

  cost(): number {
    return 1000 + this.beverage.cost();
  }
}

// 휘핑 크림
class Whip extends CondimentDecorator {
  beverage: Beverage;

  constructor(beverage: Beverage) {
    super();
    this.beverage = beverage;
  }

  getDescription(): string {
    return `휘핑크림 ${this.beverage.getDescription()}`;
  }

  cost(): number {
    return 300 + this.beverage.cost();
  }
}

// 얼음
class Ice extends CondimentDecorator {
  beverage: Beverage;

  constructor(beverage: Beverage) {
    super();
    this.beverage = beverage;
  }

  getDescription(): string {
    return `아이스 ${this.beverage.getDescription()}`;
  }

  cost(): number {
    return 500 + this.beverage.cost();
  }
}
```

## 테스트

```typescript
// 아이스 아메리카노
const americano = new Americano();
const icedAmericano = new Ice(americano);
console.log(icedAmericano.getDescription(), "￦", icedAmericano.cost());

// 모카 2개와 휘핑크림 올린 당 폭탄 하우스 블렌드
let sugarBomb = new HouseBlend();
sugarBomb = new Mocha(sugarBomb);
sugarBomb = new Mocha(sugarBomb);
sugarBomb = new Whip(sugarBomb);
console.log(sugarBomb.getDescription(), "￦", sugarBomb.cost());
```

### 결과

음료에 첨가물을 추가하기 쉽고, 추가된 첨가물에 따라 음료 가격이 자동으로 계산된다.

```
아이스 아메리카노 ￦ 4300
휘핑 크림 모카 모카 하우스 블렌드 커피 ￦ 6900
```
