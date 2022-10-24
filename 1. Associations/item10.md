### Item10. 하이버네이트 지정 @Where 애너테이션을 이용하여 연관관계 필터링하는 방법

> 주의
> 
> JOIN FETCH WHERE(item39)나 @NamedEntityGraph(item7과 item8)을 사용하지 못하는 경우에만 이것을 사용하라.

이 방식은 자동으로 Lazy처리된다. 즉, 해당 연관관계를 로딩할 때 별도의 SELECT 쿼리가 수행된다. ...
또, Where 조건이 파라미터화되지 않은 상태로 전달된다.
> 
> 그러므로 JOIN FETCH WHERE로 처리하는 것이 다음 두 가지 측면에서 더 효율적이다.
> - 하나의 SELECT 쿼리로 같이 조회할 수 있다.
> - 쿼리 바인딩 파라미터를 이용할 수 있다.
>
> item109에서 @Where 애너테이션이 더 유용한 경우를 소개한다. - Soft Deletes 구현

