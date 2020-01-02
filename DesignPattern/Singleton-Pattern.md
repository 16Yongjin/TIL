# 싱글턴 패턴

클래스의 인스턴스를 하나만 만들고, 어디서든 그 인스턴스에 접근할 수 있게 하기 위한 패턴

## 클래식한 구현

생성자를 `private`으로 선언해서 싱글턴에서만 인스턴스를 만들 수 있게 한다.

필요할 때만 인스턴스를 게으르게(Lazy) 생성한다.

```typescript
class Singleton {
  private static instance: Singleton;

  private constructor() {}

  public static getInstance(): Singleton {
    if (this.instance === null) {
      this.instance = new Singleton();
    }

    return this.instance;
  }
}
```

## 자바에서 생길 수 있는 문제점

자바스크립트는 싱글 쓰레드에서 작동하기에 걱정할 필요가 없지만

자바에서 클래식한 구현을 그대로 적용하면 멀티스레딩 환경에서는 쓰레드마다 다른 인스턴스가 만들어질 수 있다.

이를 해결하기 위한 3가지 방법이 있다.

### 1. `synchronized` 키워드를 추가해서 쓰레드 간 메서드 호출을 동기화하기

### 2. 인스턴스를 처음부터 만들어 버린다.

```typescript
class Singleton {
  private static eagerInstance: Singleton = new Singleton();

  private constructor() {}

  public static getInstance(): Singleton {
    return this.eagerInstance;
  }
}
```

### 3. DCL(Double-Checking Locking)을 사용해서 메서드에서 동기화되는 부분을 줄인다.

`volatile` 키워드로 멀티쓰레딩 환경 상에서 변수 초기화가 적절하게 될 수 있도록 한다.

인스턴스가 있는지 확인하고 동기화된 블럭(`synchronized` 내부)으로 들어간다.

```java
public class Singleton {
  private volatile static Singleton uniqueInstance;

  private Singleton() {}

  public static Singleton getInstance() {
    if (uniqueInstance == null) {
      synchronized(Singleton.class) {
        if (uniqueInstance == null) {
          uniqueInstance = new Singleton();
        }
      }
    }
    return uniqueInstance;
  }
}

```
