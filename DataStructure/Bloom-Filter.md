# Bloom Filter

**블룸 필터**는 원소가 집합에 속하는지 여부를 검사하는데 사용되는 공간 효율적인 확률적 자료 구조입니다. 긍정 오류가 나올 수 있는 대신 매우 빠르고 메모리를 최소화하도록 설계되었습니다. 대신에 부정 오류는 절대 발생하지 않습니다. 즉, 쿼리를 하면 "집합에 있을 수도 있다" 또는 "절재로 집합에 없다"를 반환합니다.

"기존의" 오류 없는 해싱 기법을 적용했을 경우 메모리 사용량이 과한 프로그램을 위해 만들어진 자료구조입니다.

## 알고리즘 설명

빈 Bloom 필터는 `0`으로 채워진 `m` 비트의 비트 배열입니다.
`k`개의 다른 해시 함수들이 있고, 각각의 함수는 특정 집합 요소를 `m`개의 배열 위치 중 하나에 위치시킵니다.
일반적으로 `k`는 `m`보다 훨씬 작은 상수이고, `m`은 추가할 원소 수에 비례합니다.
`k`의 정확한 값과 `m`의 비례 상수는 필터의 긍정 오류율 설정에 의해 결정됩니다.

다음은 집합 {x, y, z} 나타내는 블룸 필터의 예입니다. 컬러 화살표는 각 집합 원소가 매핑된 비트 배열의 위치를 ​​나타냅니다.
원소 `w`는 `0`이 포함된 1 비트 배열 위치로 해시되므로 {x, y, z} 세트에 없습니다.
이 그림의 경우 `m = 18` 및 `k = 3` 입니다.

