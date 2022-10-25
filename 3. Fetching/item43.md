## Item43. Paginated JOIN 방법

JOIN + DTO 투사를 이용하면서 Paging 처리도 하는 방법은 다음과 같다.

```java
// interface 기반 DTO
public interface AuthorBookDto {
	public String getName();  // of author
	public int getAge();      // of author
	public String getTitle(); // of book
	public String getIsbn();  // of book
}

// Pageable을 이용하는 Repository 메소드
// JOIN + DTO를 이용할 것이므로 LEFT JOIN 으로 충분하다. LEFT JOIN FETCH 는 필요치 않다.
@Transactional(readOnly = true)
@Query(value = "SELECT a.name AS name, a.age AS age, b.title AS title, b.isbn AS isbn " 
    + "FROM Author a LEFT JOIN a.books b " 
    + "WHERE a.genre = ?1")
Page<AuthorBookDto> fetchPageOfDto(String genre, Pageable pageable);

// PageRequest를 이용하여 Pageable instance를 생성, 사용한다.
public Page<AuthorBookDto> fetchPageOfAuthorsWithBooksDtoByGenre( int page, int size) {
	Pageable pageable = PageRequest.of(page, size, Sort.by(Sort.Direction.ASC, "name"));
	Page<AuthorBookDto> pageOfAuthors = authorRepository.fetchPageOfDto("Anthology", pageable);
	return pageOfAuthors ;
}
```

조회 결과로 org.springframework.data.domain.Page<T> 타입의 인스턴스가 리턴된다.

이 과정에서 COUNT 쿼리와 LIMIT 를 이용하는 조회 쿼리, 두 개의 SELECT 문이 실행된다.

> COUNT(*) OVER() 구문을 이용하여 별도의 COUNT 쿼리를 수행하지 않고 전체 데이터 수를 구할 수 있다.
> 
> MySQL의 OVER() 구문에 대한 자세한 설명은 [이 글](https://dev.mysql.com/doc/refman/8.0/en/window-functions-usage.html)을 참고하라.

Page<T> 대신 Slice<T>나 List<T>를 이용할 수도 있다. 자세한 내용은 생략...

> 여기서 언급된 접근방법은 부모 테이블에 대해 Paging을 하는 것이 아니라 DTO 단위로 Paging을 하므로, 부모 입장에서는 원하는 자식 데이터를 모두 조회할 수 없다는 문제가 생긴다.
> 
> 이 문제를 해결하기 위해 DENSE_RANK() 쿼리를 사용한다.

---

#### DENSE_RANK() 윈도우 함수를 이용하기

DENSE_RANK에 대한 상세한 사용법은 [이 글](https://oncodesign.io/2017/04/22/paging-one-to-many-results-in-sql/)을 참고하라.

