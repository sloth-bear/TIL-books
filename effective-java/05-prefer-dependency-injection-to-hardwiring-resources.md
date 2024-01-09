# Today I Learned

| 구분 | 내용                      |
| ---- | ------------------------|
| DATE | 2024.01.06              |
| PART | 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라 |

# 2장 객체 생성과 파괴
* 객체를 만들어야 할 떄와 만들지 말아야 할 때를 구분하는 방법
* 올바른 객체 생성 방법
* 불필요한 생성을 피하는 방법
* 제때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업을 관리하는 요령 

## Item5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

## 자원을 직접 명시한 예
클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 static utility class는 사용하지 않는 것이 좋다. 이 자원들을 클래스가 만들게 해서도 안 된다. 대신 필요한 자원(혹은 그 자원을 만들어주는 팩터리)을 생성자(혹은 정적 팩터리나 빌더)에 넘겨주는 것이 좋다. 

### static utility 잘못 사용한 예
```java
public class SpellChecker {
  private static final Lexicon dictionary = ...;

  private SpellChecker() {}

  public static boolean isValid(String word) { ... }
  public static List<String> suggestions(String typo) { ... }
}
```

### Singleton 잘못 사용한 예
```java
public class SpellChecker {
  private final Lexicon dictionary = ...;

  private SpellChecker() {}
  public static SpellChecker INSTANCE = new SpellChecker(...);

  public boolean isValid(String word) { ... }
  public List<String> suggestions(String typo) { ... }
}
```

두 가지 방식 모두 단 하나만 사용한다고 가정한다는 점에서 그리 훌륭해보이지 않는다. 언어별 사전, 특수 어휘용 사전 등 별도로 두기도 한다. 테스트용 사전이 필요할 수 있다. 


## Dependency Injection 
생성자, 정적 팩터리(Item 1), 빌더(Item 2) 모두에 똑같이 응용할 수 있다. 

### 예제 
여러 자원 인스턴스를 지원하게 해준다. 또한 불변(Item 17)을 보장하여 같은 자원을 사용하려는 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있기도 하다. 
```java
public class SpellChecker {
  private final Lexicon dictionary;

  public SpellChecker(Lexicon dictionary) {
    this.dictionary = Objects.requireNonNull(dictionary);
  }

  public boolean isValid(String word) { ... }
  public List<String> suggestions(String typo) { ... }
}
```


### Factory Method pattern
생성자에 자원 팩터리를 넘겨주는 방식으로, 이 패턴의 쓸만한 변형이다. 팩터리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 의미한다. 

#### `Supplier<T>`
`Supplier<T>` 인터페이스가 팩터리를 표현한 완벽한 예다. 이를 입력으로 받는 메서드는 일반적으로 한정적 와일드카드 타입(bounded wildcard type, Item 31)을 사용해 팩터리의 타입 매개변수를 제한해야 한다. 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다. 
```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```


### 장점
1. 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다. 

### 단점
1. 의존성이 수천 개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들 수 있다. Dagger, Guice, Spring 같은 의존 객체 주입 프레임워크를 사용하면 이런 어질러짐을 해소할 수 있다. 