# 시스템 다크 모드 확인하기

## 다크 모드 확인

```js
if (
  window.matchMedia &&
  window.matchMedia("(prefers-color-scheme: dark)").matches
) {
  // dark mode
}
```

## 다크 모드 변경 확인

```js
window
  .matchMedia("(prefers-color-scheme: dark)")
  .addEventListener("change", (e) => {
    const newColorScheme = e.matches ? "dark" : "light";
  });
```
