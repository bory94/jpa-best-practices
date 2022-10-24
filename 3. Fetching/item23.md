## Hibernate Bytecode Enhancement를 이용하여 엔티티 속성을 Lazy Load하는 방법

#### 속성을 Lazy Loading하기

단일 속성 중 @Lob 같은 데이터를 Lazy Loading 하려면 다음과 같이 attribute lazy loading을 활성화해야 한다.
```xml
<!-- Maven 설정-->
<plugin>
  <groupId>org.hibernate.orm.tooling</groupId>
  <artifactId>hibernate-enhance-maven-plugin</artifactId>
  <version>${hibernate.version}</version>
  <executions>
    <execution>
      <configuration>
        <failOnError>true</failOnError>
        <enableLazyInitialization>true</enableLazyInitialization> <!-- 여기 -->
      </configuration>
      <goals>
        <goal>enhance</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

> 참고
> 
> 이 역시 Hibernate의 Bytecode Enhancement를 이용하므로 자세한 내용은 [이 글](https://docs.jboss.org/hibernate/orm/5.4/topical/html_single/bytecode/BytecodeEnhancement.html)에서 확인하라.

이런 다음 해당 속성에 다음과 같이 @Basic 애너테이션을 지정한다.
```java
@Lob
@Basic(fetch = FetchType.LAZY)
private byte[] avatar;
```

---

#### 속성 Lazy Loading과 N+1

Lazy Loading은 근본적으로 N+1 조회 가능성을 내표하고 있다. 
더군다나 실제 쿼리 수행 횟수를 조회하기 전까지 N+1 쿼리가 수행되었는지 알지 못하는 경우가 대부분이다.

N+1 쿼리를 피하는 방법은 다음과 같다.

- 하위 엔티티를 이용한다. [Item24](./item24.md)
- Lazy Loading할 데이터만 별도의 DTO를 이용하여 직접 조회한다.

```java
// 두 번째 방법
public interface AuthorDto {
	public String getName();
	public byte[] getAvatar();
}
@Transactional(readOnly = true)
@Query("SELECT a.name AS name, a.avatar AS avatar
		FROM Author a WHERE a.age >= ?1") " 
List<AuthorDto> findDtoByAgeGreaterThanEqual(int age);
```

---

#### 속성 Lazy Loading과 Lazy Initialization Exception

Lazy Loading은 특정 상황에서 LazyInitializationException이 발생할 수 있다.
(예를 들어 Open-Session-In-View가 비활성화된 경우...)

이에 대한 해결 방법은 다음과 같다.

- Lazy Loading되는 속성에 대한 기본값 지정하기
- Custom JSON Filter를 이용하여 Null인 경우 무시하기

