# Knuth–Morris–Pratt 알고리즘

중복 비교를 건너뛰어 문자열 속 단어를 빠르게 찾을 수 있는 KMP 알고리즘

Knuth–Morris–Pratt 문자열 검색 알고리즘(또는 KMP 알고리즘)은
기본 "텍스트 문자열" `T`내에서 "워드" `W`의 발생을 검색합니다.
불일치가 발생할 때, 다음 일치가 어디서 시작할지에 대한 충분한 정보를 갖는다는 점을 알기 때문입니다.
그 덕분에 이전에 일치한 문자를 다시 검사하지 않아도 됩니다.

## 복잡도

- **시간:** `O(|W| + |T|)` (`O(|W| * |T|)`보다 훨씬 빠름)
- **공간:** `O(|W|)`

## 구현

```javascript
/**
 * @param {string} word
 * @return {number[]}
 */
function buildPatternTable(word) {
  const patternTable = [0];
  let prefixIndex = 0;
  let suffixIndex = 1;

  while (suffixIndex < word.length) {
    if (word[prefixIndex] === word[suffixIndex]) {
      patternTable[suffixIndex] = prefixIndex + 1;
      suffixIndex += 1;
      prefixIndex += 1;
    } else if (prefixIndex === 0) {
      patternTable[suffixIndex] = 0;
      suffixIndex += 1;
    } else {
      // 접두사와 접미사가 더 이상 일치하지 않을 때
      // word[0, prefixIndex - 1]가 가장 긴 접두사 겸 접미사임을 안다.
      // patternTable[prefixIndex - 1]를 통해 다음으로 긴 접미사를 찾는다.
      // 접두사와 접미사 비교 시 중복을 피하기 위함이다.

      prefixIndex = patternTable[prefixIndex - 1];
    }
  }

  return patternTable;
}

/**
 * @param {string} text
 * @param {string} word
 * @return {number}
 */
function knuthMorrisPratt(text, word) {
  if (word.length === 0) {
    return 0;
  }

  let textIndex = 0;
  let wordIndex = 0;

  const patternTable = buildPatternTable(word);

  while (textIndex < text.length) {
    if (text[textIndex] === word[wordIndex]) {
      // We've found a match.
      if (wordIndex === word.length - 1) {
        return textIndex - word.length + 1;
      }
      wordIndex += 1;
      textIndex += 1;
    } else if (wordIndex > 0) {
      wordIndex = patternTable[wordIndex - 1];
    } else {
      wordIndex = 0;
      textIndex += 1;
    }
  }

  return -1;
}
```

## 참고

- [Wikipedia](https://en.wikipedia.org/wiki/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm)
- [YouTube](https://www.youtube.com/watch?v=GTJr8OvyEVQ&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)

## 출처

[Github](https://github.com/trekhleb/javascript-algorithms/tree/master/src/algorithms/string/knuth-morris-pratt)
