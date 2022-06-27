# 람다를 이용한 도메인 전용 언어
```
- 도메인 전용 언어`domain-specific languages, DSL`란 무엇이며 어떤 형식으로 구성되는가?
- DSL을 API에 추가할 때의 장단점
- JVM에서 활용할 수 있는 자바 기반 DSL을 깔끔하게 만드는 대안
- 최신 자바 인터페이스와 클래스에 적용된 DSL에서 배움 
- 효과적인 자바 기반 DSL을 구현하는 패턴과 기법
- 이들 패턴을 자바 라이브러리와 도구에서 얼마나 흔히 사용하는가?
```

* 개발자들은 보통 프로그래밍 언어도 결국 언어라는 사실을 잊곤 한다. 언어의 주요 목표는 메시지를 명확하고, 안전한 방식으로 전달하는 것이다.

* DSL은 작은, 범용이 아니라 특정 도메인 대상으로 만들어진 특수 프로그래밍 언어다. DSL은 도메인의 많은 특성 용어를 사용한다. 
 - `Maven`, `Ant`: 빌드 과정을 표현하는 DSL로 간주할 수 있다.
 - `HTML`: 웹페이지의 구조를 정의하도록 특화된 언어다.

* "메뉴에서 400칼로리 이하의 모든 요리를 찾으시오" 같은 쿼리를 프로그램적으로 구현한다고 가정하자. 
    ```
    while (block != null) {
        read(block, buffer)
        for(every record in buffer) {
            if (record.calorie < 400) {
                System.out.println(record.name);
            }
        }
        block = buffer.next();
    }
    ```

    위 코드에는 두 가지 문제가 있다. `locking`, `I/O`, 디스크 할당`disk allocation` 등과 같은 지식이 필요해 경험이 부족한 프로그래머가 구현하기엔 조금 어렵다는 점, 그리고 애플리케이션 수준이 아니라 시스템 수준의 개념을 다루어야 한다는 점이다. `SELECT name FROM menu WHERE calorie < 400`과 같이 표현하면 안 될까? 이는 자바가 아닌 DSL을 이용해 데이터베이스를 조작하자는 의미로 통하게 된다. 기술적으로 이런 종류의 DSL을 외부적 `external`이라 한다. 이는 데이터베이스가 텍스트로 구현된 SQL 표현식을 파싱하고 평가하는 API를 제공하는 것이 일반적이기 때문이다. 

    자바의 스트림 API를 이용하면 아래와 같이 줄일 수 있다.  스트림 API의 특성인 메서드 체인을 보통 자바의 루프의 복잡함 제어와 비교해 유창함을 의미하는 플루언트 스타일`fluent style`이라고 부른다. 
    ```
    menu.stream()
        .filter(d -> d.getCalories() < 400)
        .map(Dish::getName)
        .forEach(System.out::println)
    ```
    여기서 DSL은 외부적이 아니라 내부적`internal`이다. `SELECT FROM` 구문처럼 애플리케이션 수준의 기본값이 자바 메서드가 사용할 수 있도록 데이터베이스를 대표하는 한 개 이상의 클래스 형식으로 노출된다. 기본적으로 DSL을 만들려면 애플리케이션 수준 프로그래머에 어떤 동작이 필요하며, 이들을 어떻게 프로그래머에게 제공하는지 고민이 필요하다. 


## 1. 도메인 전용 언어 
* DSL은 특정 비즈니스 도메인의 문제를 해결하려고 만든 언어다. 범용 프로그래밍 언어가 아니다.

* DSL은 평문 언어가 아니다. 도메인 전문가가 저수준 비즈니스 로직을 구현하도록 만드는 것은 DSL의 역할이 아니다. 다음 두 가지 필요성을 생각하면서 DSL을 개발해야 한다. 
  - 의사 소통의 왕: 코드의 의도가 명확히 전달되어야 하며, 프로그래머가 아닌 사람도 이해할 수 있어야 한다. 
  - 한 번 코드를 구현하지만 여러 번 읽는다: 가독성은 유지보수의 핵심이다. 쉽게 이해할 수 있또록 코드를 구현해야 한다. 


