# 이터레이터 패턴

컬랙션의 세부사항을 몰라도 그 안에 들어있는 모든 항목에 접근하는 방법을 제공한다.

리스트, 배열, 해시맵 등 어떤 종류의 컬랙션이든 `Iterator` 인터페이스만 구현했다면 간단하게 내부 항목을 순회할 수 있다.

## `Iterator` 인터페이스

```java
public interface Iterator {
  boolean hasNext();
  Object next();
}
```

`hasNext()` 메서드에서 다음 항목이 있는지 확인한다.

`next()` 메섣드는 다음 항목을 반환한다.

`Iterator` 인터페이스를 직접 구현하는 것보단 `java.util.Iterator` 인터페이스를 가져다 쓰는 게 일반적이다.

## 이터레이터 사용

아래와 같이 `while`문과 함깨 사용한다.

```java
while (iterator.hasNext()) {
  process(iterator.next())
}
```

사실 `for(변수 : 컬랙션)`문을 사용하면 반복자를 만들지 않아도 컬랙션을 순회할 수 있다

```java
for (Item item: items) {
  process(item)
}
```

## 자바스크립트의 이터레이터

객채에 `[Symbol.iterator]()`에 이터레이터 프로토콜(약속, 이렇게만 해주면 알아서 해줄게)을 구현하면 이터레이터다.

이터레이터 프로토콜만 맞추면 `for..of`문이나 전개 연산자를 사용할 수 있다.

## 이터레이터 프로토콜 맞추기

이터레이터 프로토콜은 `next()` 메서드를 가지고, 그 결과는 `{ value: 값, done: true/false }`여야 한다.

결과 객체의 `value` 속성은 다음 항목의 값이고 `done` 속성은 컬랙션을 다 돌았으면 `true` 값을 갖는다.

## 간단한 이터레이터 구현

python의 `range` 함수처럼 `start`와 `end`, `step` 값을 주면 이터레이터를 만드는 함수이다.

```javascript
function makeRangeIterator(start = 0, end = Infinity, step = 1) {
  let nextIndex = start;
  let iterationCount = 0;

  const rangeIterator = {
    next: function() {
      let result;
      if (nextIndex < end) {
        // 아직 반복이 안 끝나서 done이 false임
        result = { value: nextIndex, done: false };
        nextIndex += step;
        iterationCount++;
        return result;
      }
      // 반복 끝!
      return { value: iterationCount, done: true };
    }
  };
  return rangeIterator;
}
```

사용할 때는 아래와 같이 `while`문을 사용하거나 `for..in`, 전개연산자를 사용해도 된다.

```javascript
let it = makeRangeIterator(1, 10, 2);

let result = it.next();
while (!result.done) {
  console.log(result.value); // 1 3 5 7 9
  result = it.next();
}
```

이터레이터는 내부 상태를 가지고 있어서 이터레이터를 만들 땐 신경 쓸 게 많다.

그래서 제너레이터 함수를 사용하면 코드가 더 간단하고 편하다.

## 제너레이터 함수

제너레이터 함수는 연속해서 실행이 되지 않는 특성이 있다.

제너레이터 함수가 제너레이터라는 이터레이터를 반환한다.

제너레이터의 `next()` 메서드가 호출되어 값이 사용될 때마다 제너레이터 함수는 `yield` 키워드를 만날 때까지 코드를 실행한다.

위와 똑같은 함수를 제너레이터를 사용해서 만들 수 있다.

```javascript
function* makeRangeIterator(start = 0, end = 100, step = 1) {
  let iterationCount = 0;
  for (let i = start; i < end; i += step) {
    iterationCount++;
    yield i;
  }
  return iterationCount;
}
```

## 내장 이터러블

`String`, `Array`, `TypedArray`, `Map`, `Set`가 내장 이터러블이다.

모두 `prototype` 객체에 `Symbol.iterator` 메서드가 들어있다.

## 기타

### 배열을 제너레이터 함수로 반환하기

`yield*` 키워드를 사용한다.

```javascript
function* gen() {
  yield* ["a", "b", "c"];
}
```

### `Number` 객체에 이터레이터 집어넣기

아래와 같이 `Number.prototype[Symbol.iterator]`를 구현하고

```javascript
Number.prototype[Symbol.iterator] = function*() {
  for (var i = 0; i < this; i++) {
    yield i;
  }
};
```

아래와 같이 실행하면 `[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]`가 반환된다.

```javascript
console.log([...10]);
```

근데 `Number` 같은 내장 전역 객체를 수정하는 행위는 금지사항이라 알고리즘 문제 풀 때는 사용해도 실무에서 사용하면 안된다.
