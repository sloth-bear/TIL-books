# Today I Learned

| 구분 | 내용                                  |
| ---- | ------------------------------------|
| DATE | 2023.12.27                          |
| PART | 02. 생성자에 매개변수가 많다면 빌더를 고려하라 |

# 2장 객체 생성과 파괴

## Item02. 생성자에 매개변수가 많다면 빌더를 고려하라
정적 팩터리와 생성자에는 똑같은 제약이 하나 있다. 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 점이다. 많은 선택 항목을 가지는 클래스에 대해 프로그래머들은 어떤 패턴을 자주 사용했을까? 
1) telescoping constructor pattern(점층적 생성자 패턴) - 필요한 매개변수만 받는 생성자를 점층적으로 늘린다. 
 - 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다. 
2) JavaBeans Pattern: 매개변수가 없는 생성자로 객체를 만든 후, setter 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식이다. 
 - 객체 하나를 만들기 위해 메서드를 여러 개 호출해야 하고, 객체가 완성되기 전까지 일관성(consistency)이 무너진 상태에 놓이게 된다. 심각한 단점이다. 
 - 단점을 보완하기 위해 객체를 수동으로 freezing 하고, 얼리기 전에는 사용할 수 없도록 하기도 하지만 다루기 어려워서 실전에서는 거의 쓰이지 않는다. 또한 런타임 오류에도 취약하다.
3) Builder pattern
 - 점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 겸비했다. 
 - 필요한 객체를 직접 만드는 대신, 필요 매개변수만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻는다.

### Builder Pattern
```Java
public class NutritionFacts {
  private final int servingSize;
  private final int servings;
  private final int calories;
  private final int fat;

  public static class Builder {
      private final int servingSize;
      private final int servings;

      private int calories = 0;
      private int fat = 0;

      public Builder(int servingSize, int servings) {
          this.servingSize = servingSize;
          this.servings = servings;
      }

      public Builder calories(int val) {
          this.calories = val;
          return this;
      }

      public Builder fat(int val) {
          this.fat = val;
          return this;
      }

      public NutritionFacts build() {
          return new NutrionFacts(this);
      }
  }

  private NutritionFacts(Builder builder) {
    this.servingSize = builder.servingSize;
    this.servings = builder.servings;
    this.calories = builder.calories;
    this.fat = builder.fat;
  }
}
```

#### 장점
* Builder의 setter method들은 자신을 반환하므로 연쇄적으로 호출할 수 있다: `fluent API`|`Method chaining`
* 쓰기 쉽고, 읽기 쉽다. (파이썬과 스칼라의 Named optional parameter를 흉내낸 것)
* 잘못된 매개변수를 일찍 발견하기 위해서는 빌더의 생성자와 메서드에서 입력 매개변수를 검사하고 build 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 <i><strong>불변식(invariant)</strong></i>을 검사하자. `IllegalArgumentException` 예외를 던지면 된다.
  ```java
  public class Pizza {
    
    private final List<String> toppings;

    private Pizza(Builder builder) {
      this.toppings = new ArrayList<>(builder.toppings);

      if (toppings.isEmpty()) {
        throw new IllegalStateException("피자에는 최소 한 가지 토핑이 필요합니다.");
      }
    }

    public static class Builder {
      private List<String> toppings = new ArrayList<>();

      public Builder addTopping(String topping) {
        if (topping == null) {
          throw new IllegalArgumentException("토핑은 null일 수 없습니다.");
        }

        this.toppings.add(topping);
        return this;
      }

      public Pizza build() {
        return new Pizza(this);
      }
    }
  }
  ```
* 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다. 

#### 단점
* 객체를 만들려면, 그에 앞서 빌더부터 만들어야 한다. 생성 비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있다. 
* 점층적 생성자 패턴에 비해 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 한다. But API는 시간이 지날수록 매개변수가 많아지는 경향이 있음을 명심하자. 생성자나 정적 팩터리로 시작해 전환할 수 있지만, 이전에 만들어둔 것들이 도드라져 보일 것이다. 그러니 애초에 빌더로 시작하는 편이 나을 때가 많다. 즉, 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다. (매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다.)


## One more thing... 불변식에 관해 
* Item 50. 적시에 방어적 복사본을 만들라 참조
* 가변(Mutable) - 불변식(Invariants) - 불변(Immutable)

### `Immutable`|`Immutability` 불변
* 어떠한 변경도 허용하지 않는다.
* 주로 변경을 허용하는 가변(Mutable) 객체와 구분하는 용도로 쓰인다.
* ex) `String`: 한 번 만들어지면 절대 값을 바꿀 수 없는 불변 객체다.

### `invariant` 불변식
* 프로그램이 실행되는 동안, 혹은 정해진 기간 동안 반드시 만족해야 하는 조건을 말한다.
* 불변은 아니다. 즉 변경을 허용할 수는 있으나 주어진 조건 내에서만 허용한다.
* ex) List의 크기는 반드시 0 이상이어야 하니, 만약 한순간이라도 음수 값이 된다면 불변식이 깨진 것이다. 
* 가변 객체에도 불변식은 존재할 수 있다. 넓게 보면 불변은 불변식의 극단적인 예라 할 수 있다. 
