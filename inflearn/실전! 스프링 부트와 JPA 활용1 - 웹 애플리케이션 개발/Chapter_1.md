
## 1. 프로젝트 환경설정

### 1.1 프로젝트 생성

> 201013 (Tue)

* Pass


---

### 1.2 라이브러리 살펴보기

> 201013 (Tue)

* gradle 의존관계 보기

``` gradle
./gradlew dependencies
```


---

### 1.3 View 환경 설정

> 201013 (Tue)

* 스프링 부트에서 기본적인 thymeleaf viewName 환경

``` java
Prefix : resources:templates/
Suffix : .html

resources:templates/ + {ViewName} + .html
```

* spring-boot-devtools 라이브러리를 추가 시 

  수정된 html 파일을 Recompile 해주면 서버 재시작 없이 View 파일 변경이 가능하다.

> Intellij 

```
Menu -> build -> Recompile
```


---

### 1.4 H2 데이터베이스 설치

> 201013 (Tue)

* Pass

---

### 1.5 JPA와 DB 설정, 동작확인

> 201013 (Tue)

#### JPA와 DB 설정

> main/resources/application.yml

``` yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/jpashop
    username: sa
    password:
    driver-class-name: org.h2.Driver
 
  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        # show_sql: true // Real에서는 사용 X 권장
        format_sql: true

  logging.level:
    org.hibernate.SQL: debug
    # org.hibernate.type: trace
```

* 모든 로그 출력은 가급적 **logger**를 통해 남기는게 좋다.

  show_sql : System.out 에 하이버네이트 실행 SQL을 남긴다.

  org.hibernate.SQL : logger를 통해 하이버네이트 실행 SQL을 남긴다.

---

#### 쿼리 파라미터 로그로 남기기

* 따로 설정을 해주지 않으면 

  SQL에서 파라미터 부분에는 **?** 로 로그가 찍히게 된다.

  해당 파라미터 값을 보고 싶을 경우엔 추가적으로 설정이 필요하다.

  그 방법으로 2가지를 알아본다.

---

##### 1. org.hibernate.type

> main/resources/application.yml

``` yml
spring:
  logging.level:
    org.hibernate.SQL: debug
    org.hibernate.type: trace
```

* org.hibernate.type값을 trace로 준다.


---

##### 2. 외부 라이브러리

* [https://github.com/gavlyukovskiy/spring-boot-data-source-decorator](https://github.com/gavlyukovskiy/spring-boot-data-source-decorator) 라이브러리를 사용한다.

``` java
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.6.2'
```

* 주의할 점은 개발 단계에서는 편하게 사용해도 되지만
  
  하지만 운영시스템에 적용하려면 꼭 성능테스트를 하고 사용해야한다.
