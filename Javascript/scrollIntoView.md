# `element.scrollIntoView()`

해당 요소가 화면에 나타나도록 상위 컨테이너를 스크롤한다.

## 사용법

```js
element.scrollIntoView();
element.scrollIntoView(alignToTop); // 불리언 인자
element.scrollIntoView(scrollIntoViewOptions); // 객체 인자
```

## 인자로 불리언을 넣을 경우

`true`인 경우 인자를 `{block: "start", inline: "nearest"}`로 넣는 것과 동일한데

보이는 영억의 가장 위쪽을 보여준다.

`false`인 경우 인자를 `{ block: "end", inline: "nearest"}`로 넣은 것과 동일하고

보이는 영역의 가장 아래쪽을 보여준다.

## 인자로 객체를 넣을 경우

`behavior` 속성은 트랜지션 애니메이션을 정의한다.

`auto`와 `smooth` 중 하나이고 기본값은 `auto`이다.

`block` 속성은 **수직** 정렬을 정의한다.

`start`, `center`, `end`, `nearest` 중 하나이고 기본값은 `start`이다.

`inline` 속성은 **수평** 정렬을 정의한다.

`start`, `center`, `end`, `nearest` 중 하나이고 기본값은 `nearest`이다.