### 1-1. DSL의 장점과 단점 
* 장점
  - 간결함: API는 비즈니스 로직을 간편하게 캡슐화하므로 반복을 피할 수 있고, 코드를 간결하게 만들 수 있다. 
  - 가독성: 도메인 영역의 용어를 사용하므로 비 도메인 전문가도 코드를 쉽게 이해할 수 있다. 결과적으로 다양한 조직 구성원 간에 코드와 도메인 영역이 공유될 수 있다. 
  - 유지보수: 잘 설계된 DSL로 구현한 코드는 쉽게 유지보수하고 바꿀 수 있다. 유지보수는 비즈니스 관련 코드, 즉 가장 빈번히 바뀌는 애플리케이션 부분에 특히 중요하다. 
  - 높은 수준의 추상화: DSL은 도메인과 같은 추상화 수준에서 동작하므로 도메인의 문제와 직접적으로 관련되지 않은 세부 사항을 숨긴다. 
  - 집중: 비즈니스 도메인의 규칙을 표현할 목적으로 설계된 언어이므로 프로그래머가 특정 코드에 집중할 수 있다. 
  - 관심사 분리`Separation of concerns`: 지정된 언어로 비즈니스 로직을 표현함으로써 애플리케이션의 인프라구조와 관련된 문제와 독립적으로 비즈니스 관련된 코드에서 집중하기가 용이하다. 결과적으로 유지보수가 쉬운 코드를 구현한다. 

* 단점 
  - DSL 설계의 어려움: 간결하게 제한적인 언어에 도메인 지식을 담는 것이 쉬운 작업은 아니다. 
  - 개발 비용: 코드에 DSL을 추가하는 작업은 초기 프로젝트에 많은 비용과 시간이 소모되는 작업이다. 
  - 추가적인 우회 레이어`Additional indirection layer`: DSL은 추가적인 레이어로 도메인 모델을 감싸며 이때 계층을 최대한 작게 만들어 성능 문제를 회피한다. 
  - 새로 배워야 하는 언어
  - 호스팅 언어 한계: 일부 자바 같은 범용 프로그래밍 언어는 장황하고 엄격한 문법을 가졋다. 이런 언어로는 사용자 친화적 DSL을 만들기가 힘들다. 


### 1-2. JVM에서 이용할 수 있는 다른 DSL 해결책
* DSL의 카테고리를 구분하는 가장 흔한 방법은 마틴 파울러가 소개한 방법으로, 내부 DSL과 외부 DSL을 나누는 것이다. 내부 DSL(임베디드 DSL이라 불림)은 순수 자바 코드와 같은 기존 호스팅 언어를 기반으로 구현하는 반면, 스탠드얼론`standalone`이라 불리는 외부 DSL은 호스팅 언어와는 독립적으로 자체의 문법을 가진다. 


#### 내부 DSL
* 자바로 구현한 DSL을 의미한다. 


#### 다중 DSL
* JVM에서 실행되는 언어는 100개가 넘는다. 스칼라, 루비처럼 유명한 언어들, 그리고 코틀린, 실론 같이 스칼라와 호환성을 유지하면서 단순하고 쉽게 배울 수 있다는 강점을 가진 새 언어도 있다. 이들은 모두 자바보다 젊고, 제약을 줄이고, 간편한 문법을 지향하도록 설계되었다. 


#### 외부 DSL
* 자신만의 문법과 구문으로 새 언어를 설계해야 한다. 새 언어를 파싱하고, 파서의 결과를 분석하고, 외부 DSL을 실행할 코드를 만들어야 한다. 정 이 방법을 택해야 한다면 ANTLR 같은 자바 기반 파서 생성기를 이용하면 도움이 된다. 

* 외부 DSL을 개발하는 가장 큰 장점은 외부 DSL이 제공하는 무한한 유연성이다. 자바로 개발된 인프라구조 코드와 외부 DSL로 구현한 비즈니스 코드를 명확하게 분리한다는 것도 장점이다. 



## 2. 최신 자바 API의 작은 DSL 
### 2-1. Stream API: 컬렉션을 조작하는 DSL
* Stream 인터페이스는 native java API에 작은 내부 DSL을 적용한 좋은 예다. "ERROR"라는 단어로 시작하는 파일의 첫 40행을 수집하는 작업을 수행한다고 가정하는 코드를 만들어보자. 
  ```java
  List<String> errors = new ArrayList<>();
  int errorCount = 0;
  BufferedReader br = new BufferedReader(new FileReader(fileName));

  String line = br.readLine();
  while (errorCount < 40 && line != null) {
    if (line.startsWith("ERROR")) {
        errors.add(line);
        errorCount++;
    }

    line = br.readLine();
  }
  ```

  코드는 이미 장황해 의도를 한 눈에 파악하기 어렵다. 문제가 분리되지 않아 가독성과 유지보수성 모두 저하되었다. 같은 의무를 가진 코드가 여러 행에 분산되어있다.

  ```java
  List<String> errors = Files.lines(Paths.get(fileName))
                             .filter(line -> line.startsWith("ERROR"))
                             .limit(40)
                             .collect(toList());
  ```


