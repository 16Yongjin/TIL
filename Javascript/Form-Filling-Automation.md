# 양식 채우기 자동화

## hidden 타입이 아닌 모든 인풋 태그

```javascript
$$("form input:not([type=hidden])");
```

## 입력가능한 텍스트, 비밀번호, 체크박스

```javascript
$$("form input[type=text]:not([type=hidden])");

$$("form input[type=password]:not([type=hidden])");

$$("form input[type=checkbox]:not([type=hidden])");
```

## `이름`이라는 플레이스홀더를 가진 인풋 태그

```javascript
$$(`form input[type=text][placeholder*='이름']:not([type=hidden])`);
```

## 로그인

```javascript
$$(`form input[type=text][placeholder*='이름']:not([type=hidden])`)[0].value =
  "홍길동";

$$(
  `form input[type=text][placeholder*='생년월일']:not([type=hidden])`
)[0].value = "1997.01.01";

$$(`form input[type=text][placeholder*='이메일']:not([type=hidden])`)[0].value =
  "hongil@gmail.com";

$$(`form input[type=password]:not([type=hidden])`).forEach(
  e => (e.value = "testtest1!")
);

$$("form input[type=checkbox]:not([type=hidden])")
  .map(e => e.id)
  .map(id => $$(`label[for='${id}']`)[0].click());
```

## 모든 텍스트입력 양식 키 밸류로 만들기

```javascript
$$(`form input[type=text]:not([type=hidden])`).reduce(
  (acc, v) => ({ ...acc, [v.placeholder]: v }),
  {}
);
```
