## Item37. @Subselect를 이용하여 쿼리에 엔티티 매핑하기

@Subselect는 엔티티들의 관계는 유지하면서 엔티티의 일부 정보만 가져오고 싶을 때 사용한다.
이를 통해 얻어지는 엔티티는 불변<sub>Immutable</sub>이고 읽기 전용<sub>Read Only</sub>이다.

@Subselect의 사용방법은 다음과 같다.

- 필요한 필드만 갖는 엔티티를 새로 생성한다. (이 때 연관관계 필드도 같이 지정해야 한다.)
- 모든 필드에 대해 Getter 메소드만 만든다.
- @Immutable 애너테이션을 이용하여 불변 엔티티로 만든다.
- @Synchronize 애너테이션을 이용하여 사용된 엔티티의 처리 대기 상태를 DB에 적용<sub>Flush</sub>한다. 
- @Subselect 애너테이션을 이용하여 필요한 쿼리를 등록한다.

@Subselect를 이용하는 엔티티는 다음과 같다.
```java
@Entity
@Subselect(
    "SELECT a.id AS id, a.name AS name, a.genre AS genre FROM Author a")
@Synchronize({"author", "book"})
@Immutable
public class AuthorSummary implements Serializable {
    private static final long serialVersionUID = 1L;
    @Id
    private Long id;
    private String name;
    private String genre;
    @OneToMany(mappedBy = "author")
    private Set<Book> books = new HashSet<>();
    public Long getId() {
        return id;
    }
    public String getName() {
        return name;
    }
    public String getGenre() {
        return genre;
    }
    public Set<Book> getBooks() {
        return books;
    }
}
```

> 참고 - 이 엔티티 데이터를 가져오기 위해 Author와 Book에 대한 <b>두 개 SELECT 문이 실행</b>된다.