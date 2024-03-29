# Today I Learned
| 구분  | 내용                       |
|------|---------------------------|
| DATE | 2023.01.09                |
| PART | 1. Introduce JPA          |


## 1. 기억하고 싶은 내용

### 1.1. SQL을 직접 다룰 때 발생하는 문제점
#### 반복, 반복, 그리고 반복
* SQL 직접 다룰 경우 `DAO - SQL - JDBC API - ResultSet mapping`까지 지속해서 반복되는 코드가 발생한다. 
    ```java
    ResultSet rs = stmt.executeQuery(sql);
    ...
    String id = rs.getString("id");
    String name = rs.getString("name");

    Member member = new Member(id, name);
    ```

* 데이터베이스가 아니라 자바 컬렉션에 보관한다면 코드는 어떻게 변할까?
    ```java
    members.add(member);
    ```

* <strong><u>데이터베이스는 객체 구조와는 다른 데이터 중심의 구조, 객체를 데이터베이스에 직접 저장하거나 조회할 수는 없다. 따라서 개발자가 객체지향 애플리케이션과 데이터베이스 중간에서 SQL과 JDBC API를 사용해서 변환 작업을 직접 해주어야 한다. 그러나, CRUD를 위해 너무 많은 SQL과 JDBC API를 코드로 작성해야 한다.</u></strong>


#### SQL에 의존적인 개발 
* 필드가 추가된다면? 엔티티 뿐만 아니라, 등록 코드, 조회 코드 등에 대해 해당 필드를 모두 반영해주어야 하는 문제가 생긴다. 

* 연관 객체가 생긴다면? 연관된 팀을 위해 또 다른 SQL을 추가해야 하며, 추가 전까지 팀을 찾을 수 없단 에러가 생긴다면 `DAO`에서 SQL을 직접 확인해야 원인을 알 수 있다. 

* 즉, 데이터 접근 계층을 사용해 SQL을 숨기더라도 어쩔 수 없이 DAO를 열어서 어떤 SQL이 실행돼야 하는지 확인해야 한다. 

* SQL에 모든 것을 의존하는 상황에서는 엔티티를 신뢰하고 사용할 수 없다. DAO를 열어서 어떤 SQL이 실행되고 어떤 객체들이 함께 조회되는지 일일이 확인해야 한다. 이는 진정한 의미의 계층 분할이 아니다. <strong><u>물리적으로 SQL과 JDBC API를 데이터 접근 계층에 숨기는 데 성공했을지 몰라도, 논리적으로 엔티티와 아주 강한 의존관계를 가지고 있다. 이런 강한 의존관계 때문에 회원을 조회할 때는 물론이고 회원 객체에 필드를 하나 추가할 때도 DAO의 CRUD 코드와 SQL 대부분을 변경해야 하는 문제가 발생한다.</u></strong>

* 애플리케이션에서 SQL을 직접 다룰 때 발생하는 문제점을 요약하면 다음과 같다. 
    - 진정한 의미의 계층 분할이 어렵다. 
    - 엔티티를 신뢰할 수 없다. 
    - SQL에 의존적인 개발을 피하기 어렵다. 


#### JPA와 문제 해결 
* JPA는 이런 문제들을 어떻게 해결할까? JPA를 사용하면 객체를 데치터베이스에 저장하고 관리할 때, 개발자가 직접 SQL을 작성하는 것이 아니라 JPA가 제공하는 API를 사용하면 된다. 그러면 JPA가 개발자 대신에 적절한 SQL을 생성해서 데이터베이스에 전달한다. 
    ```
    jpa.persist(member); // 저장 
    jpa.find(Member.class, id); // 조회
    jpa.setName("Changed Name"); // 수정 
    member.getTeam(); // 연관 객체 조회 
    ```



### 1.2. 패러다임의 불일치
* 객체지향 프로그래밍은 시스템의 복잡성을 제어할 수 있는 다양한 장치들을 제공한다. 비즈니스 요구사항을 정의한 도메인 모델도 객체로 모델링하면 객체지향 언어가 가진 장점들을 활용할 수 있다. 문제는 이렇게 정의한 도메인 모델을 저장할 때 발생한다. 메모리가 아닌 어딘가에 영구 보관해야 한다. 

