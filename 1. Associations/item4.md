## Item4: @ManyToMany 관계를 효과적으로 구현하기
> 전제: Author와 Book이 @ManyToMany 관계로 연결되어 있다. 
> 
> 즉, 하나의 Book에 대해 공저자가 있음을 의미한다. (당연히 한 명의 저자는 한 권이상의 저서를 저술...)
> 
> > 다대다 관계는 author -|-O< author_book >O-|- book 형태로 각 엔티티가 연결<sub>junction</sub> 엔티티을 통해야 한다.

### 어느 쪽이 소유자<sub>Owner</sub>인지 결정하라.
Author와 Book 중 소유하는 쪽을 결정하고, 그 반대편을 mappedBy로 설정해준다.
 
항상 소유자 쪽에서의 변경사항만이 데이터베이스로 전파<sub>propagate</sub>된다.
 
> 만약 Author가 소유하는 쪽이라고 한다면 다음과 같이 Book에 mappedBy 설정을 한다.
```java
@ManyToMany(mappedBy = "books")
private Set<Author> authors = new HashSet<>();
```

### List가 아니라 Set을 이용하라.
특히 삭제 기능이 연관되어 있다면 List 대신 Set을 이용하는 것을 권장한다.
 
[Item5](./item5.md)에서 왜 Set이 더 효율적인지 설명한다. 

### 데이터 변경 시, 양쪽 엔티티에서 관계가 동기화되도록 하라.

### CascadeType.ALL과 CascadeType.REMOVE를 <b>사용하지 말라</b>
부모-자식 간의 관계가 아니기 때문에 대부분의 경우 Cascading 삭제는 좋은 생각이 아니다.  

### Join Table 설정을 하라.

### 양쪽 모두 FetchType.LAZY를 사용하다.
@ManyToMany의 기본 FetchType은 LAZY이다. 
절대로 EAGER로 설정하지 말아야 한다.

```java
// 지금까지를 정리한 코드
@ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
@JoinTable(
    name = "author_book",
    joinColumns = @JoinColumn(name = "author_id"),
    inverseJoinColumns = @JoinColumn(name = "book_id")
)
private Set<Book> books = new HashSet<>();
```

### equals()와 hashCode() 메소드를 적절히 재정의하라.
이에 대해서는 [Ultimate Guide to Implementing equals() and hashCode() with Hibernate](https://thorben-janssen.com/ultimate-guide-to-implementing-equals-and-hashcode-with-hibernate/) 글을 참조

### 참고: @ManyToMany 관계를 양방향 @OneToMany 관계를 이용하여 처리할 수도 있다. 이 경우 연결 엔티티가 @Entity로 실체화되어 양방향 @OneToMany를 지원하게 된다.