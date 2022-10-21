## Item7. JPA Entity Graph를 이용하여 연관관계 조회하기

> 참고1
> 
> JOIN FETCH를 이용해서도 lazy loading exception과 N+1 이슈를 피할 수 있다. 
> 
> 하지만 JPA Entity Graph 역시 JOIN FETCH 쿼리와는 별개로 연관관계 로딩에 사용될 수 있다.

### @NamedEntityGraph를 이용하여 Entity Graph 정의 / 사용하기
Entity에 @NamedEntityGraph를 설정한다.
```java
@Entity
@NamedEntityGraph(
    name = "author-books-graph",
    attributeNodes = {
        @NamedAttributeNode("books")
    }
)
public class Author implements Serializable {
  // ... 생략
}
```
설정된 author-books-graph를 Query 메소드에서 사용한다.
```java
@EntityGraph(value = "author-books-graph",
             type = EntityGraph.EntityGraphType.FETCH)
public List<Author> findAll();
```
이렇게 하면 Lazy Loading 설정된 List<Book> books 연관관계도 같이 LEFT OUTER JOIN으로 로딩된다. 

### Query 빌더 메커니즘도 이용할 수 있다.
아래와 같이 메소드명을 지정하면 WHERE 조건과 ORDER BY 항목이 지정된다.
```java
@EntityGraph(value = "author-books-graph",
             type = EntityGraph.EntityGraphType.FETCH)
public List<Author> findByAgeLessThanOrderByNameDesc(int age); 
```

### Ad Hoc Entity Graph 이용하기
@NamedEntityGraph와는 달리 Repository Method에 바로 @EntityGraph를 정의할 수 있다.
```java
@Override
@EntityGraph(attributePaths = {"books"},
             type = EntityGraph.EntityGraphType.FETCH)
public List<Author> findAll();
```

이하 생략