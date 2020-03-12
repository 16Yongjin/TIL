# CSS Variables

## 선언

`:root` 가짜 클래스에 변수를 선언하면 HTML 전체에서 변수를 사용할 수 있다.

```css
:root {
  --main-bg-color: brown;
}
```

## 사용

`var` 함수로 CSS 변수를 사용한다.

```css
.one {
  background-color: var(--main-bg-color);
}
```

## Fallback

`var` 함수 두 번째인자를 폴백으로 설정가능하다.

```css
.two {
  color: var(--my-var, red);
}
```

## Javascript에서 사용하기

`style`의 `getPropertyValue()` 메서드로 CSS 변수를 가져올 수 있다.

```javascript
element.style.getPropertyValue("--my-var");

// get variable from wherever
getComputedStyle(element).getPropertyValue("--my-var");
```

`style`의 `setProperty()` 메서드로 CSS 변수를 지역적으로 설정할 수 있다.

```javascript
element.style.setProperty("--my-var", jsVar + 4);
```

전역적으로 CSS 변수를 바꾸려면 접근하려면 `getComputedStyle(document.documentElement)`의 `style` 프로퍼티를 사용한다.
