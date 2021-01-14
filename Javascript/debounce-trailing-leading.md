# 디바운스 함수 `trailing`과 `leading` 옵션

아래의 코드펜을 체험해보면 바로 알 수 있다.

[https://codepen.io/sumolari/pen/MWKzwgb](https://codepen.io/sumolari/pen/MWKzwgb)

`debounce` 함수는 기본적으로 `trailing`은 `true`, `leading`은 `false`이다.

`leading`은 즉시 실행 옵션이고 `trailing`은 마지막에 실행된 함수를 저장하는 옵션이다.

기본 동작은 함수가 호출돼도 가만히 있다가 시간이 지나면 함수가 실행된다.

## `_.debounce(func, 1000, { leading: true })`

함수를 실행하자마자 함수가 실행되고 이후의 함수 실행은 일정 시간 동안 무시된다.

이런 동작 때문에 `Underscore`에서는 이 옵션을 `immediate`이라고도 한다.

한편, `trailing`이 `true`이므로 무시됐던 함수가 일정 시간이 지나면 실행된다.

뒤끝 있다.

## `_.debounce(func, 1000, { leading: true, trailing: false })`

`trailing` 옵션을 껐다.

위와 같이 함수를 실행하자마자 함수가 실행되고 이후의 함수 실행은 일정 시간 동안 무시된다.

대신, 무시됐던 함수가 실행되지 않는다.

## `_.debounce(func, 1000, { leading: false, trailing: false })`

`_.debounce`가 먹통이 된다.
