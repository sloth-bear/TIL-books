# `null` 대신 `Optional` 클래스

## Index
* `null` 참조의 문제점과 `null`을 멀리해야 하는 이유
* `null` 대신 `Optional`: `null`로부터 안전한 도메인 모델 재구현하기
* `Optional` 활용: null 확인 코드 제거하기
* `Optional`에 저장된 값을 확인하는 방법
* 값이 없을 수도 있는 상황을 고려하는 프로그래밍


## null 참조의 문제점과 null을 멀리해야 하는 이유
1965년 토니 호어`Tony Hoare`라는 영국 컴퓨터과학자가 힙에 할당되는 레코드를 사용하며 형식을 갖는 최초의 프로그래밍 언어 중 하나인 알골`ALGOL W`을 설계하면서 처음 `null` 참조가 등장했다. 그는 '구현하기가 쉬웠기 때문에 `null`을 도입했다.'라고 그 당시를 회상한다. '컴파일러의 자동 확인 기능으로 모든 참조를 안전하게 사용할 수 있을 것'을 목표로 정했다. 그 당시에는 `null` 참조 및 예외로 값이 없는 상황을 가장 단순하게 구현할 수 있다고 판단했고 결과적으로 `null` 및 관련 예외가 탄생했다. 여러 해가 지난 후 호어는 당시 `null` 및 예외를 만든 결정을 가리켜 '십억 달러짜리 실수'라고 표현했다. 

자바를 퐇마해서 최근 수십 년간 탄생한 대부분의 언어 설계에는 `null` 참조 개념을 포함한다. -가장 형식적인 함수형 언어인 하스켈, ML 등은 예외적으로 `null` 참조 개념을 사용하지 않는 언어다.-

### 값이 없는 상황을 어떻게 처리할까? 
```java
public class Person {
    private Car car;
    public Car getCar() { return car; }
}

public class Car {
    private Insurance insurance;
    public Insurance getInsurance() { return insurance; }
}

public class Insurance {
    private String name;
    public String getName() { return name; }
}
```

아래 코드에서는 어떤 문제가 발생할까? 
```java
public Insurace getCarInsuranceName(final Person person) {
    // Person, Car, Insurance 중 하나라도 null이라면 NullPointerException이 발생할 것이다.
    return person.getCar().getInsurance().getName();
}
```

#### 보수적인 자세로 `NullPointerException` 줄이기 
`NullPointerException`을 방지하려면 필요한 곳에 다양한 `null` 확인 코드를 추가해서 `null` 예외 문제를 해결하려 할 것이다. (더 보수적이라면 반드시 필요하지 않은 코드에까지)
```java
// recurring pattern, deep doubt 
public String getCarInsuranceName(final Person person) {
    if (person != null) {
        Car car = person.getCar();
        if (car != null) {
            Insurance insurance = car.getInsurace();
            if (insurance != null) {
                return insurance.getName();
            }
        }
    }
    return "Unknown";
}
```

이렇게도 표현할 수 있다. 이 경우엔 너무 많은 출구가 생겨 유지보수가 어려워진다.
```java
private static final String UNKNOWN_NAME = "Unknown";

public String getCarInsuranceName(final Person person) {
    if (person == null) { return UNKNOWN_NAME; }

    Car car = person.getCar();
    if (car == null) { return UNKNOWN_NAME; }

    Insurance insurance = car.getInsurance();
    if (insurance == null) { return UNKNOWN_NAME; }

    return insurance.getName();
}
```

앞의 코드는 쉽게 에러를 일으킬 수 있다. `null`로 값이 없다는 사실을 표현하는 것은 좋은 방법이 아니다. 값이 있거나 없음을 표현할 수 있는 좋은 방법이 필요하다. 


#### `null` 때문에 발생할 수 있는 문제
* 에러의 근원이다. 
* 코드를 어지럽힌다. 
* 아무 의미가 없다. `null`은 아무 의미도 표현하지 않는다. 특히 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다. 
* 자바 철학에 위배된다. 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 예외가 있는데, 그것이 바로 `null` 포인터다.
* 형식 시스템에 구멍을 만든다. `null`은 무형식이며 정보를 포함하고 있지 않으므로 모든 참조 형식에 `null`을 할당할 수 있다. 이런식으로 `null`이 할당되기 시작하면서 시스템의 다른 부분으로 `null`이 퍼졌을 때 애초에 `null`이 어떤 의미로 사용되었는지 알 수 없다. 
형식 시스템에 구멍을 만든다. 


#### 다른 언어는 `null` 대신 무얼 사용하나?
최근 그루비 같은 언어에서는 안전 네비게이션 연산자`safe navigation operator`(?.)를 도입해서 `null` 문제를 해결했다. 그루비 안전 내비게이션 연산자를 이용하면 `null` 참조 예외 걱정 없이 객체에 접근할 수 잇다. 이때 호출 체인에 `null`인 참조가 있음변 결과로 `null`이 반환된다. 
```groovy
def carInsuranceName = person?.car?.insurance?.name
```

* 하스켈, 스칼라 등의 함수형 언어: 아예 다른 관점에서 `null` 문제를 접근
  * 하스켈: 선택형 값 `Maybe` 형식 제공
  * 스칼라: T 형식의 값을 갖거나 아무 값도 갖지 않을 수 있는 `Option[T]` 구조 제공 


