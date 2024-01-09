# Today I Learned

| 구분 | 내용                            |
| ---- | ------------------------------|
| DATE | 2024.01.07                    |
| PART | 14. `Comparable`을 구현할지 고려하라 |

# 4장 Methods common to All Objects (모든 객체의 공통 메서드)
* `Object`: 객체를 만들 수 있는 구체 클래스지만, 기본적으로는 상속해서 사용하도록 설계됨 
* `final`이 아닌 메서드(`equals, hashCode, toString, clnoe, finalize`)는 모두 overriding을 염두에 두고 설계된 것이라 재정의 시 지켜야 하는 일반 규약이 명확히 정의되어 있음. 
* 그래서 `Object`를 상속하는 클래스, 즉 모든 클래스는 이 메서드들을 일반 규약에 맞게 재정의해야 함 -> 만약 메서드를 잘못 구현하면 대상 클래스가 이 규약을 준수한다고 가정하는 클래스(`HashMap`과 `HashSet` 등)를 오작동하게 만들 수 있다. 
* ***`final`이 아닌 `Object` 메서드들을 언제 어떻게 재정의해야 하는가에 초점***

## Item14. `Comparable`을 구현할지 고려하라

### `Comparable<T>`  - `int compareTo(T o)`
* `Comparable` 인터페이스의 유일한 메서드: `compareTo`
* 이 클래스를 구현하는 각 클래스의 객체에 전체 순서를 적용
* `Object`의 클래스는 아니지만, 성격은 두 가지만 빼면 `Object`의 `equals`와 같다. 
* 이를 구현했다는 것은 그 클래스의 인스턴스들에는 natural order(자연적인 순서)가 있음을 뜻한다.
* 사실상 자바 플랫폼 라이브러리의 모든 값 클래스와 열거타입(Item 34)이 `Comparable`을 구현했다.
* 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 `Comparable` 인터페이스를 구현하자.
* 

### `equals`와의 차이
1. `equals`의 단순 동치성 비교에 더해 순서까지 비교할 수 있으며 
2. 제네릭하다.
3. 다만, `equals`와 달리 `compareTo`는 타입이 다른 객체를 신경 쓰지 않아도 된다. 타입이 다른 객체가 주어지면 간단히 `ClassCastException`을 던져도 되며, 대부분 그렇게 한다. (물론 규약에선 다른 타입 사이의 비교도 허용하지만 보통은 비교할 객체들이 구현한 공통 인터페이스를 매개로 이루어진다.)


### Specification
이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 <u>객체보다 작으면 음의 정수를</u>, <u>같으면 0을</u>, <u>크면 양의 정수</u>를 반환한다. 이 객체와 비교할 수 없는 타입의 객체가 주어지면 `ClassCastException`을 던진다. 
`compareTo` 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못한다. 
  e.g. 정렬된 컬렉션: `TreeSet`, `TreeMap`
  e.g. 검색과 정렬 알고리즘을 활용하는 유틸리티 클래스: `Collections`, `Arrays`

다음 설명에서 `sgn(표현식)` 표기는 수학에서 말하는 부호 함수(signum function)를 뜻하며, 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환하도록 정의했다. 

정리하면 아래와 같다. 
1. `Comparable`을 구현한 클래스는 모든 x, y에 대해 `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`여야 한다. (따라서 `x.compareTo(y)`는 `y.compareTo(x)`가 예외를 던질 때에 한해 예외를 던져야 한다.)
2. `Comparable`을 구현한 클래스는 추이성을 보장해야 한다. 즉, `(x.compareTo(y) > 0 && y.compareTo(z) > 0)`이면 `x.compareTo(z) > 0`이다.
3. `Comparable`을 구현한 클래스는 모든 z에 대해 `x.compareTo(y) == 0`이면 `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`이다.
4. 이번 권고가 필수는 아니지만 꼭 지키는 게 좋다. `(x.compareTo(y) == 0) == (x.equals(y))`여야 한다. `Comparable`을 구현하고 이 권고를 지키지 않는 모든 클래스를 그 사실을 명시해야 한다. 다음과 같이 명시하면 적당할 것이다. ***"주의: 이 클래스의 순서는 `equals` 메서드와 일관되지 않다."***

#### 1. 대칭성: 두 객체의 참조의 순서를 바꾸어 비교해도 예상한 결과가 나와야 한다. 
* 1번 객체가 2번 객체보다 작으면 2번은 1번보다 커야 한다. 
* 1번과 2번이 같으면 2번도 1번과 같아야 한다. 
* 1번이 2번보다 크면 2번은 1번보다 작아야 한다.
```java
Comparable을 구현한 클래스는 모든 x, y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다. (따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질 때에 한해 예외를 던져야 한다.)
```

