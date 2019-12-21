`# 스프링 부트 개발 일지

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

를 해봤는데도 안됐는데

테이블 생성 시 마지막에 `DEFAULT CHARSET=utf8;` 붙이면 해결되는 문제였다.

## 2019년 12월 17일

### AOP

AOP방법은 핵심 기능과 공통 기능을 분리 시켜놓고, 공통 기능을 필요로 하는 핵심 기능들에서 사용하는 방식이다.

게시판 핵심 기능을 구현하는데 권한, 로깅, 트랜잭션 같은 공통 기능을 추가하고

계좌이체 핵심 기능을 구현하는데 다시 권한, 로깅, 트랜잭션 같은 공통 기능을 추가해야 했던 것을

권한, 로깅, 트랜잭션 등 공통 기능들은 알아서 실행되게 하고 핵심 기능 구현에만 집중하도록 하는 기법이다.

#### AOP의 용어

| 용어       | 의미                                                 |
| ---------- | ---------------------------------------------------- |
| 관점       | 공통적으로 적용될 기능(권한, 로깅, ...)              |
| 어드바이스 | 관점의 구현체, 조인포인트에 삽입되어 동작함          |
| 조인포인트 | 어드바이스 적용 지점, 스프링에서는 메서드 실행단계만 |
| 포인트컷   | 조인포인트 선별 과정                                 |
| 타깃       | 어드바이스를 받는 대상                               |
| 위빙       | 어드바이스를 적용, 삽입하는 것                       |

#### 어드바아스

동작 시점에 따라 다섯 종류로 구분된다.

- Before Advice - 메서드 실행 전
- After returning Advice - 메서드 실행 성공 후
- After throwing Advice - `try/catch`의 `catch` 같음
- After Advice - `finally` 같음
- Around Advice - 범용적

#### 포인트컷의 `execution`

- `*` - 모든 값
- `..` - 0개 이상

`execution(void select*(..))`

리턴이 `void` 메서드 이름이 `select`로 시작, 파라미터가 0개 이상 호출될 때

`execution(* board..select*(**))`

`board` 패키지의 모든 하위 패키지에 있는 `select`로 시작하고 파라미터가 두 개인 모든 메서드가 호출될 때

### 트랜잭션

스프링의 트랙잭션 처리 방식은 세 종류로 구분된다.

- XML - xml 파일에 트랜잭션 쿼리 짜서 실행하는 듯
- 에노테이션
- AOP

#### @Transaction 어노테이션 이용하기

DB 설정 클래스에 `@EnableTransactionManagement` 어노테이션과 아래 메서드를 추가한다.

```java
@Bean
public PlatformTransactionManager transactionManager() throws Exception {
	return new DataSourceTransactionManager(dataSource());
}
```

그리고 트랜잭션 처리를 원하는 곳에 `@Transactional` 어노테이션을 추가하면 된다.

이렇게 간단하지만 새로운 클래스를 만들 때마다 `@Transactional` 어노테이션을 붙여 줘야 하므로 확장성이 떨어진다.

#### AOP로 트랜잭션 설정하기

`TransactionAspect` 클래스를 만들고 르랜잭션 이름, 롤백 룰, 포인트컷을 설정한다.

#### 두 방식의 차이

|      | `@Transaction 어노테이션`            | AOP                                      |
| ---- | ------------------------------------ | ---------------------------------------- |
| 장점 | 무설정                               | 트랜잭션 누락될 일 없음                  |
|      | 원하는 곳에만 설정, 성능 영향 최소화 | 외부 라이브러리도 적용 가능              |
| 단점 | 어노테이션 누락 가능                 | 필요없는 곳까지 트랜잭션 적용, 성능 영향 |
|      | 외부라이브러리에 적용 불가           | 원하는 곳에 트랜잭션 적용하기 어려움     |

### 예외처리하기

1. `try/catch` 이용
2. 각 컨트롤러단에서 `@ExceptionHandler` 이용 - 코드 중복 많아짐
3. `@ControllerAdvice`를 이용한 전역 예외처리

```java
@Slf4j
@ControllerAdvice
public class ExceptionHandler {

