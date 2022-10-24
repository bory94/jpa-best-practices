## Item19. Boolean을 Yes/No로 매핑하기

> 참고
> 
> Java의 Boolean Type은 BIT JDBC Type으로 매핑된다.

@Converter를 이용한다.

```java
@Converter(autoApply = true)
public class BooleanConverter
       implements AttributeConverter<Boolean, String> {
    @Override
    public String convertToDatabaseColumn(Boolean attr) {
       return attr == null ? "No" : "Yes";
    }
    @Override
    public Boolean convertToEntityAttribute(String dbData) {
       return !"No".equals(dbData);
    }
}
```

> autoApply를 하면 모든 Boolean에 대해 적용되므로 주의해야 한다.
> 
> 만약 특정 필드에만 적용하고 싶다면 autoApply를 false로 하고 다음과 같이 필드에 Converter를 적용한다.
> ```java
> @Convert(converter = BooleanConverter.class)
> private Boolean bestSelling;
> ```