![Bloom Filter](https://upload.wikimedia.org/wikipedia/commons/a/ac/Bloom_filter.svg)

## 연산

_삽입_ 과 _검색_ 이 블룸 필터가 수행하는 주요 연산입니다.
검색하면 긍정 오류가 발생할 수 있고 삭제가 불가능합니다.

다르게 말하면, 필터가 원소를 가져올 수 있습니다.
원소가 이전에 삽입되었는지 확왼 시 "아니오"또는 "아마도"를 알려줍니다.

삽입과 검색은 모두 `O(1)` 연산입니다.

## 필터 만들기

블룸 필터는 특정 크기를 할당하여 생성됩니다.
이 예에서는 기본 길이로 `100`을 사용합니다.
모든 위치는 `false`로 초기화됩니다.

### 삽입

삽입 시, 입력을 해시하는데 해시 함수 여러 개(예시에선 3개)가 사용됩니다. 이 해시 함수들은 인덱스을 출력합니다.
출력된 모든 인덱스의 블룸 필터의 값을 `true`로 변경하면 됩니다.

### 검색

검색 시, 같은 해시 함수들이 입력을 해시하는 데 사용됩니다.
그런 다음 출력된 인덱스가 _모두_ 블룸 필터 내에서 `true` 값을 갖는지 확인합니다. _모두_ `true` 값을 가지면 블룸 필터에 이전에 삽입된 값일 수 있다는 것을 알게됩니다.

그러나 이전에 삽입된 다른 값이 값을 `true` 뒤집었을 가능성이 있기 때문에 확실하지 않습니다.
현재 검색중인 원소 때문에 값이 반드시 `true`인 것이 아닙니다.
원소가 1개만 삽입되있지 않은 이상 절대적으로 확실하지 않습니다.

해시 함수에 의해 반환 된 인덱스에 대한 블룸 필터를 검사하는 동안 그 중 하나라도 false 값을 갖는 경우 항목이 이전에 삽입되지 않았 음을 확실히 알 수 있습니다.

## 긍정 오류

긍정 오류 확률은 블룸 필터의 크기, 사용하는 해시 함수의 수 및 필터에 삽입된 항목 수, 이 세 가지 요소에 의해 결정됩니다.

긍정 오류 확률을 계산하는 공식은 다음과 같습니다.

( 1 - e <sup>-kn/m</sup> ) <sup>k</sup>

`k` = 해시 함수의 수

`m` = 필터 크기

`n` = 삽입된 항목 수

이러한 변수 `k` , `m` 및 `n`은 얼마나 긍정 오류를 허용할 수 있는지에 따라 선택해야합니다.
값이 선택됐는데 결과 확률이 너무 높으면 값을 조정하고 확률을 다시 계산해야합니다.

## 응용

블로깅 필터는 블로그 웹 사이트에서 사용할 수 있습니다.
독자에게 이전에 본 적이없는 기사만 표시하는 것이 목표라면 블룸 필터가 완벽합니다.
기사를 기반으로 해시 값을 저장할 수 있습니다.
사용자가 몇 개의 기사를 읽은 후 기사를 필터에 삽입 할 수 있습니다.
다음에 사용자가 사이트를 방문하면 해당 기사를 결과에서 필터링 할 수 있습니다.

실수로 일부 기사가 필연적으로 필터링되지만 비용은 허용되는 수준입니다.
사용자가 사이트를 방문할 때마다 볼 수 있는 새 기사가 있는 한 기사를 몇 개 정도 보지 않아도 괜찮습니다.

## 구현

```javascript
class BloomFilter {
  /**
   * @param {number} size - 저장공간 크기
   */
  constructor(size = 100) {
    // 블룸 필터의 크기는 긍정 오류 확률에 직접적인 영향을 미친다.
    // 크기가 클수록 긍정 오류일 확률이 낮아진다.
    this.size = size;
    this.storage = this.createStore(size);
  }

  /**
   * @param {string} item
   */
  insert(item) {
    const hashValues = this.getHashValues(item);

    // 각 hashValue 인덱스를 true로 설정한다.
    hashValues.forEach(val => this.storage.setValue(val));
  }

  /**
   * @param {string} item
   * @return {boolean}
   */
  mayContain(item) {
    const hashValues = this.getHashValues(item);

    for (let hashIndex = 0; hashIndex < hashValues.length; hashIndex += 1) {
      if (!this.storage.getValue(hashValues[hashIndex])) {
        // item이 분명히 삽입된 적 없다.
        return false;
      }
    }

    // item이 삽입됐거나 안 됐다.
    return true;
  }
  /**
   * 필터가 사용할 데이터 저장소를 만든다.
   * 데이터를 캡슐화해서 필요한 메서드만 데이터에 접근할 수 있게한다.
   *
   * @param {number} size
   * @return {Object}
   */
  createStore(size) {
    const storage = [];

    // 모든 인덱스를 false로 초기화한다
    for (
      let storageCellIndex = 0;
      storageCellIndex < size;
      storageCellIndex += 1
    ) {
      storage.push(false);
    }

    const storageInterface = {
      getValue(index) {
        return storage[index];
      },
      setValue(index) {
        storage[index] = true;
      }
    };

    return storageInterface;
  }

  /**
   * @param {string} item
   * @return {number}
   */
  hash1(item) {
    let hash = 0;

    for (let charIndex = 0; charIndex < item.length; charIndex += 1) {
      const char = item.charCodeAt(charIndex);
      hash = (hash << 5) + hash + char;
      hash &= hash; // 32비트 정수로 변환한다.
      hash = Math.abs(hash);
    }

    return hash % this.size;
  }

  /**
   * @param {string} item
   * @return {number}
   */
  hash2(item) {
    let hash = 5381;

    for (let charIndex = 0; charIndex < item.length; charIndex += 1) {
      const char = item.charCodeAt(charIndex);
      hash = (hash << 5) + hash + char; /* hash * 33 + c */
    }

    return Math.abs(hash % this.size);
  }

  /**
   * @param {string} item
   * @return {number}
   */
  hash3(item) {
    let hash = 0;

    for (let charIndex = 0; charIndex < item.length; charIndex += 1) {
      const char = item.charCodeAt(charIndex);
      hash = (hash << 5) - hash;
      hash += char;
      hash &= hash; // 32비트 정수로 변환한다.
    }

    return Math.abs(hash % this.size);
  }

  /**
   * 입력에 해시 함수 3개를 다 적용하고 결과를 배열로 반환한다.
   *
   * @param {string} item
   * @return {number[]}
   */
  getHashValues(item) {
    return [this.hash1(item), this.hash2(item), this.hash3(item)];
  }
}
```

## 참고

- [Wikipedia](https://en.wikipedia.org/wiki/Bloom_filter)
- [Bloom Filters by Example](http://llimllib.github.io/bloomfilter-tutorial/)
- [Calculating False Positive Probability](https://hur.st/bloomfilter/?n=4&p=&m=18&k=3)
- [Bloom Filters on Medium](https://blog.medium.com/what-are-bloom-filters-1ec2a50c68ff)
- [Bloom Filters on YouTube](https://www.youtube.com/watch?v=bEmBh1HtYrw)

## 출처

[Github](https://github.com/trekhleb/javascript-algorithms/tree/master/src/data-structures/bloom-filter)
