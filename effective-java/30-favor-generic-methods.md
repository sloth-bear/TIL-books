# Today I Learned

| 구분 | 내용                                |
| ---- | ----------------------------------- |
| DATE | 2024.01.10                          |
| PART | 30. 이왕이면 제네릭 메서드로 만들라 |

# 5장 Generic

- 자바 5부터 지원된 generic - 지원 전에는 컬렉션에서 객체를 꺼낼 때마다 형변환을 해야 했다.
- 제네릭 사용 시 컬렉션이 담을 수 있는 타입을 컴파일러에게 알려주게 된다: 컴파일 타임에서 형변환 오류를 차단하므로 더 안전하고 명확한 프로그램을 만들어준다.
- 컬렉션이 아니더라도 관련 이점을 누릴 수 있다.
- 이번 장에서는 제네릭의 이점을 최대로 살리고 단점을 최소화하는 방법을 이야기한다.

## Item30. 이왕이면 제네릭 메서드로 만들라

### Generic method

1. 타입 매개변수 목록(타입 매개변수들을 선언하는)은 메서드의 제한자와 반환 타입 사이에 온다.
2. 타입 매개변수의 명명 규칙은 제네릭 메서드나 제네릭 타입이나 같다.

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet<>(s1);
  result.addAll(s2);
  return result;
}
```

3. unbounded wildcard type(Item 31)을 사용하면 더 유연하게 만들 수 있다.

#### Generic singleton factory

- 불변 개체를 여러 타입으로 활용할 수 있게 만들어야 될 때 사용
- 제네릭은 런타임에 타입 정보가 소거(Item 28)되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다.
- 이렇게 하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔 주는 정적 팩터리를 만들어야 한다. 이 팩터리를 Generic singleton factory라고 한다.
- e.g. Collections.reverseOrder(함수 개체;Item 42), Collections.emptySet

#### Identify function

- 항등함수(identify function)를 담은 클래스를 만들고 싶다면 `Function.identify`를 사용하면 되지만, 직접 한 번 작성해보자. 항등함수 객체는 상태가 없으니 요청할 때마다 새로 생성하는 것은 낭비다.
- 자바의 제네릭이 실체화된다면 항등함수를 타입별로 하나씩 만들어야 했겠지만, 소거 방식을 사용한 덕에 제네릭 싱글턴 하나면 충분하다.

  ```java
  private static UnaryOperator<Object> IDENTIFY_FN = (t) -> t;

  @SuppressWarnings("unchecked")
  public static <T> UnaryOperator<T> identifyFunction() {
    return (UnaryOperator<T>) IDENTIFY_FN;
  }
  ```

- 입력 값을 수정 없이 그대로 반환하는 특별한 함수이므로, `<T>`가 어떤 타입이든 `<T>`를 사용해도 타입 안전하다.

#### Recursive type bound(재귀적 타입 한정)

- 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다. 이것이 재귀적 타입 한정이라는 개념이다.
- 주로 타입의 자연적 순서를 정하는 `Comparable` 인터페이스(Item 14)와 함께 쓰인다. 여기서 타입 매개변수 T는 `Comparable<T>`를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다.

  ```java
  public interface Comparable<T> {
      int compareTo(T o);
  }
  ```

  - 또는 아래와 같이 제약을 둘 수도 있다. 타입 한정인 `<E extends Comparable<E>>`는 "모든 타입 E는 자신과 비교할 수 있다"라고 읽을 수도 있다.

  ```java
  public static <E extends Comparable<E>> E max(Collection<E> c);
  ```

### 정리

- 재귀적 한정 타입은 훨씬 복잡해질 가능성이 있지만, 다행히 잘 일어나지 않는다.
- 관용구, 와일드카드를 사용한 변형(Item 31), 시뮬레이트한 셀프 타입 관용구(Item 2)를 이해하고 나면 실전에서 마주하는 대부분의 재귀적 타입 한정을 무리 없이 다룰 수 있을 것이다.
- 제네릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다.
- 타입과 마찬가지로, 형변환을 해줘야 하는 기존 메서드는 제네릭하게 만들자.