* 관계형 데이터베이스는 데이터 중심으로 구조화되어 있고, 집합적인 사고를 요구한다. 그리고 객체지향에서 이야기하는 추상화, 상속, 다형성 같은 개념이 없다. 

* 객체와 관계형 데이터베이스는 지향하는 목적이 서로 다르므로, 둘의 기능과 표현 방법도 다르다. 이것을 객체와 관계형 데이터베이스의 패러다임 불일치 문제라고 한다. 따라서 객체 구조를 테이블 구조에 저장하는 데는 한계가 있다. 

* 자바라는 객체지향 언어로 개발하고 데이터는 관계형 데이터베이스에 저장해야 한다면, 패러다임의 불일치 문제를 개발자가 중간에서 해결해야 한다. 문제는 이런 객체와 관계형 데이터베이스 사이의 패러다임 불일치 문제를 해결하는 데 너무 많은 시간과 코드를 소비하는 데 있다. 


#### 상속 
* 객체는 상속 가능하지만, 테이블을 상속이라는 기능이 없다. 그나마 데이터베이스 모델링의 super-sub type 관계를 사용하면 객체 상속과 가장 유사한 형태로 테이블을 설계할 수 있다. 

* JDBC API를 사용해 부모와 자식 객체를 저장하려면, 각 객체를 super, sub type으로 분해해 따로 저장하고, 조회 시에는 join해 그 결과로 객체를 생성해야 한다. 모든 과정이 모두 패러다임의 불일치를 해결하고자 소모하는 비용이다. 해당 객체들을 데이터베이스가 아닌 자바 컬렉션에 보관한다면 그냥 사용하면 된다. 
    ```java
    list.add(album);
    list.add(movie);

    Item album = list.get(albumId);
    ```

* JPA는 상속과 관련된 패러다임의 불일치 문제를 대신 해결해주며, 자바 컬렉션에 객체를 저장하듯 저장하면 된다. 
    ```java
    jpa.persis(album);
    ```


#### 연관관계
* 객체는 참조를 사용해서 다른 객체과 연관관게를 가지고, 참조에 접근해 연관된 객체를 조회한다. 반면 테이블은 외래 키를 사용해 다른 테이블과 연관관계를 가지고 조인을 사용해 연관된 테이블을 조회한다. 이 패러다임 불일치는 객체지향 모델링을 거의 포기하게 만들 정도로 극복하기 어렵다. 

* 객체의 연관관계는 참조가 있는 방향으로만 조회 가능하다. (ex: `member.getTeam()`은 가능, `team.getMemeber()`는 member가 없다면 불가능 -> 데이터베이스는 조인 가능)

* JPA는 연관관계와 관련된 패러다임의 불일치 문제를 해결해준다. 
    ```java
    member.setTeam(team);
    jpa.persist(member);
    ```


#### 객체 그래프 탐색 
* 객체에서 회원이 소속된 팀을 조회할 때는 참조를 사용해 연관된 팀을 찾으면 되는데, 이 연관된 객체들은 꼬리를 물고 늘어난다. 즉, 객체는 마음껏 객체 그래프를 탐색할 수 있어야 한다. 그러나 SQL을 실행해 조회한 결과에서는 마음껏 객체 그래프를 탐색할 수 없다. 실제로 조회된 데이터만 탐색 가능하다. 즉, SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할지 정해지는 것이다. 언제 끊어질지 모를 객체 그래프를 함부로 탐색할 수는 없다. 

* `SQL을 직접 다룰 때 발생하는 문제점`에서도 언급했듯 JPA는 연관된 객체를 사용하는 시점에 적절한 SELECT SQL을 실행한다. 따라서 JPA를 사용하면 연관된 객체를 신뢰하고 마음껏 조회할 수 있다. 이 기능은 실제 객체를 사용하는 시점까지 데이터베이스 조회를 미룬다고 해서 `lazy loading`이라 한다. 
    ```java
    Member member = jpa.find(Member.class, id);

    Order order = member.getOrder();
    order.getOrderDate(); // Order 사용 시점에 SELECT ORDER SQL
    ```


#### 비교 
* 데이터베이스는 기본 키의 값으로 각 로우`row`를 구분한다. 반면에 객체는 동일성`identity` 비교와 동등성`equality` 비교라는 두 가지 비교 방법이 있다. 따라서 테이블의 로우를 구분하는 방법과 객체를 구분하는 방법에는 차이가 있다. 

