# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.13             |
| PART | 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라 |

# 5장 Generic
* 자바 5부터 지원된 generic - 지원 전에는 컬렉션에서 객체를 꺼낼 때마다 형변환을 해야 했다. 
* 제네릭 사용 시 컬렉션이 담을 수 있는 타입을 컴파일러에게 알려주게 된다: 컴파일 타임에서 형변환 오류를 차단하므로 더 안전하고 명확한 프로그램을 만들어준다. 
* 컬렉션이 아니더라도 관련 이점을 누릴 수 있다. 
* 이번 장에서는 제네릭의 이점을 최대로 살리고 단점을 최소화하는 방법을 이야기한다.

## Item32. 제네릭과 가변인수를 함께 쓸 때는 신중하라

### Varargs와 Generic

- 가변인수(varargs) 메서드와 제네릭은 자바 5때 함께 추가되었으나 함께 잘어우러지진 않는다.
- varargs
  - 메서드에 넘기는 인수의 개수를 client가 조절할 수 있게 해준다.
  - 구현 방식
    - 가변인수 메서드 호출 시 가변인수를 담기 위한 배열이 자동으로 하나 만들어진다.
    - 이 배열은 내부로 감춰야했으나, 이 배열이 client에 노출하는 문제가 생겼다.
    - varargs parameter에 generic / 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생한다. (거의 모든 generic과 매개변수화 타입은 은 실체화 불가 타입이므로)
      ```
      warning: [unchecked] Possible heap pollution from parameterized vararg type List<String>
      ```
- Heap pollution (힙 오염): 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다. 이렇게 다른 타입 객체를 참조하는 상황에서는 컴파일러가 자동 생성한 형변환이 실패할 수 있으니, 제네릭 타입 시스템이 약속한 타입 안정성의 근간이 흔들린다.

### Heap pollution

- Generic type의 변수가 예상과 다른 type의 객체를 참조하게 되면서, 실행 시각(runtime)에 type casting error가 발생할 위험이 있는 상태
- 주로 generic type 정보가 컴파일 시점에서만 존재하고 런타임에는 type 정보가 제거되는 자바의 type erasure(타입 소거) 메커니즘과 관련이 있다.

#### e.g. varargs with generic

```java
static void dangerous(List<String>... stringLists) {
  List<Integer> intList = List.of(42);
  Object[] objects = stringLists;
  objects[0] = intList; // 힙 오염 발생
  String s = stringLists[0].get(0); // ClassCastException
}
```

- 형변환하는 곳이 보이지 않는데도 인수를 건네 호출하면 `ClassCastExcpetion`을 던진다. 마지막 라인에 컴파일러가 생성한 (보이지 않는) 형변환이 숨어 있기 때문이다.
- **_즉, 타입 안전성이 깨진다. 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다._**

### 대안

- **<u>가변인수 메서드를 호출할 때, varargs 매개변수를 담는 제네릭 배열이 만들어진다.</u>** 메서드가 이 배열에 아무것도 저장하지 않고(그 매개변수들을 덮어쓰지 않고), 그 배열의 참조가 밖으로 노출되지 않는다면(신뢰할 수 없는 코드가 배열에 접근할 수 없다면) 타입 안전하다.
- 즉, 이 varargs 매개변수 배열이, 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면(varargs의 목적대로) 그 메서드는 안전하다.

#### 주의 케이스 - varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하는 경우

- 아래와 같은 경우는 위험하다. (varargs 매개변수 배열에 아무것도 저장하지 않지만 타입 안전성을 깰 수 있는 경우) - 가변인수로 넘어온 매개변수들을 배열에 담아 반환하는 제너릭 메서드
  - 이 메서드가 반환하는 배열의 타입: 이 메서드가 인수를 넘기는 컴파일타임에 결정 -> 그 시점에서는 컴파일러에게 충분한 정보가 주어지지 않아 타입을 잘못 판단할 수 있음.
  - 따라서 자신의 varargs 매개변수 배열을 그대로 반환하면 힙 오염을 이 메서드를 호출한 쪽의 콜스택으로까지 전이하는 결과를 낳을 수 있음.
  ```java
  static <T> T[] toArray(T... args) {
  	return args;
  }
  static <T> T[] pickTwo(T a, T b, T c) {
  	switch(ThreadLocalRandom.current().nextInt(3)) {
  		case 0: return toArray(a, b);
  		case 1: return toArray(a, c);
  		case 2: return toArray(b, c);
  	}
  	throw new AssertionError(); // 도달할 수 없다.
  }
  public static void main(String[] args) {
  	// ClassCastException: Object[] -> String[]
  	String[] attributes = pickTwo("좋은", "빠른", "저렴한");
  }
  ```
  - @SafeVarargs로 제대로 애노테이트된 또 다른 varargs 메서드에 넘기는 것은 안전하다.
  - 이 배열 내용의 일부 함수를 호출만 하는(varargs를 받지 않는) 일반 메서드에 넘기는 것도 안전하다.

#### 대안 예시

1. `@SafeVarargs`: 제너릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 달아라. 단, 반드시 안전해야 한다. 안전하지 않은 varargs 메서드는 절대 작성해서는 안 된다. e.g. `List.of`

```java
@SafeVarargs
static <T> List<T> flatten(List<? excends T>... lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists) {
    result.addAll(list);
  }
  return result;
}
```

2. varargs 매개변수 -> List 매개변수로 변경
   - 장점: 컴파일러가 이 메서드의 타입 안전성을 검증할 수 있음
   - 단점: 클라이언트 코드가 살짝 지저분해지고 속도가 조금 느려질 수 있음

```java
static <T> List<T> flatten(List<List<? excends T>> lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists) {
    result.addAll(list);
  }
  return result;
}
```

#### 규칙

- varargs 매개변수 배열에 아무것도 저장하지 않는다.
- 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.
- `@SafeVarargs` 애너테이션은 재정의할 수 없는 메서드에만 달아야 한다. 재정의한 메서드도 안전할지는 보장할 수 없기 때문이다. (java8 - static method, final instance method / java9: private instance method 허용)

### 정리

- 가변인수와 제네릭은 궁합이 좋지 않다.
  1.  가변인수: 배열을 노출하여 추상화가 완벽하지 못함
  2.  배열과 제네릭의 타입 규칙이 서로 다름
- 제네릭 varargs 매개변수는 타입 안전하지 않지만 허용된다. 메서드에 제네릭 (혹은 매개변수화 된) varargs 매개변수를 사용하고자 한다면, 그 메서드는 타입 안전해야 하며 `@SafeVarargs` 애너테이션을 달아 사용하는 데 불편함이 없게끔 하자.
