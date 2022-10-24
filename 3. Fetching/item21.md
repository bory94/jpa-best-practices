## Item21. 직접 가져오기

> 엔티티를 ID로 조회하는 것을 의미한다.

조회하려는 엔티티의 ID를 알고 있고, 연관 엔티티들이 로딩될 필요가 없는 경우 가장 선호되는 조회 방법이다.

> 어떤 연관관계든 FetchType.LAZY로 설정하고 필요한 경우 별도의 수동 조회 전략을 사용하는 것이 바람직하다. (item39, item41, item32) 

다음 세 가지 방법으로 직접 가져오기를 수행할 수 있다.

- Spring Repository의 findById(id)
- EntityManager의 find(Class, id)
- Hibernate Session을 이용하기 위한 EntityManager.unwrap() + Session.get(id)

> 어떤 방식이든 다음 순서로 조회한다.
> 1. 현재 영속성 컨텍스트
> 2. Second Level Cache
> 3. 데이터베이스

---

#### 직접 가져오기와 세션 레벨 Repeatable-Read

Hibernate가 현재 영속성 컨텍스트를 가장 먼저 확인하는 이유는 세션 레벨에서의 Repeatable-Read를 보장하기 위함이다.

> Repeatable-Read에 대한 상세한 내용은 [이 글](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)을 참고하라.
> 
> 참고로, MySQL의 기본 격리 수준<sub>Isolation Level</sub>은 REPEATABLE READ이다.

---

### 여러 개의 ID로 여러 개의 엔티티를 직접 조회하기

여러 개의 ID를 이용하는 경우 다음 네 가지 방법을 이용할 수 있다.
- Spring Data의 findAllById(List<ID> ids)를 이용
- Spring Repository에 @Query를 이용하여 메소드 생성
- Specification 이용 (생략)
- Hibernate의 MultiIdentifierLoadAccess 인터페이스 사용 (생략)

```java
// findAllById를 이용하기
List<Book> books = bookRepository.findAllById(List.of(1L, 2L, 5L));

// Repository에 @Query를 이용한 새로운 메소드 등록하기
@Query("SELECT b FROM Book b WHERE b.id IN ?1")
List<Book> fetchByMultipleIds(List<Long> ids);
```
