## Item1: @OneToMany 관계를 효과적으로 구현하기
> 전제: Author와 Book Entity가 양방향<sub>bidirectional</sub> lazy @OneToMany 관계로 연결되어있다.
> > 주의사항: 일방향 연관관계 대신 양방향 @OneToMany 관계를 사용하라. 이에 대해 [Item2](item2.md)에서 설명한다.

#### 항상 부모 측에서 자식 측으로 Cascade <sub>종속</sub> 설정을 하라.

#### 부모 측에 항상 mappedBy 설정을 하라.

#### 부모 측에 orphanRemoval 설정을 하라.

```java 
// 부모 측 결과 코드
@OneToMany(
  cascade = CascadeType.ALL,
  mappedBy = "author", // Author가 Book에 author란 필드명으로 연결되었다.
  orphanRemoval = true  
)
private List<Book> books;

// 자식 측 결과코드
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "author_id")
private Author author;
```

### 데이터 변경 시, 양쪽 엔티티에서 관계가 동기화되도록 하라.
> Author에서 특정 Book을 빼면 그 Book에서도 Author를 빼라는 얘기

### equals()와 hashCode() 메소드를 재정의하라.
> 이에 대해서는 [Ultimate Guide to Implementing equals() and hashCode() with Hibernate](https://thorben-janssen.com/ultimate-guide-to-implementing-equals-and-hashcode-with-hibernate/) 글을 참조

### 양쪽 모두 Lazy Fetching을 이용하라.
> @OneToMany는 기본적으로 FetchType.LAZY를 이용하고 있으니 바꾸지 말 것
>
> @ManyToOne은 기본적으로 FetchType.EAGER를 이용하고 있으니 바꿀 것
```java
@ManyToOne(fetchType = FetchType.LAZY)
```

### ToString()을 제대로 재정의하였는지 주의하라.
> Lazy 관계 Entity를 toString()에서 사용하면 LazyInitializationException이 발생할 수 있다.