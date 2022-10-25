## Item32. DTO 내의 엔티티를 가져올 때 생성자 표현식을 피해야 하는 이유

다음과 같은 DTO가 있다고 가정한다. 
```java
public class BookstoreDto implements Serializable {
    private static final long serialVersionUID = 1L;
    private final Author author; // <-- DTO 내의 엔티티
    private final String title;
    public BookstoreDto(Author author, String title) {
        this.author = author;
        this.title = title;
    }
    public Author getAuthor() {
        return author;
    }
    public String getTitle() {
        return title;
    }
}
```
> 참고: 이 예에서는 Author와 Book 간에 아무런 실체적인 연관관계는 없고 genre라는 컬럼으로 값 join을 할 수 있다고 가정한다.

이 DTO로 조회하기 위해 다음과 같이 생성자 표현식을 이용하면 Book과 Author를 별도로 조회한다. 
즉, 두 번의 SELECT 문이 실행된다.

```java
@Repository
@Transactional(readOnly = true)
public interface AuthorRepository extends JpaRepository<Author, Long> {
    @Query("SELECT new com.bookstore.dto.BookstoreDto(a, b.title)"
             + "FROM Author a JOIN Book b ON a.genre=b.genre ORDER BY a.id")
    List<BookstoreDto> fetchAll();
}
```

이 방법은 비효율적이다.