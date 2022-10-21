## Item2: 단방향 @OneToMany를 사용하면 안 되는 이유
> <b>주의사항</b>
> 
> 원서에 설명이 완벽하게 잘 되어 있지는 않지만, @OneToMany 관계인 Entity 간에 단방향 @OneToMany를 사용하지 말라는 의미이다.
> 
> @ManyToMany에 대한 주의사항이 아니며, @ManyToMany에 대해서는 [Item4](./item4.md)에서 별도로 다룬다.

> 전제: Author와 Book Entity가 일방향<sub>unidirectional</sub> lazy @OneToMany 관계로 연결되어있다.
>> 일방향 @OneToMany이기 때문에 author -|-O< author_books >O-|- book 형태로 엔티티가 구성된다.
>>
>> 즉, author_books라는 연결<sub>junction</sub> 엔티티가 필요하다. 

### 연결 테이블이 두 개의 외래키를 보유하며, 그래서 메모리를 더 많이 사용하게 된다. 또, 모든 CRUD 작업에 더 많은 비용이 소비된다. 

### 하나의 Author에 세 개 Book을 신규 등록하는 경우 
4 <sub>1 + 3</sub>개의 INSERT 문이 아니라 7 <sub>1 + 3 + 3</sub> 개의 INSERT 문이 실행된다.

### 세 개 Book을 갖고 있는 기존 Author에 책 하나 추가하기
1개의 INSERT 문이 아니라 5 <sub>1 DELETE + 4 INSERT</sub> 개의 DML이 수행된다. 
양방향 @OneToMany는 1개의 INSERT로 충분하다.

---

... 몇 가지 반복된 내용 생략 ...

---

### @OrderColumn을 사용하는 경우
ORDER BY 하기 위한 별도의 컬럼을 연결 엔티티<sub>author_books</sub>에 갖고 있어야 한다.

그로 인해 데이터 등록/삭제 시 별도의 추가 DML이 실행되어야 한다. 

### @JoinColumn을 사용하는 경우
@JoinColumn을 사용하면 다음과 같은 코드를 이용하게 되고 그 결과 연결<sub>author_books</sub> 엔티티가 필요 없게 된다.
> 하지만 단방향 @OneToMany를 사용하고 있다는 전제는 여전하다. 
```java
@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
@JoinColumn(name = "author_id")
private List<Book> books = new ArrayList<>();
```

#### 양방향 @OneToMany가 아니라면 관계 설정을 위해 역시 별도의 DML이 수행되어야 한다. (이하 생략)

### 결론: @OneToMany는 양방향을 이용하라. 즉, <b>반드시 </b>부모 측에 @OneToMany를, 자식 측에 @ManyToOne을 지정하라. 

