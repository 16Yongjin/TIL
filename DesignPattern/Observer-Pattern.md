# 옵저버 패턴

한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들한테 연락이 가고 자동으로 내용이 갱신되는 방식으로 일대다 의존성을 정의한다.

## 클래스 다이어그램

![class-diagram](https://user-images.githubusercontent.com/22253556/71555172-4e7ef880-2a6c-11ea-8bc7-4f674a2e71bc.png)

## 느슨한 결합

두 객체가 느슨하게 결합되어 있다는 것은, 그 둘이 상호작용을 하긴 하지만 서로에 대해 서로 잘 모른다는 것을 의미한다.

옵저버 패턴에서는 서브젝트(옵저버블)와 옵저버가 느슨하게 결합되어 있는 객체 디자인을 제공한다.

- 서브젝트는 옵저버가 특정 인터페이스만 구현한다는 것만 안다.
- 옵저버를 추가하려 할 때도 서브젝트는 변경할 필요가 없다.

## 구현

뉴스센터와 구독자를 구현해본다.

옵저버를 추가, 삭제하고 옵저버에게 정보를 전달하는 메서드를 가진 `Subject` 인터페이스

```typescript
interface Subject {
  registerObserver(observer: Observer): void;
  removeObserver(observer: Observer): void;
  notifyObservers(): void;
}
```

옵저버 인터페이스, 구독자들은 뉴스를 받을 `update()` 메서드를 구현해야 한다.

```typescript
interface Observer {
  update(news: String): void;
}
```

`Subject` 인터페이스를 구현하는 `NewsCenter`

```typescript
class NewsCenter implements Subject {
  observers: Observer[] = [];
  news = "";

  registerObserver(observer: Observer) {
    this.observers = [...this.observers, observer];
  }

  removeObserver(observer: Observer) {
    this.observers = this.observers.filter(o => o !== observer);
  }

  notifyObservers() {
    this.observers.forEach(observer => {
      observer.update(this.news);
    });
  }

  setNews(news: string) {
    this.news = news;
    console.log(`새로운 뉴스: ${this.news}`);

    // 새로운 뉴스가 생기면 구독자에게 알려준다.
    notifyObservers();
  }
}
```

`Observer`를 구현하는 `Subscriber` 클래스

```typescript
class Subscriber implements Observer {
  name: string;
  news: string;

  constructor(name: string) {
    this.name = name;
    this.news = "";
  }

  update(news: string) {
    this.news = news;
    console.log(`${this.name}이 읽은 뉴스: ${this.news}`);
  }
}
```

## 테스트 코드

뉴스센터에 구독자 3명을 등록하고 뉴스를 전달한다.

```typescript
const newsCenter = new NewsCenter();

const s1 = new Subscriber("Samuel");
const s2 = new Subscriber("John");
const s3 = new Subscriber("Alice");

newsCenter.registerObserver(s1);
newsCenter.registerObserver(s2);
newsCenter.registerObserver(s3);

newsCenter.setNews("탈모 치료제 개발");
```

결과, 뉴스센터에 뉴스를 등록했는데 구독자들도 그 뉴스를 읽게 됐다.

```
새로운 뉴스: 탈모 치료제 개발
Samuel이 읽은 뉴스: 탈모 치료제 개발
John이 읽은 뉴스: 탈모 치료제 개발
Alice이 읽은 뉴스: 탈모 치료제 개발
```

## 응용

`Vue`, `RxJS`, `Flutter`, `React`에서 옵저버 패턴이 넓게 쓰인다.

`Vue`에서는 컴포넌트나 스토어 내 데이터 값을 변경하고 `React`와 `Flutter`에서는 `setState`에 변경되는 값을 넣고 메서드를 호출하면 그 값에 의존하는 UI가 변경된다.

`RxJS`에서는 반응형 프로그래밍을 사용해 복잡한 비동기 로직을 구현할 때 유용하다.
