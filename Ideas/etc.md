# 빨래분배

빨래 주인 찾아주기 번거롭다.

빨래물이 다 똑같이 생겨서 그렇다.

빨래를 시각적으로 구별할 수 있으면 빠르게 빨래를 나눌 수 있다.

유성매직으로 색칠하기, 형형색색의 실로 자수 넣기 등의 방법이 있다.

실현이 쉽고 안정성, 신속성을 다 만족하는 방안을 계속 생각 해봐야겠다.

# 약속 장소 잡기

서로 다른 지역에 사는 친구들끼리 만날 때

각자의 지역 입력하면 중간 지역을 알려주는 앱

은 이미 있구나.. 위밋플레이스

# 조엘이 권장하는 '대학생이 갖춰야 할 지식'목록

1. 졸업 전에 작문법을 배운다.
2. 졸업 전에 C를 배운다.
3. 졸업 전에 미시 경제학을 공부한다.
4. 따분하다고 비 전산 과목을 등한시하지 마라.
5. 프로그래밍 심화과정을 수강하라.
6. 모든 작업이 인도로 넘어간다는 걱정은 그만둬라.
7. 무엇을 하든 여름 인턴과정을 거쳐라.

# 20191223

해커랭크 쉬운문제 8문제 풀었다.

# 20191224

해커랭크 레밸 2를 달성했다.

---

"함수형 언어 산책"이라는 책이 새로 나왔다.

대표적인 함수형 언어들(스칼라, 하스켈, 클로저, 엘릭서) 예제들을 다룬다.

"순수 함수형 데이터 구조:불변성과 지연 계산을 활용한 함수형 데이터 구조" 라는 책도 나온 거 같은데 재밌어 보인다.

# 20191225

## 자주 틀리는 띄어쓰기

해야 한다.
안 하다.
싶어 하다

# 20200105

[Glowing Gradient Button Animation Effects on Hover](https://codepen.io/16yongjin/pen/abzEorY)

완전 이쁘다.

다음의 검색창 테두리도 이 효과를 볼 수 있다.

길다란(400px) 그레디언트 배경을 만들고 `background-position` 속성을 바꾸는 애니메이션을 줘서 배경이 움직이게 해 버튼의 색상이 형형색색으로 변한다.

`filter: blur()` 속성으로 빛나는 효과도 추가했다.

# 20200121

1. repl.it의 멀티플레이어 기능으로 팀 단위 코딩테스트를 준비하면 좋을 것 같다.
2. 잡코리아 이력서 자동입력 중인데 학교구분 드롭다운 버튼이 안 눌린다.

# 20200122

1. 잡코리아 이력서 등록, 학력정보 자동입력 부분을 마쳤다.

# 20200123

1. 잡코리아 이력서 등록, 경력정보 자동입력 부분을 마쳤다.
   - 다만, 직무 부분은 선택지가 너무 많아서 스킵하고 나중에 서비스가 상용화될 때 쯤이나 다뤄야겠다.

# 20200124

1. 잡코리아 자격증 부분 입력 자동화
2. 타입스크립트로 리팩터링 후 기능별로 함수와 파일을 나눴다.
3. 다른 항목마다 공통된 부분이 보이기 시작해서 그 부분을 함수화 해야겠다.

# 20200125

1. 잡코리아 어학 부분을 자동화 중인데 또 구분 드롭다운 버튼이 안 눌린다.
2. 같은 이름의 js 파일과 ts 파일이 있으면 js 파일이 먼저 선택되는것 같다.
3. front-end 라이브러리로 buefy 당첨이다.
4. 이력서 등록 UI Flowchart Draft를 완성했다.

# 20200129

1. 코딩테스트를 봤다. 엄청 쉬운 문제였는데 너무 떨어서 실수가 많았다.
2. 사람인 마이프로필 자동화, 타입스크립트 리팩터링을 위한 구조를 잡았다.
3. 사람인 경력사항 부분 경력을 복수로 받을 수 있게 리팩터링했다.
4. 만들어 놓은 puppeteer 유틸 라이브러리가 정말 유연해서 유용하다.
5. 잡코리아 로그인 시 메인 페이지 대신 로그인용 페이지로 가게해서 로딩시간이 10초 이상 줄었다.

# 20200130

1. ctrl cv 서비스 시퀀스 다이어그램을 작성했다.
2. swagger로 API와 데이터 모델을 문서화를 해야겠다.
3. 아키텍쳐 다이어그램도 만들어서 팀원들과 소통해야겠다.

# 20200131

1. Flask 대체로 FastAPI가 좋아보인다. 깃헙 스타도 8.8로 웹 프레임워크 중 제일 많다. 타입힌트 덕분에 IDE 도움도 받을 수 있고 문서도 자동으로 생성된다. 튜토리얼이 잘 되어있어서 기능 구현도 원할할 것 같다.

# 20200204

1. 잡플래닛 학력부분 자동화했다.
2. 폼 삭제 시 서버와 데이터 동기화가 일어나는 점을 이용해서 폼 초기화 시 networkidle0을 기다리는 함수로 진행 동기화를 했다.

# 20200620

`Auto Annotaion`에 `Onnx.js` 모델 연동을 시도했다가 다 갈아엎었다.

자바스크립트 정수는 32비트만 취급하는데 Onnx 모델 Zoo의 모델 파라미터는 64비트 정수로 되어있어서 Onnx.js에서 모델 로딩 시 int64 에러가 발생한다.

64비트로 훈련된 모델을 바로 32비트로 캐스팅하는 방법은 아직 없는 것 같다.

직접 32비트 모델을 훈련시켜서 가져다 쓰면 되겠지만 굳이 그러고 싶지 않다.

또한 Onnx.js 사용 시 모델마다 전처리와 후처리를 다 다르게 해야하므로 그냥 Tensorflow.js 만 쓰기로 했다.

# 20200930

현재 수강 중인 컴퓨터구조 강의를 40명 가까이 드롭했다.

교수님도 많이 당황하셨는데

수강신청 드롭 전후의 신청인원을 수집해서 강의를 탈출순으로 정렬하면 재밌을 거 같다.
