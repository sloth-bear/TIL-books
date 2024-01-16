# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.15             |
| PART | 34. int 상수 대신 열거 타입을 사용하라 |

# 6장 Enums and Annotations
* Java에는 특수한 목적의 참조 타입이 두 가지 있다:
  1. 클래스의 일종인 enum type
  2. interfacec의 일종인 annotation 
* 이 타입들을 올바르게 사용하는 방법을 알아보자. 


## Item34. int 상수 대신 열거 타입을 사용하라

### int enum pattern
* 쓰지 말자. 상당히 취약하다.
```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
```

### Enum type
* 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입
* e.g. the seasons of the year, 행성, 카드게임의 카드 종류 
* 상수 하나당 자신의 인스턴스를 만들어 public static final field로 만들어 공개한다. 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final 이다. 
```java
public enum Apple { FUJI, PIPPIN }
public enum Orange { Navel, Temple }
```

#### 장점
* 컴파일타임의 타입 안전성 
* 다른 타입과 이름이 같은 상수도 평화롭게 공존한다. 
* `toString` 메서드는 출력하기에 적합한 문자열을 내어준다. 
* 임의의 메서드나 필드를 추가할 수 있고, 임의의 인터페이스를 구현할 수도 있다. 

#### 선언
* 열거 타입을 선언한 클래스 혹은 그 패키지에서만 유용한 기능은 private나 package-private 메서드로 구현한다. 그 기능을 클라이언트에 노출해야 할 합당한 이유가 없다면 private로, 혹은 package-private로 선언하라. (Item 15)
* e.g. `java.math.RoundingMode`: `BigDecimal`과 관련 없는 영역에도 유용한 개념이라 톱레벨로 올렸다. 
* `valueOf(String)`: 상수 이름을 입력 받아 그 이름에 해당하는 상수를 반환해주는 메서드(자동 생성)
* `toString`이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 `fromString` 메서드도 함께 제공하는 걸 고려해보자. 

#### 활용 예시 1 - constant-specific method implementation
* 상수별 클래스 몸체를 가지는, 즉, 각 상수에서 자신에 맞게 재정의할 수 있다. 
* 단, 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다. -> 메서드 내에서 분기처리하는 것은 위험할 수 있으니, 모든 상수에 중복해서 넣거나 도우미 메서드로 작성한 다음 각 상수가 호출하면? 코드가 점점 복잡해진다. -> 활용 예시 2 참조 
```java
public enum Operation {
  PLUS {
    public double apply(double x, double y) {
      return x + y;
    }
  },
  ...
}
```

#### 활용 예시 2. The strategy enum pattern
```java
enum PayrollDay {
  MONDAY, TUESDADY, SATURDAY(PayType.WEEKEND);

  private final PayType payType;

  PayrollDay(PayType payType) {
    this.payType = payType;
  }

  int pay(int minutesWorked, int payRate) {
    return payType.pay(minutesWorked, payRate);
  }

  enum PayType {
    WEEKDAY {
      int overtimePay(int minsWorked, int payRate) {
        return minsWorked <= MINS_PER_SHIFT ? 0 : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
      }
    },
    WEEKEND {
      int overtimePay(int minsWorked, int payRate) {
        return minsWorked * payRate / 2;
      }
    };

    abstract int overtimePay(int mins, int payRate);
    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minsWorked, int payRate) {
      int basePay = minsWorked * payRate;
      return basePay + overtimePay(minsWorked, payRate);
    }
  }
}
```


#### When use? 
* 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합일 때 
* 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다. 나중에 상수가 추가돼도 바이너리 수준에서 호환되도록 설계되었다. 


### 정리
1. 열거 타입은 정수 상수보다 뛰어나다. 읽기 쉽게 안전하고 강력하다. 
2. 대다수의 열거 타입은 명시적 생성자, 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나, 상수마다 다르게 동작할 때는 필요하다. 
3. 하나의 메서드가 상수별로 다르게 동작할 때는 상수별 메서드 구현을 사용하자. 
4. 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자. 