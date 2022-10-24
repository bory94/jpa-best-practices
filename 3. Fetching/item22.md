## Item22. 미래의 영속성 컨텍스트 데이터에 대해 데이터베이스로 변경사항을 저장할 때 항상 읽기 전용 엔티티를 사용하는 이유

> 전제
> 
> 하나의 별도 트랜잭션에서 데이터를 로딩하고 그와는 다른 트랜잭션에서 데이터를 저장하는 시나리오이다.

다음과 같은 방식으로 엔티티를 로딩하면 실제로 필요하지 않음에도 영속성 컨텍스트에 엔티티가 적재된다.
```java
@Transactional
public Author fetchAuthorReadWriteMode() {
    Author author = authorRepository.findByName("Joana Nimar");
    return author;
}
```
하지만 이 엔티티는 fetchAuthorReadWriteMode() 메소드가 실행된 후 바로 영속성 컨텍스트에서 제거되므로 쓸데없이 영속성 컨텍스트를 사용하여 자원을 낭비하는 꼴이 된다.

이럴 떄에는 다음과 같이 읽기 전용 모드로 로딩해야 한다.
```java
@Transactional(readOnly = true)
public Author fetchAuthorReadWriteMode() {
    Author author = authorRepository.findByName("Joana Nimar");
    return author;
}
```

---

#### Update하는 경우

조회한 데이터를 별도의 트랜잭션에서 등록해야 하므로, 다음과 같이 처리할 수 있다.
```java
// modify the read-only entity in detached state
Author authorRO = bookstoreService.fetchAuthorReadOnlyMode();
authorRO.setAge(authorRO.getAge() + 1);
bookstoreService.updateAuthor(authorRO);
// merge the entity
@Transactional
public void updateAuthor(Author author) {
    // behind the scene it calls EntityManager#merge()
    authorRepository.save(author); // <-- 내부적으로 SELECT + UPDATE 방식으로 처리된다.
}
```

> 이런 처리 방식은 HTTP long Conversation에서 많이 볼 수 있다. 
> 일반적인 웹 애플리케이션에서는 이런 시나리오는 두 개의 별도 Request로 처리가 될 것이다. 