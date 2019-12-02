# Little Lisp interpreter

> 출처: https://maryrosecook.com/blog/post/little-lisp-interpreter

Little Lisp 인터프리터는 함수 호출과 람다, let, if, 라이브러리 함수들, 리스트를 지원한다.

자바스크립트 코드 116줄로 구현되어있다.

먼저 Lisp이 뭔지부터 알아본다.

가장 단순한 아톰(Atom)인 숫자:
```lisp
1
```

문자열도 아톰이다:
```lisp
"a"
```
빈 리스트:
```lisp
()
```

아톰이 들어있는 리스트:
```lisp
(1)
```

아톰이 2개 들어있는 리스트:
```lisp
(1 2)
```

아톰과 리스트가 들어있는 리스트:
```lisp
(1 (2))
```

함수 호출

첫 번째 요소가 함수, 나머지 요소는 인수인 리스트로 구성된다.

`first`는 인자 `(1, 2)`를 받아서 첫 번째 요소인 `1`을 반환한다.

```lisp
(first (1 2))
  => 1
```
람다, 함수 정의

인자로 `x`를 받아서 그대로 반환한다.

```lisp
(lambda (x)
  x)
```
람다 호출

첫 번째 요소가 람다, 나머지 요소가 인자인 리스트로 구성된다.

람다가 인자로 `"Lisp"`을 받아서 그대로 반환한다.

```lisp
((lambda (x)
   x)
  "Lisp")

  => "Lisp"
```

---
### Little Lisp 두 부분:
1. 파서
2. 인터프리터


## Parser

파싱의 두 단계: 
1. 토크나이징
2. 괄호로 묶기

`tokenize()` 함수는 Lisp 코드를 받아서 모든 괄호 주변에 공백을 넣고 공백을 기준으로 나눈다. 

예를 들어, 

`((lambda (x) x) "Lisp")`를 받으면 

` ( ( lambda ( x ) x ) “Lisp” ) `으로 변환한 뒤 (괄호 주변에 공백 넣기)

`['(', '(', 'lambda', '(', 'x', ')', 'x', ')', '"Lisp"', ')']`으로 변환한다. (공백 기준으로 나누기)

```javascript
const tokenize = input => 
  input.replace(/\(/g, ' ( ')
    .replace(/\)/g, ' ) ')
    .trim()
    .split(/\s+/)
```

`parenthesize()` 함수는 `tokenize()`가 생성한 토큰들을 받아서 Lisp 코드의 구조를 따르는 중첩 배열을 생성한다.

예를 들어, `['(', '(', 'lambda', '(', 'x', ')', 'x', ')', '"Lisp"', ')']`은 

```javascript
[[{ type: 'identifier', value: 'lambda' }, [{ type: 'identifier', value: 'x' }],
  { type: 'identifier', value: 'x' }],
  { type: 'literal', value: 'Lisp' }]
```

으로 변환된다.

`parenthesize()`는 토큰을 하나씩 보면서 현재 토큰이 여는 괄호면 새 배열을 만든다. 

현재 토큰이 아톰인 경우 해당 타입으로 이름 붙이고 현재 배열에 추가한다. 

현재 토큰이 닫는 괄호면 현재 배열 작성을 중지하고 바깥 배열 작성을 계속한다.

```javascript
const parenthesize = (input, list) => {
  if (list === undefined) {
    return parenthesize(input, []);
  } else {
    var token = input.shift();
    if (token === undefined) {
      return list.pop()
    } else if (token === "(") {
      list.push(parenthesize(input, []))
      return parenthesize(input, list)
    } else if (token === ")") {
      return list
    } else {
      return parenthesize(input, list.concat(categorize(token)))
    }
  }
};
```

`parenthesize()`가 처음 호출되면 `input` 매개변수에는 `tokenize()`가 반환한 토큰 배열이 포함된다. 

예를 들면

```javascript
['(', '(', 'lambda', '(', 'x', ')', 'x', ')', '"Lisp"', ')']
```