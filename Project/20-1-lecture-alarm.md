# 20-1학기 한국외대 수강신청 빈자리 알람

6학기째 만들고 있는 서비스

다른 사람이 Todo List나 게시판으로 기본기를 다진다면, 나는 강의 알람으로 기본기를 다지고 있다.

## 프로젝트 결과

171명에게 666개의 빈자리 알람을 보냈다.

서버는 한 번도 안 멈췄다.

## 지난 학기 버전과 다른 점

### 1. 서버에 신경을 많이 썼다.

저번에 숙련도가 떨어지는 스칼라로 서버를 만들었다가 버그로 변경기간 5일 중 4일동안 서비스가 정지됐다.

그래서 잘 알고 있는 Node.js를 사용했고, 서버 이상 파악을 위해 **모니터링 대시보드**도 만들었다.

### 2. 안드로이드 앱 리워드 광고 추가

근데 \$0.89 밖에 못 벌었다.

### 3. 문자 버전 추가

내 서비스는 아이폰 지원이 안 된다는 치명적인 결점이 있었다.

ncloud의 SMS 서비스를 이용해서 전화번호만 있으면 문자로 알람을 받을 수 있게 했다.

### 4. 강의 검색 기능 추가

# [Alarm Server](https://github.com/16Yongjin/20-1-lecture-alarm-node) (`Node.js`)

## 하는 일

- 강의 목록 제공
- 유저 알람 보기, 저장, 삭제
- 빈자리 확인 후 알람(FCM)
- 서버 로그, 상태 제공

등의 기능을 담당한다.

## 프로젝트 구조

