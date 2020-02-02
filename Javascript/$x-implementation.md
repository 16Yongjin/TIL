# \$x 콘솔 유틸리티 구현

```javascript
function $x(xpath) {
  const elements = document.evaluate(
    xpath,
    document,
    null,
    XPathResult.ANY_TYPE,
    null
  );

  const nodes = [];

  let node = null;
  while ((node = result.iterateNext())) {
    nodes.push(node);
  }

  return nodes;
}
```
