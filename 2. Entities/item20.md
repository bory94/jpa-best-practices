## Item20. Aggregate Root에서 도메인 이벤트를 게시하는 가장 좋은 방법

직접 @DomainEvents와 @AfterDomainEventPublication 메소드를 구현해도 되지만 AbstractAggregateRoot를 상속받는 것이 더 효과적이다.

> 주의
> 
> AbstractAggregateRoot를 상속하면 registerEvent로 바로 이벤트를 등록할 수 있지만, repository의 save 계열 메소드가 호출되어야만 등록된 이벤트가 게시된다.

트랜잭션과 관련된 이벤트라면 @TransactionalEventListener를 이용하라.
> 이 애너테이션의 phase 프러퍼티 기본값은 TransactionPhase.AFTER_COMMIT이다.

---

#### 동기적 실행<sub>Synchronous Execution</sub>
일반적으로 도메인 이벤트를 이용하는 경우 TransactionPhase.AFTER_COMMIT을 이용하지만, BEFORE_COMMIT이나 AFTER_COMPLETION 등 상황에 맞는 phase를 사용할 수 있다.

> 참고: 진행 중인 트랜잭션이 없다면 @TransactionEventListener 메소드는 실행되지 않는다. 
> 
> 실행되게 하려면 fallbackExecution이 true로 설정되어야 한다.

AFTER_COMMIT을 이용한다면 명시적으로 진행 중인 트랜잭션 컨텍스트가 없기 때문에, 별도의 트랜잭션에서 실행될 것 같지만, <b>실제로는 그렇지 않다.</b>
별도의 지시가 없으면 기존과 동일한 트랜잭션 컨텍스트에서 실행이 되지만, 해당 트랜잭션은 이미 COMMIT이 되었기 떄문에 이벤트 리스너의 처리를 데이터베이스로 COMMIT 되지 않는다.
그래서 이 경우, 다음과 같이 별도의 트랜잭션 컨텍스트를 이용하도록 지정해야 한다.

```java
@TransactionalEventListener
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void handleCheckReviewEvent(CheckReviewEvent event) {
    ...
}
```

<b>그런데 이렇게 할 경우 또 다른 문제(?)가 발생한다.</b>

- 기존의 트랜잭션이 실행 연기<sub>suspended</sub> 상태가 되고 이벤트 리스너의 처리가 완료되길 기다린다.
- 새로운 트랜잭션 컨텍스트가 실행되므로 새로운 데이터베이스 커넥션이 사용된다.
- 이벤트 리스너의 처리가 완료(COMMIT)되면 새로운 데이터베이스 커넥션이 반환된 후 기존 트랜잭션의 실행이 재개<sub>Resumed</sub>되고 처리 완료된다.
- 기존 트랜잭션을 위한 데이터베이스 커넥션이 반환된다.

즉, <b>두 개의 Long-Running 트랜잭션이 생성된다.</b>

이 문제를 해결하려면 AFTER_COMMIT 대신 BEFORE_COMMIT을 이용해야 한다.
```java
@TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT)
public void handleCheckReviewEvent(CheckReviewEvent event) {
    ...
}
```

하지만 이 방법이 반드시 옳은 것은 아니다. 기존 트랜잭션이 COMMIT 된 후에 이벤트 리스너가 실행되길 바라는 경우에는 사용할 수 없는 옵션이기 때문이다.

마지막으로, AFTER_COMMIT을 이용하면서 기존 트랜잭션이 먼저 종료되도록 하는 방법이 있다. (item60)
```properties
# applicaiton.properties에서 auto-commit 관련된 내용들을 모두 false로 지정한다.
spring.datasource.hikari.auto-commit=false
spring.jpa.properties.hibernate.connection.provider_disables_autocommit=true
```
```java
// 그리고 그냥 Propagation.REQUIRES_NEW를 이용한다.
@TransactionalEventListener
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void handleCheckReviewEvent(CheckReviewEvent event) {
	...
}
```

---

#### 비동기적 실행<sub>Asynchronous Execution</sub>

Spring Boot에 @EnableAsync 설정을 하고 TransactionEventListener 메소드에 @Async 애너테이션을 설정하면 비동기 처리가 수행되며 위에서 말한 성능 이슈가 해결될 수 있다.
```java
@Async // <-- 이 애너테이션으로 인해 이 메소드의 실행은 별도 스레드에서 별도 트랜잭션 컨텍스트를 이용하여 비동기적으로 처리된다.
@TransactionalEventListener
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void handleCheckReviewEvent(CheckReviewEvent event) {
    ...
}
```

비동기 처리와 관련하여 주의할 점은 다음과 같다.
- TransactionPhase.AFTER_COMMIT이 목적이라면 비동기 처리를 하는 것이 더 낫다. 
- 만약 이벤트 핸들러의 작업이 데이터베이스 트랜잭션과 관련이 없다면 @Transactional을 사용하지 말라.
- 만약 (읽기든 쓰기든) 데이터베이스 트랜잭션과 관련이 있다면 Propagation.REQUIRES_NEW를 사용해야 한다.
- 데이터베이스에서 읽기만 한다면 readOnly=true 설정도 추가하라.
- 비동기 처리에서는 TransactionPhase.BEFORE_COMMIT을 사용하면 안 된다.
- 상황에 따라 이벤트 핸들러 스레드의 실행 완료를 가로채야<sub>intercept</sub> 할 수도 있다.

동기 처리와 관련하여 주의할 점은 다음과 같다.
- 비동기 처리를 고려하라.
- TransactionPhase.BEFORE_COMMIT은 Long-Running 트랜잭션을 유발한다.
- TransactionPhase.AFTER_COMPLETION은 빠르게 처리될 수 있는 작업이거나 데이터베이스 쓰기 작업이 없을 떄에만 사용하라.
- TransactionPhase.BEFORE_COMMIT을 이용할 때, 이벤트 핸들러 내에서 예외가 발생하면 전체 트랜잭션이 ROLLBACK 된다.

---

> 주의
> 
> - 도메인 이벤트는 Spring Data를 이용할 때에만 사용할 수 있다.
> - 도메인 이벤트는 Repository의 save 메소드를 호출할 때에만 게시된다.
> - 이벤트를 게시하는 과정에서 예외가 발생한다면 이벤트 리스너는 실행되지 않는다. 이 경우, <b>이벤트가 유실된다.</b>