  @org.springframework.web.bind.annotation.ExceptionHandler(Exception.class)
  public ModelAndView defaultExceptionHandler(HttpServletRequest request, Exception exception) {
    ModelAndView mv = new ModelAndView("/error/error_default");

    mv.addObject("exception", exception);

    log.error("defaultExceptionHandler", exception);

    return mv;
  }
}
```

전역적으로 에러 발생 시 유저에게 에러 로그를 보여주는 에러 핸들러다.

여기선 모든 에러 처리를 하지만 실제 프로젝트에서는 다양한 에러에 맞는 각각의 에러처리 필요하다

추가로, 위와 같이 예외 로그를 화면에 노출시키면 프로그램의 취약점이 드러나 공격받을 수 있다.

### 헌글 처리를 위한 인코딩

스프링 부트 2.1.x 버전부터는 이미 인코딩 필터가 적용되있다.

굳이 적용하면 왜 적용하냐고 경고를 준다.

## 2019년 12월 19일

### 파일 업로드와 다운로드

스프링에는 파일 업로드를 위한 `MultipartResolver` 인터페이스가 정의되어 있어

파일 업로드 기능 구현시 아파치의 `CommonsMultipartResolver`나 서블릿 API의 `StandardServletMultipartResolver` 구현체를 사용하면 된다.

`CommonsMultipartResolver`를 구현하고 첨부파일 관련 구성에서 스프링의 특성인 자동구성이 되지 않게 하기 위해 아래 코드를 추가한다.

```java
@SpringBootApplication(exclude = { MultipartAutoConfiguration.class })
```

### 뷰 변경

폼으로 데이터 전송 시 파일도 같이 첨부되도록 `form` 태그에 `enctype="multopart/form-data"` 속성을 추가한다.

`type`이 `file`인 `input` 태그도 추가한다.

### 파일 업로드 유틸

1. 파일이 업로드될 폴더를 생성한다.
2. 파일확장자를 확인해서 서버에 저장될 파일 이름을 생성한다.(중복 방지를 위해 나노초
   사용)
3. BoardFileDto에 데이터베이스에 저장할 파일정보 담기
4. 업로드된 파일을 새로운 이름으로 바꾸어 지정된 경로에 저장

### 여러 개의 값 넣는 매퍼 구문

```xml
<insert id="insertBoardFileList" parameterType="board.board.dto.BoardFileDto">
	<![CDATA[
		INSERT INTO t_file
		(
			board_idx,
			original_file_name,
			stored_file_path,
			file_size,
			creator_id,
			created_datetime
		)
		VALUES
	]]>
	<foreach collection="list" item="item" separator=",">
		(
			#{item.boardIdx},
			#{item.originalFileName},
			#{item.storedFilePath},
			#{item.fileSize},
			'admin',
			NOW()
		)
	</foreach>
