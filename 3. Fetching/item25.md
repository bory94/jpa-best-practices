## Item25. Spring 투사<sub>Projection</sub>를 통해 DTO 가져오기

> 주의사항
> 
> 효율성을 위해, 엔티티를 수정할 이유가 없다면 읽기-쓰기 모드로 엔티티를 로딩하지 않아야 한다. 
> 읽기-쓰기 모드 엔티티는 더 많은 메모리와 CPU를 사용하게 된다.
> 
> 또, 수정할 대상인 엔티티는 읽기 전용 모드로 로딩하지 않아야 한다.
> 읽기 전용 모드의 엔티티를 수정하는 경우 별도의 SELECT를 수행해야 하므로 비효율적이다.

읽기 전용 모드를 이용할 것이라면 Data Transfer Object<sub>DTO</sub>를 이용하는 것이 가장 바람직하다.
또, 애초에 DTO가 필요했다면 엔티티를 로딩한 뒤에 DTO로 변환하는 것도 매우 비효율적이다.

> 참고
> 
> DTO는 DTO 클래스를 이용하거나 Getter Method 만들오 이루어진 인터페이스로 만들 수도 있다.

---

#### JPA Named Query는 Spring 투사와 연결될 수 있다.

생략

#### Class 기반 투사

생략

#### Spring 투사 재사용하기

생략

#### 동적 Spring 투사 사용하기

다음과 같이 Generic Type을 이용하여 동적으로 투사할 수 있다.

```java
// Repository에 아래와 같이 등록한다.
<T> T findByName(String name, Class<T> type);

// 다음과 같이 사용할 수 있다.
Author author = authorRepository.findByName("Joana Nimar", Author.class);
AuthorGenreDto author = authorRepository.findByName("Joana Nimar", AuthorGenreDto.class);
AuthorNameEmailDto author = authorRepository.findByName("Joana Nimar", AuthorNameEmailDto.class);
```