#### 2. 추이성: 1번이 2번보다 크고, 2번이 3번보다 크면 1번은 3번보다 커야 한다.
```java
Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, (x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0이다.
```

#### 3. 반사성: 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.
```java
Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))이다.
```

#### 4. (선택) 동치성: `compareTo` 메서드로 수행한 동치성 테스트의 결과가 `equals`와 같아야 한다.
* 이를 잘 지키면 `compareTo`로 줄지은 순서와 `equals` 결과가 일관되게 같다.
* 일관되지 않더라도 동작은 하지만, 이 클래스의 객체를 정렬된 컬렉션에 넣으면 해당 컬렉션이 구현한 인터페이스(Collection, Set, Map)에 정의된 동작과 엇박자를 낼 것이다.
    * e.g.
      * `BigDecimal`: `compareTo`와 `equals`가 일관되지 않다. 정렬된 컬렉션들은 동치성을 비교할 때 `equals` 대신 `compareTo`를 사용한다. `new BigDecimal("1.0")`과 `new BigDecimal("1.00")`을 `HashSet`에 추가하면, 원소를 2개 가지지만 `TreeSet`을 사용하면 원소를 하나만 갖게 된다. `compareTo`로 비교하면 인스턴스가 같기 때문이다.
```java
(x.compareTo(y) == 0) == (x.equals(y))여야 한다. Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스를 그 사실을 명시해야 한다. 다음과 같이 명시하면 적당할 것이다. "주의: 이 클래스의 순서는 `equals` 메서드와 일관되지 않다."
```


### 주의사항
* 구체 클래스에서 새로운 갑 컴포넌트를 추가했다면 `compareTo` 규약을 지킬 방법이 없다. `equals`와 마찬가지로 반사성, 대칭성, 추이성을 충족시켜야 하기 때문이다. 객체 지향적 추상화의 이점을 포기할 생각이 아니라면 말이다. 우회하려면? 확장 대신 독립된 클래스를 만들고, 이 클래스에 원래 클래스의 인스턴스를 가리키는 필드를 두자. 그런 다음 내부 인스턴스를 반환하는 '뷰' 메서드를 제공하면 된다. 이렇게 하면 바깥 클래스에 우리가 원하는 `compareTo` 메서드를 구현해 넣을 수 있다. 
* 박신된 기본 타입 클래스들에 새로 추가된 정적 메서드인 `compare`를 <, > 대신 적극 활용하자. `compareTo` 메서드에서 관계 연산자 <와 >를 사용하는 이전 방식은 거추장 스럽고 오류를 유발한다. (2판에서 권한 방식)

#### 추이성 위반 사례
가끔 '값의 차'를 기준으로 첫 번째 값이 두 번째 값보다 작으면 음수를, 두 값이 같으면 0을, 첫 번째 값이 크면 양수를 반환하는 `compareTo`나 `compare` 메서드와 마주하는데, 해시코드 값의 차를 기준으로 하는 비교자 - 추이성을 위반하므로 사용하면 안 된다.
정수 오버플로를 일으키거나 부동소수점 계산 방식에 따른 오류를 낼 수 있다. 
(해시코드 오버플로우: `hashCode` 메서드가 반환하는 int 값에서 오버플로우/언더플로우가 발생할 수 있다. 최대값을 초과함으로써 결과 오류를 낼 수 있다.)
```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
    return o1.hashCode() - o2.hashCode();
  }
};
```

대신 아래 방법을 사용하자.
```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
    return Integer.compare(o1.hashCode(), o2.hashCode());
  }
}
```
```java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```

### 구현 방법
1. 인수가  `null`일 경우 예외처리 한다.
2. 각 필드가 동치인지를 비교하는 게 아니라 그 순서를 비교하므로 객체 참조 필드를 비교하려면 `compareTo` 메서드를 재귀적으로 호출한다. 이를 구현하지 않았거나 표준이 아닌 순서로 비교해야 한다면 `Comparator`를 대신 사용한다.
    ```java
    public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
      public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
      }
    }
    ```

#### 핵심 필드가 여러 개라면?
* 어느 것을 먼저 비교하냐가 중요하다: 가장 핵심적인 필드부터 비교해나가자. 
  ```java
  public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0) {
      result = Short.compare(prefix, pn.prefix);
      if (result == 0)
        result = Short.compare(lineNum, pn.lineNum);
    }
    return result;
  }
  ```

##### method chaining 방식
* 약간의 성능 저하는 뒤따른다. 
```java
private static final Comparator<PhoneNumber> COMPARATOR = comparingInt(
(PhoneNumber pn) -> pn.areaCode
  .thenComparingInt(pn -> pn.prefix)
  .thenComparingInt(pn -> pn.lineNum));

public int compareTo(PhoneNumber pn) {
  return COMPARATOR.compare(this, pn);
}
```