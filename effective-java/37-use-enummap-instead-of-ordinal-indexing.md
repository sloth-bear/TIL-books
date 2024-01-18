# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.17             |
| PART | 37. ordinal 인덱싱 대신 EnumMap을 사용하라 |

# 6장 Enums and Annotations
* Java에는 특수한 목적의 참조 타입이 두 가지 있다:
  1. 클래스의 일종인 enum type
  2. interfacec의 일종인 annotation 
* 이 타입들을 올바르게 사용하는 방법을 알아보자. 


## Item37. ordinal 인덱싱 대신 EnumMap을 사용하라

### Using ordinal() to index into an array 
#### 문제점
* 배열은 제네릭과 호환되지 않으니 (Item28) 비검사 형변환 수행해야 하고, 깔끔히 컴파일되지 않을 것이다.
* 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다. 
* 가장 심각한 문제는 정확한 정수값을 사용한다는 것을 직접 보증해야 한다는 점이다. 정수는 열거 타입과 달리 타입 안전하지 않기 때문이다. 잘못된 값을 사용하면 잘못된 동작을 묵묵히 수행하거나 (운이 좋다면) `ArrayIndexOutOfBoundsException`을 던질 것이다. 

#### Example
```java
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycle.length; i++)
  plantsByLifeCycle[i] = new HashSet<>();

for (Plant p : garden)
  plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);

for (int i = 0; i < plantsByLifeCycle.length; i++) {
  System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

### Mapping data and Enum type by using EnumMap
#### Example
```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values())
  plantsByLifeCycle.put(lc, new HashSet<>());

for (Plant p : garden)
  plantsByLifeCycle.get(p.lifeCycle).add(p);

System.out.println(plantsByLifeCycle);
```

#### 장점
* 더 짧고 명료, 안전하다. 성능도 원래 버전과 비등하다.
* 안전하지 않은 형변환은 사용하지 않고, 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블을 달 필요도 없다. 
* 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천봉쇄된다. 
* `EnumMap`의 성능이 `ordinal`을 쓴 배열에 비견되는 이유는 그 내부에서 배열을 사용하기 때문이다. 내부 구현 방식을 숨겨서 `Map`의 타입 안전성과 배열의 성능 모두를 얻어낸 것이다. 
* `EnumMap`의 생성자가 받는 키 타입의 Class 객체는 한정적 토큰 타입으로, 런타임 제네릭 타입 정보를 제공한다. (Item 33)


#### Example2-1 - Stream 1
```java
System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle)));
```
* `EnumMap`이 아닌 고유의 맵 구현체를 사용했으므로 `EnumMap`을 써서 얻은 공간과 성능 이점이 사라진다는 문제가 있다. 

#### Example 2-2 - Stream 2
```java
System.out.println(
  Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle, () -> new EnumMap<>(LifeCycle.class), toSet())));
```
* 단순한 프로그램에서는 최적화가 굳이 필요 없으나 맵을 빈번히 사용하는 프로그램에서는 꼭 필요할것이다. 

#### Stream을 사용하면?
* `EnumMap`만 사용했을 때와는 살짝 다르게 동작한다: `EnumMap` 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트미 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다.


### 정리
* 배열의 인덱스를 얻기 위해 `ordinal`을 사용하는 것은 일반적으로 좋지 않으니 대신 `EnumMap`을 사용하라.
* 다차원 관계는 `EnumMap<..., EnumMap<...>>`으로 표현하라. 
* 애플리케이션 프로그래머는 `Enum.ordinal`을 웬만해선 사용하지 말아야 한다(Item 35)는 일반 원칙의 특수한 사례다.