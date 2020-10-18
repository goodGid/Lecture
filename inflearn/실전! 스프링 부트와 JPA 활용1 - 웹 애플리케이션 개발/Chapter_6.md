
## 6. 주문 도메인 개발

### 6.1 주문, 주문상품 엔티티 개발

> 201018 (Sun)

* Pass

---

### 6.2 주문 리포지토리 개발

> 201018 (Sun)

* Pass

---

### 6.3 주문 서비스 개발

> 201018 (Sun)

<h2> [도메인 모델 패턴](http://martinfowler.com/eaaCatalog/domainModel.html)</h2>

* 비즈니스 로직 대부분이 엔티티에 있다. 

  Service Layer에서 단순히 엔티티에 필요한 **요청을 위임**하는 역할을 한다. 

* 이처럼 엔티티가 비즈니스 로직을 가지고 
  
  객체 지향의 특성을 적극 활용하는 것을 **[도메인 모델 패턴](http://martinfowler.com/eaaCatalog/domainModel.html)**이라 한다. 

---

<h2> 트랜잭션 스크립트 패턴(http://martinfowler.com/eaaCatalog/transactionScript.html) </h2>

* 도메인 모델 패턴과는 반대로 

  엔티티에는 비즈니스 로직이 거의 없고 
  
  Service Layer에서 대부분의 비즈니스 로직을 처리하는 것을 **[트랜잭션 스크립트 패턴](http://martinfowler.com/eaaCatalog/transactionScript.html)**이라 한다.

---


### 6.4 주문 기능 테스트

> 201018 (Sun)

* Pass


---

### 6.5 주문 검색 기능 개발

> 201018 (Sun)

* **동적 쿼리**는 QueryDSL을 사용하자.