* JPA는 같은 트랜잭션일 떄 같은 객체가 조회되는 것을 보장한다. 객체 비교하기는 분산 환경이나 트랜잭션이 다른 상황까지 고려하면 더 복잡해진다. 자세한 내용은 추후 더 알아보자. 



### 1.3. JPA란 무엇인가? 
* JPA`Java Persistence API`는 자바 진영의 `ORM` 기술에 대한 API 표준 spec이다. 
    `application - jpa - jdbc api - db`

* ORM`Object-Relation Mapping`은 객체와 테이블을 매핑한다는 뜻이다. ORM 프레임워크는 객체와 테이블을 매핑해 패러다임의 불일치 문제를 개발자 대신 해결해준다. 

* ORM 프레임워크는 단순히 SQL을 개발자 대신 생성해서 데이터베이스에 전달해주는 것뿐만 아니라, 앞서 이야기한 다양한 패러다임의 불일치 문제들도 해결해준다. 따라서 객체 측면에서는 정교한 객체 모델링을 할 수 있고 관계형 데이터베이스는 데이터베이스에 맞도록 모델링하면 된다. 그리고 둘을 어떻게 매핑해야 하는지 매핑 방법만 ORM 프레임워크에게 알려주면 된다. 

* 어느 정도 성숙한 객체지향 언어에는 대부분 ORM 프레임워크들이 있는데 각 프레임워크의 성숙도에 따라 단순히 객체 하나를 CRUD하는 정도의 기능만 제공하는 것부터 패러다임 불일치 문제를 대부분 해결해주는 ORM 프레임워크도 있다. (Java에서 가장 많이 사용하는 프레임워크: `Hibernate`)

* JPA 사용 이유 
    - 생산성
    - 유지보수 
    - 패러다임의 불일치 해결 
    - 성능 
    - 데이터 접근 추상화와 벤더 독립성 
    - 표준


## 2. 소감
이 책의 초반부에서는 JPA 도입 이전의 문제 상황들에 대해 이야기한다. 그러나 단순 CRUD의 반복을 피하기 위해서 JPA를 사용하는 것은 아닐 것이다. 또한 패러다임 불일치라는 것은 결국 서로 다른 목적, 필요에 의해 태어났다는 것인데, 그것을 맞추고자 강제하는 게 꼭 바람직하지만은 않을 것이다. 결국 JPA의 사용 목적을 제대로 이해하고자 한다면 객체 지향 프로그래밍을 잘 이해해야 하는데, 아직까지는 크게 와닿지는 않는다. 더불어 쿼리가 복잡해질수록 쓰기 어려워지고, 결국 native query나 QueryDSL까지 써야 하는데 굳이 써야 되나? 싶은 생각이 들 때도 있었다. 

객체 그래프 탐색은 사실 kotlin의 `safe call operator`가 있다면 조금 더 안전하게 탐색할 수 있고, 해당 비즈니스에 최적화된 쿼리와 객체를 통해 참조하는 것이 더 바람직하지 않나 싶은 생각도 든다. 다시 말해서, 해당 비즈니스에 필요하지 않은 필드들이 꼭 select 되어야 하냐는 것이다. 더불어서 지나치게 깊은 depth로 가져오는 것 또한 지양하는 것이 좋다고 알고 있는데, JPA의 필요성이 무엇인지 써야 하는 이유는 무엇인지가 아직은 와닿지 않는다. 이전에 한 번 보았던 내용들이지만, `jpa`를 비롯해 `spring data jdbc` + `mybatis`을 사용해보았으니 조금 더 현실적인 관점에서, 실용적인 관점에서 이 책을 볼 수 있을 것 같다. 내가 지금껏 `jpa`를 잘 이해하고 있었던 것인지, 제대로 쓰고 있었던 것인지에 대해서 말이다.1장 후반부에 나오는 아래 내용이 아마 중심이지 않을까 싶다. 
```
더 어려운 문제는 객체지향 애플리케이션답게 정교한 객체 모델링을 할수록 패러다임의 불일치 문제가 더 커진다는 것이다. 그리고 이 틈을 메우기 위해 개발자가 소모해야 하는 비용도 점점 더 많아진다. 결국, 객체 모델링은 힘을 잃고 점점 데이터 중심의 모델로 변해간다. 
```