## Item40. JOIN과 JOIN FETCH 중 어떤 것을 선택할지 결정하는 방법

> 전제 - Author와 Book은 양방향 Lazy @OneToMany 관계로 연결되어 있다.
> 
> 지정된 가격보다 더 비싼 Book에 대해 Author와 Book을 조회하려 한다. -> 추후 변경할 것을 목표로 함  

#### Author 기준으로 Book 목록 조회하기 + Price 조건 추가 

아래 JOIN 과 JOIN FETCH 를 이용하는 Repository 메소드 두 가지가 있다.
```java
@Repository
@Transactional(readOnly = true)
public interface AuthorRepository extends JpaRepository<Author, Long> {
    // JOIN FETCH
    @Query(value = "SELECT a FROM Author a JOIN FETCH a.books b WHERE b.price > ?1")
    List<Author> fetchAuthorsBooksByPriceJoinFetch(int price);
		
    // INNER JOIN
    @Query(value = "SELECT a FROM Author a INNER JOIN a.books b WHERE b.price > ?1")
    List<Author> fetchAuthorsBooksByPriceInnerJoin(int price);
}
```

##### JOIN FETCH 동작 방식

```sql
SELECT
  author0_.id AS id1_0_0_,
  books1_.id AS id1_1_1_,
  author0_.age AS age2_0_0_,
  author0_.genre AS genre3_0_0_,
  author0_.name AS name4_0_0_,
  books1_.author_id AS author_i5_1_1_,
  books1_.isbn AS isbn2_1_1_,
  books1_.price AS price3_1_1_,
  books1_.title AS title4_1_1_,
  books1_.author_id AS author_i5_1_0__,
  books1_.id AS id1_1_0__
FROM author author0_
INNER JOIN book books1_
  ON author0_.id = books1_.author_id
WHERE books1_.price > ?
```
카테시안 곱 조회를 수행하는 하나의 SELECT 문을 호출한 뒤 엔티티와 연관 엔티티에 데이터를 채워 넣는다<sub>Hydrate</sub>.

##### JOIN 동작 방식

```sql
-- 최초 조회 시 
SELECT
  author0_.id AS id1_0_,
  author0_.age AS age2_0_,
  author0_.genre AS genre3_0_,
  author0_.name AS name4_0_
FROM author author0_
INNER JOIN book books1_
  ON author0_.id = books1_.author_id
WHERE books1_.price > ?

-- getBook() 메소드로 Book 필드에 접근 시
SELECT
    books0_.author_id AS author_i5_1_0_,
    books0_.id AS id1_1_0_,
    books0_.id AS id1_1_1_,
    books0_.author_id AS author_i5_1_1_,
    books0_.isbn AS isbn2_1_1_,
    books0_.price AS price3_1_1_,
    books0_.title AS title4_1_1_
FROM book books0_
WHERE books0_.author_id = ?
```

이 때 두 가지 문제가 발생한다.

- Lazy 연관 데이터<sub>Book</sub>에 대해 접근할 때 새로운 SELECT 문이 실행된다.
- <b>Book 조회 시 Price 조건이 빠진다.</b>

그래서 이런 목적이 아니라면 JOIN을 사용해서는 안 된다.

---

#### Book 기준으로 Author 조회하기

다음과 같은 세 가지 Repository 메소드가 작성되어 있다.

```java
@Repository
@Transactional(readOnly = true)
public interface BookRepository extends JpaRepository<Book, Long> {
    // INNER JOIN BAD
    @Query(value = "SELECT b FROM Book b INNER JOIN b.author a")
    List<Book> fetchBooksAuthorsInnerJoinBad();
    // INNER JOIN GOOD
    @Query(value = "SELECT b, a FROM Book b INNER JOIN b.author a")
    List<Book> fetchBooksAuthorsInnerJoinGood();
    // JOIN FETCH
    @Query(value = "SELECT b FROM Book b JOIN FETCH b.author a")
    List<Book> fetchBooksAuthorsJoinFetch();
}
```

##### JOIN FETCH 동작 방식

예상한 대로 동작한다.

##### JOIN 동작 방식

INNER JOIN BAD 라 주석처리되어 있는 fetchBooksAuthorsInnerJoinBad() 메소드는 N+1 문제를 일으키는 문제가 인다. 
> 단 특정 Author 엔티티가 이미 영속성 컨텍스트 내에 저장되어 있다면 해당 엔티티에 대해서는 실제 SELECT 문을 실행하지는 않는다.

반대로, 
INNER JOIN GOOD 이라 주석처리되어 있는 fetchBooksAuthorsInnerJoinGood() 메소드는 <b>JOIN FETCH 와 동일한 방식으로 동작</b>한다.

단순 INNER JOIN을 사용할 것이라면 이 방식을 사용해야 한다.

---

결론적으로 이런 류의 처리는 항상 JOIN FETCH를 이용하여야 한다.

단 읽기 전용 데이터를 조회할 때에는 JOIN + DTO 방식을 이용해야 한다.
