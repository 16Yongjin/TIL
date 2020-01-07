# 템플릿 메서드 패턴

전체 알고리즘은 상위 클래스에서 구현하면서 특정 부분은 서브 클래스에서 구현하게 하는 패턴

알고리즘의 구조는 유지하면서 바뀌는 부분만 따로 구현할 수 있다.

일부분만 구현하면 전체 알고리즘을 사용할 수 있으니 코드 재사용에 도움이 된다.

제어 역전 특성 덕분에 프레임워크에서 많이 사용되는 패턴이다.

## 클래스 다이어그램

`AbstractClass`에 템플릿 메서드가 들어있다.

이 템플릿 메서드에서는 알고리즘 구현 시 `primitiveOperation()`을 활용한다.

`abstract`로 선언된 `primitiveOperation()`들을 서브 클래스인 `ConcreteClass` 에서 구현한다.

![template-method](https://user-images.githubusercontent.com/22253556/71810981-f9b82f00-30b6-11ea-9c8a-47512a75e21d.png)

## 훅(Hook)

훅는 상위 클래스에서 선언되어 기본적인 내용만 구현되어 있거나 아무것도 들어있지 않은 메서드이다.

템플릿 메서드에 중간중간에 후크를 넣고 서브 클래스에서 오버라이드 해서 알고리즘 실행 중에 원하는 기능이 수행되게 할 수 있다.

React의 `componentDidMount`나 Vue의 `created` 같은 컴포넌트 라이프사이클 훅이 자주 쓰는 템플릿 메서드 패턴의 훅이다.

필수적인 부분이 아니라 구현해도 되고 안 해도 되기에 훅 메서드를 `abstract`로 선언하지 않는다.

## 할리우드 원칙

먼저 연락하지 마세요. 저희가 연락 드리겠습니다.

> 오디션에 실패한 배우들이 많이 듣던 말이라고 한다.

저수준 구성요소를 고수준 구성요소에만 사용하게 해서 의존성이 복잡하게 꼬이는 **의존성 부패를 방지**한다.

서브 클래스가 구현한 메서드는 호출을 "당해야" 추상 클래스를 사용할 수 있다.

## 템플릿 메서드로 정렬하기

자바에서는 `Comparable` 인터페이스를 구현하면 `Arrays`의 `sort()` 템플릿 메서드를 사용할 수 있다.

자바스크립트에서는 `Array.prototype.sort()`에 알맞는 비교함수를 넣으면 뭐든지 정렬을 할 수 있다.

> 이 경우는 상속이나 구현을 사용하지 않아서 맞는 예제인지 애매하다..

## 구현

딥러닝 모델을 훈련하는 프레임워크를 만든다.

추상 클래스인 `ModelTrainer`르 정의한다.

`train()` 메서드는 템플릿 메서드로 서브 클래스에서 구현되어 일부 기능을 수행할 메서드들을 호출한다.

```typescript
abstract class ModelTrainer {
  train() {
    this.predict();
    this.calcLoss();
    this.backPropagate();
    this.updateWeight();
  }

  updateWeight() {
    console.log("가중치 업데이트");
  }

  abstract predict(): void;

  abstract calcLoss(): void;

  abstract backPropagate(): void;
}
```

`SimpleModelTrainer`는 `ModelTrainer`를 상속받아 필요한 부분만 구현한다.

```typescript
class SimpleModelTrainer extends ModelTrainer {
  predict() {
    console.log("값 에측");
  }

  calcLoss() {
    console.log("손실 계산");
  }

  backPropagate() {
    console.log("역전파");
  }
}
```

## 테스트

```typescript
const trainer = new SimpleModelTrainer();

trainer.train();
```

### 결과

```
값 에측
손실 계산
역전파
가중치 업데이트
```
