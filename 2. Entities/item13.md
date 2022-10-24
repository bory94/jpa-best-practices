## Item13. Entity에 Fluent API를 채택하기

> Fluent API는 Builder Pattern이나 항수형 프로그래밍에서 사용하는 메소드 연쇄를 이용하는 API를 말한다.
> 
> 예를 들어 다음과 같다.
> ```java
> // Builder Pattern 예제
> Author author = new Author()
>    .setName("Joana Nimar")
>    .setAge(34)
>    .setGenre("History")
>    .addBook(new Book()
>        .setTitle("A History of Ancient Prague")
>        .setIsbn("001-JN"))
>    .addBook(new Book()
>        .setTitle("A People's History")
>        .setIsbn("002-JN"));
> ```

각종 Setter 메소드나 사용자 정의 (도메인) 메소드의 리턴 타입을 엔티티 자신으로 하고 return this를 추가한다.
또, Setter 메소드에 set이라는 prefix를 제거함으로써 보다 읽기 쉬운 코드를 만들 수 있다. 

```java
@Entity
public class Author implements Serializable {
	private static final long serialVersionUID = 1L;
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;
	private String name;
	private String genre;
	private int age;
	@OneToMany(cascade = CascadeType.ALL,
			mappedBy = "author", orphanRemoval = true)
	private List<Book> books = new ArrayList<>();
	public Author addBook(Book book) {
		this.books.add(book);
		book.setAuthor(this);
		return this; // <-- 여기
	}
	public Author removeBook(Book book) {
		book.setAuthor(null);
		this.books.remove(book);
		return this;
	}
	public Author id(Long id) {
		this.id = id;
		return this; // <-- 여기
	}
	public Author name(String name) {
		this.name = name;
		return this; // <-- 여기
	}
	public Author genre(String genre) {
		this.genre = genre;
		return this; // <-- 여기
	}
	public Author age(int age) {
		this.age = age;
		return this; // <-- 여기
	}
	public Author books(List<Book> books) {
		this.books = books;
		return this; // <-- 여기
	}
	// getters and setters omitted for brevity
}
```