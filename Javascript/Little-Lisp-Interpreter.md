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


## Parser 부분

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

예를 들어

```javascript
['(', '(', 'lambda', '(', 'x', ')', 'x', ')', '"Lisp"', ')']
```

`parenthesize()`가 처음 호출되면, `list` 매개변수는 `undefined`이다.

3행이 실행되고 `parenthesize()`는 빈 배열로 설정된 `list`가 빈 배열로 설정된 상태에서 재귀실행된다.



재귀에서 5행이 실행되어 `input`에서 첫 번째 여는 괄호를 제거한다.

9행이 새로운 빈 배열로 재귀해서 새로운 빈 배열을 시작한다.



재귀에서 5행이 실행되어 `input`에서 다른 여는 괄호를 제거한다.

9행은 빈 배열로 재귀해서 새로운 빈 배열을 시작한다.



재귀에서 `input`은 `['lambda', '(', 'x', ')', 'x', ')', '"Lisp"', ')']`이다.

14행은 `lambda`로 설정된 `token`과 함께 실행된다.

`categorize()`에 `lambda`를 인자로 넣고 호출한다.

`categorize()`의 7행은 `type`이 `identifier`이고 `value`가 `lambda`인 객체를 반환한다.

```javascript
var categorize = function(input) {
  if (!isNaN(parseFloat(input))) {
    return { type:'literal', value: parseFloat(input) };
  } else if (input[0] === '"' && input.slice(-1) === '"') {
    return { type:'literal', value: input.slice(1, -1) };
  } else {
    return { type:'identifier', value: input };
  }
};
```

`parenthesize()`의 14번 줄이 `categorize()`가 반환한 객체를 `list`에 추가하고 나머지 `input` 및 `list`와 함꼐 재귀한다.


```javascript
var parenthesize = function(input, list) {
  if (list === undefined) {
    return parenthesize(input, []);
  } else {
    var token = input.shift();
    if (token === undefined) {
      return list.pop();
    } else if (token === "(") {
      list.push(parenthesize(input, []));
      return parenthesize(input, list);
    } else if (token === ")") {
      return list;
    } else {
      return parenthesize(input, list.concat(categorize(token)));
    }
  }
};
```

제귀 중에, 다음 토큰은 괄호이다. 

`parenthesize()`의 9행은 빈 배열로 재귀해서 새로운 빈 배열을 시작한다.

재귀에서 `input` 은 `['x', ')', 'x', ')', '"Lisp"', ')']`이다.

14행은 `x`로 설정된 `token`과 함께 실행된다.

`type`이 `identifier`이고 `value`가 `x`인 객체를 만들어서 `list`에 추가하고 재귀한다.



재귀에서 다음 토큰은 닫는 괄호다.

12행이 실행되어 완성된 `list`를 반환한다: `[{ type: 'identifier', value: 'x' }]`.



`parenthesize()`는 모든 입력 토큰을 처리할 때까지 계속 재귀하고

마지막엔 타입이 붙여진 아톰들의 중첩된 배열을 반환한다.



`parse()`는 `tokenize()`와 `parenthesize()`를 연속해서 실행한 것이다:

```javascript
const parse = (input) => parenthesize(tokenize(input))
```

`((lambda (x) x) "Lisp")`을 입력으로 주면 이런 결과가 나온다:

```javascript
[[{ type: 'identifier', value: 'lambda' }, [{ type: 'identifier', value: 'x' }],
  { type: 'identifier', value: 'x' }],
  { type: 'literal', value: 'Lisp' }]
```

---

## Interpreter 부분

파싱(구문 분석)이 끝나면 해석이 시작된다.



`interpret()`은 `parse()`의 결과를 받아서 실행한다.

위의 파싱 예제에서 나온 결과를 가지고 생각해보면 `interpret()`은 람다를 생성해서 `"Lisp"`을 인수로 넣고 호출할 거다.

람다 호출은 프로그램의 최종 결과인 `"Lisp"`을 반환한다.



`interpret()`는 실행 입력뿐만 아니라 실행 컨텍스트를 받는다.

실행 컨텍스트는 변수와 값이 저장되는 곳이다.

Lisp 코드가 `interpret()` 에 의해 실행될 때, 실행 컨텍스트는 해당 코드에 액세스할 수 있는 변수를 가지고 있는다.



