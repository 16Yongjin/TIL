# 러스트와 객체 지향 프로그래밍

## 객체 지향 언어의 특성

객체 지향 언어는

- 객체
- 캡슐화
- 상속

을 지원한다.

## GOF의 OOP 정의

> 객체-지향 프로그램은 객체로 구성된다. 객체는 데이터 및 이 데이터를 활용하는 프로시저를 묶는다. 이 프로시저들은 보통 메소드 혹은 연산 (operation) 으로 불린다.

이 정의에 따라 러스트도 객체 지향적이라고 할 수 있다.

구조체와 열거형으로 데이터를 표현하고 `impl` 블럭으로 메서드를 제공한다.

## 캡슐화

객체 사용자가 그 객체의 세부 구현에 접근하는 것을 막는 것이다

사용자가 코드 내부 데이터나 동작을 변경하는 것을 막고 공개된 API만 사용하게 하면, 사용자 코드는 그대로 두면서 객체 내부를 변경할 수 있다.

## 러스트의 캡슐화

모듈, 타입, 함수, 메서드는 기본적으로 비공개이고 `pub` 키워드로 공개 여부를 결정한다.

아래 코드는 벡터의 값의 평균값을 캐시하는 구조체이다.

```rust
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}
```

구조체 자체는 공개적으로 사용할 수 있지만, 구조체 안의 항목들은 비공개이다.

```rust
impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            },
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
```

사용자는 공개 메서드인 `add`, `remove`, `average`를 통해서만 객체의 인스턴스를 수정할 수 있다.

나중에 벡터를 해시맵으로 리팩터링한다고 해도, 이 객체를 사용하는 코드는 변경할 필요가 없다.

## 상속

한 객체가 다른 객체의 정의를 상속받아서, 자식 객체는 부모 객체의 데이터와 동작을 재정의할 필요없이 가져다 쓸 수 있는 매커니즘이다.

러스트로 필드와 메서드 구현을 상속 받는 구조체를 만들 수 없다.

대신에 다른 방법으로 상속과 비슷한 행위를 할 수 있다. --

## 상속의 두 가지 장점

- 코드를 재사용할 수 있다.

러스트는 기본 트레잇 메서드 구현으로 코드를 공유할 수 있다..

- 자식 타입을 부모 타입처럼 사용할 수 있다.(다형성)

트레잇 바운드라는 제약이 있는 제네릭 타입으로 매개변수형 다형성을 구현한다.
