# 크롬 콘솔 유틸리티

[출처](https://developers.google.com/web/tools/chrome-devtools/console/utilities)

## `$_`

가장 최근 평가된 식의 결과를 반환함

## `$0` - `$4`

검사한 돔 요소 중 가장 최근 5개 레퍼런스

$0이 가장 최근 검사한 요소, $1이 그 다음..

## `$(selector, [startNode])`

CSS 선택자와 일치하는 첫 번째 요소 반환

`document.querySelector()`와 동일

## `$$(selector, [startNode])`

CSS 선택자외 일치하는 모든 요소를 배열로 반환

`document.querySelectorAll()`과 동일

## `$x(path, [startNode])`

XPath문에 일치하는 돔 요소 배열 반환

## `clear()`

콘솔의 히스토리 지움

## `copy(object)`

클립보드에 특정 객체 복사

## `debug(function)`

특정 함수 실행 시 디버거 실행

## `dir(object)`

특정 객체의 모든 프로퍼티를 객체 스타일로 보여줌

## `dirxml(object)`

특정 객체를 XML로 표현해서 출력함

## `inspect(object/function)`

요소나 객체를 알맞는 패널에서 보여줌

돔 요소는 Elements 패널에서, 자바스크립트 힙 객체는 Profiles 패널에서 보여줌

## `getEventListeners(object)`

특정 객체에 등록된 이벤트 리스너를 반환

## `keys(object)`, `values(object)`

특정 객체의 키나 값을 배열로 반환

## `monitor(function)`

특정 함수 호출 시 함수 이름과 전달된 인자를 콘솔에 로깅

`unmonitor(function)`으로 모니터링 취소 가능

## `monitorEvents(object[, events])`

특정 객체의 특정한 이벤트 발생 시 콘솔에 로깅

`unmonitorEvents(object[, events])` 함수로 모니터링 취소

## `profile([name])` and `profileEnd([name])`

`profile([name])`로 자바스크립트 CPU 프로파일을 시작하고 `profileEnd([name])`로 프로파일을 끝내고 Profile 패널에 결과를 보여줌

## `table(data[, columns])`

객체 데이터를 테이블 형식으로 보여줌

## `undebug(function)`

특정 함수의 디버그를 중단함
