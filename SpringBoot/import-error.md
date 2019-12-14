# 스프링 부트 프로젝트 생성 후 org.springframework.web을 못 찾을 때

build.gradle 파일, dependencies에 아래 코드를 추가하면 `org.springframework.web` 패키지를 못 찾는 에러를 고칠 수 있다.

```groovy
compile("org.springframework.boot:spring-boot-starter-web")
```
