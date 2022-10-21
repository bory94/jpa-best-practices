## @ManyToMany 관계에서 Set이 List보다 좋은 이유

> 참고: Hibernate는 @ManyToMany 관계를 두 개의 단방향 @OneToMany 관계로 처리한다.
>
> 그래서 엔티티의 삭제 혹은 재정렬을 할 경우, 연결 테이블의 모든 데이터를 삭제하고 영속성 컨텍스트 내의 데이터만 다시 등록한다.

### List일 때 Book 하나를 삭제한다면...
DELETE ALL -> INSERT Remains 방식으로 동작한다.
남은 항목이 세 개라면 총 DML을 4 <sub>1 * DELETE + 3 * INSERT</sub> 회 수행한다.

### Set일 때 Book 하나를 삭제한다면...
해당 author_book 연결 테이블에 대한 DELETE 1회만 수행한다.

### ResultSet의 순서를 보존하는 방법
java.util.ArrayList는 입력된 요소들의 순서가 잘 보존되지만 java.util.HashSet은 그렇지 않다.
 
입력 순서를 보존하고 싶으면 다음 두 가지 중 하나를 선택하라.

- @OrderBy를 이용하고 어떤 컬럼으로 정렬할지 지정한다.
- 별도의 정렬용 컬럼을 생성하고 @OrderColumn으로 가리킨다.
> 참고: Set을 이용하면서 정렬을 수행한다면 java.util.LinkedHashSet을 이용하라. 

### 결론: @ManyToMany 관계에서는 항상 List 대신 Set을 이용하라.
다른 연관 관계에서는 상황에 맞게 사용하면 된다. 다만 List를 이용하는 경우 [HHH-3855](https://hibernate.atlassian.net/browse/HHH-5855) 이슈를 주의해야 한다.
