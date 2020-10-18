
## 2. 도메인 분석 설계

### 2.1 요구사항 분석

> 201014 (Wed)

* Pass

---

### 2.2 도메인 모델과 테이블 설계

> 201014 (Wed)

* Pass

---

### 2.3 엔티티 클래스 개발1

> 201015 (Thu)

* Pass


---


### 2.4 엔티티 클래스 개발2

> 201015 (Thu)

* Pass


---


### 2.5 엔티티 설계시 주의점

> 201015 (Thu)

<h2> Lazy Fetch </h2>

* N+1 문제를 방지하기 위해서는 LAZY로 Fetch 해야한다.

* xToMany 는 기본이 **LAZY** Fetch이다.

``` java
public @interface OneToMany {
    FetchType fetch() default LAZY;
}

public @interface ManyToMany {
    FetchType fetch() default LAZY;
}
```

* xToOne은 기본이 **EAGER** Fetch이다.

  그러므로 xToOne 사용 시 Fetch 전략을 LAZY로 수정해주자.

``` java
public @interface OneToOne {
    FetchType fetch() default EAGER;
}

public @interface ManyToOne {
    FetchType fetch() default EAGER;
}
```

---

<h2> SpringPhysicalNamingStrategy </h2>

* **SpringPhysicalNamingStrategy**에 의해서 

  Entity에서 사용하는 필드명과 DB의 필드명은 다를 수 있다.

```
1. 카멜 케이스 -> 언더스코어 (ex. memberPoint -> member_point)
2. .(점) -> _(언더스코어)
3. 대문자 -> 소문자
```

> Example

``` java
@Entity
@Getter
@Setter
public class OrderItem {
    private BigDecimal orderPrice;
}
```

* 위와 같이 *orderPrice* 라고 선언을 하여도 

  기본 전략에 의해 실제 DB에는 

  **order_price**로 컬럼이 생성된다.

---

<h2> CascadeType </h2>

* Order : OrderItem = 1 : N 이라고 할 때

  CascadeType.ALL 설정을 주면

  OrderItem 값을 하나하나 persist 해줄 필요가 없다.

> Entity

``` java
@Entity
@Table(name = "orders")
@Getter
@Setter
public class Order {
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();
}
```

> CascadeType.ALL 설정 유무

``` java
// Not CascadeType.ALL
persist(orderItemA)
persist(orderItemB)
persist(orderItemC)
persist(order)

// CascadeType.ALL
persist(order)
// 자동으로 persist(orderItemA, orderItemB, orderItemC) 작업을 해준다. 
```
