# Longest common subsequence

최장 공통 부분 수열(LCS) 알고리즘은 시퀀스 여러 개(보통 2개)에서 모든 시퀀스에 공통적으로 최장 부분 수열을 찾는 알고리즘입니다.
가장 긴 공통 하위 문자열 알고리즘과 다릅니다.
하위 문자열과 달리 부분 수열은 원래 수열 내에서 문자가 연속적으로 있을 필요가 없습니다.

## 응용

최장 공통 부분 수열 문제는 고전적인 컴퓨터 과학 문제입니다.
`diff` 유틸리티와 같은 데이터 비교 프로그램의 기초이며 생물 정보학에 응용됩니다.
또한 Git 같은 버전 관리 시스템에서 파일에 대한 여러 변경 사항을 조정하기 위해서도 사용됩니다.

## Example

- 입력 수열 `ABCDGH`와 `AEDFHR`의 LCS는 `ADH`이고 길이는 3이다.
- 입력 수열 `AGGTAB`와 `GXTXAYB`의 LCS는 `GTAB`이고 길이는 4이다.

## 구현

```javascript
/**
 * @param {string[]} set1
 * @param {string[]} set2
 * @return {string[]}
 */
function longestCommonSubsequence(set1, set2) {
  // LCS 행렬을 초기화한다. (set1 길이 by set2 길이 행렬)
  const lcsMatrix = Array(set2.length + 1)
    .fill(null)
    .map(() => Array(set1.length + 1).fill(null));

  // 첫 번째 열을 0으로 채운다.
  for (let columnIndex = 0; columnIndex <= set1.length; columnIndex += 1) {
    lcsMatrix[0][columnIndex] = 0;
  }

  // 첫 번째 행을 0으로 채운다.
  for (let rowIndex = 0; rowIndex <= set2.length; rowIndex += 1) {
    lcsMatrix[rowIndex][0] = 0;
  }

  // 두 문자열이 일치하는 행을 채운다.
  for (let rowIndex = 1; rowIndex <= set2.length; rowIndex += 1) {
    for (let columnIndex = 1; columnIndex <= set1.length; columnIndex += 1) {
      // 문자가 같으면
      if (set1[columnIndex - 1] === set2[rowIndex - 1]) {
        // 대각선으로 왼쪽 위에 있는 값 + 1 한 값을 저장한다.
        lcsMatrix[rowIndex][columnIndex] =
          lcsMatrix[rowIndex - 1][columnIndex - 1] + 1;
      } else {
        // 문자가 다르면 왼쪽이나 위에 있는 값 중 큰 값을 저장한다.
        lcsMatrix[rowIndex][columnIndex] = Math.max(
          lcsMatrix[rowIndex - 1][columnIndex],
          lcsMatrix[rowIndex][columnIndex - 1]
        );
      }
    }
  }

  // LCS 행렬을 보면서 LCS를 만든다.
  if (!lcsMatrix[set2.length][set1.length]) {
    // 가장 긴 공통 문자열의 길이가 0이면 빈 문자열을 리턴한다.
    return [""];
  }

  const longestSequence = [];
  let columnIndex = set1.length;
  let rowIndex = set2.length;

  while (columnIndex > 0 || rowIndex > 0) {
    // 왼쪽과 위쪽의 문자가 같다면
    if (set1[columnIndex - 1] === set2[rowIndex - 1]) {
      // 대각선으로 왼쪽 위로 감
      longestSequence.unshift(set1[columnIndex - 1]);
      columnIndex -= 1;
      rowIndex -= 1;
    } else if (
      lcsMatrix[rowIndex][columnIndex] === lcsMatrix[rowIndex][columnIndex - 1]
    ) {
      // 왼쪽으로
      columnIndex -= 1;
    } else {
      // 위쪽으로
      rowIndex -= 1;
    }
  }

  return longestSequence;
}
```

## References

- [Wikipedia](https://en.wikipedia.org/wiki/Longest_common_subsequence_problem)
- [YouTube](https://www.youtube.com/watch?v=NnD96abizww&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)
