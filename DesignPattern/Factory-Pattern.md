# 팩토리 패턴

팩토리 메서드 패턴과 추상 팩토리 패턴

## 간단한 팩토리

조건에 따라 객체를 다르게 생성하는데 사용된다.

옛날에 정리한 [팩토리 패턴](https://yongj.in/design%20pattern/simple-factory/)은 알고보니 패턴이 아니라 그냥 **팩토리**였다.

팩토리는 디자인 패턴은 아니지만, 클라이언트와 구체 클래스를 분리시켜 변경에 유연하게 대처하는데 유용하다.

# 팩토리 메서드 패턴

팩토리 메서드 패턴은 인스턴스를 만드는 일을 서브클래스에 넘긴다.

일을 넘길 때는 **상속**을 사용한다.

서브 클래스는 팩토리 메서드를 구현해서 객체를 생산하는 역할을 한다.

추상 클래스/인터페이스에 맞게 코딩할 수 있게 한다.

추상 클래스에서 생성될 객체가 뭔지 몰라도 인터페이스만 알면 그 객체를 사용할 수 있다.

## 구현

교촌치킨과 BHC가 M&A로 한 회사가 되어 가맹점들의 치킨 제조과정을 일원화하려고 한다.

모든 치킨의 기초가 될 `Chicken` 추상 클래스다.

```typescript
abstract class Chicken {
  name: string;
  sause: string;

  prepare() {
    console.log(`${this.name} 주문 준비`);
    console.log("|  치킨 반죽하기");
  }

  fry() {
    console.log("|  치킨 튀기기");
  }

  seasoning() {
    console.log("|  양념 칠하기");
  }

  box() {
    console.log("|  치킨 포장");
  }
}
```

치킨집의 베이스가 될 `ChickenStore` 추상 클래스다.

`createChicken()` 추상 메서드가 선언되어 서브 클래스에서 이를 구현한다.

`orderChicken()`에서 `createChicken()`가 뭔지는 몰라도 이 메서드를 호출해서 치킨 주문을 처리한다.

```typescript
abstract class ChickenStore {
  // 팩토리 메서드
  abstract createChicken(type: string): Chicken;

  orderChicken(type: string): Chicken {
    // createChicken()이 어떤 치킨을 반환할 지 몰라도 일단 사용가능함
    const chicken: Chicken = this.createChicken(type);

    chicken.prepare();
    chicken.fry();
    chicken.seasoning();
    chicken.box();

    return chicken;
  }
}
```

교촌치킨 시리즈를 파는 `KyochonChickenStore`다.

`createChicken()` 메서드를 구현하여 브랜드에 맞는 치킨을 만든다.

```typescript
class KyochonChickenStore extends ChickenStore {
  createChicken(type: string): Chicken {
    switch (type) {
      case "교촌허니오리지날":
        return new 교촌허니오리지날();
      case "교촌레드스틱":
        return new 교촌레드스틱();
      case "교촌살살치킨":
        return new 교촌살살치킨();
    }
  }
}

class 교촌허니오리지날 extends Chicken {
  name = "교촌허니오리지날";
  sause = "간장";
}

class 교촌레드스틱 extends Chicken {
  name = "교촌레드스틱";
  sause = "레드";
}

class 교촌살살치킨 extends Chicken {
  name = "교촌살살치킨";
  sause = "잠발라야";
}
```

맛초킹과 뿌링클을 만드는 `BHCChickenStore`다.

마찬가지로, `createChicken()` 메서드를 구현하여 브랜드에 맞는 치킨을 만든다.

```typescript
class BHCChickenStore extends ChickenStore {
  createChicken(type: string): Chicken {
    switch (type) {
      case "맛초킹":
        return new 맛초킹();
      case "뿌링클":
        return new 뿌링클();
    }
  }
}

class 맛초킹 extends Chicken {
  name = "맛초킹";
  sause = "맛초킹소스";
}

class 뿌링클 extends Chicken {
  name = "뿌링클";
  sause = "치즈시즈닝";
}
```

## 테스트

교촌에는 교촌레드스틱을, BHC에는 뿌링클을 주문한다.

```typescript
console.log("교촌치킨 주문");
const kyochon = new KyochonChickenStore();
const kyochonChicken = kyochon.orderChicken("교촌레드스틱");
console.log(kyochonChicken.name);

console.log("\nBHC치킨 주문");
const bhc = new BHCChickenStore();
const bhcChicken = bhc.orderChicken("뿌링클");
console.log(bhcChicken.name);
```

결과

```
교촌치킨 주문
교촌레드스틱 주문 준비
|  치킨 반죽하기
|  치킨 튀기기
|  양념 칠하기
|  치킨 포장
교촌레드스틱

BHC치킨 주문
뿌링클 주문 준비
|  치킨 반죽하기
|  치킨 튀기기
|  양념 칠하기
|  치킨 포장
뿌링클
```

치킨 제조과정은 일원화하면서 브랜드에 맞는 치킨을 만들 수 있게 했다.

치킨을 더 오래 튀기거나 제품 박스를 바꾸고 싶을 때 `ChickenStore` 클래스만 수정하면 된다.

## 의존성 뒤집기 원칙(Dependency Inversion Principle)

**구체적인 것 보다 추상적인 것에 의존적이게 만들기**

`createChicken()`을 추상화함으로써 치킨 종류에 신경쓰지 않고 치킨 주문을 받을 수 있게 됐다.

### 이 원칙을 위한 가이드라인

- 어떤 변수에도 구체 클래스에 대한 레퍼런스를 저장하지 않는다.
- 구체 클래스에서 유도된 클래스를 만들지 않는다.
- 베이스 클래스에 이미 구현된 메서드를 오버라이드하지 않는다.

# 추상 팩토리 패턴

인터페이스를 이용하여 서로 연관되거나 의존하는 객체를 구체 클래스를 사용하지 않아도 만들 수 있다.

객체의 조합을 활용하고 객체 생성은 팩토리 인터페이스에서 선언한 메서드들에서 구현된다.

들어가는 부품이 같은 비슷한 제품이 있을 때 각 부품들을 제품에 따라 다르게 만들고 싶을 때 사용한다.

## 구현

본사에서 치킨에 필요한 소스와 생닭, 무, 박스를 직접 가맹점에 공급해서 원재료의 품질을 관리하려고 한다.

우선, 원재료를 생성할 팩토리 인터페이스를 정의한다.

```typescript
interface ChichenIngredientFactory {
  createRawChicken(): RawChicken;
  createSauce(): Sauce;
  createBox(): Box;
  createRadis(): Radis;
}
```

교촌치킨에 맞는 원재료를 생성하는 공장을 구현한다.

```typescript
class KyochonChichenIngredientFactory implements ChichenIngredientFactory {
  createRawChicken() {
    return new 교촌생닭();
  }

  createSauce() {
    return new 교촌소스();
  }

  createBox() {
    return new 교촌박스();
  }

  createRadis() {
    return new 교촌무();
  }
}
```

BHC 치킨에 맞는 원재료를 생성하는 공장을 구현한다.

```typescript
class BHCChichenIngredientFactory implements ChichenIngredientFactory {
  createRawChicken() {
    return new BHC생닭();
  }

  createSauce() {
    return new BHC소스();
  }

  createBox() {
    return new BHC박스();
  }

  createRadis() {
    return new BHC무();
  }
}
```

치킨을 만들 때는 원재료 공장에서 나온 재료를 사용하게 한다.

신메뉴를 출시할 때, 하나의 치킨에 각 브랜드에 맞는 재료를 제공함으로써 브랜드에 맞는 치킨을 만들어 낼 수 있다.

예를 들어, 마라맛치킨을 출시할 때

```typescript
class 마라맛치킨 extends Chicken {
  ingredientFactory: ChichenIngredientFactory;

  constructor(ingredientFactory: ChichenIngredientFactory) {
    super();
    this.ingredientFactory = ingredientFactory;
  }

  prepare() {
    this.name = "마라맛치킨";
    this.sause = this.ingredientFactory.createSauce();
    this.box = this.ingredientFactory.createBox();
    this.radis = this.ingredientFactory.createRadis();
    this.rawChicken = this.ingredientFactory.createRawChicken();
  }
}
```

교촌치킨에 맞는 재료를 제공해서 교촌마라맛치킨을 만들 수 있고

```typescript
const 교촌마라맛치킨 = new 마라맛치킨(new KyochonChichenIngredientFactory());
```

BHC에 맞는 재료를 제공해서 bhc마라맛치킨도 만들 수 있다.

```typescript
const bhc마라맛치킨 = new 마라맛치킨(new BHCChichenIngredientFactory());
```

## 결론

두 패턴 모두 객체 생성을 캡슐화해서 코드와 구체 타입을 분리할 수 있게 한다.
