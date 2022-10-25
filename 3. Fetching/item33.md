## Item33. JPA Tuple을 이용하여 DTO를 가져오는 방법

JPA는 javax.persistence.Tuple이라는 클래스를 제공한다. 이것을 이용하면 DTO 없이도 Object[] 보다 손쉽게 데이터를 가져올 수 있다.

- Tuple은 Query에 지정되어 있는 Alias를 보유하고 있다.
- Tuple은 자동으로 데이터 캐스팅(casting)을 수행한다.
- TupleElement는 Java Generics를 지원한다.

> 저자는 값 투사<sub>Scalar Projection</sub> 방법 중 Tuple을 이용하는 것이 최고의 방법이라고 한다.
> 
> Tuple은 JPQL, Native SQL, Criteria API 모두 지원한다.

```java
// Tuple을 이용하는 Repository 메소드
@Repository
@Transactional(readOnly = true)
public interface AuthorRepository extends JpaRepository<Author, Long> {
	@Query(value = "SELECT a.name AS name, a.age AS age FROM Author a")
	List<Tuple> fetchAuthors();
}

// 다음과 같이 결과를 조회할 수 있다. (Map과 매우 유사하다)
List<Tuple> authors = ...;
for (Tuple author : authors) {
	System.out.println("Author name: " + author.get("name")
		+ " | Age: " + author.get("age"));
}
```