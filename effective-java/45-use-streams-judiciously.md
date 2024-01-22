# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.22             |
| PART | 45. 스트림은 주의해서 사용하라  |

# 7장 Lambdas and Streams
* Java 8 - Functional interface, lambda, method reference 개념이 추가되면서 함수 개체를 더 쉽게 만들 수 있게 되었다. 
* 그리고 Stream API까지 추가되어 Data element의 시퀀스 처리를 라이브러리 차원에서 지원하기 시작했다. 
* 이 기능들을 효과적으로 사용하는 방법을 알아보자. 


## Item45. 스트림은 주의해서 사용하라 

### Ease the task of performing Bulk operations 
Stream API는 bulk operation, 다량의 데이터 처리 작업(순차적/병렬적)을 돕고자 자바 8에서 추가되었다. 

#### 핵심 추상 개념
1. Stream: Data elements의 유한, 혹은 무한 sequence를 표상한다. 
    * elements: 어디서든 올 수 있다. e.g. collections, arrays, files, regular expression pattern matchers, pseudorandom number generators, other streams
    * `int`, `long`, `double`: 3가지의 기본 타입 값도 지원한다.
2. Stream pipeline: 이 elements들로 수행하는 다단계 연산을 표상한다. 

### Stream pipeline 
#### 구성
* source stream -- 하나 이상의 중간 연산(*Intermediate operations*) -- 하나의 종단 연산(*terminal operation*)

#### Intermediate operation
* 각 중간 연산은 stream을 어떤 방식으로 transform한다. 
    * e.g. mapping each element to a function of that element
    * e.g. filtering out all elements that do not satisfy som condition 
* 중간 연산은 모두 한 스트림을 다른 스트림으로 바꾼다. 변환된 스트림의 원소 타입은 변환 전 원소 타입과 같을 수도, 다를 수도 있다. 

#### Terminal operation
* final computation on the stream - 마지막 중간 연산의 결과 stream에 최종 연산 수행 
* e.g. storing its elements into a collection
* e.g. returning a certain element or printing all of its elements

#### lazy evaluation (지연 평가)
* 스트림 파이프라인은 지연 평가된다: 종단 연산이 호출될 떄 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다. 
* 이러한 지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠다. 
* 종단 연산이 없는 스트림 파이프라인은 아무 일도 하지 않는 명령어인 No-op과 같으니 종단 연산을 빼먹는 일이 절대 없도록 하자. 

#### Fluent API
* Stream API는 메서드 연쇄를 지원하는 fluent API다. 
* 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다. 파이프라인 여러 개를 연결해 표현식 하나로 만들 수도 있다. 

### Sequentially 
* 기본적으로 Stream API는 순차적으로 수행된다. 
* 병렬로 실행하고자 한다면 파이프라인을 구성하는 stream 중 하나에서 `parallel` 메서드를 호출해주기만 하면 되지만, 효과를 볼 수 있는 상황은 많지 않다. (Item 48)

### 사용 노하우 
#### Example 1. Stream 적용 전
```java
public class Anagrams {  
  
  public static void main(String[] args) throws IOException {  
    File dictionary = new File(args[0]);  
    int minGroupSize = Integer.parseInt(args[1]);  
  
    Map<String, Set<String>> groups = new HashMap<>();  
    try (Scanner s = new Scanner(dictionary)) {  
      while (s.hasNext()) {  
        String word = s.next();  
        // 각 키에 다수의 값을 매핑하는 맵을 쉽게 구현할 수 있다. 
        groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);
      }  
    }  
  
    for (Set<String> group : groups.values())  
      if (group.size() >= minGroupSize) {  
        System.out.println(group.size() + ": " + group);  
      }  
  }  
  
  private static String alphabetize(String word) {  
    char[] a = word.toCharArray();  
    Arrays.sort(a);  
    return new String(a);  
  }  
}
```

#### Example 2. Stream 적용 후 
```java
public class Anagrams {
  public static void main(String[] args) throws IOException {
    Path dictionary = Paths.get(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);

    try (Stream<String> words = Files.lines(dictionary)) {
      // lambda 사용 시에는 매개변수 이름을 잘 지어야 가독성이 유지된다. (타입이름이 자주 생략되므로)
      words.collect(groupingBy(word -> alphabetize(word)))
        .values().stream()
        .filter(group -> group.size() >= minGroupSize)
        .forEach(group -> System.out.println(group.size() + ": " + g));
    }
  }
}
```
* 일부 method를 분리해서 가독성을 높였다. 
* alphabetize 도 stream으로 구현할 수 있겠으나, 오히려 성능이 떨어질 수 있고 잘못 구현할 가능성이 높아진다. `char` 값들을 처리할 때는 스트림을 삼가는 편이 낫다. 

#### Refactoring Tip
* Stream으로 바꾸는 게 가능하더라도 코드 가독성과 유지보수 측면에서 손해를 볼 수 있으므로 중간 정도 복잡한 작업에도 스트림과 반복문을 적절히 조합하는 것이 최선이다. 
* 기존 코드는 스트림을 사용하도록 리팩터링하되, 새 코드가 더 나아보일 때만 반영하자. 

##### Stream과 맞지 않는 케이스 
1. 코드 블록 내에서는 scope 내 지역 변수를 읽고 수정할 수 있다. 하지만 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있고, 지역변수를 수정하는 건 불가능하다. 
2. 코드 블록에서는 return 문을 사용해 메서드에서 빠져나가거나, `break`, `continue` 문으로 블록 바깥의 반복문을 종료하거나 반복을 한 번 건너뛸 수 있다. 또한 메서드 선언에 명시된 검사 예외를 던질 수 있다. 하지만 람다는 이 중 어떤 것도 할 수 없다. 

##### Stream과 잘 어울리는 케이스 
1. 원소들의 시퀀스를 일관되게 변환한다. 
2. 원소들의 시퀀스를 필터링한다. 
3. 원소들의 시퀀스를 하나의 연산을 사용해 결합한다. (더하기, 연결, 최소값 구하기 등)
4. 원소들의 시퀀스를 컬렉션에 모은다 (아마 공통된 속성을 기준으로 묶으면서)
5. 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다. 

##### Stream으로 처리하기 어려운 경우 
1. 한 데이터가 파이프라인의 여러 단계(stage)를 통과할 때, 이 데이터의 각 단계에서의 값들에 동시에 접근하기는 어려운 경우다. 스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조이기 때문이다. 
2. 원래 값과 새로운 값의 쌍을 저장하는 객체를 사용해 우회할 수 있겠지만, 점점 더 복잡해지므로 스트림을 쓰는 주목적에서 완전히 벗어난다. 가능한 경우라면, 앞 단계의 값이 필요할 때 매핑을 거꾸로 수행하는 방법이 나을 것이다. 

### 정리 
* 스트림과 반복문 중 선택이 애매한 경우라면, 둘 다 해보고 더 나아 보이는 것을 선택해라. 혹은 동료들이 스트림 코드를 이해할 수 있고 선호한다면 스트림 방식을 사용하자. 
* 스트림을 사용해야 멋지게 처리할 수 있는 일이 있고, 반복 방식이 더 알맞은 일도 있다. 수많은 작업이 이 둘을 조합했을 때 가장 멋지게 해결된다. 