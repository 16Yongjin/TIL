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

- 크롬 브라우저

- 구글, 스택오버플로 사용 불가

- 라이브러리 사용 불가

- IDE 사용 불가, 메모장 사용

## 제한사항

게시판에 `hwp` 파일만 업로드할 수 있어서

배포 용이성을 위해 모든 코드와 자원은 `html` 파일 하나에 들어가야 했다.

&nbsp;

# 게임 소개

## [미로 찾기](https://html-games.surge.sh/maze)

&nbsp;

<img alt="maze" src="https://user-images.githubusercontent.com/22253556/82049751-dac74a80-96f1-11ea-96d8-f3fff2f002a9.png" width="400px"/>

- 리얼월드 알고리즘 1장 과제에서 DFS로 미로 생성 프로그램을 만들라고 해서 만들었다.

  - 스택기반 DFS는 코드가 정말 짧다.

- 프린터로 출력해서 펜으로 풀 수 있게 `인쇄하기` 버튼을 추가했다.

- 사진처럼 방향키로 미로 탐색도 가능하다.

&nbsp;

## [스와이프 벽돌 깨기](https://html-games.surge.sh/brick-breaker)

<img alt="sbb" src="https://user-images.githubusercontent.com/22253556/81933996-2cf06900-9629-11ea-9b4e-31cee7502d29.png " width="400px"/>

- 자바스크립트 버전은 세상에서 유일한 것 같다.

- 공 발사는 예전에 만들었던 당구 게임에서 착안

- 객체 지향으로 구조화 (+ 다이어그램 추가)

![SBB-diagram](https://user-images.githubusercontent.com/22253556/90336638-1ab30c00-e018-11ea-9dde-f204c7c1cc08.png)

- 파티클 시스템

- 후임이 당직 때 10시간 동안 플레이해서 997점 달성했다. (내가 본 최고점수)

&nbsp;

## [장기](https://html-games.surge.sh/janggi)

<img alt="janggji" src="https://user-images.githubusercontent.com/22253556/81934129-645f1580-9629-11ea-89b9-5bdf53918eb2.png" width="400px"/>

- 말의 경로를 함수형 프로그래밍으로 모델링했다.

  - `for` 문이 없다.

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

* CSS 변수를 사용했고 이 변수에 다른 변수들이 의존하게 했다.

  - 그래서 값 하나가 바뀌면 다른 값도 자동으로 바뀐다. (forward reference)

  - 하드 코딩을 했다면 값을 하나 바꾸기 위해 프로그램 전체를 뒤지며 나머지 값을 일일이 바꿔야 한다.

&nbsp;

## [오목](https://html-games.surge.sh/omok)

<img alt="gomoku" src="https://user-images.githubusercontent.com/22253556/81934177-7ccf3000-9629-11ea-8478-fb1894a93e84.png" width="400px"/>

- 3시간만에 만들었다.

- 바둑 판자 / 바둑 선 / 바둑알 놓는 곳 3개로 레이어를 나눠서 쉽게 UI를 구현했다.

  - 다른 구현을 보면 위 3개를 한 번에 하려다 보니 코드가 길고 보기 어렵다.

- 승리 확인 부분도 함수형 프로그래밍으로 `map`, `filter`, `reduce`만 사용해서 구현했다.

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

      const fiveInRow = !![
        right,
        left,
        top,
        bottom,
        topRight,
        topLeft,
        bottomRight,
        bottomLeft,
      ]
        .filter((poss) => poss.every(isInBoard))
        .filter((poss) => poss.map(toTurn).every(isSameTurn)).length;

      if (fiveInRow) {
        return ui.win(this.turn);
      }
    }
  }
  ```

  </details>

* 놓은 바둑알을 스택에 넣어서 물리기 기능을 추가했다.

&nbsp;

## [테트리스](https://html-games.surge.sh/tetris)

<img alt="tetris" src="https://user-images.githubusercontent.com/22253556/81934265-a5572a00-9629-11ea-9f8f-6d807f4f37cb.png" width="400px"/>

- 100줄 테트리스의 로직과 데릭 아저씨의 테트리스의 UI를 합쳤다.

- 거기에 다음 블록보기, 블록 한번에 내리기 기능 추가

## [트렐로 클론](https://html-games.surge.sh/kanban)

<img alt="canban1" src="https://user-images.githubusercontent.com/22253556/81934493-fff08600-9629-11ea-92e1-eadd3c0f3393.png" width="400px"/>

<img alt="canban2" src="https://user-images.githubusercontent.com/22253556/81934425-e51e1180-9629-11ea-85d4-bff5f6e671c8.png" width="400px"/>

- 오프라인 저장을 위해 `html`을 통짜로 덤프 떠서 `LocalStorage`에 저장한다.

  - 다시 불러올 땐 `html`을 컨테이너의 `innerHTML`으로 설정하고 이벤트 리스너만 붙이면 끝

  - SPA 프레임워크를 쓴다면 상상도 못할 일

- CSS 변수를 사용해서 간단하게 다크 모드를 구현했다.

&nbsp;

## [스택!](https://html-games.surge.sh/stack)

<img alt="stack" src="https://user-images.githubusercontent.com/22253556/86470747-07edbc00-bd77-11ea-9457-aee76662e62c.png" width="400px"/>

- CSS 스택에 영감을 받음

* 게임 로직 만드는 데 3일, 게임 종료 애니메이션을 만드는 데 일주일 걸렸다.

  - 게임이 끝나면 화면이 서서히 축소되면서 지금까지 쌓아올린 블록을 보여준다.

  - 캔버스의 `scale`과 `translate`를 조정하는 게 힘들었다.

&nbsp;

## 이외에

### 2048

싸지방에서 인쇄한 2048 코드를 하나하나 입력해서 구현했다.

기존 프로토타입 기반 코드를 ES6 클래스 문법으로 바꾸고

메서드는 함수형으로 리팩터링해서 1000줄짜리 코드를 700줄로 줄였다.

### 스네이크

33줄로 구현하는 리액트를 구현하고

블로그에 예제로 나온 스네이크 게임도 그대로 가져다가 구현했다.

기존 코드에 최고 점수 기능, PC에 맞게 CSS 레이아웃을 추가했다.

## 느낀 점

- 게임 개발이 남들 다 하길래 쉬울 줄 알았는데 넘모 어려웠다.

- 근데 하다보면 되는 게 신기하다.

- 크롬 최고
