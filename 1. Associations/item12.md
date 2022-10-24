## Item12. 단 하나의 연관관계만 Null이 아님을 검증하는 방법

> 전체
> 
> Review 엔티티는 Book, Article, Magazine 세 가지 엔티티에 @ManyToOne으로 연결되어 있다.

아래와 같이 사용자 정의 애너테이션과 Validator를 이용할 수 있다.

```java
// Custom Annotation
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = {JustOneOfManyValidator.class})
public @interface JustOneOfMany {
	String message() default "A review can be associated with either a book, a magazine or an article";
	Class<?>[] groups() default {};
	Class<? extends Payload>[] payload() default {};
}

// Validator
public class JustOneOfManyValidator
		implements ConstraintValidator<JustOneOfMany, Review> {
	@Override
	public boolean isValid(Review review, ConstraintValidatorContext ctx) {
		return Stream.of(
						review.getBook(), review.getArticle(), review.getMagazine())
				.filter(Objects::nonNull)
				.count() == 1;
	}
}

// 사용하기
@Entity
@JustOneOfMany
public class Review implements Serializable {
    ...
}
```