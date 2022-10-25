## Item31. 생성자 표현식으로 DTO 가져오기

다음과 같이 DTO를 생성하고 Query Builder 방식으로 SQL을 실행할 수 있다.
그리고 이 방법은 꽤 효율적이다.

```java
// Immutable DTO with Constructor
public class AuthorDto implements Serializable {
    private static final long serialVersionUID = 1L;
    private final String name;
    private final int age;
    public AuthorDto(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public String getName() {
        return name;
    }
    public int getAge() {
        return age;
    }
}

// Repository
@Repository
@Transactional(readOnly = true)
public interface AuthorRepository extends JpaRepository<Author, Long> {
	List<AuthorDto> findByGenre(String genre);
}
```

다만 이 방식을 직접 사용할 수 없는 경우라면 다음과 같이 JPQL에 직접 생성자 표현식을 지정하여 DTO 투사를 실행할 수 있다.

```java
// JPQL에 생성자 표현식을 직접 입력한 Repository 메소드
@Repository
@Transactional(readOnly = true)
public interface AuthorRepository extends JpaRepository<Author, Long> {
    @Query(value="SELECT new com.bookstore.dto.AuthorDto(a.name, a.age) FROM Author a")
    List<AuthorDto> fetchAuthors();
}
```

이 방법 역시 효율적이다.