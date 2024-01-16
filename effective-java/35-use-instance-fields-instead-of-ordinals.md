# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.16             |
| PART | 35. ordinal 메서드 대신 인스턴스 필드를 사용하라 |

# 6장 Enums and Annotations
* Java에는 특수한 목적의 참조 타입이 두 가지 있다:
  1. 클래스의 일종인 enum type
  2. interfacec의 일종인 annotation 
* 이 타입들을 올바르게 사용하는 방법을 알아보자. 


## Item35. ordinal 메서드 대신 인스턴스 필드를 사용하라

###  Ordinal method 
* 상수 선언을 바꾸는 순간 ordinal 값은 바뀐다. 
* 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다. 
* 값을 중간에 비워둘 수 없다. 
* 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고 인스턴스 필드에 저장하자.
* `EnumSet`, `EnumMap` 같이 열거 타입 기반의 범용 자료구죠에 쓸 목적으로 설계되었다. 이 경우가 아니라면 사용하지 말자. 

### Instance Field
```java
public enum Ensemble {
  SOLO(1), DUET(2);

  private final int numberOfMusicians;
  Ensemble(int size) { this.numberOfMusicians = size; }
  public int numberOfMusicians() {
    return numberOfMusicians;
  }
}
```