</insert>
```

`insert` 내부에 `foreach`를 사용하고 각 항목을 지정하는 별칭을 통해 데이터에
접근한다. `seperator`로 값 사이를 구분해준다.

### 첨부된 파일 목록 보여주기

1. 파일 목록을 조회하는 쿼리를 추가한다.
2. `BoardDto`에 fileList 속성을 추가한다.
3. 뷰에 th:each 속성으로 파일 리스트를 렌더링한다.

### 파일 다운로드

1. 파일 정보를 조회하는 쿼리를 추가한다.

쿼리 작성 시 파라미터 타입으로 `map`을 사용한다. 파라미터 전달만을 목적으로 DTO를 만들기 애매하기 때문이다.

매퍼 인터페이스에서 `@Param` 어노테이션을 사용하면 해당 파라미터들이 `Map`에 저장되어 쿼리에 파라미터로 전달할 수 있다.

2. 뷰에 다운로드 링크를 삽입한다.

함수를 호출하는것 같은 아래 코드는 렌더링되면

`th:href="@{/board/downloadBoardFile.do(idx=${list.idx}, boardIdx=${list.boardIdx})}"`

`/board/downloadBoardFile.do?idx=파일번호&boardIdx=글번호` 같이 파라미터가 추가되어 화면에 나타난다.

3. 파일의 바이너리 데이터를 사용자에게 전달한다.

컨트롤러 메서드에 `HttpServletResponse`를 파라미터로 설정하고 이를 적절히 설정하면 사용자에게 전달할 데이터를 원하는 대로 만들 수 있다.

DB에서 파일 정보를 가져오고, 파일 경로에서 파일을 읽고 `byte[]` 형태로 변환한다.

`response`의 헤더에 컨텐츠 타입, 크기, 형태 설정하고 파일 이름은 UTF-8로 인코딩한다.

헤더 작성 시 띄어쓰기와 대소문자를 주의한다.

바이트 배열을 `response`에 작성하고 버퍼를 정리 후 닫아준다.

## 2019년 12월 20일

### RESTful 게시판으로 변경하기

컨트롤러를 아래와 같이 작성한다.

| 기능              | 요청 방식 | URL           |
| ----------------- | --------- | ------------- |
| 게시판 목록       | GET       | /board        |
| 게시글 작성 화면  | GET       | /board/write  |
| 게시글 작성       | POST      | /board/write  |
| 게시글 상세 화면  | GET       | /board/글번호 |
| 게시글 수정       | PUT       | /board/글번호 |
| 게시글 삭제       | DELETE    | /board/글번호 |
| 첨부파일 다운로드 | GET       | /board/file   |

### `PUT`과 `DELETE` 방식 지원하기

`HTML`의 `form`은 `POST와` `GET` 방식의 요청만 지원하고 `PUT`과 `DELETE` 방식은 지원하지 않는다.

스프링(2.1.x)에는 `HiddenHttpMethodFilter` 필터가 이미 등록되어있다.

`_method`라는 이름의 파라미터가 존재할 경우 그 값을 요청 방식으로 사용한다.

자바스크립트로 아래의 요소의 `value`를 `PUT`이나 `DELETE`로 설정하면 된다.

```html
<input type="hidden" id="method" name="_method" />
```

### REST API로 변경하기

일반적으로 애플리케이션은 백엔드 서버와 클라이언트로 나뉜다.

API와 화면이 구분된 진정한 REST API를 구현해본다.

컨트롤러에 `@RestController` 어노테이션을 사용한다.

`@RestController`는 `@Controller`와 `@ResponseBody` 어노테이션을 합친 것이다. 해당 API의 응답결과를 웹 응답 바디를 이용해 JSON 형식으로 보내준다.

`@RequestBody` 어노테이션은 메서드 파리미터가 HTTP 패킷의 바디에 담겨 있어야 한다는 것을 나타낸다.

그래서 `POST`와 `PUT` 메서드는 `@RequestBody` 어노테이션을, `GET` 메서드는 `@RequestParam` 어노테이션을 사용한다.

## 2019년 12월 21일

### JPA란?

JPA(Java Persistence API)란 자바 객체와 DB 테이블 간의 매핑을 처리하는 ORM 기술의 표준이다.

JPA는 ORM 기능이 어떻게 동작되어야 한다는 것을 정의한 기술 명세이며 사용하기 위해서는 JPA의 구현체가 필요하다.

이런 구현체로는 하이버네이트, 이클립스링크 등이 있고 이를 JPA 프로바이더라고 한다.

실제 프로젝트에서는 하이버네이트를 많이 사용하지만 스프링과의 연동이 간단하지 않아 하이버네이트를 래핑한 스프링 데이터 JPA를 사용한다.

### JPA의 장점

- 개발이 편하다. 반복적으로 CRUD SQL을 직접 작성하지 않아도 된다.
- 데이터베이스에 독립적인 개발이 가능하다. DB가 바뀌어도 상관없다.
- 유지보수가 쉽다. 데이터 스키마 변경 시 JPA 엔티티만 수정하면 돼서 데이터테이블 변경이 쉽다.

### JPA의 단점

- 학습 곡선이 크다. SQL을 직접 작성하지 않아 튜닝이 어려울 수 있다.
- 특정 DB의 기능을 사용할 수 없다.
- 객체 위주의 설계보단 데이터베이스의 테이블에 맞게 객체나 로직이 설계되기 때문에 객체지향적 설계가 어려울 수 있다.

### 스프링 데이터 JPA

스프링에서 JPA를 쉽게 사용하도록 해 주는 라이브러리다.

`Repository`라는 인터페이스를 상속받아 규칙에 맞게 구현하면 된다.

### 기본 설정 추가

`application.properties`에 아래 설정을 추가한다.

```
spring.jpa.database=mysql
// 기본 엔진이 MyISAM이라 InnoDB로 변경한다.
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
// 실제 개발 시 꼭 false로 해야한다. 예상치 못하게 DB 변경 시 데이터가 삭제될 수 있기 떄문이다.
spring.jpa.generate-ddl=true
// MySQL의 자동 증가 속성을 사용하기에 필요 없어서 false
spring.jpa.hibernate.use-new-id-generator-mappings=false
```

`BoardApplication` 클래스에 아래 어노테이션을 추가해서 같이 `Jsr310JpaConverters` 클래스를 적용한다.

```java
@EntityScan(basePackageClasses = { Jsr310JpaConverters.class }, basePackages = { "board" })
```

DB의 `DATETIME`과 자바 8의 날짜 API를 문제없이 사용하기 위함이다.

### 엔티티 생성하기

`@Entity` 어노테이션으로 해당 클래스가 JPA 엔티티임을 나타낸다.

`@Table(name= "table_name")` 어노테이션으로 테이블 매핑이 되도록 한다.

`@Id` 어노테이션으로 기본키임을 나타낸다.

`@GeneratedValue(strategy = GenerationType.AUTO)` 어노테이션으로 기본키 생성 시 DB에서 제공하는 전략을 따른다.
MySQL은 자동증가, 오라클의 경우 시퀀스를 생성한다.

`@Column(nullable = false)` 어노테이션으로 칼럼에 `NOT NULL` 소서을 지정한다.

`@OneToMany` 어노테이션으로 1:N 관계를 표현한다.

`@JoinColumn` 어노테이션으로 릴레이션 관계가 있는 테이블의 칼럼을 지정한다.

### 리포지터리 작성하기

`CrudRepository` 인터페이스를 상속해서 리포지터리를 작성한다.

`CrudRepository` 인터페이스는 도메인 클래스와 도메인 id 타입을 파라미터로 받는다.

`findAllByOrderByBoardIdxDesc`와 같이 규칙에 맞게 메서드 이름을 추가하면 실행 시 메서드의 이름에 따라 쿼리가 생성되어 실행된다.

`@Query` 어노테이션으로 실행하고 싶은 쿼리를 직접 정의할 수 있다.

나머지 필요한 메서드(`save`, `find`, `delete`, ...)는 `CrudRepository`에 이미 정의되어서 따로 정의할 필요없다.

### 서비스 작성하기

리포지터리의 `save` 메서드는 `insert`와 `update` 두 가지 역할을 수행한다.

저장할 내용이 새로 생성된 내용이면 `insert`를, 기존의 내용에서 변경되었을 경우 `update`를 수행한다.

### 스프링 데이터 JPA 리포지터리 인터페이스

```
Repository<T, ID>
		 ↑ extends