### 2-2. Collectors: 데이터를 수집하는 DSL
* `Collector` 인터페이스는 데이터 수집을 수행하는 DSL로 간주할 수 잇다. 특히 `Comparator` 인터페이스는 다중 필드 정렬을 지원하도록 합쳐질 수 있으며, `Collectors`는 다중 수준 그룹화를 달성할 수 있도록 합쳐질 수 있다. 
  ```java
  Map<String, Map<Color, List<Car>>> carsByBrandAndColor = cars.stream().collect(groupingBy(Car::getBrand, groupingBy(Car::getColor)));
  ```


## 3. 자바로 DSL을 만드는 패턴과 기법
* DSL은 특정 도메인 모델에 적용할 친화적이고 가독성 높은 API를 제공한다. 

### 3-1. Method chain
```java
final var order = forCustomer("BigBank")
        .buy(80)
        .stock("IBM")
        .on("NYSE")
        .end();
```

```java
public class OrderBuilder {

    private final Order order = new Order();

    private OrderBuilder(final String customer) {
        order.setCustomer(customer);
    }

    public static OrderBuilder forCustomer(final String customer) {
        return new OrderBuilder(customer);
    }

    public TraderBuilder buy(final int quantity) {
        return new TradeBuilder(this, Trade.Type.BUY, quantity);
    }

    public OrderBuilder addTrade(final Trade trade) {
        order.addTrade(trade);
        return this;
    }

    public Order end() {
        return order;
    }

}
```

* `TradeBuilder`, `StockBuilder` 등을 추가적으로 정의해줄 수 있는데, OrderBuilder가 끝날 때까지 다른 거래를 플루언트 방식으로 추가할 수 있다. 주문에 사용한 파라미터가 빌더 내부로 국한된다는 다른 잇점도 제공한다. 정적 메서드 사용을 최소화하고 메서드 이름이 인수의 이름을 대신하도록 만듦으로써 이런 형식의 DSL 가독성을 개선하는 효과를 더한다. 이런 기법을 적용한 플루언트 DSL에는 문법접 잠음이 최소화된다. 

* 빌더를 구현해야 한다는 것이 메서드 체인의 단점이다. 상위 수준의 빌더를 하위 수준의 빌더와 연결할 많은 접착 코드가 필요하다. 


### 3-2. Nested functions
```java
final order = order(
    "BigBank",
    buy(80, stock("IBM", on("NYSE")), at(125.00)),
    sell(50, stock("GOOGLE", on("NASDAQ")), at(375.00))
);
```

```java
public class OrderBuilder {

    public static Order order(String customer, Trade... trades) {
        Order order = new Order();
        order.setCustomer(customer);
        Stream.of(trades).forEach(order::addTrade);
        return order;
    }

    public static Trade buy(int quantity, Stock stock, double price) {
        return buildTrade(quantity, stock, price, Trade.Type.BUY);
    }

    public static Trade sell(int quantity, Stock stock, double price) {
        return buildTrade(quantity, stock, price, Trade.Type.SELL);
    }

    private static Trade buildTrade(int quantity, Stock stock, double price, Trade.Type tradeType) {
        Trade trade = new Trade();
        trade.setQuantity(quantity);
        trade.setType(tradeType);
        trade.setStock(stock);
        trade.setPrice(price);
        return trade;
    }

    public static double at(double price) {
        return price;
    }

    public static Stock stock(String symbol, String market) {
        Stock stock = new Stock();
        stock.setSymbol(symbol);
        stock.setMarket(market);
        return stock;
    }

    public static String on(String market) {
        return market;
    }
}
```

* 이는 인수 목록을 정적 메서드에 넘겨줘야 한다는 제약이 있다. 인수의 역할을 확실하게 만드는 여러 더미 메서드를 이용해 이 문제를 조금은 완화할 수 있다. 


### 3-3. 람다 표현식을 이용한 함수 시퀀싱
```java
var order = order(o -> {
    o.forCustomer("BigBank");

    o.buy(t -> {
        t.quantity(80);
        t.price(125.00);
        t.stock(s -> {
            s.symbol("IBM");
            s.market("NYSE");
        })
    });

    o.sell(t -> {
        t.quantity(50);
        t.price(375.00);
        t.stock(s -> {
            s.symbol("GOOGLE");
            s.market("NASDAQ");
        })
    })
})
```

* 메서드 체인 패턴처럼 플루언트 방식으로 거래 주문을 정의할 수 잇다. 또한 중첩 함수 형식처럼 다양한 람다 표현식의 중첩 수준과 비슷하게 도메인 객체의 계층 구조를 유지한다. 

* 그러나 많은 설정 코드가 필요하며, DSL 자체가 자바 8 람다 표현식 문법에 의한 잡음의 영향을 받는다는 것이 이 패턴의 단점이다. 


### 3-4. 조합
```java
var order = forCustomer(
    "BigBank",
    buy(t -> t.quantity(80).stock("IBM").on("NYSE").at(125.00)),
    sell(t -> t.quantity(80).stock("GOOGLE").on("NASDAQ").at(125.00)),
);
```

