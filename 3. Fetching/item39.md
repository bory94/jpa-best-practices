## Item39. 하나의 SELECT 만으로 부모와 연관 관계 데이터를 효과적으로 가져오는 방법

> 전제: Author와 Book이 양방향 Lazy @OneToMany 관계로 연결되어 있다. 
> 
> 두 엔티티 모두 조회 후 데이터를 변경하려 한다.

이런 상황에서 어떤 방향(부모 측, 자식 측)에서 조회하든 연관 엔티티를 조회하기 위해 두 개의 쿼리가 수행되어야 한다. 
하지만 이런 방식은 두 가지 단점이 있다.

- 두 번의 SELECT 문이 실행된다. 
- Lazy 연관 데이터 조회를 위해 활성화된 Hibernate 세션이 필요하다. 그렇지 않다면 LazyInitializationException이 발생한다.

하나의 SELECT 만으로 처리되면 좋겠지만, 데이터 변경을 목적으로 조회하는 것이기 떄문에 JOIN + DTO 방법을 사용할 수는 없다.

> 이런 측면에서 봤을 때 엔티티 모두 영속성 컨텍스트에서 관리되어야 한다.

SQL JOIN을 이용하는 것도 실용적인 선택사항이라 할 수 없다. (item40)

LAZY 대신 EAGER로 바꾸는 것은 최악의 선택이니 <b>절대로 그래서는 안 된다.</b>

이런 경우에는 JOIN FETCH 가 가장 바람직한 해결책이다.

> <b>영속성 컨텍스트 없이 연관관계를 조회할 때에는 JOIN + DTO를, 
> 영속성 컨텍스트가 필요한 경우에는 JOIN FETCH를 이용하는 것이 가장 좋은 선택이다.</b>

다음과 같이 JOIN FETCH 를 이용하는 Repository Method를 만들어 사용한다.

```java
// Author Repository
@Repository
@Transactional(readOnly = true)
public interface AuthorRepository extends JpaRepository<Author, Long> {
    @Query(value = "SELECT a FROM Author a JOIN FETCH a.books WHERE a.name = ?1")
    Author fetchAuthorWithBooksByName(String name);
}

// Book Repository
@Repository
@Transactional(readOnly = true)
public interface BookRepository extends JpaRepository<Book, Long> {
    @Query(value = "SELECT b FROM Book b JOIN FETCH b.author WHERE b.isbn = ?1")
    Book fetchBookWithAuthorByIsbn(String isbn);
}
```

> 참고
> 
> 테이블 JOIN 은 카테시안 곱<sub>Cartesian Product</sub>를 유발하고 단순 Lazy 조회는 N+1 쿼리를 유발한다.
> 
> 둘 중 선택하라면 데이터베이스 조회를 수행하는 것보다 단일 쿼리 테이블 JOIN 을 선택하라. 
> 
> 다만, 카테시안 곱으로 인해 과도하게 많은 데이터가 조회되는 상황이 있다면 몇 개의 SELECT 문을 이용하는 것이 나을 수도 있다.  