# Today I Learned

| 구분 | 내용                      |
| ---- | ------------------------|
| DATE | 2023.12.30              |
| PART | 7. 다 쓴 객체 참조를 해제하라 |

# 2장 객체 생성과 파괴
* 객체를 만들어야 할 떄와 만들지 말아야 할 때를 구분하는 방법
* 올바른 객체 생성 방법
* 불필요한 생성을 피하는 방법
* 제때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업을 관리하는 요령 

## Item7. 다 쓴 객체 참조를 해제하라

### Java와 메모리 관리
자바에서는 메모리를 직접 관리하지 않고 가비지 컬렉터가 대신해준다. 가비지 컬렉터가 다 쓴 객체를 알아서 회수해간다. 그러나 메모리 관리를 더 이상 신경 쓰지 않아도 된다는 것은 아니다. 

### 다 쓴 객체 참조를 해제하지 않으면? 메모리 누수 예제
```java
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  ...
  public Object pop() {
    if (size == 0) {
      throw new EmptyStackException();
    }
    return elements[--size];
  }
}
```

#### 문제점
* 이 스택을 사용하는 프로그램을 오래 사용하다보면 점차 가비지 컬렉션 활동과 메모리 사용량이 늘어나 결국 성능이 저하될 것이다. 드물지만 심할 경우 디스크 페이징이나 OutOfMemoryError를 일으켜 프로그램이 예기치 않게 종료될 수 있다. 
* 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다. 스택이 그 객체들의 다 쓴 참조(obsolete reference)를 여전히 가지고 있기 때문이다. 

#### 보완
메모리 누수 문제를 해결하기 위해서는 객체의 다 쓴 참조를 해제해야 된다. 
```java
public Object pop() {
  if (size == 0) {
    throw new EmptyStackException();
  }
  elements[size] = null;
  return elemetns[--size];
}
```

* 가비지 컬렉션 언어에서는 (의도치 않게 객체를 살려두는) 메모리 누수를 찾기 아주 까다롭다. 객체 참조 하나를 살려두면 가비지 컬렉터는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체(그리고 또 그 객체들이 참조하는 모든 객체)를 회수해가지 못한다. 잠재적으로 이는 성능에 악영향을 준다. 
* ***가장 좋은 방법은 객체 참조를 담은 변수를 scope 밖으로 밀어내는 것이다.* 예외 상황에서만 null 처리를 해야 한다. (여기서 Stack은 배열로 저장소 풀을 만들어 원소를 직접 관리하므로 null 처리를 했다.)** 변수의 범위를 최소로 지정했다면 자연스럽게 이루어질 것이다. (Item 57)

### 메모리 누수 주의 필요한 상황들
#### 1. 자기 메모리를 직접 관리하는 클래스 
원소를 다 사용한 즉시 그 원소가 참조한 객체들을 null 처리한다. (위의 Stack 예제 참조)

#### 2. 캐시
객체 참조를 캐시에 넣고 잊은 채 그 객체를 다 쓴 후로도 한참을 그냥 두는 경우 메모리 누수가 일어난다. 해결 방법은 크게 3가지가 있다.

1. WeakHashMap - 캐시 외부에서 key를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 경우 
```java
public class CacheWithWeakHashMap {
  private final Map<Object, Object> cache = new WeakHashMap<>();

  public void addToCache(final Object key, final Object value) {
    cache.put(key, value);
  }

  public Object getFromCache(Object key) {
    return cache.get(key);
  }
}
```

외부에서 해당 객체에 대한 참조가 사라지면 자동으로 해당 객체는 가비지 컬렉션의 대상이 된다.

2. 시간 기반 Cache 청소 
시간이 지나면서 캐시 엔트리의 가치가 떨어지는 경우, 주기적으로 또는 특정 조건에서 캐시를 청소할 수 있다. 

* 백그라운드 스레드 활용
```java
public class CacheCleaner {
  private final Map<Object, Object> cache = new ConcurrentHashMap<>();
  private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);

  public CacheCleaner() {
    scheduler.scheduleAtFixedRate(() -> cache.clear(), 60, 60, TimeUnit.SECONDS);
  }
  ...
}
```

* 캐시에 새 엔트리 추가 시 부수 작업으로 처리 
```java
public class CacheWithCleanupOnAdd {
  private static final int MAX_ENTRY_SIZE = 100;
  private final Map<Object, Object> cache = new LinkedHashMap<>(MAX_ENTRY_SIZE + 1, .75F, true) {
    @Override
    protected boolean removeEldestEntry(Map.Entry<Object, Object> eldestEntry) {
      return size() > MAX_ENTRY_SIZE;
    }
  }

  public void addToCache(Object key, Object value) {
    cache.put(key, value);
    // 또는 여기서 추가적인 청소 로직 구현
  }
  ...
}
```

#### 3. Listener / Callback 
* 클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는 경우 - 조치 전까지 콜백이 쌓인다. 
* 콜백을 약한 참조(weak reference)로 저장하면 가비지 컬렉터가 즉시 수거해간다. 