* 패턴들의 장점은 살릴 수 있지만, 사용자가 배우는 데 오랜 시간이 걸릴 수 있다.


### 3-5. DSL에 메서드 참조 사용하기 
* 세금 계산 기능을 추가한다고 가정해보자. 이 구현의 가독성 문제는 쉽게 드러난다. 
```java
double value = calculate(order, true, false, true);
```

* 플루언트 방식으로 boolean flag를 설정하는 최소 DSL을 제공하는 것이 더 좋은 방법이다. 
```java
double value = new TaxCalculator().withTaxRegional()
        .withTaxSurcharge()
        .calculate(order);
```

* 코드가 장황하다는 것이 위 기법의 문제다. 함수형으로 바꾸면 더 간결하고 유연한 방식으로 사용할 수 있다.
```java
double value = new TaxCalculator().with(Tax::regional)
        .with(Tax::surcharge)
        .calculate(order);
```


## 4. 실생활의 자바 8 DSL 
| Pattern                          | Pros                                                                                                                                                                                    | Cons                                                                                                   |
|----------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| Method chain                     | - 메서드 이름이 키워드 인수 역할 - 선택형 파라미터와 잘 동작  - DSL 사용자가 정해진 순서로 메서드를 호출하도록 강제 가능 - 정적 메서드를 최소화하거나 없앨 수 있음 - 문법적 잡음 최소화 | - 구현이 장황  - 접착 코드 필요  - 들여쓰기 규칙으로만 도메인 객체 계층 정의                           |
| Nested functions                 | - 구현 장황함 줄일 수 O - 함수 중첩으로 도메인 객체 계층 반영                                                                                                                           | - 정적 메서드 사용 빈번 - 이름이 아닌 위치로 인수 정의 - 선택형 파라미터를 처리할 메서드 오버로딩 필요 |
| Function sequencing with lambdas | - 선택형 파라미터와 잘 동작 - 정적 메서드를 최소화하거나 없앨 수 있음 - 람다 중첩으로 도메인 객체 계층 반영 - 빌더 접착 코드 X                                                          | - 구현이 장황 - 람다 표현식으로 인한 문법적 잡음                                                       |


### 4-1. 라이브러리 
* SQL 매핑 도구, behavior-driven 개발 프레임워크, Enterprise Integration Patterns를 구현하는 세 가지 자바 라이브러리


#### 4-1-1. jOOQ
* SQL 질의 작성, 실행하는데 필요한 DSL 제공 

* Stream API와 조합해 사용 가능 

```java
create.selectFrom(BOOK)
      .where(BOOK.PUBLISHED_IN.eq(2016))
      .orderBy(BOOK.TITLE);
```


#### 4-1-2. 큐컴버
* 동작 주도 개발`Behavior-driven development(BDD)`은 테스트 주도 개발의 확장으로 다양한 비즈니스 시나리오를 구조적으로 서술하는 간단한 도메인 전용 스크립팅 언어를 사용함 

* Cucumber는 다른 BDD 프레임워크와 마찬가지로 이들 명령문을 실행할 수 있는 테스트 케이스로 변환한다. 결과적으로 이 개발 기법으로 만든 스크립트 결과물은 실행할 수 있는 테스트임과 동시에 비즈니스 기능의수용 기준이 된다. 
```
Feature: Buy stock
  Scenario: Buy 10 IBM stocks
    Given the price of a "IBM" stock is 125$
    When I buy 10 "IBM"
    Then the order value should be 1250$
```

* 전체 조건 정의(Given), 시험하려는 도메인 객체의 실질적 호출(When), 테스트 케이스의 결과를 확인하는 assertion(Then)

* Cucomber 어노테이션을 이용해 테스트 시나리오 구현이 가능하다. 또한 람다 표현식을 지원하면서 어노테이션을 제거하는 다른 문법ㅇ르 큐컴버로 개발할 수 있다. 


#### 4-1-3. 스프링 통합 `Spring integration`
* 유명한 엔터프라이즈 통합 패턴을 지원할 수 있도록 의존성 주입에 기반한 스프링 프로그래밍 모델을 확장한다. 핵심 복표는 복잡한 엔터프라이즈 통합 솔루션을 구현하는 단순한 모델을 제공하고 비동기, 메시지 주도 아키텍처를 쉽게 적용할 수 있게 돕는 것이다. 

* Channel, endpoints, pollers, channel interceptors 등 메시지 기반의 애플리케이션에 필요한 가장 공통 패턴을 모두 구현한다. 여러 엔드포인트를 한 개 이상의 메시지 흐름으로 조합해서 통합 과정이 구성된다. 


