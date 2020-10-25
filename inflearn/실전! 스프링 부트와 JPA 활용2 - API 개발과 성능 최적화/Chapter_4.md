
## 4. API 개발 고급 - 컬렉션 조회 최적화

### 4.1 주문 조회 V1: 엔티티 직접 노출

> 201024 (Sat)

* Pass

---

### 4.2 주문 조회 V2: 엔티티를 DTO로 변환

> 201024 (Sat)

* Pass

---

### 4.3 주문 조회 V3: 엔티티를 DTO로 변환 - 페치 조인 최적화

> 201024 (Sat)

<h2> distinct </h2>

* 모든 상황의 1:N 경우이다.

  1:N이 아닌 경우엔 fetch join으로 대부분 해결되므로 크게 신경 쓰지 않아도 된다.

* **distinct**는 DB에게 distinct 속성을 알려주고 결과 데이터를 받는다.

  그리고 Application Level에서 자체적으로 

  PK가 같다면 중복되는 값들에 대해서 List로 생성하여 넘겨준다.

  // 이해가 안 된다면 강의를 다시 들어보자!

> Example

``` java
public List<Order> findAllWithItem() {
    return em.createQuery("select distinct o from Order o"
                          + " join fetch o.member m"
                          + " join fetch o.delivery d"
                          + " join fetch o.orderItems oi"
                          + " join fetch oi.item i", Order.class)
              .getResultList();
}
```

<h2> 단점 </h2>

* 1:N 관계의 DB를 조회 시 데이터 뻥튀기가 된다.

  그걸 Apllication Level에서 JPA가 똑똑하게 중복을 제거해주는데

  여기에 페이징 기능을 사용하려고 하면 동작하지 않는다.

  마치 JPA가 중복은 어찌해줄 수 있지만 페이징은 못 해준다고 이해하면 될 듯싶다.

* 그런데도 사용한다면 Hibernate는 Warn Log를 남기면서 

  읽어온 데이터를 메모리에서 페이징해버린다.

  만약 데이터가 엄청나게 많을 땐 OOM이 발생할 수도 있다.

* 또한 Collection Fetch Join은 1개의 1:N에 대해서만 사용할 수 있다.

  만약 1:N:M과 같은 관계라면 절대로 사용하면 안 된다.

  설령 동작하더라도 부적합하게 조회될 수 있다.

``` java
public List<Order> findAllWithItem() {
    return em.createQuery("select distinct o from Order o"
                          + " join fetch o.member m"
                          + " join fetch o.delivery d"
                          + " join fetch o.orderItems oi"
                          + " join fetch oi.item i", Order.class)
              .setFirstResult(1)
              .setMaxResults(100)
              .getResultList();
}

// Output
2020-10-24 22:32:47.888  WARN 51727 --- [nio-8080-exec-2] o.h.h.internal.ast.QueryTranslatorImpl   : HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!
```

---

### 4.4 주문 조회 V3.1: 엔티티를 DTO로 변환 - 페이징과 한계 돌파

> 201024 (Sat)

<h2> default_batch_fetch_size </h2>

* DB간 관계는 다음과 같다.

```
Order : OrderItem = 1 : N
OrderItem : Item = N : 1
```

> application.yml

``` java
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
        format_sql: true
        default_batch_fetch_size: 10 // Set `default_batch_fetch_size` value
```

* **default_batch_fetch_size** 값을 설정해준다.

  그리고 설정 유무에 따른 차이를 알아보자.

> default_batch_fetch_size 설정 X

