# 군대에서 만든 게임들

부대에 있을 때 코로나19 때문에 병사들이 휴가도 못 나가고 일도 없어서 지루한 시간을 보내고 있었다.

모두가 힘든 상황에서 상황병으로서 뭘 할 수 있을까 고민하다가 자바스크립트로 게임을 만들기 시작했고

게임 공장처럼 평균 2주에 한 개씩 찍어낸 게임이

- 스와이프 벽돌 깨기
- 2048
- 장기
- 오목
- 테트리스
- 미로 찾기
- 스네이크
- 스택

으로 총 8개에 트렐로 클론까지 만들었다.

덕분에 선후임들은 신나게 일과시간을 녹일 수 있었다.

## 개발환경

- 크롬 브라우저 + 메모장

- Google, Stack Overflow 사용 불가

- NPM 라이브러리 사용 불가

- IDE 사용 불가

## 제한사항

게시판에 파일 하나만 업로드할 수 있어서 모든 코드와 자원은 `html` 파일 하나에 들어가야 했다.

> 생각이 있었다면, 인트라넷에 있는 파이썬으로 css, js을 html 파일 하나로 모아주는 번들러를 만들었을 것 같다.

&nbsp;

# 게임 소개

## [스와이프 벽돌 깨기](https://html-games.surge.sh/brick-breaker)

&nbsp;

<img alt="sbb" src="https://user-images.githubusercontent.com/22253556/81933996-2cf06900-9629-11ea-9b4e-31cee7502d29.png " width="400px"/>

### 자바스크립트 버전은 세상에서 유일한 것 같다.

- 있었으면 그냥 배꼈을 텐데, 없어서 게임을 백번 씩 플레이 해보고 코드, 로직 하나하나 직접 만들었다.

### 공 발사는 예전에 만들었던 당구 게임에서 착안했다.

- 공 발사 각도는 두 점으로 각도를 구하는 아크 탄젠트(`Math.atan2`) 함수를 사용해서 구했다.

- 그 뒤로 공 벽 반사, 공과 벽돌 충돌 감지, 보너스 공, 벽돌과 바닥 충돌 감지 등의 기능을 하나씩 추가하다보니 완성했다.

### 객체 지향으로 코드 구조화