## null 대신 `Optional`: null로부터 안전한 도메인 모델 재구현하기

자바 8은 '선택형값' 개념의 영향을 받아서 `java.util.Optional<T>`라는 새로운 클래스를 제공한다. `null`을 `Optional`로 바꿀 때 우리 도메인 모델에서 선택형 값에 접근하는 방법도 달라져야 한다. 궁극적으로 더 좋은 API를 설계하는 방법을 취득하게 된다. 즉, 우리 API 사용자는 메서드의 시그니처만 보고도 선택형 값을 기대해야 하는지 판단할 수 있다. `Optional`은 선택형 값을 캡슐화하는 클래스다. `Optional` 클래스를 사용함으로써 모델의 의미`semantic`가 더 명확해졌음을 확인할 수 있다. `Optional`의 역할은 더 이해하기 쉬운 API를 설계하도록 돕는 것이다. 즉, 메서드의 시그니처만 보고도 선택형 값이닞 여부를 구별할 수 있다. 

```java
public class Person {
    private Optional<T> car;
    public Optional<Car> getCar() { return car; }
}

public class Car {
    private Optional<Insurance> insurance;
    public Optional<Insurance> getInsurance() { return insurance; }
}

public class Insurance {
    private String name;
    public String getName() { return name; }
}
```


## `Optional` 활용: null 확인 코드 제거하기 -- 적용 패턴
```java
// 암묵적 도메인 지식에 의존하지 않고 명시적으로 형식 시스템을 정의할 수 있다. 
public String getCarInsuranceName(final Optional<Person> person) {
    return person.flatMap(Person::getCar) // Optional 평준화: 두 Optional을 합치면서 둘 중 하나라도 null일 경우 빈 Optional 생성하는 연산
        .flatMAp(Car::getInsurance)
        .map(Insurance::getName)
        .orElse("Unknown");

}
```

* 도메인 모델에 `Optional`을 사용했을 때 데이터를 직렬화할 수 없는 이유 
  * Java 언어 아키텍트인 브라이언 고츠`Brian Goetz`는 `Optional`의 용도가 '선택형 반환값을 지원하는 것'이라고 못박았다. 즉, 도메인 모델에서 값이 꼭 있어야 하는지 값이 없을 수 있는지 여부를 구체적으로 표현할 수는 있지만, 이와 다른 용도로만 `Optional` 클래스를 사용할 것을 가정했다. 
  * `Optional` 클래스는 필드 형식으로 사용할 것을 가정하지 않았으므로 `Serializable` 인터페이스를 구현하지 않는다. 따라서 우리 도메인 모델에 `Optional`을 사용한다면 직렬화 모델을 사용하는 도구나 프레임워크에서 문제가 생길 수 있다.
  * 이와 같은 단점에도 불구하고 여전히 `Optional`을 사용해서 도메인 모델을 구성하는 것이 바람직하다고 생각한다. 특히 객체 그래프에서 일부 또는 전체 객체가 `null`일 수 있는 상황이라면 더욱 그렇다. 
  * 직렬화 모델이 필요하다면 다음 예제에서 보여주는 것처럼 `Optional`로 값을 반환받을 수 있는 메서드를 추가하는 방식을 권장한다.
  ```java
  public class Person {
      private Car car;
      public Optional<Car> getCarAsOptional() { return Optional.ofNullable(car); }
  }
  ```


* Optional Stream, default actions, Optional unwrap 등의 기능을 더 활용할 수 있다 
  * [Optional java 11 api](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html)


## 정리
* 역사적으로 프로그래밍 언어에서는 `null` 참조로 값이 없는 상황을 표현해왔다. 
* 자바 8에서는 값이 있거나 없음을 표현할 수 있는 클래스 `java.util.Optional<T>`를 제공한다. 
* 팩토리 메서드 `Optional.empty`, `Optional.of`, `Optional.ofNullable` 등을 통해서 `Optional` 객체를 만들 수 있다. 
* `Optional` 클래스는 스트림과 비슷한 연산을 수행하는 `map`, `flatMap`, `filter` 등의 메서드를 제공한다. 
* `Optional`로 값이 없는 상황을 적절하게 처리하도록 강제할 수 있다. 즉, `Optional`로 예상치 못한 `null` 예외를 방지할 수 있다. 
* `Optional`을 활용하면 더 좋은 API를 설계할 수 있다. 즉, 사용자는 메서드의 시그니처만 보고도 `Optional`값이 사용되거나 반환되는지 예측할 수 있다. 


## 궁금한 내용
* ["Optional" should not be used for parameters](https://rules.sonarsource.com/java/RSPEC-3553)
* [Java Optional에 대한 생각](https://bistros.tistory.com/entry/Java-Optional%EC%97%90-%EB%8C%80%ED%95%9C-%EC%83%9D%EA%B0%81)
* [Opinions on using Optional<> as parameter](https://www.reddit.com/r/java/comments/sat1j4/opinions_on_using_optional_as_parameter/)
* [Should Java 8 getters return optional type?](https://stackoverflow.com/questions/26327957/should-java-8-getters-return-optional-type/26328555#26328555)