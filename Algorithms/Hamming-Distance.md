# Hamming Distance

길이가 같은 문자열 두 개 사이의 해밍 거리는 해당 심볼이 다른 위치의 개수입니다.
즉, 한 문자열을 다른 문자열로 변경하는데 필요한 최소 대체 개수 또는 한 문자열을 다른 문자열로 변환할 수있는 최소 오류수를 측정합니다.
보다 일반적인 의미로 해밍 거리는 두 시퀀스 사이의 편집 거리를 측정하기위한 여러 문자열 메트릭 중 하나입니다.

## 예제

- "ka**rol**in"과 "ka**thr**in"의 해밍거리는 **3**.
- "k**a**r**ol**in"과 "k**e**r**st**in"의 해밍거리는 **3**.
- 10**1**1**1**01과 10**0**1**0**01의 해밍거리는 **2**.
- 2**17**3**8**96과 2**23**3**7**96의 해밍거리는 **3**.

## 구현

```javascript
/**
 * @param {string} a
 * @param {string} b
 * @return {number}
 */
function hammingDistance(a, b) {
  if (a.length !== b.length) {
    throw new Error("Strings must be of the same length");
  }

  let distance = 0;

  for (let i = 0; i < a.length; i += 1) {
    if (a[i] !== b[i]) {
      distance += 1;
    }
  }

  return distance;
}
```

## 참고

[Wikipedia](https://en.wikipedia.org/wiki/Hamming_distance)

## 출처

[Github](https://github.com/trekhleb/javascript-algorithms/tree/master/src/algorithms/string/hamming-distance)
