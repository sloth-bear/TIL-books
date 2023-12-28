# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2023.12.28             |
| PART | 17. 변경 가능성을 최소화하라 |

# 4장 Class, Interface
* `Class`, `Interface`: 추상화의 기본 단위, 자바 언어의 심장과도 같다. 
* 클래스와 인터페이스 설계에 사용하는 강력한 요소가 많이 있으므로, 이 요소를 적절히 활용해 클래스와 인터페이스를 쓰기 편하고, 견고하며 유연하게 만드는 방법을 안내한다.

## Item17. 변경 가능성을 최소화하라 

### `Immutable class`: 불변 클래스
* 불변 클래스:  그 인스턴스의 내부 값을 수정할 수 없는 클래스
* 불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다. 
* ex) `String`, `BigInteger`, `BigDecimal`

### 왜 불변으로 설계했을까?
가변 클래스보다 설계, 구현, 사용이 쉽고 오류가 생길 여지도 적고 훨씬 안전하다. 

### 클래스를 불변으로 만들기 위한 5가지 규칙 
1. `setter` 미제공: 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
2. `final class` 선언: 클래스를 확장할 수 없도록 한다. 
  - 하위 클래스에서 부주의하게 혹은 나쁜 의도로 객체의 상태를 변하게 만드는 사태를 막아준다. 
  - 상속을 막는 대표적인 방법. 다른 방법도 뒤에서 나온다. 
3. `final field`: 모든 필드를 `final`로 선언한다. 
  - 시스템이 강제하는 수단을 이용해 설계자의 의도를 명확히 드러내는 방법이다. 
  - 새로 생성된 인스턴스를 동기화 없이 다른 스레드로 건네도 문제 없이 동작하게끔 보장하는 데도 필요하다.
4. `private field`: 모든 필드를 `private`로 선언한다. 
  - 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아준다. 
  - 기술적으로는 기본 타입 필드나 불변 객체를 참조하는 필드를 `public final`로만 선언해도 불변 객체가 되지만, 이렇게 하면 다음 릴리스에서 내부 표현을 바꾸지 못하므로 권하진 않는다.
5. `내부 가변 컴포넌트 접근 불가`: 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다. 
  - 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야 한다. 
  - 절대 클라이언트가 제공한 객체 참조를 가리키게 해선 안 되며, 접근자 메서드가 그 필드를 그대로 반환해서도 안 된다. 생성자, 접근자, `readObject` 메서드 모두에서 방어적 복사를 수행하라.

```java
public final class Complex {
  private final double re;
  private final double im;

  public Complex(double re, double im) {
    this.re = re;
    this.im = im;
  }

  public double realPart() { return re; }
  public double imaginaryPart() { return im; }

  public Complex plus(Complex c) {
    return new Complex(re + c.re, im + c.im);
  }

  public Complex minus(Complex c) {
    return new Complex(re - c.re, im - c.im);
  }

  public Complex times(Complex c) {
    return new Complex(re * c.re - im * c.im, re * c.im + im * c.re);
  }

  public Complex dividedBy(Complex c) {
    double tmp = c.re * c.re + c.im * c.im;
    return new Complex((re * c.re + im * c.im) / tmp, (im * c.re - re * c.im) / tmp);
  }

  @Override
  public boolean equals(Object o) {
    if (o == this)
      return true;
    if (!(o instanceof Complex))
      return false;
    Complex c = (Complex) o;

    return Double.compare(c.re, re) == 0 && Double.compare(c.im, im) == 0;
  }

  @Override
  public int hashCode() {
    return 31 * Double.hashCode(re) + Double.hashCode(im);
  }

  @Override
  public String toString() { return "(" + re + " + " + im + "i)"; }
}
```
* 이 클래스는 복소수(실수부와 허수부로 구성된 수)를 표현한다.
* 사칙연산 메서드들: 인스턴스 자신은 수정하지 않고, 새로운 `Complex` instance를 반환한다. 
  * `functional programming`: 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴
  * `절차적(명령형) 프로그래밍`: 피연산자인 자신을 수정해 자신의 상태가 변함
* 메서드 이름을 동사(add...) 대신 전치사(plus...) 사용: 해당 메서드가 객체의 값을 변경하지 않는다는 사실을 강조 
  * 이 명명 규칙을 따르지 않은 클래스: `BigInteger`, `BigDecimal` - 클래스를 사람들이 잘못 사용해 오류가 발생하는 일이 자주 있다. 


### 불변 객체의 특징 
* 불변 객체는 단순하다. 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다. 
* 모든 생성자가 클래스 불변식(class invariant)을 보장한다면 그 클래스를 사용하는 프로그래머가 다른 노력을 들이지 않더라도 영원히 불변으로 남는다. 
* <b>근본적으로 Thread safe하여 따로 동기화 할 필요가 없다.</b> 클래스를 Thread safe하게 만드는 가장 쉬운 방법이기도 하다. 안심하고 공유할 수 있다. 따라서 불변 클래스라면 한 번 만든 인스턴스를 최대한 재활용하기를 권한다. 가장 쉬운 재활용 방법은 자주 쓰이는 값들을 상수(public static final)로 제공하는 것이다. 
    ```java
    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE = new Complex(1, 0);
    public static final Complex I = new Complex(0, 1);
    ```
  * 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩터리를 제공할 수 있는데, 박싱된 기본 타입 클래스 전부와 `BigInteger`가 여기 속한다. -> 메모리 사용량, 가비지 컬렉션 비용 감소 
