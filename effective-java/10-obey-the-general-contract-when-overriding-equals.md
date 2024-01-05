# Today I Learned

| 구분 | 내용                            |
| ---- | ------------------------------|
| DATE | 2024.01.06                    |
| PART | 10. equals는 일반 규약을 지켜 재정의하라 |

# 4장 Methods common to All Objects (모든 객체의 공통 메서드)
* `Object`: 객체를 만들 수 있는 구체 클래스지만, 기본적으로는 상속해서 사용하도록 설계됨 
* `final`이 아닌 메서드(`equals, hashCode, toString, clnoe, finalize`)는 모두 overriding을 염두에 두고 설계된 것이라 재정의 시 지켜야 하는 일반 규약이 명확히 정의되어 있음. 
* 그래서 `Object`를 상속하는 클래스, 즉 모든 클래스는 이 메서드들을 일반 규약에 맞게 재정의해야 함 -> 만약 메서드를 잘못 구현하면 대상 클래스가 이 규약을 준수한다고 가정하는 클래스(`HashMap`과 `HashSet` 등)를 오작동하게 만들 수 있다. 
* ***`final`이 아닌 `Object` 메서드들을 언제 어떻게 재정의해야 하는가에 초점***

## Item10. `equals`는 일반 규약을 지켜 재정의하라

`equals` 메서드는 재정의하기 쉬워 보이지만 곳곳에 함정이 도사리고 있어 자칫하면 끔찍한 결과를 초래한다. 문제를 회피하는 가장 쉬운 길은 아예 재정의하지 않는 것이다. (=그 클래스의 인스턴스는 자기 자신과만 같게 된다.) 즉, `equals`를 재정의 할 상황이 아니라면 하지 마라.

### `equals` 재정의가 필요하지 않은 상황 
1. 각 인스턴스가 본질적으로 고유할 때 (값을 표현하는 게 아닌 동작하는 개체를 표현하는 클래스)
     e.g. `Thread`
 2. 인스턴스의 logical equality 검사할 일이 없을 때 
     * `java.util.regex.Pattern`: `equals`를 재정의해서 두 `Pattern`의 인스턴스가 같은 정규표현식을 나타내는지(논리적 동치성)를 검사하고 싶을 수 있다. 그러나 설계자는 필요하지 않거나 클라이언트가 원하지 않는다고 판단할 수 있다. 
  3. 상위 클래스에서 재정의한 `equals`가 하위 클래스에도 딱 들어맞을 때
    * e.g. 대부분의 `Set` 구현체, `List`/`Map` 구현체: `AbstractSet`이 구현한 `equals` 상속받아 사용
  4. 클래스가 private/package-private 이면서 `equals` 메서드를 호출할 일이 없을 때
    * 위험을 회피하려면? `equals`에 `throw new AssertionError();` 까지 작성해두자.


### `equals` 재정의가 필요한 상황 
1. 논리적 동치성을 확인해야 하는데, 상위 클래스의 `equals`가 논리적 동치성을 비교하도록 재정의되지 않았을 때 
    * e.g. value object (`Integer`, `String`, ...)
    * object identify(객체 식별성, 두 객체가 물리적으로 같은가) 아님
    * 객체가 같은지가 아닌 값이 같은지를 알고 싶어 할 것
    * `Map`의 key, `Set`의 element로도 사용 가능
    * 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스(Item 1)라면 재정의하지 않아도 됨 e.g. Enum (Item 34)


### Specification in `Object`
`equals` 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다. 
1. 반사성(reflexivity)
    * `null`이 아닌 모든 참조 값 x에 대해 `x.equals(x) == true`
2. 대칭성(symmetry)
    * `null`이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`가 true면 `y.equals(x)`
3. 추이성(transitivity)
    * `null`이 아닌 모든 참조 값 x, y, z에 대해 `x.equals(y)`, `y.equals(x)`가 true면 `x.equals(z)`도 true
4. 일관성(consistency)
    * `null`이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`를 반복해서 호출해도 일관된 값
