# Today I Learned

| 구분 | 내용                        |
| ---- | --------------------------|
| DATE | 2023.12.27                |
| PART | 50. 적시에 방어적 복사본을 만들라 |

# 8장 Method

## Item50. 적시에 방어적 복사본을 만들라 
자바는 안전한 언어다. 이것이 자바를 쓰는 즐거움 중 하나다. 네이티브 메서드를 사용하지 않으니 C, C++ 같이 안전하지 않은 언어에서 흔히 보이는 버퍼 오버런, 배열 오버런, 와일드 포인터 같은 메모리 충돌 오류에서 안전하다. 자바로 작성한 클래스는 시스템의 다른 부분에서 무슨 짓을 하든 그 불변식이 지켜진다. 메모리 전체를 하나의 거대한 배열로 다루는 언어에서는 누릴 수 없는 강점이다. 그러나 아무런 노력 없이 다른 클래스의 침범을 다 막을 순 없다. 클라이언트가 불변식을 깨뜨리려 한다고 가정하고 방어적으로 프로그래밍해야 한다.

### 불변식을 지키지 못한 경우 
```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException("시작 시각이 종료 시각보다 늦을 수 없습니다.");
        }
        this.start = start;
        this.end = end;
    }

    public Date start() { return start; }
    public Date end() { return end; }
}
```
* 이 클래스는 불변처럼 보이고, 시작 시각이 종료 시각보다 늦을 수 없다는 불변식이 지켜질 것 같지만 `Date`가 가변이라는 사실을 이용하면 어렵지 않게 그 불변식을 깨뜨릴 수 있다.
  ```java
  Date end = new Date();
  Period period = new Period(new Date(), end);
  end.setYear(78);
  ```
  이 경우, `Date` 대신 불변인 `Instant`|`LocalDateTime` 등을 사용하면 된다. `Date`는 낡은 API이므로 새로운 코드를 작성할 때는 더 이상 사용하면 안 된다. 여전히 많은 API 내부 구현에 그 잔재가 남아 있으므로, 방어적으로 작성할 필요가 있다.

### 불변식을 지켜보자
외부 공격으로부터 Period 인스턴스 내부를 보호하려면 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사(defensive copy)해야 한다. 그리고 인스턴스 내부에서는 원본 대신 복사본을 이용한다. 
```java
public Period(Date start, Date end) {
  this.start = new Date(start.getTime());
  this.end = new Date(end.getTime());

  if (this.start.compareTo(this.end) > 0) {
    throw new IllegalArgumentException("시작 시각이 종료 시각보다 늦을 수 없습니다.");
  }
}
```

#### <strong><i>왜 유효성 검사 전에 방어적 복사본을 만들고, 복사본으로 유효성 검사를 했을까?</i></strong><br />
순서가 부자연스러워 보이지만 반드시 필요한 부분이다. 멀티스레딩 환경이라면 원본 객체의 유효성을 검사한 후 복사본을 만드는 그 찰나의 취약한 순간에 다른 스레드가 원본 객체를 수정할 위험이 있기 때문이다: TOCTOU 공격(time-of-check/time-of-use attack) 

#### <strong><i>왜 방어적 복사에 `Date`의 `clone` 메서드를 사용하지 않았을까?</i></strong><br />
`Date`는 `final`이 아니므로 `clone`이 `Date`가 정의한 게 아닐 수 있다. 즉, `clone`이 악의를 가진 하위 클래스의 인스턴스를 반환할 수 있다. 이 하위 클래스는 start와 end 필드의 참조를 private 정적 리스트에 담아뒀다가 공격자에게 이 리스트에 접근하는 길을 열어줄 수 있다. 결국 공격자에게 `Period` 인스턴스 자체를 송두리째 맡기는 꼴이 된다. <u>이런 공격을 막기 위해서는 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 `clone`을 사용해서는 안 된다.</u>

그리고 추가적으로 접근자 메서드가 내부의 가변 정보를 직접 드러내지 않도록 수정하자.
```java
public Date start() { return new Date(start.getTime()); }
public Date end() { return new Date(end.getTime(); }
```

이제 `Period`는 완전한 불변이 되었고 불변식을 위배할 방법은 없다. (native method, reflection 제외)
생성자와 달리 접근자 메서드에서는 방어적 복사에 `clone`을 사용해도 된다. `Period`가 가지고 있는 `Date` 객체는 `java.util.Date`임이 확실하기 때문이다. 물론, 그렇더라도 인스턴스를 복사하는 데는 일반적으로 생성자나 정적 팩터리를 쓰는 것이 좋다. 

불변 객체를 만들기 위해서만이 아니라 메서드, 생성자 모두 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 할 때면 항시 그 객체가 잠재적으로 변경될 수 있는지를 생각해야 한다. 임의로 변경된다 하더라도 클래스가 문제 없이 동작할지를 생각해보아야 하며, 확신할 수 없다면 복사본을 만들어 저장해야 한다.

이제 "되도록 불변 객체들을 조합해 객체를 구성해야 방어적 복사를 할 일이 줄어든다"는 교훈을 얻을 수 있다.