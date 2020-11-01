
## 5. 스프링 데이터 JPA 분석

### 5.1 스프링 데이터 JPA 구현체 분석

> 201101 (Sun)

<h2> @Repository </h2>

* JPA 예외를 Spirng이 추상화한 예외로 반환한다.

---

<h2> @Transactional </h2>

* JPA의 모든 변경은 **@Transactional** 안에서 동작해야한다.

* 그런데 Spring Data JPA를 사용할 때 

  따로 @Transactional을 선언하지 않고 사용한다.

  그럼에도 정상적으로 동작한다.

> Why? 

* Repository에 @Transactional가 적용되어 있다.

> SimpleJpaRepository

``` java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
    ...
}
```

* 서비스 계층에서 트랜잭션을 시작하지 않으면 Repository에서 트랜잭션이 시작된다.

* 서비스 계층에서 트랜잭션을 시작하면 Repository에서 트랜잭션을 전파 받아서 사용한다.

---

### 5.2 새로운 엔티티를 구별하는 방법

> 201101 (Sun)

* JPA 식별자 생성 전략이 @GenerateValue면 

  save()를 호출하는 시점에 식별자가 없으므로 새로운 엔티티로 인식해서 정상 동작한다.
   
* 그런데 JPA 식별자 생성 전략이 

  @Id만 사용해서 직접 할당이면 이미 식별자값이 있는 상태로 save() 를 호출한다. 
  
  따라서 이 경우 merge()가 호출된다. 
  
* merge() 는 우선 DB를 호출해서 값을 확인하고 

  DB에 값이 없으면 새로운 엔티티로 인지하므로 매우 비효율 적이다. 
  
  따라서 **Persistable**를 사용해서 새로운 엔티티 확인 여부를 직접 구현하게는 효과적이다.

* 참고로 등록시간(@CreatedDate)을 조합해서 사용하면 

  해당 필드로 새로운 엔티티 여부를 편리하게 확인할수 있다. 
  
  (= @CreatedDate에 값이 없으면 새로운 엔티티로 판단한다.)

> Persistable

``` java
package org.springframework.data.domain;

public interface Persistable<ID> {

	/**
	 * Returns the id of the entity.
	 *
	 * @return the id. Can be {@literal null}.
	 */
	@Nullable
	ID getId();

	/**
	 * Returns if the {@code Persistable} is new or was persisted already.
	 *
	 * @return if {@literal true} the object is new.
	 */
	boolean isNew();
}
```