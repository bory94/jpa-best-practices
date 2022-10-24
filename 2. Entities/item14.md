## Hibernate 지정 Proxy를 통해 자식 측의 부모 연관관계를 채우기

> 참고
> 
> 원문의 Populate를 "채우기"로 변역하였다. 

findById() 메소드와는 달리 getOne() 메소드는 Hibernate 지정 Proxy를 리턴한다.
그리고 이 경우 자식 데이터를 저장할 때 불필요하게 부모 데이터를 데이터베이스에서 로딩하지 않고 바로 외래키를 채워줄 수 있다.

다음 코드는 단 하나의 INSERT 문을 수행한다.

```java
// 실행 메소드
@Transactional
public void addBookToAuthor() {
  Author proxy = authorRepository.getOne(1L);
  Book book = new Book();
  book.setIsbn("001-MJ");
  book.setTitle("The Canterbury Anthology");
  book.setAuthor(proxy);
  bookRepository.save(book);
}
```

```sql
-- 실제 실행되는 DML
INSERT INTO book (author_id, isbn, title)
VALUES (?, ?, ?)
```

> 결론
> 
> 자식 데이터를 저장하기 위해 부모 데이터 전체가 필요하지 않은 경우라면 <b>getOne을 이용하는 것이 findById를 이용하는 것보다 효율적</b>이다. 