## Item30. 모든 Entity를 Spring 투사로 가져오기

중간 생략...

결론적으로 말해 모든 데이터를 추출하는 경우 JPQL을 사용하면서 모든 컬럼을 명시하는 것이 가장 성능이 좋다.

```java
@Repository
@Transactional(readOnly=true)
public interface AuthorRepository extends JpaRepository<Author, Long> {
    @Query("SELECT a.id AS id, a.age AS age, a.name AS name,
                   a.genre AS genre FROM Author a")
    List<AuthorDto> fetchAsDtoColumns();
}
```

> 심지어 Native Query를 이용할 때보다 위의 JPQL + 컬럼 명시 방법이 더 빠르다.
> ```java
> // Native Query를 사용하는 방식
> @Repository
> @Transactional(readOnly=true)
> public interface AuthorRepository extends JpaRepository<Author, Long> {
>   @Query(value = "SELECT id, age, name, genre FROM author", nativeQuery = true)
>   List<AuthorDto> fetchAsDtoNative();
> 
> ```