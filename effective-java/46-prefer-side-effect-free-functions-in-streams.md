# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.22             |
| PART | 46. 스트림에서는 부작용 없는 함수를 사용하라  |

# 7장 Lambdas and Streams
* Java 8 - Functional interface, lambda, method reference 개념이 추가되면서 함수 개체를 더 쉽게 만들 수 있게 되었다. 
* 그리고 Stream API까지 추가되어 Data element의 시퀀스 처리를 라이브러리 차원에서 지원하기 시작했다. 
* 이 기능들을 효과적으로 사용하는 방법을 알아보자. 


## Item46. 스트림에서는 부작용 없는 함수를 사용하라

### Functional programming 
* 스트림은 그저 또 하나의 API가 아닌, 함수형 프로그래밍에 기초한 패러다임이다. 
* 스트림이 제공하는 표현력, 속도, 상황에 따라 병렬성을 얻기 위해서는 API는 말할 것도 없고, 이 패러다임까지 함께 받아들여야 한다. 

### Streams paradigm - Structure your computation as a sequence of transformations 
* 스트림 패러다임에서 가장 중요한 부분은 "***계산을 일련의 변환(transformation)으로 재구성***"하는 부분이다. 
* 각 stage의 결과는 가능한 한 이전 단계의 결과를 받아 처리하는 ***pure function***여야 한다. 
* pure function: 오직 입력만이 결과에 영향을 주는 함수로, 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다. 이를 위해서는 중간 단계든, 종단 단계는 스트림 연산에 건네는 함수 객체는 모두 side effect가 없어야 한다. 

#### Example. Stream을 가장한 반복적 코드 
```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
  words.forEach(word -> {
    // 다른 상태를 변경하고 있다. 
    freq.merge(word.toLowerCase(), 1L, Long::sum);
  })
}
```
* 스트림 패러다임을 이해하지 못한 채 API를 사용한 코드다. 외부 상태를 수정하는 람다를 실행하면서 문제가 생긴다. forEach가 그저 스트림이 수행한 연산 결과를 보여주는 일 이상을 하는 것을 보니 code smell이 느껴진다. 
* forEach 연산은 for-each 반복문과 유사하게 생겼으며, 종단 연산 중 기능이 가장 적고 가장 '덜' 스트림답다. 대놓고 반복적이므로 병렬화할 수도 없다. 이는 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 사용하지 맞라. 

#### Example. Stream을 올바르게 활용할 경우 
```java
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
  freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

### Collectors (수집기)
* Stream을 올바로 사용하려면 수집기를 잘 알아둬야 한다. 
* 가장 중요한 수집기 팩터리는 `toList`, `toSet`, `toMap`, `groupingBy`, `joining`이다. 