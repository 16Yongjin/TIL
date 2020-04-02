# 수평 스크롤

[Code Pen](https://codepen.io/16yongjin/pen/XWbQepp)

```scss
.horizontal-scroll-wrapper {
  direction: rtl;
  width: 250px;
  max-height: 1000px;
  overflow-y: auto;
  overflow-x: hidden;
  transform: rotate(-90deg) translateY(-$finalHeight);
  transform-origin: right top;

  & > div {
    transform: rotate(90deg);
    transform-origin: right top;
  }
}
```