5. `null`-아님
    * `null`이 아닌 모든 참조 값 x에 대해 `x.equals(null)`은 false


#### 동치관계란?
* 집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산
* 이 부분집합을 동치류(equivalence class)라고 한다. 
* `equals` 메서드가 쓸모 있으려면 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다. 


#### 반사성(reflexivity)
* 객체는 자기 자신과 같아야 함
* 일부러 어기지 않는 이상 지키지 않는 것이 어려움. 
* cf) 어길 시 `Collection`의 `contains` 호출 시 instance 못 찾을 것

#### 대칭성(symmetry)
* 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 함. 
* e.g. 잘못된 예
    ```java
    public final class CaseInsensitiveString {
      private final String s;
      ...

      @Override
      public boolean equals(Object o) {
        if (o instance of CaseInsensitiveString)
          return s.equalsIgnoreCase(
            ((CaseInsensitiveString) o).s);
        if (o instanceof String) 
          return o.equalsIgnoreCase((String) o);
        return false;
      }
    }
    ```
    * 이 예제에서 `String`과도 비교하는데, String은 `CaseInsensitiveString`의 존재도 모른다. 
    * `"Polish".equals(new CaseInsensitiveString("Polish")`는 false다.
    * Collection의 contains에서도 어떻게 동작할지 확답할 수 없다.
  * e.g. 올바른 예
    ```java
    @Override
      public boolean equals(Object o) {
        return o instance of CaseInsensitiveString &&
          ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
      }
    ```

#### 추이성(transitivity)
1. 1번 객체와 2번 객채가 같고, 2번 객체와 3번 객체가 같으면 1번 객체와 3번 객체는 같아야 한다.
2. 잘못하면 어기기 쉬우니 주의하자. 
3. e.g. `x, y` 필드를 가진 `Point` 클래스를 확장한 구체 클래스 `ColorPoint`와 `equals`를 지키려면?
    * `ColorPoint`, `Point`, `ColorPoint` 3가지를 같다고 할 수 있는 방법이 있을까?
    * 리스코프 치환 원칙에 따르면, 상위에서 중요한 속성은 하위에서도 중요하다. 따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 동작해야 된다. `Point`의 하위 클래스는 정의상 여전히 `Point`이므로 어디서든 `Point`로 활용될 수 있어야 한다. 
  4. ***모든 객체 지향 언어의 동치 관계에서 나타나는 근본적인 문제다. 구체 클래스를 확장해 새로운 값을 추가하면서 `equals` 규약을 만족시킬 방법은 존재하지 않는다.*** 
  5. 상속 대신 위임을 사용하라 (Item 18) 조언을 따르면 우회할 수 있다. 
  6. e.g. 위반 사례
    * `java.sql.Timestamp`는 `java.util.Date`를 확장해 `nanoseconds` 필드를 추가했다. 그 결과 `Timestamp`의 `equals`는 대칭성을 위배한다. `Date`와 섞어쓰면 엉뚱하게 동작할 수 있다.
7. 추상 클래스의 하위 클래스에서는 `equals` 규약을 지키면서도 값을 추가할 수 있다. 클래스 계층 구조(Item 23)에서는 중요한 사실이다. 즉, 상위 클래스를 직접 인스턴스로 만드는 게 불가능하다면 지금까지 이야기한 문제들은 일어나지 않는다.


