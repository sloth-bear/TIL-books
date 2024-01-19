# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.19             |
| PART | 40. @Override 애너테이션을 일관되게 사용하라  |

# 6장 Enums and Annotations
* Java에는 특수한 목적의 참조 타입이 두 가지 있다:
  1. 클래스의 일종인 enum type
  2. interfacec의 일종인 annotation 
* 이 타입들을 올바르게 사용하는 방법을 알아보자. 


## Item40. @Override 애너테이션을 일관되게 사용하라

### `@Override`
1. 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달자. 
2. 예외 케이스는 한 가지뿐이다. 구체 클래스에서 상위 클래스의 추상메서드를 재정의할 때는 굳이 달지 않아도 된다. 컴파일러가 그 사실을 바로 알려주기 때문이다. 물론 재정의 메서드 모두에 일괄로 붙여두는 게 좋아보인다면 그래도 상관없다.

#### @Override 미사용으로 인한 버그 예시
```java
class Bigram {  
    private final char first;  
    private final char second;  
  
    public Bigram(char first, char second) {  
        this.first = first;  
        this.second = second;  
    }  
  
    public boolean equals(Bigram b) {  
        return b.first == first && b.second == second;  
    }  
  
    public int hashCode() {  
        return 31 * first + second;  
    }

	public static void main(String[] args) {
		Set<Bigram> s = new HashSet<>();  
		for (int i = 0; i < 10; i++)  
		    for (char ch = 'a'; ch <= 'z'; ch++)  
		        s.add(new Bigram(ch, ch));  
		  
		System.out.println(s.size()); // 260
	}
}
```
* `Set` 자료형을 사용했는데도 중복제거가 되지 않고 총 260개의 element가 저장되었다. 이유가 무엇일까?
* `equals`를 재정의한 것이 아니라 `overloading` 하였다. -> 매개변수 타입이 `Object`가 아니다. 결국 객체 식별성(identify)만을 확인하여 각각 서로 다른객체로 인식되었다.  
* `@Override`를 추가하면 컴파일 타임에 이 에러를 찾아낼 수 있었을 것이다.