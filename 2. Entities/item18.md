## Item18. 변경 트래킹을 사용하는 이유와 활성화하는 방법

> Dirty Checking의 목적은 엔티티 내부 정보가 바뀌었는지 확인하여 추후 변경 데이터를 저장하기 위해 UPDATE DML를 수행하기 위함임

Hibernate5 이전에는 Reflection을 이용하여 Dirty Checking을 했음.
-> Entity가 작을 때는 별 문제가 없지만 커지면 성능에 큰 영향을 줌

Hibernate5 이후로는 변경 트래킹<sub>Dirty Tracking</sub> 메커니즘을 이용하는데 이것은 엔티티가 직접 자신의 속성의 변경사항을 트래킹하는 능력임
-> 당연히 이것이 성능이 더 뛰어남
> 대신 Hibernate Bytecode Enhancement를 이용해야 하기 떄문에 다음과 같은 설정을 해줘야 함
> ```xml
> <plugin>
>    <groupId>org.hibernate.orm.tooling</groupId>
>    <artifactId>hibernate-enhance-maven-plugin</artifactId>
>    <version>${hibernate.version}</version>
>    <executions>
>        <execution>
>            <configuration>
>                <failOnError>true</failOnError>
>                <enableDirtyTracking>true</enableDirtyTracking> <!-- 여기 -->
>            </configuration>
>            <goals>
>                <goal>enhance</goal>
>            </goals>
>        </execution>
>    </executions>
> </plugin>
> ```

> 참고
> 
> Gradle에서도 설정할 수 있는데 자세한 내용은 [이 글](https://docs.jboss.org/hibernate/orm/5.4/topical/html_single/bytecode/BytecodeEnhancement.html)을 참고할 것
> 
> Hibernate Plugin을 적용한 뒤 enhance 설정을 추가해줘야 함 (enhancement는 기본 설정이 아님)