![SBB-diagram](https://user-images.githubusercontent.com/22253556/90336638-1ab30c00-e018-11ea-9dde-f204c7c1cc08.png)

- 명령형으로 작성한 코드가 길어질 수록 이해하기 힘들고 스크롤만 위아래로 왔다갔다하는 일이 많아졌다.

- 그래서 기능별로 코드를 쪼개고 계층화하고 권한을 위임했다.

- `Ball`들은 `Balls` 클래스, `Brick`들은 `Bricks` 클래스에 들어있고

- 각 요소 간 상호작용과 렌더링은 `GameManager` 클래스에서 담당한다.

> 객체 지향으로 구조를 잡고 함수형으로 메서드를 구현하는 방식이 나름 합리적인 개발 방식인 것 같다.

### 이외에

- 파티클 시스템으로 `Brick`이 깨졌을 때 타격감 있는 이펙트를 제공한다.

- 오리지날에 없는 공 내리기 기능으로 게임 진행 속도를 높였다.

- 후임이 당직 때 10시간 동안 플레이해서 997점 달성했다. (내가 본 최고점수)

&nbsp;

## [장기](https://html-games.surge.sh/janggi)

&nbsp;

<img alt="janggji" src="https://user-images.githubusercontent.com/22253556/81934129-645f1580-9629-11ea-89b9-5bdf53918eb2.png" width="400px"/>

### 말의 경로를 **함수형 프로그래밍**으로 모델링했다.

- `for` 문과 변수가 없다.

- 코드도 짧아서 경로 계산만 100줄 내외

  <details>
    <summary>경로 계산 코드</summary>

```js

calcPath () {
    const canForward = ([x, y]) => !table[y][x] || (table[y][x].team !== this.team && 'stop');
    const isEmptyPlace = ([x, y]) => !table[y][x];
    const isEnemy = ([x, y]) => table[y][x] && table[y][x].team !== this.team;
    const isEmptyOrEnemy = path => isEmptyPlace(path) || isEnemy(path);
    const toAbsoultePos = ([x, y]) => [this.x + x, this.y + y];

    if (this.text === '兵') {
        const paths = [
            [[-1, 0], [1, 0], [0, 1]].map(toAbsoultePos).filter(isInBoard).filter(canForward),
            isIn([[3, 7], [5, 7]])(this.pos) ? [[4, 8]].filter(canForward) : [],
            this.posIs([4, 8]) ? [[3, 9], [5, 9]].filter(canForward) : [],
        ].flat()

        return paths
    }

    if (this.text === '卒') {
        const paths = [
            [[-1, 0], [1, 0], [0, -1]].map(toAbsoultePos).filter(isInBoard).filter(canForward),
            isIn([[3, 2], [5, 2]])(this.pos) ? [[4, 1]].filter(canForward) : [],
            this.posIs([4, 1]) ? [[3, 0], [5, 0]].filter(canForward) : [],
        ].flat()

        return paths
    }

    else if (this.text === '漢' || this.text === '楚' || this.text === '士') {
        const isInPalace = this.team === 'red' ?
            ([x, y]) => 3 <= x && x <= 5 && 0 <= y && y <= 2 :
            ([x, y]) => 3 <= x && x <= 5 && 7 <= y && y <= 9;

        const isPalaceSideMiddle = this.team === 'red' ?
            ([x, y]) => (y === 1 && (x === 3 || x === 5)) || (x === 4 && (y === 0 || y === 2)) :
            ([x, y]) => (y === 8 && (x === 3 || x === 5)) || (x === 4 && (y === 7 || y === 9))

        const paths = [[-1, -1], [-1, 0], [-1, 1], [0, -1], [0, 1], [1, -1], [1, 0], [1, 1]]
            .map(toAbsoultePos)
            .filter(isInPalace)
            .filter(([x, y]) => !isPalaceSideMiddle(this.pos) || !isPalaceSideMiddle([x, y]))
            .filter(canForward)

        return paths
    }

    else if (this.text === '車') {
        const paths = [
            range(this.x - 1, 0).map(x => [x, this.y]).filter(isInBoard).takeWhile(canForward),
            range(this.x + 1, 8).map(x => [x, this.y]).filter(isInBoard).takeWhile(canForward),
            range(this.y - 1, 0).map(y => [this.x, y]).filter(isInBoard).takeWhile(canForward),
            range(this.y + 1, 9).map(y => [this.x, y]).filter(isInBoard).takeWhile(canForward),
            this.posIs([3, 7]) ? [[4, 8], [5, 9]].takeWhile(canForward) : [],
            this.posIs([3, 9]) ? [[4, 8], [5, 7]].takeWhile(canForward) : [],
            this.posIs([5, 7]) ? [[4, 8], [3, 9]].takeWhile(canForward) : [],
            this.posIs([5, 9]) ? [[4, 8], [3, 7]].takeWhile(canForward) : [],
            this.posIs([4, 8]) ? [[3, 7], [3, 9], [5, 7], [5, 9]].filter(canForward) : [],
            this.posIs([3, 0]) ? [[4, 1], [5, 2]].takeWhile(canForward) : [],
            this.posIs([3, 2]) ? [[4, 1], [5, 0]].takeWhile(canForward) : [],
            this.posIs([5, 0]) ? [[4, 1], [3, 2]].takeWhile(canForward) : [],
            this.posIs([5, 2]) ? [[4, 1], [3, 0]].takeWhile(canForward) : [],
            this.posIs([4, 1]) ? [[3, 0], [3, 2], [5, 0], [5, 2]].filter(canForward) : [],
        ].flat()

        return paths
    }

    else if (this.text === "包") {
        const calcPoPath = pathList => pathList
            .filter(isInBoard)
            .skipWhile(([x, y]) => !table[y][x] || (table[y][x] && table[y][x].text === '包'  && 'skipAll'))
            .slice(1)
            .takeWhile(([x, y]) => !table[y][x] || table[y][x].text !== '包' && (table[y][x].team !== this.team && 'stop'));

        const paths = [
            calcPoPath(range(this.y - 1, 0).map(y => [this.x, y])),
            calcPoPath(range(this.y + 1, 9).map(y => [this.x, y])),
            calcPoPath(range(this.x - 1, 0).map(x => [x, this.y])),
            calcPoPath(range(this.x + 1, 8).map(x => [x, this.y])),
            this.posIs([3, 7]) ? calcPoPath([[4, 8], [5, 9]]) : [],
            this.posIs([3, 9]) ? calcPoPath([[4, 8], [5, 7]]) : [],
            this.posIs([5, 7]) ? calcPoPath([[4, 8], [3, 9]]) : [],
            this.posIs([5, 9]) ? calcPoPath([[4, 8], [3, 7]]) : [],
            this.posIs([3, 0]) ? calcPoPath([[4, 1], [5, 2]]) : [],
            this.posIs([3, 2]) ? calcPoPath([[4, 1], [5, 0]]) : [],
            this.posIs([5, 0]) ? calcPoPath([[4, 1], [3, 2]]) : [],
            this.posIs([5, 2]) ? calcPoPath([[4, 1], [3, 0]]) : [],
        ].flat()

        return paths
    }

    else if (this.text === '馬') {
        const paths = [[0, -1], [-1, 0], [0, 1], [1, 0]]
            .flatMap(p => [[p, p.map(i => i || -1)], [p, p.map(i => i || 1)]])
            .map(([[x1, y1], [x2, y2]]) =>  [[x1, y1], [x1 + x2, y1 + y2]])
            .map(ps => ps.map(toAbsoultePos))
            .filter(ps => ps.every(isInBoard))
            .filter(([[x1, y1], [x2, y2]]) => isEmptyPlace([x1, y1]) && isEmptyOrEnemy([x2, y2]))
            .map(i => i[1])

        return paths
    }

    else if (this.text === '象') {
        const paths = [[0, -1], [-1, 0], [0, 1], [1, 0]]
            .flatMap(p => [[p, p.map(i => i || -1), p.map(i => i || -1)], [p, p.map(i => i || 1), p.map(i => i || 1)]])
            .map(([[x1, y1], [x2, y2], [x3, y3]]) => [[x1, y1], [x1 + x2, y1 + y2], [x1 + x2 + x3, y1 + y2 + y3]])
            .map(ps => ps.map(toAbsoultePos))
            .filter(ps => ps.every(isInBoard))
            .filter(([[x1, y1], [x2, y2], [x3, y3]]) =>
                isEmptyPlace([x1, y1]) && isEmptyPlace([x2, y2]) && isEmptyOrEnemy([x3, y3])
            )
            .map(i => i[2])

        return paths
    }

    return []
}

```

  </details>

### CSS 변수를 사용했고 이 변수에 다른 변수들이 의존하게 했다.

- 그래서 값 하나가 바뀌면 다른 값도 자동으로 바뀐다. (forward reference)

- 하드 코딩을 했다면 값을 하나 바꾸기 위해 프로그램 전체를 뒤지며 나머지 값을 일일이 바꿔야 한다.

### 인공지능과 대전하기 기능을 추가하려고 했다.

- 자신에겐 유리하고 상대에겐 불리한 선택 경로를 찾는 미니맥스 알고리즘과 불필요한 경로는 무시하는 가지치기 알고리즘을 공부했다.

- 한편, 프로그램의 모델 부분과 뷰 부분이 강결합돼있는 바람에 미니맥스 알고리즘을 구현하려면 코드를 다 갈아엎어야 돼서 포기했다.

- 다음부턴 뷰와 모델 사이에 컨트롤러를 꼭 추가해야겠다.

&nbsp;

## [오목](https://html-games.surge.sh/omok)

&nbsp;

<img alt="gomoku" src="https://user-images.githubusercontent.com/22253556/81934177-7ccf3000-9629-11ea-8478-fb1894a93e84.png" width="400px"/>

### 바둑 판자 / 바둑 선 / 바둑알 놓는 곳 3개로 레이어를 나눠서 쉽게 UI를 구현했다.

- 다른 구현을 보면 위 3개를 한 번에 하려다 보니 코드가 길고 보기 어렵다.

### 승리 확인 부분도 함수형 프로그래밍으로 `map`, `filter`, `reduce`만 사용해서 구현했다.

  <details>

  <summary>승리 확인 코드</summary>

```js
// 상하좌우+대각선 5줄이 모두 같은 색이면 승리

checkWin() {
  const turnPositions = this.grid.flatMap((row, y) =>
    row.reduce(
      (acc, turn, x) => (turn === this.turn ? [...acc, [x, y]] : acc),
      []
    )
  );

  for (const [x, y] of turnPositions) {
    const right = zip(range(x, x + 4), repeat5(y));
    const left = zip(range(x, x - 4), repeat5(y));
    const top = zip(repeat5(x), range(y, y + 4));
    const bottom = zip(repeat5(x), range(y, y - 4));
    const bottomLeft = zip(range(x, x + 4), range(y, y + 4));
    const bottomRight = zip(range(x, x - 4), range(y, y + 4));
    const topRight = zip(range(x, x + 4), range(y, y - 4));
    const topLeft = zip(range(x, x - 4), range(y, y - 4));

    const toTurn = ([x, y]) => this.grid[y][x];
    const isSameTurn = (turn) => turn === this.turn;

    const fiveInRow = !![right, left, top, bottom, topRight, topLeft, bottomRight, bottomLeft]
      .filter((poss) => poss.every(isInBoard))
      .filter((poss) => poss.map(toTurn).every(isSameTurn)).length;

    if (fiveInRow) {
      return ui.win(this.turn);
    }
  }
}
```

  </details>
&nbsp;

### 놓은 바둑알을 순서대로 스택에 넣어서 물리기 기능을 추가했다.

- `undo` 기능을 처음으로 구현해봤다.

&nbsp;

## [테트리스](https://html-games.surge.sh/tetris)

&nbsp;

<img alt="tetris" src="https://user-images.githubusercontent.com/22253556/81934265-a5572a00-9629-11ea-9f8f-6d807f4f37cb.png" width="400px"/>

### [100줄 테트리스](https://github.com/Alaricus/SimpleTetris)의 로직과 [데릭 아저씨의 테트리스](https://www.youtube.com/watch?v=QDp8BZbwOqk)의 UI를 합쳤다.

- 100줄 테트리스는 함수형 프로그래밍스러운 로직을 짜서 코드 이해가 쉽지만 UI가 아쉽다.

- 데릭 아저씨의 테트리스는 UI가 보기는 좋지만 로직에 오류가 있다.

- 두 코드를 믹스 & 매치했다.

- 남이 잘 짜놓은 코드 가져다가 내 것으로 만드는게 제일 행복하다.

### 다음 블록보기, 블록 한번에 내리기 기능 추가

- 후임들의 요청으로 넣은 기능이다.

&nbsp;

## [스택!](https://html-games.surge.sh/stack)

&nbsp;

<img alt="stack" src="https://user-images.githubusercontent.com/22253556/86470747-07edbc00-bd77-11ea-9457-aee76662e62c.png" width="400px"/>

### [Pure CSS Stack](https://codepen.io/finnhvman/pen/xJRMJp)에 영감을 받았다.

- Pure CSS Stack은 HTML과 CSS만 사용해서 3D Stack 애니메이션을 구현했다.

- 대신에 50칸 밖에 쌓을 수 없어서 JS 버전으로 새로 만들었다.

- `three.js` 없이 바닐라 `Canvas`로는 3D 구현이 어려워서 2D로 구현했다.

- 3D 구현을 위해서는 3D 공간을 모델링하고 이를 레이 트레이싱한 것을 2D 캔버스에 투영하거나

- `WebGL`을 사용해야하는데 이렇게 하려면 전문하사 신청해야 끝낼 수 있어서 안 했다.

### 게임 로직 만드는 데 3일 밖에 안 걸렸지만 게임 종료 애니메이션을 만드는 데 일주일 걸렸다.

<img alt="stack end" src="https://user-images.githubusercontent.com/22253556/90395078-d5054a80-e0ce-11ea-8153-4f5621c8bac7.gif" width="400px"/>

- 게임이 끝나면 화면이 서서히 축소되면서 지금까지 쌓아올린 블록을 보여준다.

- 캔버스의 `scale`과 `translate`를 프레임마다 적절히 조정하는 게 어려웠다.

&nbsp;

## [미로 찾기](https://html-games.surge.sh/maze)

&nbsp;

<img alt="maze" src="https://user-images.githubusercontent.com/22253556/82049751-dac74a80-96f1-11ea-96d8-f3fff2f002a9.png" width="400px"/>

- 리얼월드 알고리즘 1장, **DFS로 미로 생성 프로그램을 만들기** 과제를 풀다가 만들었다.

  <details>
    <summary>스택기반 DFS는 코드가 정말 짧다.</summary>

  ```js
  const initialCell = grid[0][0]

  initialCell.visit()

  const stack = [initialCell]

  let currentCell

  while (stack.length) {
    currentCell = stack.pop()

    const neighborCell = currentCell.randomNeighbor(grid)

    if (neighborCell) {
      stack.push(currentCell)
      removeWall(currentCell, neighborCell)
      neighborCell.visit()
      stack.push(neighborCell)
    }
  }
  ```

  </details>

* 프린터로 출력해서 펜으로 풀 수 있게 `인쇄하기` 버튼을 추가했다.

* 사진처럼 방향키로 미로 탐색도 가능하다.

&nbsp;

## [트렐로 클론](https://html-games.surge.sh/kanban)

<img alt="canban1" src="https://user-images.githubusercontent.com/22253556/81934493-fff08600-9629-11ea-92e1-eadd3c0f3393.png" width="400px"/>

<img alt="canban2" src="https://user-images.githubusercontent.com/22253556/81934425-e51e1180-9629-11ea-85d4-bff5f6e671c8.png" width="400px"/>

### 오프라인 저장을 위해 `html`을 통짜로 덤프 떠서 `LocalStorage`에 저장한다.

- 다시 불러올 땐 `html` 덤프를 컨테이너의 `innerHTML`으로 설정하고 이벤트 리스너만 붙이면 끝

- SPA 프레임워크를 쓴다면 이런 방식이 불가능하다.

### CSS 변수를 사용해서 간단하게 다크 모드를 구현했다.

&nbsp;

# 이외에

## `2048`

&nbsp;

<img alt="canban2" src="https://user-images.githubusercontent.com/22253556/90393773-5dceb700-e0cc-11ea-88ba-0b190188ba81.png" width="400px"/>

싸지방에서 인쇄한 2048 코드를 하나하나 입력해서 구현했다.

기존 프로토타입 기반 코드를 ES6 클래스 문법으로 바꾸고

메서드는 함수형으로 리팩터링했더니 1000줄짜리 코드가 700줄로 줄였다.

&nbsp;

## 스네이크

&nbsp;

<img alt="canban2" src="https://user-images.githubusercontent.com/22253556/90393982-bf8f2100-e0cc-11ea-8087-08002e90aaea.png" width="400px"/>

33줄로 구현하는 리액트를 구현하고

<script src="https://gist.github.com/16Yongjin/c5a8be2f7df740e76450d4dcb149dcf1.js"></script>

블로그에 예제로 나온 스네이크 게임도 그대로 가져다가 구현했다.

기존 코드에 최고 점수 기능, PC에 맞게 CSS 레이아웃을 추가했다.

## 느낀 점

- 남들 다 장기나 공 튀기기 하나씩 만들길래 게임 제작을 쉽게 봤는데 힘든 점이 많았다.

- 근데 하다보면 된다.

- 크롬 최고