CrudRepository<T, ID>
		 ↑ extends
PagingAndSortingRepository<T, ID>
		 ↑ extends
JpaRepository<T, ID>
```

`Repository`는 아무런 기능이 없어서 잘 안 쓴다.

`CrudRepository`는 CRUD 기능을 제공한다.

`PagingAndSortingRepository`는 `CrudRepository` + 페이징 및 정렬 기능이 추가되어 있다.

`JpaRepository`는 JPA에 특화된 기능이 추가되어 있다.

### 쿼리 메서드

규칙에 메서드를 추가하면 그 메서드 이름으로 쿼리를 생성하는 기능이다.

쿼리 메서드는 `find...By`, `read...By`, `query...By`, `count...By`, `get...By`로 시작해야 한다.

`By`뒤는 컬럼 이름으로 구성된다. (ex, `findByTitle(String title)`)

두 개 이상의 속성을 조합하려면 `And` 키워드를 사용한다.

스프링 JPA에서 제공하는 비교연산자 목록은 [여기](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)서 찾아 볼 수 있다.

### `@Query` 사용하기

1. [?숫자] 형식으로 파라미터를 지정한다. 순서대로 파라미터가 지정된다.

2. :[변수이름]으로 파라미터를 지정한다. 변수이름은 메서드의 `@Param` 어노테이션에 대응된다. `:boardIdx`의 `boardIdx` 변수는 `@Param("boardIdx")` 어노테이션이 있는 메서드의 파라미터를 사용한다.

파라미터 개수가 많아지거나 쿼리의 길이가 많이질 경우 쿼리, 메서드 파라미터를 알아보기 힘들고 파라미터 순서를 바꿔 입력하는 실수를 할 수 있기에 두 번째 방법을 사용하는 것이 좋다.
