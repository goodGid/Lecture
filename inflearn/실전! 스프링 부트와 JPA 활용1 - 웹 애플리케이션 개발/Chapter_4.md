
## 4. 회원 도메인 개발

### 4.1 회원 리포지토리 개발

> 201018 (Sun)

* Pass

---

### 4.2 회원 서비스 개발

> 201018 (Sun)

<h2> readOnly() </h2>

* Read를 하기 위한 Method에는 **readOnly** 속성을 true로 주는게 성능상 이점이 있다.

> Docs : readOnly

``` java
	/**
	 * A boolean flag that can be set to {@code true} if the transaction is
	 * effectively read-only, allowing for corresponding optimizations at runtime.
	 * <p>Defaults to {@code false}.
	 * <p>This just serves as a hint for the actual transaction subsystem;
	 * it will <i>not necessarily</i> cause failure of write access attempts.
	 * A transaction manager which cannot interpret the read-only hint will
	 * <i>not</i> throw an exception when asked for a read-only transaction
	 * but rather silently ignore the hint.
	 * @see org.springframework.transaction.interceptor.TransactionAttribute#isReadOnly()
	 * @see org.springframework.transaction.support.TransactionSynchronizationManager#isCurrentTransactionReadOnly()
	 */
	boolean readOnly() default false;
```

> Example : Service

``` java
@Service
@Transactional(readOnly = true) // readOnly = true 넣어주는게 성능상 좋다.
public class MemberService {

    @Autowired
    private MemberRepository memberRepository;

    /**
     * [주의]
     * @Transactional 선언을 override 해주지 않으면
     * readOnly 속성에 의해서 insert가 되지 않는다.
     */
    @Transactional
    public Long join(Member member) {
        memberRepository.save(member);
        return member.getId();
    }

    public List<Member> findMembers() {
        return memberRepository.findAll();
    }

    public Member findOne(Long memberId) {
        return memberRepository.findById(memberId);
    }

}
```
 
---

### 4.3 회원 기능 테스트

> 201018 (Sun)

<h2> Spring Boot Test </h2>

* 실제 DB에 Data Access가 발생하면 좋지 않다.

  그러므로 In-memory DB로 실제 DB에는 영향이 없는 테스트를 진행하는게 좋다.

> main/resources/application.yml

``` yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/jpashop # 실제 DB URL
    username: sa
    password:
    driver-class-name: org.h2.Driver
```

> test/resources/application.yml

``` yml
spring:
  datasource:
    url: jdbc:h2:mem:test # In-memory DB URL
    username: sa
    password:
    driver-class-name: org.h2.Driver
```

* 실제 DB를 가리키는 URL이 아니라

  Test 상황에서는 **jdbc:h2:mem:test**라는 In-momory DB를 사용하여 Test가 가능하다.

  ref : https://www.h2database.com/html/cheatSheet.html

* 하지만 Spring Boot에서는

  해당 설정이 없어도 **자동으로** jdbc:h2:mem:test를 사용한다.

  즉 *test/resources/application.yml* 파일이 없어도 In-memory로 Test가 동작한다.