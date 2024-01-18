# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.18             |
| PART | 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라 |

# 6장 Enums and Annotations
* Java에는 특수한 목적의 참조 타입이 두 가지 있다:
  1. 클래스의 일종인 enum type
  2. interfacec의 일종인 annotation 
* 이 타입들을 올바르게 사용하는 방법을 알아보자. 


## Item38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

### Enum type과 확장
* effective java 초판에서 소개한 typesafe enum pattern에 비해 Enum은 우수하다.
* 그러나 Enum type은 확장 불가능하다: 대부분 상황에서 Enum type을 확장하는 것은 좋지 않은 생각이다.
* 연산 코드(operation code/opcode)는 확장할 수 있는 Enum type이 어울린다. 
  * 연산 코드의 각 원소: 특정 기계가 수행하는 연산 (e.g. API가 제공하는 기본 연산 외 사용자 확장 연산을 추가할 수 있도록 열어줘야 할 때)

#### Example
```java
public interface Operation {
  double apply(double x, double y);
}

public enum BasicOperation implements Operation {
  PLUS("+") {
    public double apply(double x, double y) { return x + y; }
  };

  private final String symbol;
  ...
}
```
* `BasicOperation`은 확장할 수 없지만 인터페이스인 `Operation`은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다. `Operation`을 구현한 또 다른 열거 타입을 정의해 기본 타입인 `BasicOperation`을 대체할 수 있다. 
* 공유하는 기능(혹은 중복 기능)이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있을 것이다. 
* e.g. `java.nio.file.LinkOption`는 `CopyOption`과 `OpenOption` 인터페이스를 구현했다. 

##### Testing
```java
public static void main(String[] args) {
  test(BasicOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
  for (Operation op : opEnumType.getEnumConstants())
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}

// 조금 더 유연하다.
private static <T extends Enum<T> & Operation> void test(Collection<? extends Operation> opSet, double x, double y) {
  for (Operation op : opSet)
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```


### 정리
* 열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 결과를 낼 수 있다: 클라이언트는 이 인터페이스를 구현해 자신만의 열거 타입(혹은 다른 타입)을 만들 수 있다. 
* 그리고 API가 인터페이스 기반으로 작성되었다면 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다. 