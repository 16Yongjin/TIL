# 스와이프 벽돌깨기 클론

## 1. 마우스 위치에 따라 공 진행 방향 점선으로 표시

### 마우스 이벤트

```js
function handleMouseMove(evt) {
  const canvasScale = Canvas2D.scale;
  const canvasOffset = Canvas2D.offset;
  const mx = (evt.pageX - canvasOffset.x) / canvasScale.x;
  const my = (evt.pageY - canvasOffset.y) / canvasScale.y;
  Mouse._position = new Vector2(mx, my);
}
```

### 진행 방향 각도 구하기

```js
const opposite = Mouse.position.y - this.position.y;
const adjacent = Mouse.position.x - this.position.x;
this.rotation = Math.atan2(opposite, adjacent);
```

### 점선 그리기

```js
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

// Dashed line
ctx.beginPath();
ctx.setLineDash([5, 15]);
ctx.moveTo(0, 50);
ctx.lineTo(300, 50);
ctx.stroke();
```

## 2. 진행 각도에 맞게 공 발사

### 공 각도에 맞는 힘 벡터 계산

```js
Ball.prototype.shoot = function (angle) {
  this.moving = true;

  this.velocity = calculateBallVelocity(angle);
};

const calculateBallVelocity = (angle) =>
  new Vector2(100 * Math.cos(angle) * POWER, 100 * Math.sin(angle) * POWER);
```

### 벽돌 충돌 탐지