이러한 변수는 계층 구조로 저장된다.

현재 범위(스코프)의 변수는 계층의 맨 아래에 있는다. 

주변 범위의 변수는 그 위의 수준에 있는다. 

주변 범위의 주변 범위에 있는 변수는 그 위의 수준에 있는 셈이다.

예를 들어, 다음에 코드에서:

```lisp
((lambda (a)
  ((lambda (b)
      (b a))
    "b"))
 "a")
```

3행에서 실행 컨텍스트는 2개의 활성 범위가 있다. 

내부 람다는 현재 범위를 형성하고 바깥 쪽 람다는 주변 범위를 형성한다.

현재 범위는 `b` 가 `"b"`로 바인딩되어있다. 

주변 범위는 `a`에 `"a"`가 바인딩된다. 

3행이 실행되면 인터프리터는 컨텍스트에서 `b`를 찾으려고 시도한다. 현재 범위를 확인하고 `b` 찾아서 그 값을 반환한다. 여전히 3행에서 인터프리터는 `a`를 찾는다. 현재 범위를 확인하고 `a`를 못 찾아서 주변 범위에서 찾기를 시도한다. 거기서 `a`를 발견하고 그 값을 반환한다.

Little Lisp에서 실행 컨텍스트는 `Context` 생성자 호출로 만들어진 객체로 모델링된다. 현재 범위의 변수와 해당 값을 포함하는 객체인 scope 인자로 받는다. 그리고 `parent`를 받아서 `parent` 가 `undefined` 경우, 현재 범위는 최상위, 전역 또는 가장 바깥쪽이 된다.

```javascript
class Context {
  constuctor(scope, parent) {
    this.scope = scope
    this.parent = parent
  }

  get(identifier) {
    if (identifier in this.scope) {
      return this.scope[identifier]
    } else if (this.parent !== undefined) {
      return this.parent.get(identifier)
    }
  }
}
```

`((lambda (x) x) "Lisp")`가 어떻게 파싱되는지 봤으니 파싱된 코드가 실행되는 방식을 살펴보자.

```javascript
const interpret = (input, context) => {
  if (context === undefined) {
    return interpret(input, new Context(library))
  } else if (input instanceof Array) {
    return interpretList(input, context)
  } else if (input.type === "identifier") {
    return context.get(input.value)
  } else {
    return input.value
  }
}
```

`interpret()`이 처음 호출될 때 `context`는 `undefined`이다. 

2-3행은 실행 컨텍스트를 만들기 위해 실행된다.



초기 컨텍스트가 인스턴스화되면 생성자 함수는 `library` 객체를 사용한다. 여기에는 언어에 내장된 기능이 포함되어있다: `first`와 `rest`, `print`이 해당된다. 이 함수들은 자바스크립트로 작성됐다.


`interpret()`은 원래의 `input`과 새 `context`를 가지고 재귀한다.


`input`에는 파싱 부분 예제의 결과값이 들어있다:

```javascript
[[{ type: 'identifier', value: 'lambda' }, [{ type: 'identifier', value: 'x' }],
  { type: 'identifier', value: 'x' }],
  { type: 'literal', value: 'Lisp' }]
```

`input`이 배열이고 `context`가 정의되어 있으므로 4-5행이 실행되고 `interpretList()` 가 호출된다.


```javascript
const interpretList = (input, context) => {
  if (input.length > 0 && input[0].value in special) {
    return special[input[0].value](input, context)
  } else {
    const list = input.map(x => interpret(x, context))
    if (list[0] instanceof Function) {
      return list[0].apply(undefined, list.slice(1))
    } else {
      return list
   }
  }
}
```
> 여기까지 번역함

`interpretList()`에서 5행은 입력 배열을 맵핑하고 각 요소에 `interpret()`을 호출한다. 람다 정의에서 `interpretList()`가 호출되면 `interpretList()`가 다시 호출된다. 이번에 `interpretList()`에 대한 `input` 인수는 다음과 같다.

```javascript
[{ type: 'identifier', value: 'lambda' }, [{ type: 'identifier', value: 'x' }],
 { type: 'identifier', value: 'x' }]
```

배열의 첫 번째 요소 인 `lambda` 가 특수한 형식이기 때문에 `interpretList()` 3 행이 호출됩니다. `lambda` 함수를 만들기 위해 `special.lambda()` 가 호출됩니다.