* 방어적 복사도 필요 없게 된다. 그러니 불변 클래스는 `clone` 메서드나 복사 생성자를 제공하지 않는 것이 좋다. (`String` 클래스의 복사 생성자는 자바 초창기 때 이 사실을 잘 이해하지 못해서 만들어진 것으로, 되도록 사용하지 말아야 한다.)
* <strong><i>불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.</i></strong>
  * `BigInteger`의 `negate` 메서드는 크기는 같고 부호만 반대인 새로운 `BigInteger`를 생성하는데, 이떄 배열은 비록 가변이지만 복사하지 않고 원본 인스턴스와 공유해도 된다: `BigInteger`는 불변 클래스이고, 한 번 생성되면 그 상태가 더 이상 변경되지 않기 때문이다. 즉, 내부 상태(int 배열)도 불변이다.
* 값이 바뀌지 않는 구성요소들로 이뤄진 객체이므로 맵의 key, Set의 원소로 쓰기에 안성맞춤이다. 안에 담긴 값이 바뀌면 불변식이 허물어지는데, 불변 객체를 사용하면 그런 걱정은 하지 않아도 된다.


### 불변 객체의 단점
* 값이 다르면 반드시 독립된 객체로 만들어야 한다. 값의 가짓수가 많다면 이들을 모두 만드는 데 큰 비용을 치러야 한다.
-> 다단계 연산(multistep operation)을 예측하여 기본 기능으로 제공: 각 단계마다 객체를 생성하지 않아도 된다. 
-> ex) `BigInteger`: 모듈러 지수 같은 다단계 연산 속도를 높여주는 가변 동반 클래스(companion class)를 package-private으로 두고 있다. 그런데, 이 가변 동반 클래스를 사용하기란 BigInteger를 쓰는 것보다 훨씬 어렵다. 
-> ex2) `String`: `StringBuilder`가 가변 동반 클래스다. (구닥다리 `StringBuffer`도 있다.)


### 불변 클래스 - `final class` 혹은 `private`|`package-private` 생성자로 만들고, `public` 정적 팩터리 제공
* `final` 클래스로 선언하는 방법이 가장 쉽지만, `private`|`package-private` 생성자로 만들고, `public` 정적 팩터리 제공이 더 유연한 방법이다.
* 예시
  ```java
  public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
      this.re = re;
      this.im = im;
    }

    public static Complex valueOf(double re, double im) {
      return new Complex(re, im);
    }
  }
  ```
* 이 방식이 최선일 때가 많다. 바깥에서 볼 수 없는 package-private 구현 클래스를 원하는 만큼 만들어 활용할 수 있다.
* 패키지 밖의 클라이언트에서 바라본 이 불변 객체는 사실상 final 이다. `public`, `protected` 생성자가 없으니 다른 패키지에서는 이 클래스를 확장하는 게 불가능하다.
* `BigInteger`, `BigDecimal` 설계 당시에는 final이어야 한다는 생각이 널리 퍼지지 않아서, 재정의 가능하게 설계되었다. 하위 호완성이 발목을 잡아 고치지 못했다. 그러니 신뢰할 수 없는 클라이언트에게서 이 인스턴스를 인수로 받는다면 주의해야 한다. 아래와 같이 복사해서 사용해야 한다.
  ```java
  public static BigInteger safeInstance(BigInteger val) {
    return val.getClass() == BigInteger.class ? val : new BigInteger(val.toByteArray());
  }
  ```


### 정리
* 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.
* 단순한 값 객체는 항상 불변으로 만들자. 
* `String`, `BigInteger` 처럼 무거운 값 객체도 불변으로 만들 수 있는지 고심해야 한다. 성능 때문에 어쩔 수 없다면, 불변 클래스와 쌍을 이루는 가변 동반 클래스를 public 클래스로 제공하자. 
* 모든 클래스를 불변으로 만들 순 없지만, 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자. 꼭 변경해야 할 필드를 뺀 나머지 모두를 final로 선언하자. 
* 즉, 다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다. 
* 객체를 재활용할 목적으로 다시 초기화하는 메서드도 안 된다. 복잡성만 커지지 성능 이점은 거의 없다. 
* 가변 클래스지만 가질 수 있는 상태가 많지 않은 클래스 예시: `java.util.concurrent.CountDownLatch`



## One more thing - BigInteger 다단계 연산
* 다단계 연산에서 중간 결과를 계속 불변 객체로 만드는 것보다 효율적으로 동작한다. `BigInteger`의 경우, 모듈러 지수 연선과 같은 복잡한 수학 연산을 최적화 하기 위해 내부적으로 가변 동반 클래스를 활용한다. 
* 내부 정수 배열, 비트 배열 같은 것들이 가변적일 수 있다. 연산 과정에서 변경되고 최종 결과가 도출될 떄까지 불변 `BigInteger` 객체로 변환되지 않을 것이다. 즉, 중간 과정에서만 사용된다는 것이다. 외부 API로는 노출되지 않는다. 
