# 스프링 부트 개발 일지

## 2019월 12월 14일

**스프링 부트 시작하기**라는 책을 보고 있다.

### 에디터(VS Code) 설정

책에서는 이클립스를 사용하라고 하지만 VS Code에도 자바와 스프링 개발을 위한 플러그인들이 많아서 충분히 개발이 가능한 것 같다.

[Java Extension Pack](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)을 설치하고
[Spring Boot in Visual Studio Code](https://code.visualstudio.com/docs/java/java-spring-boot)을 보고 스프링 부트용 플러그인들을 설치했다.

레드햇에서 만든 [XML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-xml) 플러그인도 유용한 것 같다.
설정 시 XML을 많이 사용하는데, 태그 닫기와 스키마 자동 완성을 지원해준다.

### 데이터베이스 설정

MySQL 5.7 버전과 GUI 툴인 SQLyog를 설치했다.

### 프로젝트 설정

[Spring Initializr](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-spring-initializr)로 프로젝트를 생성했다.

`application.properties`에 Hikari CP 설정으로 드라이버 이름, DB url, 계정, 연결 테스트 쿼리를 넣었다.

`DatabaseConfiguration` 클래스를 만들고 `@Configuration`, `@PropertySource` 어노테이션를 붙여 자바로 된 설정을 했다.

이 클래스 안에 Hikari, 마이바티스 설정을 넣었다.

테스트 코드를 만들어서 DB와 마이바티스가 잘 연결되는지 확인했다.

---

## 2019년 12월 15일

4장 간단한 게시판 구현하기를 완료했다.

테이블 생성 쿼리를 실행해 게시판 테이블을 만들었다.

`getter`나 `setter`, `toString` 메서드를 자동으로 만들어주는 Lombok 플러그인을 설치했다.

데이터베이스에서 게시판 데이터를 가져와서 자바에서 사용하기 위해 `DTO`를 정의했다.

### 컨트롤러

컨트롤러는 클라이언트 요청을 받아서 필요한 비즈니스 로직을 호출하고 그 결과를 응답해주는 디스패처 역할을 한다.

`@RequestMapping` 어노테이션으로 메서드를 라우팅 한다.

`ModelAndView`를 반환해서 HTML 렌더링하거나 `String`을 반환해서 리다이렉트를 한다.

### 서비스

`Service` 인터페이스와 `ServiceImpl` 클래스로 나눠서 서비스를 구성한다.

서비스에서 DB 조회와 데이터 가공을 처리한다

### 매퍼

마이바티스 사용 시 DAO를 사용하는 대신 인터페이스만을 이용해 편리하게 개발하기 위해 매퍼를 사용한다.

매퍼 인터페이스에는 DB 조작을 위한 인터페이스만 설정하고 매퍼 XML에는 쿼리를 작성하고 이를 매퍼 인터페이스와 연결한다.

### 뷰

기본 html에 `thymeleaf` 템플릿 엔진을 사용한다.

데이터가 들어갈 자리만 정해주고 데이터 객체를 넣으면 HTML이 렌더링된다.

---

## 2019년 12월 16일

### LogBack을 사용한 로깅

`slf4j` 인터페이스에 스프링 부트 기본 로거인 `LogBack` 구현체를 사용한다.

`slf4j`라는 추상 레이어 사용으로 `LogBack`이나 `Log4j2` 같은 로깅 구현체를 마음대로 바꿔 사용할 수 있다.

### Log4JDBC로 쿼리 로그 정렬

`application.properties`에 아래 코드를 추가한 뒤

```groovy
implementation group: 'org.bgee.log4jdbc-log4j2', name: 'log4jdbc-log4j2-jdbc4.1', version: '1.16'
```

jdbc 드라이버를 `com.mysql.cj.jdbc.Driver`에서 `net.sf.log4jdbc.sql.jdbcapi.DriverSpy`로 바꿔주고

logback 설정 좀 만져주면

```bash
2019-12-16 20:41:05,503  INFO [jdbc.sqlonly] SELECT
				board_idx,
				title,
				contents,
				hit_cnt,
				DATE_FORMAT(created_datetime, '%Y.%m.%d %H:%i:%s') AS created_datetime,
				creator_id
			FROM
				t_board
			WHERE
				board_idx = 3
				AND deleted_yn = 'N'

2019-12-16 20:41:05,505  INFO [jdbc.resultsettable]
|----------|---------------|----------|--------|--------------------|-----------|
|board_idx |title          |contents  |hit_cnt |created_datetime    |creator_id |
|----------|---------------|----------|--------|--------------------|-----------|
|3         |Hello world!!! |hello!!!! |6       |2019.12.15 19:46:18 |admin      |
|----------|---------------|----------|--------|--------------------|-----------|
```

이런 식으로 로그가 쿼리부터 실행결과 테이블까지 가지런하게 나온다.

### 인터셉터

컨트롤러 처리 전 또는 후 작업을 위해 사용된다.

#### 필터와 인터셉트 차이

필터와 기능적으로 비슷한데 필터는 디스패처 서블릿 앞에서, 인터셉트는 디스패처 서블릿 뒤 핸들러 컨트롤러 앞에서 동작한다.

인터셉트는 스프링 빈을 사용할 수 있다.

문자열 인코딩 같이 전반에서 사용되는 기능은 필터로,
클라이언트 요청 관련 처리(로그인, 인증, 권한 등)는 인터셉터로 처리한다.

#### 사용법

`HandlerInterceptorAdapter`를 상속받아
`preHandle`, `postHandle`, `afterCompletion` 세 가지 메서드 중 필요한 메서드를 구현한 뒤
스프링 빈에 인터셉트를 등록하면 된다.

### 한글 인코딩 문제 해결법

mysql 기본 인코딩이 latin으로 되있어서 생기는 문제다.

1. `C:\Program Files\MySQL\MySQL Server 5.7`로 들어가서 `my.cnf` 파일을 생성하고 아래 코드를 입력한다.

```ini
[client]
default-character-set=utf8

[mysqld]
character-set-client-handshake = FALSE
init_connect="SET collation_connection = utf8_general_ci"
init_connect="SET NAMES utf8"
character-set-server = utf8

[mysql]
default-character-set=utf8

[mysqldump]
default-character-set = utf8
```

2. 윈도우 서비스에서 MYSql 5.7 을 재시작한다.

3. 테이블은 기존의 인코딩으로 생성되어있으므로 GUI툴로 테이블 Charset을 utf8로 Collation를 utf8_general_ci 바꾼다.
