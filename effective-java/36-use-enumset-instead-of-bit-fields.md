# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.16             |
| PART | 36. 비트 필드 대신 EnumSet을 사용하라 |

# 6장 Enums and Annotations
* Java에는 특수한 목적의 참조 타입이 두 가지 있다:
  1. 클래스의 일종인 enum type
  2. interfacec의 일종인 annotation 
* 이 타입들을 올바르게 사용하는 방법을 알아보자. 


## Item36. 비트 필드 대신 EnumSet을 사용하라

### Bit field enumeration constants 
* 옛날 기법
* 열거한 값들이 단독이 아닌 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴(Item 34)을 사용해왔다. 

#### Implementation
```java
public class Text {
  public static final int STYLE_BOLD = 1 << 0; // 1
  public static final int STYLE_ITALIC = 1 << 1; // 2
  public static final int STYLE_UNDERLINE = 1 << 2; // 4

  public void applyStyles(int styles) { ... }
}
```
* 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 비트 필드(bit field)라고 한다. 

#### Client
```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

#### 장점
* 비트별 연산을 사용해 합집합/교집합 같은 집합 연산 효율적 수행 

#### 단점
* 정수 열거 상수의 단점
* 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵다. 
* 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭다. 
* 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입(보통 `int`/`long`)을 선택해야 한다.
* API를 수정하지 않고는 비트 수(32bit or 64bit)를 늘릴 수 없기 때문이다. 


### `java.util.EnumSet` - A modern replacement for bit fields
* 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다. 
* `Set` 인터페이스를 완벽히 구현하고, 타입 안전, 다른 어떤 `Set` 구현체와도 함께 사용할 수 있다. 
* `EnumSet` 내부는 비트 벡터로 구현되었다. 원소가 총 64개 이하라면, 즉 대부분의 경우에 `EnumSet` 전체를 `long` 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다. 

#### Implementation
```java
public class Text {
  public enum Style { BOLD, ITALIC, UNDERLINE }

  // 어떤 Set을 넘겨도 되지만, EnumSet이 가장 좋다.
  public void applyStyles(Set<Style> styles) { ... }
}
```

#### Client
```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```
* `applyStyles`메서드가 `EnumSet<Style>`이 아닌 `Set<Style>`을 받은 이유를 생각해보자. 이왕이면 인터페이스로 받는 것이 일반적으로 좋은 습관이다. (Item 64)


### 정리
* 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 하더라도 비트 필드를 사용할 이유는 없다. 
* `EnumSet` 클래스가 비트 필드 수준의 명료함과 성능을 제공하고 아이템 34에서 설명한 열거 타입의 장점까지 제공한다. 
* 유일한 단점은 불변 `EnumSet`을 만들 수 없으므로 `Collections.unmodifiableSet`으로 사용할 수 있다. (명확성과 성능이 조금 희생되지만.)