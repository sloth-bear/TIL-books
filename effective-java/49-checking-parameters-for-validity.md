# Today I Learned

| 구분 | 내용                        |
| ---- | --------------------------|
| DATE | 2023.12.28                |
| PART | 49. 매개변수가 유효한지 검사하라 |

# 8장 Method
* 메서드를 설계할 때 주의할 점에 관해 (대부분 생성자에도 적용됨) - 사용성, 견고성, 유연성
  * 매개변수와 반환값을 어떻게 처리해야 하는지 
  * 메서드 시그니처는 어떻게 설계해야 하는지 
  * 문서화는 어떻게 해야 하는지 

## Item49. 매개변수가 유효한지 검사하라
메서드와 생성자 대부분은 `input parameter` 특정 조건을 만족하기를 바란다. ex) index: 음수이면 안 됨 / 객체 참조: `null`이 아니어야 함.
이런 제약은 반드시 <i>문서화</i>해야 하며 메서드 몸체가 시작되기 전에 검사해야 한다. 즉, 오류는 가능한 빨리 (발생한 곳에서) 잡아야 한다는 일반 원칙의 한 사례이기도 하다.

### 매개변수 검사를 제대로 하지 못하면? 
1. 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다. 
2. 메서드가 잘 실행되고 잘못된 결과를 반환하면 더 문제다.
2. 메서드는 문제 없이 수행되었지만 어떤 객체를 이상한 상태로 만들어놓아서 미래의 알 수 없는 시점에 이 메서드와 관련 없는 오류를 낼 때다. 

즉, 매개변수 검사에 실패하면 실패 원자성(failure atomicity)을 어기는 결과를 낳을 수 있다. 

### `public`, `protected` 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화 해야 한다. 
`@throws` 자바독 태그를 사용하면 된다. 보통 `IllegalArgumentException`, `IndexOutOfBoundsException`, `NullPointerException` 중 하나가 될 것이다.
매개변수 제약을 문서화한다면 그 제약을 어겼을 때 발생하는 예외도 함께 기술해야 한다.
```java
/**
 * 항상 음이 아닌 BigInteger를 반환한다는 점에서 Remainder 메서드와 다르다.
 * 
 * @param m 계수 (양ㅅ우여야 한다.)
 * @return 현재 값 mod m
 * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
 */
public BigInteger mod(BigInteger m) {
  if (m.signum() <= 0) {
    throw new ArithmeticException("계수(m)은 양수여야 합니다.");
  }
  ...
}
```
* `m`이 `null`일 경우 `NullPointerException`을 반환하는데, 문서화 되어있지 않은 이유는 클래스 수준 주석이기 때문이다. 즉, 그 클래스의 모든 public 메서드에 적용되므로 각 메서드에 일일이 기술하는 것보다 훨씬 깔끔한 방법이다.
* `@Nullable`이나 이와 비슷한 애너테이션을 사용해 특정 매개변수는 null이 될 수 있다고 알려줄 수 있지만, 표준적인 방법은 아니다.
* java 7에 추가된 `java.util.Objects.requireNonNull` 메서드는 유연하고 사용하기 편하니 더 이상 `null` 검사를 수동으로 하지 않아도 된다.
* 메서드가 직접 사용하지 않지만 나중에 쓰기 위해 저장하는 매개변수는 특히 더 신경 써서 검사해야 한다. 그렇지 않으면, 나중에 사용할 때 에러 발생 시 추적하기 어렵다.


### `private`: 공개되지 않은 메서드라면 메서드가 호출되는 상황을 통제할 수 있다.
오직 유효한 값만이 메서드에 넘겨지리라는 것을 보증할 수 있고, <i>그렇게 해야 한다.</i> 다시 말해 public이 아닌 메서드라면 단언문`assert`을 사용해 매개변수 유효성을 검증할 수 있다. 
```java
private static void sort(long a[], int offset, int length) {
  assert a != null;
  assert offset >= 0 && offset <= a.length;
  assert length >=0 && length <= a.length - offset;
}
```
* 핵심: 자신이 단언한 조건이 무조건 참이라고 선언한다. 
* 일반적인 유효성 검사와는 다른 부분이 있다. 
  * 실패하면 `AssertionError`를 던진다.
  * 런타임에 아무런 효과도, 아무런 성능 저하도 없다. (`java --enableassertions` flag 설정하는 경우 제외)


### `constructor`: 나중에 쓰려고 저장하는 매개변수의 유효성을 검사하라는 원칙의 특수한 사례
생성자 매개변수의 유효성 검사는 클래스 불변식을 어기는 객체가 만들어지지 않게 하는 데 꼭 필요하다. 


### 메서드 몸체 실행 전 매개변수 유효성 검사의 예외 
* 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때 
* 계산 과정에서 암묵적으로 검사가 수행될 때 
* 예시
  * `Collections.sort(list)`: 리스트 내 객체들은 모두 상호 비교될 수 있어야 하며, 정렬 과정에서 비교한다. 만약 상호 비교 불가한 타입의 객체가 들어 있다면 그 객체와 비교할 때 `ClassCastException`을 던질 것이다. 따라서 비교하기 앞서 리스트 내 모든 객체가 상호 비교될 수 있는지 검사해봐야 별다른 실익이 없다. 그러나 이러한 암묵적 유효성 검사에 너무 의존하면 실패 원자성(아이템 76)을 해칠 수 있으니 주의하자.


### 유의사항
* 때로는 계산 과정에서 필요한 유효성 검사가 이뤄지지만 실패했을 때 잘못된 예외를 던지기도 한다. 달리 말하면, 계산 중 잘못된 매개변수 값을 사용해 발생한 예외와 API 문서에서 던지기로 한 예외가 다를 수 있다는 뜻이다.
  ```java
  public class ArrayProcessor {

    /**
     * 주어진 배열과 인덱스에 해당하는 값을 반환합니다.
     * @param array 대상 배열
     * @param index 접근할 배열의 인덱스
     * @return 배열의 해당 인덱스 값
     * @throws InvalidParameterException 인덱스가 배열 범위를 벗어날 때 던져집니다.
     */
    public static int getElement(int[] array, int index) {
        try {
            return array[index];
        } catch (ArrayIndexOutOfBoundsException e) {
            throw new InvalidParameterException("인덱스가 배열 범위를 벗어났습니다.");
        }
    }
  }
  ```
  * 매개변수를 따로 검증하지 않았지만 계산 과정에서 유효성 검사가 이뤄질 것이다. 그러나 API 문서에 적힌 예외를 던지지 않으므로 `InvalidParameterException`을 다시 던지고 있다.

* 이번 아이템을 "매개변수에 제약을 두는 게 좋다."고 해석해서는 안 된다. <strong><i>그 반대다. 메서드는 최대한 범용적으로 설계해야 한다.</i></strong> 메서드가 건네받은 값으로 무언가 제대로 된 일을 할 수 있다면, 매개변수 제약은 적을수록 좋다. 하지만 구현하려는 개념 자체가 특정한 제약을 내재한 경우도 드물지 않다. 


### 정리 
* 메서드나 생성자를 작성할 때면 그 매개변수들에 어떤 제약이 있을지 생각하고 문서화 및 메서드 코드 시작부분에서 명시적으로 검사해야 한다.