```javascript
var special = {
  lambda: function(input, context) {
    return function() {
      var lambdaArguments = arguments;
      var lambdaScope = input[1].reduce(function(acc, x, i) {
        acc[x.value] = lambdaArguments[i];
        return acc;
      }, {});

      return interpret(input[2], new Context(lambdaScope, context));
    };
  }
};
```

`special.lambda()` 는 람다를 정의하는 입력 부분을 사용합니다. 호출되면 일부 인수에서 람다를 호출하는 함수를 리턴합니다.

3 행은 람다 호출 함수의 정의를 시작합니다. 4 행은 람다 호출에 전달 된 인수를 저장합니다. 5 번 줄은 람다의 새로운 호출 범위를 만들기 시작합니다. 람다의 매개 변수를 정의하는 입력 부분을 줄입니다. `[{ type: 'identifier', value: 'x' }]`. `input` 및 인수의 람다에 전달 된 각 람다 매개 변수의 람다 범위에 키 / 값 쌍을 추가합니다. 10 행은 람다 본문에서 `interpret()` 을 호출하여 람다를 호출합니다. `{ type: 'identifier', value: 'x' } { type: 'identifier', value: 'x' } { type: 'identifier', value: 'x' }` . 람다의 범위와 부모 컨텍스트를 포함하는 람다 컨텍스트를 전달합니다.

람다는 이제 `special.lambda()` 반환 한 함수로 표시됩니다.

`interpretList()` 은 목록의 두 번째 요소 인 `"Lisp"` 문자열에서 `interpret()` 을 호출하여 `input` 배열에 대한 맵핑을 계속합니다. 

```javascript
var interpret = function(input, context) {
  if (context === undefined) {
    return interpret(input, new Context(library));
  } else if (input instanceof Array) {
    return interpretList(input, context);
  } else if (input.type === "identifier") {
    return context.get(input.value);
  } else {
    return input.value;
  }
}
```

이것은 리터럴 객체 `'Lisp'` 의 `value` 속성을 반환하는 `interpret()` 9 행을 실행합니다. `interpretList()` 5 행에있는 맵 조작이 완료되었습니다. `list` 은 다음과 같습니다


```javascript
 [function(args) { /* code to invoke lambda */ },
 'Lisp']
```

`interpretList()` 6 행이 실행되고 `list` 의 첫 번째 요소가 JavaScript 함수임을 찾습니다. 이는 목록이 호출임을 의미합니다. 7 행은 람다를 실행하고 호출하여 나머지 `list` 을 인수로 전달합니다.

```javascript
var interpretList = function(input, context) {
  if (input.length > 0 && input[0].value in special) {
    return special[input[0].value](input, context);
  } else {
    var list = input.map(function(x) { return interpret(x, context); });
    if (list[0] instanceof Function) {
      return list[0].apply(undefined, list.slice(1));
    } else {
      return list;
    }
  }
};
```

람다 호출 함수에서 8 행은 람다 본문 `{ type: 'identifier', value: 'x' } { type: 'identifier', value: 'x' } { type: 'identifier', value: 'x' }`.


```javascript
function() {
  var lambdaArguments = arguments;
  var lambdaScope = input[1].reduce(function(acc, x, i) {
    acc[x.value] = lambdaArguments[i];
    return acc;
  }, {});

  return interpret(input[2], new Context(lambdaScope, context));
};

```
`interpret()` 6 행은 `input` 이 식별자 원 자라는 것을 발견합니다. 7 행은 `context` 에서 식별자 `x` 찾고 `'Lisp'`을 리턴합니다.


```javascript
var interpret = function(input, context) {
  if (context === undefined) {
    return interpret(input, new Context(library));
  } else if (input instanceof Array) {
    return interpretList(input, context);
  } else if (input.type === "identifier") {
    return context.get(input.value);
  } else {
    return input.value;
  }
}
```

`'Lisp'` 는 lambda 호출 함수에 의해 리턴되는데, `interpretList()` 함수 `interpretList()` 에 의해 리턴 됨)는 `interpret()` 의해 리턴됩니다.

모든 코드를 보려면 GitHub 저장소 로 이동하십시오. Peter Norvig가 Python으로 작성한 눈부신 간단한 Scheme 해석기 lis.py를 살펴보십시오. 