#### 일관성(consistency)
1. 두 객체가 같다면 (어느 하나, 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다. 즉, 가변 객체는 시점에 따라 다를 수 있지만 불변 객체는 한 번 다르면 끝까지 달라야 한다. 
2. `equals` 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다. 
    * e.g. 위반 사례: `java.net.URL`의 `equals`: 주어진 URL, 매핑된 호스트의 IP 주소를 이용해 비교하는데, 호스트 이름을 IP 주소로 바꾸려면 네트워크를 통해야 한다. 그 결과가 항상 같다고 보장할 수 없다. 일반 규약을 어긴 것이다.
    * `equals`는 메모리에 존재하는 객체만을 사용한 결정적(deterministic) 계산만 수행해야 한다.


#### null-아님
1. 모든 객체가 `null`과 같지 않아야 한다. 
    * `null` check를 `equals` 마다 할 필요는 없고, instanceof 연산자로 매개변수가 올바른 타입인지 검사하는데, 이때 null이면 false를 반환하므로 명시적으로 할 필요는 없다.


### `equals` 메서드 구현 방법
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다. 
    * 성능 최적화용, 비교 작업이 복잡할 때 값을 함
2.  `instanceof` 연산자로 입력이 올바른 타입인지 확인한다.
    * 일반적으로 `equals`가 정의된 클래스지만, 가끔 그 클래스가 구현한 특정 인터페이스가 될 수 있다. 어떤 인터페이스는 자기를 구현한 (서로 다른) 클래스끼리도 비교할 수 있도록 `equals` 규약을 수정하기도 한다. 이런 인터페이스를 구현한 클래스라면 `equals`에서 해당 인터페이스를 사용해야 한다. 
        e.g.) `Set, List, Map, Map.Entry`
3. 입력을 올바른 타입으로 형변환한다. 
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다. 
    * `float`, `double` 제외한 primitive type: == 연산자로 비교
    * `float`, `double`: `Float.compare(float, float)`, `Double.compare(double, double)`로 비교 (특수한 부동소수 값 때문), wrapper의 `equals`도 있으나 오토박싱을 수반할 수 있으니 성능상 피한다. 
    * 배열: 원소 각각을 앞서의 지침대로 비교한다. (배열의 모든 원소가 핵심 필드라면 `Array.equals` 필드 중 하나를 사용)

### 참고 
1. 때로는 `null`도 정상 값으로 취급하는 참조 타입 필드도 있다. 이런 필드는 static method인 `Objects.equals(Object, Object)`로 비교해 NullPointerException을 예방하자.
2. 앞서의 `CaseInsensitiveString` 예처럼 비교하기가 복잡한 필드를 가진 클래스도 있다. 이럴 때는 그 필드의 표준형(canonical form)을 저장해둔 후 표준형끼리 비교하면 훨씬 경제적이다. 이 기법은 특히 불변 클래스(Item 17)가 제격이다. 값이 바뀔 때마다 표준형을 최신 상태로 갱신해줄 필요가 없다. 
3. 어떤 필드를 먼저 비교하냐가 `equals`의 성능을 좌우하기도 한다. 성능을 높이고 싶다면 다를 가능성이 더 크거나, 비교하는 비용이 저렴한 필드를 먼저 비교하자. 
4. 동기화용 락(lock) 필드 같이 객체의 논리적 상태와 관련 없는 필드는 비교하면 안 된다. 
5. 핵심 필드로부터 계산해낼 수 있는 파생 필드 역시 굳이 비교할 필요는 없지만, 이를 비교하는 쪽이 더 빠를 때도 있다. (파생 필드가 객체 전체의 상태를 대표할 때)


### Testing
1. `equals`를 다 구현했다면 단위 테스트를 작성해 돌려보자. (대칭성, 추이성, 일관성 - 나머지는 문제되는 경우가 별로 없으므로)
2. AutoValue를 이용해 작성했다면 생략해도 안심할 수 있다. 


### 주의사항
1. `equals`를 재정의할 땐 `hashCode`도 재정의하자. (item 11)
2. 너무 복잡하게 해결하려 들지 말자. 필드들의 동치성만 비교해도 어렵지 않게 규약을 지킬 수 있다. 너무 공격적으로 파고들다가 문제를 일으키기도 한다. 
3. 일반적으로 별칭(alias)는 비교하지 않는 게 좋다. e.g.) `File`: 심볼릭 링크를 비교해 같은 파일을 가리키는지 확인하려 들면 안 된다. 
4. `Object` 외의 타입을 매개변수로 받는 `equals` 메서드는 선언하지 말자. 반드시 `Object` 여야 한다. 
5. 꼭 필요한 경우가 아니라면 `equals`를 재정의하지 말자. 많은 경우 `Object`의 `equals`가 원하는 비교를 정확히 수행해준다. 