# 최장 공통 부분 문자열

최장 공통 부분 문자열 문제는 두 개 이상의 문자열에서 가장 긴 부분 문자열을 찾는 것입니다.

## 예제

`ABABC`와 `BABCA`, `ABCBA`의 최장 공통 부분 문자열은
길이가 3인 `ABC`입니다. 다른 공통 문자열은 `A`와 `AB`, `B`, `BA`, `BC`, `C`입니다.

```
ABABC
  |||
 BABCA
  |||
  ABCBA
```

## 구현

```javascript
function longestCommonSubstring(string1, string2) {
  // 유니코드 문자를 제대로 다루기 위해 문자열을 배열로 바꾼다.
  // 예를 들면
  // '𐌵'.length === 2
  // [...'𐌵'].length === 1
  const s1 = [...string1];
  const s2 = [...string2];

  // 모든 부분 문자열의 길이를 나타낼 행렬을 초기화한다.
  // 동적 프로그래밍 기법을 사용하기 위함.
  const substringMatrix = Array(s2.length + 1)
    .fill(null)
    .map(() => {
      return Array(s1.length + 1).fill(null);
    });

  // 첫 행과 첫 열을 0으로 채운다.
  for (let columnIndex = 0; columnIndex <= s1.length; columnIndex += 1) {
    substringMatrix[0][columnIndex] = 0;
  }

  for (let rowIndex = 0; rowIndex <= s2.length; rowIndex += 1) {
    substringMatrix[rowIndex][0] = 0;
  }

  // 모든 부분 문자열의 길이를 나타낼 행렬을 만든다.
  // 동적 프로그래밍 기법을 사용하기 위함.
  let longestSubstringLength = 0;
  let longestSubstringColumn = 0;
  let longestSubstringRow = 0;

  for (let rowIndex = 1; rowIndex <= s2.length; rowIndex += 1) {
    for (let columnIndex = 1; columnIndex <= s1.length; columnIndex += 1) {
      // 문자가 같은 경우
      if (s1[columnIndex - 1] === s2[rowIndex - 1]) {
        // 대각선 왼쪽 위에 있는 값 + 1
        substringMatrix[rowIndex][columnIndex] =
          substringMatrix[rowIndex - 1][columnIndex - 1] + 1;
      } else {
        // 아니면 그냥 0
        substringMatrix[rowIndex][columnIndex] = 0;
      }

      // 모든 부분 문자열 길이 중 가장 긴 것을 찾는다.
      // 그런 다음 그것의 마지막 문자 위치를 기억한다.
      if (substringMatrix[rowIndex][columnIndex] > longestSubstringLength) {
        longestSubstringLength = substringMatrix[rowIndex][columnIndex];
        longestSubstringColumn = columnIndex;
        longestSubstringRow = rowIndex;
      }
    }
  }

  if (longestSubstringLength === 0) {
    // 최장 공통 부분 문자열을 못 찾음
    return "";
  }

  // 행렬에서 최장 부분 문자열을 찾는다.
  let longestSubstring = "";

  while (substringMatrix[longestSubstringRow][longestSubstringColumn] > 0) {
    longestSubstring = s1[longestSubstringColumn - 1] + longestSubstring;
    longestSubstringRow -= 1;
    longestSubstringColumn -= 1;
  }

  return longestSubstring;
}
```

## 참고

- [Wikipedia](https://en.wikipedia.org/wiki/Longest_common_substring_problem)
- [YouTube](https://www.youtube.com/watch?v=BysNXJHzCEs&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)

## 출처

[Github](https://github.com/trekhleb/javascript-algorithms/tree/master/src/algorithms/string/longest-common-substring)
