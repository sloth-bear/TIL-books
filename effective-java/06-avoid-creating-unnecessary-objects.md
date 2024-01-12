# Today I Learned

| 구분 | 내용                      |
| ---- | ------------------------|
| DATE | 2024.01.07              |
| PART | 6. 불필요한 객체 생성을 피하라 |

# 2장 객체 생성과 파괴
* 객체를 만들어야 할 떄와 만들지 말아야 할 때를 구분하는 방법
* 올바른 객체 생성 방법
* 불필요한 생성을 피하는 방법
* 제때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업을 관리하는 요령 

## Item06. 불필요한 객체 생성을 피하라

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많다. 재사용은 빠르고 세련되며, 특히 불변 객체(Item 17)는 언제든 재사용할 수 있다.

```java
// 절대 따라하지 말자. 
String s = new String("hello");
```
"hello" 파라미터 자체가 이 생성자로 만들어내려는 `String`과 기능적으로 완전히 똑같다. 이 문장이 반복문이나 빈번히 호출되는 메서드 안에 있다면 쓸데 없는 `String` 인스턴스가 수백만 개 만들어질 수도 있다. 

```java
String s = "hello";
```
대신 위처럼 사용하자. 새로운 인스턴스를 매번 만드는 것이 아니라 하나의 `String` instance를 사용한다. [또한 같은 가상 머신 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장된다.](https://progmiscon.org/specs/jls13/3.10.5)


### 생성자 대신 Factory method 사용하기
* 불변 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다. `Boolean(String)` 생성자 대신 `Boolean.valueOf(String)` 팩터리 메서드를 사용하는 것이 좋다. (java 9에서는 deprecated로 지정되었다.)


### 생성 비용이 비싼 객체
* 생성 비용이 비싼 객체는 반복해서 필요하다면 캐싱하여 재사용하길 권한다. (안타깝게도 매번 비싼 객체인지를 명확히 확인할 수 없다.)
* 예를 들어, `String`의 `matches`로 주어진 문자열이 유효한 문자열인지 확인하는 메서드를 작성한다고 해보자. 정규표현식을 활용하는 것이 가장 쉽겠지만, 성능이 중요할 때 반복해 사용하긴 적합하지 않다. 내부적으로 사용하는 `Pattern` 인스턴스는 한 번 쓰고 버려져서 곧바로 가비지 컬렉션 대상이 된다. `Pattern`은 입력받은 정규표현식에 해당하는 유한 상태 머신(finite state machine)을 만들기 때문에 인스턴스 생성 비용이 높다. 성능을 개선하려면, 필요한 정규표현식을 표현하는 (불변인) `Pattern` 인스턴스를 클래스 초기화(정적 초기화) 과정에서 직접 생성해 캐싱해두고, 나중에 재사용한다. 
  ```java
  private static final Pattern ROMAN = Pattern.compile(
    "^(?=.)M*(C[MD]|D?C{0,3})" 
    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$"
  );
  ```
    * 추가적으로 lazy initialization을 적용할 수 있겠지만, 권장하지 않는다. 코드를 복잡하게 만들고 성능은 크게 개선되지 않을 때가 많다. (Item 67)
* boxed primitive type보다는 primitive type을 사용하자. 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.


### 주의사항
* 객체 생성은 비싸니 피해야 하는 것으로 오해하면 안 된다. 프로그램의 명확성, 간결성, 기능을 위해 객체를 추가로 생성하는 것이라면 일반적으로 좋은 일이다. 
* 아주 무거운 객체가 아닌 다음에야 단순히 객체 생성을 피하고자 여러분만의 객체 풀(pool)을 만들지는 말자.일반적으로는 코드를 헷갈리게 만들고 메모리 사용량을 늘리고 성능을 떨어뜨린다. (요즘의 JVM의 가비지 컬렉터는 상당히 잘 최적화되어서 가벼운 객체용을 다룰 때는 직접 만든 객체 풀보다 훨씬 빠르다.)
* 이번 아이템은 defensive copy를 다루는 Item 50과 대조적인데, 방어적 복사가 필요한 상황에서 객체를 재사용했을 때의 피해가 필요 없는 객체를 반복 생성했을 때의 피해보다 훨씬 크다는 사실을 기억하자. 