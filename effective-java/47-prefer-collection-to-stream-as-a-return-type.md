# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.23             |
| PART | 47. 반환 타입으로는 스트림보다 컬렉션이 낫다.  |

# 7장 Lambdas and Streams
* Java 8 - Functional interface, lambda, method reference 개념이 추가되면서 함수 개체를 더 쉽게 만들 수 있게 되었다. 
* 그리고 Stream API까지 추가되어 Data element의 시퀀스 처리를 라이브러리 차원에서 지원하기 시작했다. 
* 이 기능들을 효과적으로 사용하는 방법을 알아보자. 


## Item47. 반환 타입으로는 스트림보다 컬렉션이 낫다.

### Streams do not make iteration obsolete
* Stream을 for-each로 반복할 수 없다. `Iterable`을 구현하지 않았기 때문이다. 
* 그럼에도 불구하고, Stream은 iteration을 쓸모 없게 만드는 것은 아니다. 좋은 코드는 stream과 반복문을 적절하게 잘 섞어 쓰는 것이다.
* 재미있는 점은 `Stream` interface는 `Iterable` 인터페이스가 정의한 추상 메서드를 전부 포함할 뿐만 아니라, `Iterable` interface가 정의한 방식대로 동작한다. 

#### Adapter from `Stream<E>`  to `Iterable<E>`
* 아래와 같은 어댑터를 사용하면 형변환하지 않고도 for-each문으로 반복할 수 있다. 
* 그러나 이러한 어댑터는 반복을 사용하는 게 더 자연스러운 상황에서도 이러한 어댑터를 사용해야 하고, 코드를 어수선하게 만들 수 있고 속도도 더 느리다. 
```java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
  return stream::iterator;
}

for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
  //...
}
```

#### Adapter from `Iterable<E>` to `Stream<E>`
* 마찬가지로 아래와 같은 어댑터를 사용하면 `Iterable`을 스트림 파이프라인으로 쉽게 처리할 수 있다.
```java
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
  return StreamSupport.stream(iterable.spliterator(), false);
}
```

#### public API 설계 시에는
1. sequence of objects를 반환할 메서드를 작성하는데, 오직 스트림 파이프라인 내에서만 쓰일 걸 안다면 스트림을 반환해도 괜찮지만(혹은 그 반대의 경우에 `Iterable`), 공개 API를 작성할 때는 양쪽 모두를 배려해야 한다. 
2. `Collection` interface는 `Iterable`의 하위 타입이고, `stream` 메서드도 제공하니 반복과 스트림을 동시에 지원한다. 
3. 따라서 원소 시퀀스를 반환하는 공개 API의 return type에는 `Collection`이나 그 하위 타입을 쓰는 게 일반적으로 최선이다. `Arrays` 역시 `Arrays.asList`와 `Stream.of` 메서드로 손쉽게 반복과 스트림을 지원할 수 있다. 
4. 반환하는 시퀀스의 크기가 메모리에 올려도 안전할 만큼 작다면, `ArrayList`나 `HashSet` 같은 표준 컬렉션 구현체를 반환하는 게 최선일 수 있다. 그러나 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안 된다.