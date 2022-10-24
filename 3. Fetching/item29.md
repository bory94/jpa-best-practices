## Item29. Spring 투사에 연관된 Collection이 있는 경우 주의를 기울여야 하는 이유

> 전제: Author와 Book이 @OneToMany 관계로 연결되어 있다.
>
> 이 때, Author의 name과 genre, 그리고 book의 title만 조회하고 싶다.

#### 중첩 폐쇄 투사 이용하기

다음과 같은 DTO를 이용하는 경우..
```java
public interface AuthorDto {
    public String getName();
    public String getGenre();
    public List<BookDto> getBooks();
    interface BookDto {
        public String getTitle();
    }
}

```

##### 쿼리 빌더 메커니즘 이용하기
```java
@Repository
@Transactional(readOnly=true)
public interface AuthorRepository extends JpaRepository<Author, Long> {
	// 쿼리 빌더가 자동으로 쿼리를 만들어 줌
    List<AuthorDto> findBy();
}
```

<b>!! 절대 사용하지 말아야 함 !!</b>
이 방법은 N+1과 영속성 컨텍스트 사용까지 모두 걸린다.

##### 명시적인 JPQL을 이용하는 경우

```java
@Repository
@Transactional(readOnly=true)
public interface AuthorRepository extends JpaRepository<Author, Long> {
    @Query("SELECT a.name AS name, a.genre AS genre, b AS books "
         + "FROM Author a INNER JOIN a.books b")
    List<AuthorDto> findByViaQuery();
}
```

하나의 SELECT만 수행되지만, Book 정보가 모두 로딩되고 모든 Book들이 영속성 컨텍스트에 저장된다.

##### JPA JOIN FETCH를 이용하는 경우
```java
@Repository
@Transactional(readOnly=true)
public interface AuthorRepository extends JpaRepository<Author, Long> {
    @Query("SELECT a FROM Author a JOIN FETCH a.books") // <-- 여기에 주의
    Set<AuthorDto> findByJoinFetch();
}
```

하나의 SELECT만 수행되지만, Author 정보 모두, Book 정보 모두 로딩하며 모두 영속성 컨텍스트에 저장된다.

---

#### 단순 폐쇄 투사를 이용하는 경우
```java
// DTO
public interface SimpleAuthorDto {
    public String getName();  // of author
    public String getGenre(); // of author
    public String getTitle(); // of book
}

// Repository: JPQL을 이용해야 한다.
@Repository
@Transactional(readOnly=true)
public interface AuthorRepository extends JpaRepository<Author, Long> {
	@Query("SELECT a.name AS name, a.genre AS genre, b.title AS title "
			+ "FROM Author a INNER JOIN a.books b")
	List<SimpleAuthorDto> findByViaQuerySimpleDto();
}
```

한 번의 쿼리만 수행되고 원하는 데이터만 조회되며 영속성 컨텍스트에 저장되는 데이터도 없다.

> 이 방법을 사용하는 것이 가장 바람직하다. 결과 데이터 구조가 그럻게 중요하지 않다면...

---

#### DTO 내에서 List<Object[]>를 변환하는 방법

중간 생략

> 빠르긴 하지만 단순 폐쇄 투사보다 월등히 빠르지 않으며 작업량은 훨씬 많다.

