## Item15. Persistence Layer에서 Java8 Optional 이용하기

#### 엔티티의 Getter에서 사용할 수 있다.
> Optional을 사용하지 말아야 하는 경우
> 
> - 엔티티의 필드
> - 생성자나 Setter의 매개변수
> - 원시자료형이나 컬렉션을 리턴하는 Getter 메소드의 리턴 타입
> - 유일키<sub>Primary Key</sub>를 리턴하는 Getter 메소드의 리턴 타입

#### Repository에서 사용할 수 있다.
생략