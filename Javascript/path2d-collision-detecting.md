# Canvas 내 다각형 점 충돌 감지

`CanvasRenderingContext2D`의 `isPointInPath`와 `isPointInStroke`를 이용하면 다각형도 쉽게 충돌감지할 수 있다.

## `isPointInPath`

점이 `Path` 안에 있는지 판별한다.

### 사용법

```js
ctx.isPointInPath(path, x, y [, fillRule]);
```

`path` 인자로 `Path2D` 인스턴스를 넘긴다.

### 예제

```js
const path = new Path2D()

path.rect(0, 0, 100, 100)

ctx.isPointInPath(path, 50, 50) === true
```

## `isPointInStroke`

점이 `Path`의 `Stroke` 위에 올려져 있는지 판별한다.

### 사용법

```js
ctx.isPointInStroke(path, x, y)
```

### 예제

```js
const path = new Path2D()

path.moveTo(10, 10)
path.lineTo(20, 20)

ctx.isPointInStroke(path, 15, 15) === true
```
