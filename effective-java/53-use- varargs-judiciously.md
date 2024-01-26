# Today I Learned

| 구분 | 내용                        |
| ---- | --------------------------|
| DATE | 2024.01.26                |
| PART | 53. 가변인수는 신중히 사용하라 |

# 8장 Method
* 이번 장에서는 메서드를 설계할 때 주의할 점들을 살펴본다.
* 구체적으로는 매개변수와 반환값을 어떻게 처리해야 하는지, 메서드 시그니처는 어떻게 설계해야 하는지, 문서화는 어떻게 해야 하는지를 다룬다. 
* 상당 부분은 메서드뿐 아니라 생성자에도 적용된다. 
* 사용성, 견고성, 유연성에 집중할 것이다.

## Item53. 가변인수는 신중히 사용하라

### Wrong way to use varargs 
```java
static int min(int... args) {
  if (args.length == 0) 
    throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
  int min = args[0];
  for (int i = 1; i < args.length; i++)
    if (args[i] < min)
      min = args[i];
  return min;
}
```
* 이 방법의 문제는 무엇일까? 
* 인수를 0개만 넣어 호출하면 컴파일 타임이 아닌 런타임에 실패한다. 코드도 지저분하다. 
* args 유효성 검사를 명시적으로 해야 한다. 유효성 검사를 하지 않으면 min의 초기값을 Integer.MAX_VALUE로 설정하지 않고는 더 명료한 for-each문도 사용할 수 없다. 

#### Write way to use varargs
```java
static int min(int firstArg, int... remainingArgs) {
  int min = firstArg;
  for (int arg : remainingArgs)
    if (args < min)
      min = args;
  return min;
}
```
* 매개변수를 2개 받도록 하면 앞서의 문제가 사라진다.


### Performance
* 성능에 민감한 상황이라면 가변인수가 걸림돌이 될 수 있다. 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화한다. 다행히 이 비용을 감당할 수는 없지만, 가변인수의 유연성이 필요할 때 선택할 수 있는 멋진 패턴이 있다. 예를 들어 해당 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 하면, 인수를 0개~4개까지 총 5개를 다중정의하고, 마지막 다중정의 메서드가 인수 4개 이상인 5%의 호출을 담당하면 된다. e.g. `List.of`, `EnumSet`의 정적 팩터리 (열거 타입 집합 생성 비용 최소화 - 비트 필드를 대체하면서 성능까지 유지해야 하므로 아주 적절하게 활용한 예; Item 36)