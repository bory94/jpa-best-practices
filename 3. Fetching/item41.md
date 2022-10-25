Item41. LEFT JOIN을 하는 방법

> 전제: Author와 Book이 양방향 Lazy @OneToMany 연관관계이다.

> JOIN FETCH 의 기본 동작 방식은 INNER JOIN 이다.

JOIN FETCH 시 LEFT JOIN을 하려면 그냥 LEFT JOIN FETCH 을 이용하면 된다.

```java
@Repository
@Transactional(readOnly = true)
public interface AuthorRepository extends JpaRepository<Author, Long> {
    @Query(value = "SELECT a FROM Author a LEFT JOIN FETCH a.books")
    List<Author> fetchAuthorWithBooks();
}
```

> 어느 방향에서 조회하든 마찬가지이다.