[Production ready Node.js REST API Setup using TypeScript, PostgreSQL and Redis](https://itnext.io/production-ready-node-js-rest-apis-setup-using-typescript-postgresql-and-redis-a9525871407)

이 글을 보고 서버 구조를 잡았다.

![image](https://user-images.githubusercontent.com/22253556/77154557-479dda00-6adf-11ea-82ed-0cacce3071e8.png)

- `database`: DB 연결 코드
- `entities`: TypeORM 엔티티
- `middleware`: CORS, 로깅, 압축, 본문 파싱, 검증을 담당하는 코드
- `services`: 기능별로 분리된 라우터와 컨트롤러
- `utils`: 여러 곳에서 사용되는 에러 처리 로직, FCM 전송, Logger 유틸

구조를 잘 잡으니 기능 구현 시 파일을 헤매는 일도 없고 기능 추가하기도 쉽다.

## 로깅

`winston`으로 `morgan`에서 나오는 Express 요청부터, 에러, 완료된 알람 로그를 로그 파일에 남겼다.

로그 파일 덕분에 서버 에러를 파악하고 서버가 잘 동작하고 있음을 알 수 있었다.

항상 로깅을 소홀히 했었는데 이번 기회에 로깅의 중요성을 알게 됐다.

## 테스팅

![서버테스팅](https://user-images.githubusercontent.com/22253556/77155927-a9f7da00-6ae1-11ea-9ee3-3d8fbaa23590.png)

테스트 케이스를 안 만들고 Postman으로 직접 데이터를 넣어서 작동을 확인하는 나쁜 습관을 고치기 위해 테스트를 작성했다.

코드 수정 후 잘 작동될 거라는 생각에 실행해보지도 않고 서버에 배포하는 더 나쁜 습관이 있었는데, 코드 푸시 전 항상 실행해보는 습관도 길렀다.

## TypeORM

`User`와 `Lecture` 두 엔티티를 만들고 각각 `lectures`와 `users`로 Many To Many 연결을 했다.

Array의 filter 메서드로 Many To Many 관계 삭제 시 Postgres에서만 일어나는 미묘한 버그가 있는데

아래와 같이 쿼리 빌더를 사용해서 해결했다.

```typescript
User.createQueryBuilder()
  .relation(User, "lectures")
  .of({ id })
  .remove(lectureId);
```

강의 중 유저가 등록 중인 것만 가져올 때는 Inner Join을 사용했다.

그냥 강의를 가져오면 FULL OUTER JOIN으로 4185개의 강의를 모두 가져와서 `users`가 1 이상인 강의만 필터링해야되는데 이러면 매우 느리다.

## Cron 사용

빈자리 확인기는 3가지 요구사항이 있다.

1. 3초마다 작동
2. 10-16시 사이에 작동
3. 운영자가 작동을 멈추고 시작할 수 있음

예전엔

1. setTimeout이나 setInterval을 사용하고
2. 시간 관련 로직을 짜고
3. 시작/정지를 위해 요상한 객체를 만들었었다.

이번에는 `CronJob` 라이브러리에 `*/3 * 10-16 * * *` 인자 하나를 넣음으로 모든 요구 사항을 충족했다.

`*/3 * 10-16 * * *`는 10-16시 사이 3초 마다를 의미한다.

## 미들웨어로 검증과 비즈니스 로직 분리

![image](https://user-images.githubusercontent.com/22253556/77158062-e75e6680-6ae5-11ea-8245-0d25cb47a616.png)

유저가 알람 추가 시 `addUserAlarm` 컨트롤러가 호출된다.

알람 추가에 필요한 모든 정보가 있는지 검증하는 `checkAddUserAlarmBody`를 컨트롤러 앞에 놓아서,

컨트롤러는 잘못된 인자가 들어올 걱정 없이 비즈니스 로직에만 집중할 수 있고, 코드 보기도 편하다.

## Github Webhook으로 자동 배포

코드 Push 시 자동으로 배포가 되게 설정했다.

배포서버에 이 [nodejs-github-webhook](https://github.com/velopert/nodejs-github-webhook) 레포에 있는 코드를 실행해서 웹훅에 반응할 수 있게 했다.

웹훅 발생 시, 미리 설정해 놓은 쉘 스크립트가 최신 코드 pull, 패키지 설치, 서버 시작을 실행한다

한 번 설정해놓으면 배포가 알아서 되니, 개발에만 집중할 수 있었다.

# [Android App by Flutter](https://github.com/16Yongjin/20-1-lecture-alarm-flutter)

![Screenshot_2020-03-20-20-35-09-875_com google android apps playconsole](https://user-images.githubusercontent.com/22253556/77160536-e24fe600-6aea-11ea-8dc7-46e39795c09f.jpg)

<center>활성 사용자 129명</center>

100명 넘게 내 앱을 사용해줬다.

사실, 이번 버전을 만들까 말까 고민했는데 아래 리뷰 댓글을 보고 만들게 됐다. ㅋㅋ

![image](https://user-images.githubusercontent.com/22253556/77160620-11665780-6aeb-11ea-88ae-c847c36a2ecb.png)

## 위젯을 쪼개고 또 쪼개고

위젯의 `build` 메서드에는 최대한 위젯의 문법 강조 색인 초록색만 보이게 했다.

![image](https://user-images.githubusercontent.com/22253556/77160974-e6c8ce80-6aeb-11ea-978f-4a94d7441338.png)

<center>홈 위젯</center>

![image](https://user-images.githubusercontent.com/22253556/77160951-d6b0ef00-6aeb-11ea-8af4-7d9ca094e396.png)

<center>알람 추가 위젯</center>

위젯을 최대한 나눴더니 코드 보기도 쉽고 위젯을 재활용할 수 있다.

## 광고 추가하기

[`firebase_admob`](`https://pub.dev/packages/firebase_admob`) 패키지를 사용해서 리워드 비디오 광고를 추가했다.

광고 영상을 불러와서 보여주는 코드를 재활용하기 위해,

로딩 후 / 시작 전 / 시작 후 / 완료 후 실행돼야할 Hook 함수를 인자로 받는 클래스로 광고를 보여주는 기능을 분리했다.

## 광고를 볼 수 밖에 없도록

처음엔 선물 아이콘 버튼으로 광고를 보면 알람 등록 개수가 늘어나게 했다.

버튼 설명이 없어서 그런지 **아무도** 광고를 보지 않았다.

다음엔 알람 개수 제한에 도달 시, 알람을 더 추가하려고 하면 광고를 보라는 메시지를 띄었다.

광고만 보면 알람을 더 등록할 수 있는데도 광고를 **5명** 밖에 안 봤다.

마지막으로 알람 개수 제한 시에도 강의를 추가할 수 있게 한 뒤,

강의 선택 후, 알람을 추가하려고 할 때 광고를 보도록 했다.

지금까지 강의를 선택한 수고가 아까워서 그런지 **50명**이 광고를 봤다.

## 리워드 비디오 광고의 배신

30초 광고를 보는데도 노출 한 개당 50원도 안준다.

그냥 작은 배너 광고 하나 띄우는 게 돈을 더 많이 벌 것 같다.

## 비밀 코드 입력

![image](https://user-images.githubusercontent.com/22253556/77161864-06f98d00-6aee-11ea-9b58-40812d3c5ee0.png)

아무도 발견할 수 없는 이스터에그를 만들었다.

숨겨져있는 버튼을 길게 누르고 있으면 입력 창이 뜬다.

여기에 "포어과"를 입력하면 광고를 보지 않아도 알람을 7개를 추가할 수 있다.

## 검색 기능 추가

![KakaoTalk_20200315_185857858_06](https://user-images.githubusercontent.com/22253556/77163232-fa2a6880-6af0-11ea-920d-3c3e8672953c.jpg)

힘들게 추가했지만 1000번 이상의 강의 찾기 중, 34번만 검색이 사용되었다.

검색 아이콘으로는 검색 기능 존재를 설명하기는 부족한가보다.

사용자를 위한 매뉴얼 작성에 더 신경써야겠다.

## 기타

안드로이드 에뮬레이터에서 localhost에 접근하기 위해 주소로 `10.0.2.2`를 사용해야 한다

![image](https://user-images.githubusercontent.com/22253556/77160754-62764b80-6aeb-11ea-8e59-4e1dd9cf0a95.png)

<center>앱 업데이트 후 수정된 리뷰 ㅋㅋ</center>

# [Web App By Svelte](https://github.com/16Yongjin/20-1-lecture-alarm-svelte)

## 라이브러리 코드를 직접 수정하지 말자

아..

# [Dashboard](https://github.com/16Yongjin/20-1-lecture-alarm-dashboard)

![dashboard-screenshot](https://user-images.githubusercontent.com/22253556/77064355-b1a47980-6a22-11ea-8696-238b8e949667.png)
