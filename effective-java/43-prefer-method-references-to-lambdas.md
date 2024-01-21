# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.21             |
| PART | 43. 람다보다는 메서드 참조를 사용하라  |

# 7장 Lambdas and Streams
* Java 8 - Functional interface, lambda, method reference 개념이 추가되면서 함수 개체를 더 쉽게 만들 수 있게 되었다. 
* 그리고 Stream API까지 추가되어 Data element의 시퀀스 처리를 라이브러리 차원에서 지원하기 시작했다. 
* 이 기능들을 효과적으로 사용하는 방법을 알아보자. 


## Item43. 람다보다는 메서드 참조를 사용하라

### Method Reference 특징
1. Lambda보다 간결하다. (매개변수가 늘어날수록 메서드 참조로 제거할 수 있는 코드양도 늘어난다.)
  * 일부 Lambda는 매개변수의 이름 자체가 더 좋은 가이드가 되기도 한다.
2. 람다로 할 수 없는 일이라면 메서드 참조로도 할 수 없다. 
  * 한 가지 유일하게 람다로는 불가능하지만 참조로는 가능한 예가 있다: 제네릭 함수 타입(generic function type)
3. 기능을 잘 드러내는 이름을 지어줄 수 있고 친절한 설명을 문서로도 남길 수 있다. 

#### Lambda가 더 나은 경우
```java
service.execute(GoshThisClassNameIsHumongous::action);
service.execute(() -> action());
```
* 이처럼 람다가 메서드 참조보다 간결할 때가 있다. 주로 메서드와 람다가 같은 클래스에 있을 때 그렇다. 이런 경우는 메서드 참조가 더 짧지도 명확하지도 않으므로 람다가 더 낫다.
  * e.g. `Function.identify()` 보다 `x -> x`가 더 낫다.

### Method Reference 유형 
1. `static method`를 가리키는 메서드 참조
    * `Integer::parseInt`
    * `str -> Integer.parseInt(str)`
2. instance method를 참조하는 유형 - 수신 객체(receiving object; 참조 대상 인스턴스)를 특정하는 한정적(bound) 인스턴스 메서드 참조
    * `Instant.now()::isAfter`
    * `Instant then = Instant.now();`
        `t -> then.isAfter(t)`
3. instance method를 참조하는 유형 - 수신 객체(receiving object)를 특정하지 않는 비한정적(unbounded) 인스턴스 메서드 참조 
    * 주로 스트림 파이프라인에서의 매핑과 필터 함수에 쓰인다.
    * `String::toLowerCase`
    * `str -> str.toLowerCase()`
4. 클래스 생성자를 가리키는 메서드 참조
    * `TreeMap<K,V>::new`
    * `() -=> new TreeMap<K,V>()`
5. 배열 생성자를 가리키는 메서드 참조
    * `int[]::new`
    * `len -> new int[len]`

### 정리
* 메소드 레퍼런스는 주로 람다보다 더 간결한 대안이다. 
* 메소드 레퍼런스가 더 짧고 분명하다면, 이를 사용하라. 그러나 그렇지 않다면 lambda를 사용하라.