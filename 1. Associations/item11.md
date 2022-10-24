### Item11. 단방향/양방향 @OneToOne 관계를 @MapsId를 통해 최적화하는 방법

> 전제: Author와 Book이 @OneToOne 관계에 있다.
> 
> Book -|O--O|- Author

> 참고: @MapsId는 외래키를 유일키로 등록할 때 사용한다.

관계형 데이터베이스에서 @OneToOne 관계는 일반적으로 부모-자식 관계로 표상되며 자식 테이블에 부모의 키<sub>Primary Key</sub>를 외래키<sub>Foreign Key</sub>로 등록하여 연결한다.

이 때 @MapsId 애너테이션이 중요한 이유는 다음과 같다.

- 부모 측에도, 자식 측에도 @OneToOne 설정을 해줘야 하는데, @MapsId를 이용해야 Lazy Loading이 처리된다.
- 자식 측이 2차 캐시에 등록되어 있는 경우 2차 캐시에 등록된 정보를 로딩할 수 있다.
- 부모를 로딩할 때 쓸데없이 자식까지 같이 로딩하지 않을 수 있다.
- 동일한 유일키를 공유함으로써 메모리 사용량을 줄일 수 있다.

> 결과적으로 다음과 같은 엔티티가 만들어진다.

```java
// Author -- Book을 가지고 있지 않는다.
@Entity
public class Author implements Serializable {
	private static final long serialVersionUID = 1L;
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;
	private String name;
	private String genre;
	private int age;
	// getters and setters omitted for brevity
}

// Book -- Author를 가지고 있으며, @MapsId로 처리되어 있다.
@Entity
public class Book implements Serializable {
	private static final long serialVersionUID = 1L;
	@Id
	private Long id;
	private String title;
	private String isbn;
	@MapsId // <-- 여기에 지정
	@OneToOne(fetch = FetchType.LAZY)
	@JoinColumn(name = "author_id")
	private Author author;
	// getters and setters omitted for brevity
}
```

