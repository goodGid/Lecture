
## 3. API 개발 고급 - 지연 로딩과 조회 성능 최적화

### 3.1 간단한 주문 조회 V1: 엔티티를 직접 노출

> 201020 (Tue)

<h2> 주의 사항 </h2>

* 엔티티를 직접 노출 시

  **양방향 연관관계**가 걸린 곳은 
  
  꼭 한 곳을 **@JsonIgnore** 처리 해야 한다.
  
  안그러면 양쪽을 서로 호출하면서 무한 루프가 걸린다.

* 엔티티를 API 응답으로 외부로 노출하는 것은 좋지 않다. 

  따라서 Hibernate5Module를 사용하기 보다는 
  
  DTO로 변환해서 반환하는 것이 더 좋은 방법이다.

``` java
// Config
@Bean
Hibernate5Module hibernate5Module() {
Hibernate5Module hibernate5Module = new Hibernate5Module();
hibernate5Module.configure(Feature.FORCE_LAZY_LOADING, true);
return hibernate5Module;
}

// build.gradle
implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5'
```

---

### 3.2 간단한 주문 조회 V2: 엔티티를 DTO로 변환

> 201024 (Sat)

* Pass

---

### 3.3 간단한 주문 조회 V3: 엔티티를 DTO로 변환 - 페치 조인 최적화

> 201024 (Sat)

* fetch 조인을 통해 N+1 문제를 1번의 SQL로 해결가능.

> Example

``` java
em.createQuery(
        "select o from Order o"
        + " join fetch o.member m"
        + " join fetch o.delivery d", Order.class
).getResultList()
```

---

### 3.4 간단한 주문 조회 V4: JPA에서 DTO로 바로 조회

> 201024 (Sat)

* Pass