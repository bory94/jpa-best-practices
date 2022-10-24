## Item27. 엔티티의 정보가 아닌 가상의 속성을 이용하여 Spring 투사에 정보를 추가하기

SpEL을 이용하면 된다.

```java
public interface AuthorNameAge {
    String getName();
    @Value("#{target.age}")
    String years();
    @Value("#{ T(java.lang.Math).random() * 10000 }")
    int rank();
    @Value("5")
    String books();
}
```