``` java
2020-10-25 00:47:15.908 DEBUG 54587 --- [nio-8080-exec-1] org.hibernate.SQL                        : 
    select
        order0_.order_id as order_id1_6_0_,
        member1_.member_id as member_i1_4_1_,
        delivery2_.delivery_id as delivery1_2_2_,
        order0_.delivery_id as delivery4_6_0_,
        order0_.member_id as member_i5_6_0_,
        order0_.order_date as order_da2_6_0_,
        order0_.status as status3_6_0_,
        member1_.city as city2_4_1_,
        member1_.street as street3_4_1_,
        member1_.zipcode as zipcode4_4_1_,
        member1_.name as name5_4_1_,
        delivery2_.city as city2_2_2_,
        delivery2_.street as street3_2_2_,
        delivery2_.zipcode as zipcode4_2_2_,
        delivery2_.status as status5_2_2_ 
    from
        orders order0_ 
    inner join
        member member1_ 
            on order0_.member_id=member1_.member_id 
    inner join
        delivery delivery2_ 
            on order0_.delivery_id=delivery2_.delivery_id limit ?
2020-10-25 00:47:15.922 DEBUG 54587 --- [nio-8080-exec-1] org.hibernate.SQL                        : 
    select
        orderitems0_.order_id as order_id5_5_0_,
        orderitems0_.order_item_id as order_it1_5_0_,
        orderitems0_.order_item_id as order_it1_5_1_,
        orderitems0_.count as count2_5_1_,
        orderitems0_.item_id as item_id4_5_1_,
        orderitems0_.order_id as order_id5_5_1_,
        orderitems0_.order_price as order_pr3_5_1_ 
    from
        order_item orderitems0_ 
    where
        orderitems0_.order_id=?
2020-10-25 00:47:15.932 DEBUG 54587 --- [nio-8080-exec-1] org.hibernate.SQL                        : 
    select
        item0_.item_id as item_id2_3_0_,
        item0_.name as name3_3_0_,
        item0_.price as price4_3_0_,
        item0_.stock_quantity as stock_qu5_3_0_,
        item0_.artist as artist6_3_0_,
        item0_.etc as etc7_3_0_,
        item0_.author as author8_3_0_,
        item0_.isbn as isbn9_3_0_,
        item0_.actor as actor10_3_0_,
        item0_.director as directo11_3_0_,
        item0_.dtype as dtype1_3_0_ 
    from
        item item0_ 
    where
        item0_.item_id=?
2020-10-25 00:47:15.933 DEBUG 54587 --- [nio-8080-exec-1] org.hibernate.SQL                        : 
    select
        item0_.item_id as item_id2_3_0_,
        item0_.name as name3_3_0_,
        item0_.price as price4_3_0_,
        item0_.stock_quantity as stock_qu5_3_0_,
        item0_.artist as artist6_3_0_,
        item0_.etc as etc7_3_0_,
        item0_.author as author8_3_0_,
        item0_.isbn as isbn9_3_0_,
        item0_.actor as actor10_3_0_,
        item0_.director as directo11_3_0_,
        item0_.dtype as dtype1_3_0_ 
    from
        item item0_ 
    where
        item0_.item_id=?
2020-10-25 00:47:15.934 DEBUG 54587 --- [nio-8080-exec-1] org.hibernate.SQL                        : 
    select
        orderitems0_.order_id as order_id5_5_0_,
        orderitems0_.order_item_id as order_it1_5_0_,
        orderitems0_.order_item_id as order_it1_5_1_,
        orderitems0_.count as count2_5_1_,
        orderitems0_.item_id as item_id4_5_1_,
        orderitems0_.order_id as order_id5_5_1_,
        orderitems0_.order_price as order_pr3_5_1_ 
    from
        order_item orderitems0_ 
    where
        orderitems0_.order_id=?
2020-10-25 00:47:15.935 DEBUG 54587 --- [nio-8080-exec-1] org.hibernate.SQL                        : 
    select
        item0_.item_id as item_id2_3_0_,
        item0_.name as name3_3_0_,
        item0_.price as price4_3_0_,
        item0_.stock_quantity as stock_qu5_3_0_,
        item0_.artist as artist6_3_0_,
        item0_.etc as etc7_3_0_,
        item0_.author as author8_3_0_,
        item0_.isbn as isbn9_3_0_,
        item0_.actor as actor10_3_0_,
        item0_.director as directo11_3_0_,
        item0_.dtype as dtype1_3_0_ 
    from
        item item0_ 
    where
        item0_.item_id=?
2020-10-25 00:47:15.936 DEBUG 54587 --- [nio-8080-exec-1] org.hibernate.SQL                        : 
    select
        item0_.item_id as item_id2_3_0_,
        item0_.name as name3_3_0_,
        item0_.price as price4_3_0_,
        item0_.stock_quantity as stock_qu5_3_0_,
        item0_.artist as artist6_3_0_,
        item0_.etc as etc7_3_0_,
        item0_.author as author8_3_0_,
        item0_.isbn as isbn9_3_0_,
        item0_.actor as actor10_3_0_,
        item0_.director as directo11_3_0_,
        item0_.dtype as dtype1_3_0_ 
    from
        item item0_ 
    where
        item0_.item_id=?
```


> default_batch_fetch_size 설정 O

``` java
2020-10-25 00:44:26.304 DEBUG 54463 --- [nio-8080-exec-3] org.hibernate.SQL                        : 
    select
        order0_.order_id as order_id1_6_0_,
        member1_.member_id as member_i1_4_1_,
        delivery2_.delivery_id as delivery1_2_2_,
        order0_.delivery_id as delivery4_6_0_,
        order0_.member_id as member_i5_6_0_,
        order0_.order_date as order_da2_6_0_,
        order0_.status as status3_6_0_,
        member1_.city as city2_4_1_,
        member1_.street as street3_4_1_,
        member1_.zipcode as zipcode4_4_1_,
        member1_.name as name5_4_1_,
        delivery2_.city as city2_2_2_,
        delivery2_.street as street3_2_2_,
        delivery2_.zipcode as zipcode4_2_2_,
        delivery2_.status as status5_2_2_ 
    from
        orders order0_ 
    inner join
        member member1_ 
            on order0_.member_id=member1_.member_id 
    inner join
        delivery delivery2_ 
            on order0_.delivery_id=delivery2_.delivery_id limit ?
2020-10-25 00:44:26.307 DEBUG 54463 --- [nio-8080-exec-3] org.hibernate.SQL                        : 
    select
        orderitems0_.order_id as order_id5_5_1_,
        orderitems0_.order_item_id as order_it1_5_1_,
        orderitems0_.order_item_id as order_it1_5_0_,
        orderitems0_.count as count2_5_0_,
        orderitems0_.item_id as item_id4_5_0_,
        orderitems0_.order_id as order_id5_5_0_,
        orderitems0_.order_price as order_pr3_5_0_ 
    from
        order_item orderitems0_ 
    where
        orderitems0_.order_id in (
            ?, ?
        )
2020-10-25 00:44:26.309 DEBUG 54463 --- [nio-8080-exec-3] org.hibernate.SQL                        : 
    select
        item0_.item_id as item_id2_3_0_,
        item0_.name as name3_3_0_,
        item0_.price as price4_3_0_,
        item0_.stock_quantity as stock_qu5_3_0_,
        item0_.artist as artist6_3_0_,
        item0_.etc as etc7_3_0_,
        item0_.author as author8_3_0_,
        item0_.isbn as isbn9_3_0_,
        item0_.actor as actor10_3_0_,
        item0_.director as directo11_3_0_,
        item0_.dtype as dtype1_3_0_ 
    from
        item item0_ 
    where
        item0_.item_id in (
            ?, ?, ?, ?
        )
```

* Orderitem과 Item을 조회하는 Query의 Where 절에서 **in**이 사용되는 걸 볼 수 있다.