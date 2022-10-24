## Item28. *-to-One 관계를 포함하는 Spring 투사를 효율적으로 조회하기

> 전제: Author와 Book이 @OneToMany 관계로 연결되어 있다.
> 
> 이 때, Author의 name과 genre, 그리고 book의 title만 조회하고 싶다.

#### 중첩된 폐쇄 투사 이용하기

```java
// 다음과 같은 DTO를 중첩 폐쇄 DTO라 한다.
public interface BookDto {
    public String getTitle();
    public AuthorDto getAuthor();
    interface AuthorDto {
        public String getName();
        public String getGenre();
    }
}

// Repository
@Repository
@Transactional(readOnly=true)
public interface BookRepository extends JpaRepository<Book, Long> {
	List<BookDto> findBy();
}
```

다음과 같은 결과를 얻는다.
```json
[
   {
      "title":"A History of Ancient Prague",
      "author":{
         "genre":"History",
         "name":"Joana Nimar"
      }
   },
   {
      "title":"A People's History",
      "author":{
         "genre":"History",
         "name":"Joana Nimar"
      }
   },
  ...
]
```

결과는 원하는 대로지만, 쿼리 단계에서 쓸데없이 Author의 모든 정보를 조회하게 된다.

```sql
SELECT
  book0_.title AS col_0_0_,
  author1_.id AS col_1_0_,
  author1_.id AS id1_0_,
  author1_.age AS age2_0_,
  author1_.genre AS genre3_0_,
  author1_.name AS name4_0_
FROM book book0_
LEFT OUTER JOIN author author1_
  ON book0_.author_id = author1_.id
```

그리고 조회된 Author 엔티티를 영속성 컨텍스트에 저장한다. 그러므로 필요 이상으로 비효율적으로 동작한다.

> 동일한 DTO를 이용할 경우, 명시적인 JPQA을 이용해도 동일한 방식으로 동작한다.

--- 

#### 단순 폐쇄 투사 이용하기

```java
// DTO
public interface SimpleBookDto {
    public String getTitle(); // of book
    public String getName();  // of author
    public String getGenre(); // of author
}

// Repository: JPQL을 이용해야 한다.
@Repository
@Transactional(readOnly=true)
public interface BookRepository extends JpaRepository<Book, Long> {
	@Query("SELECT b.title AS title, a.name AS name, a.genre AS genre "
			+ "FROM Book b LEFT JOIN b.author a")
	List<SimpleBookDto> findByViaQuerySimpleDto();
}
```

다음과 같은 JSON을 얻게 된다.

```json
[
   {
      "title":"A History of Ancient Prague",
      "genre":"History",
      "name":"Joana Nimar"
   },
   {
      "title":"A People's History",
      "genre":"History",
      "name":"Joana Nimar"
   },
   ...
]
```

하지만 한 번의 SELECT문만 실행되며 영속성 컨텍스트에 저장되는 것도 없다.

---

#### 단순 개방 투사 이용하기

생략 - 데이터 구조를 유지하면서 한 번의 쿼리만으로 처리되지만 가장 느리다.