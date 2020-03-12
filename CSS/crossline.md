# HTML 대각선 그리기

SVG line 태그로 그린 대각선을 요소의 배경으로 설정한다.

```html
<style>
  .slash {
    background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg"><line x1="0" y1="100%" x2="100%" y2="0" stroke="gray" /></svg>');
  }
  .backslash {
    background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg"><line x1="0" y1="0" x2="100%" y2="100%" stroke="gray" /></svg>');
  }
  table {
    border-collapse: collapse;
    border-top: 1px solid gray;
    border-left: 1px solid gray;
  }
  th,
  td {
    border-bottom: 1px solid gray;
    border-right: 1px solid gray;
    padding: 5px;
  }
</style>

<table>
  <tr>
    <th class="backslash"></th>
    <th>점수</th>
  </tr>
  <tr>
    <td>한놈</td>
    <td>10</td>
  </tr>
  <tr>
    <td class="slash">두시기</td>
    <td>20</td>
  </tr>
  <tr>
    <td>석삼</td>
    <td class="slash"></td>
  </tr>
</table>
```

[결과](https://jsfiddle.net/jmnote/9kcacm8j/?utm_source=website&utm_medium=embed&utm_campaign=9kcacm8j)

`xmlns="http://www.w3.org/2000/svg"` 속성을 추가 안 하면 svg가 그려지지 않는다.
