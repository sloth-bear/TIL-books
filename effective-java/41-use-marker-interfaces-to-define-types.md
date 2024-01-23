# Today I Learned

| 구분 | 내용                     |
| ---- | -----------------------|
| DATE | 2024.01.19             |
| PART | 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라  |

# 6장 Enums and Annotations
* Java에는 특수한 목적의 참조 타입이 두 가지 있다:
  1. 클래스의 일종인 enum type
  2. interfacec의 일종인 annotation 
* 이 타입들을 올바르게 사용하는 방법을 알아보자. 


## Item41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

### Marker interface란?
* 아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스 
* e.g. `Serializable` interface: 자신을 구현한 클래스의 인스턴스는 `ObjectOutputStream`을 통해 write 할 수 있다(즉, 직렬화할 수 있다)고 알려준다.

### Marker annotation vs Marker interface
* Marker annotation (Item 39)가 등장하면서 Marker interface가 구식이 된 것은 아니다.

#### Marker interface의 장점
* 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있으나, 애너테이션은 그렇지 않다. 컴파일 타임에 오류를 잡을 수 있다. (자바의 직렬화는 이 타입 이점을 잘 살리지 못했다. - 메서드에서 Object 타입으로 받는다.)
* 적용 대상을 더 정밀하게 지정할 수 있다. 
	* 인터페이스처럼 특정 인터페이스를 구현한 클래스에만 적용하고 싶어도 할 수 없다.

#### Marker annotation의 장점
* 거대한 애너테이션 시스템의 지원을 받는다. 따라서 애너테이션을 적극 활용하는 프레임워크에서는 마크 애너테이션을 쓰는 쪽이 일관성을 지키는 데 유리할 것이다. 


#### 사용
* marker annotation
	* 클래스, 인터페이스 외의 프로그램 요소(모듈, 패키지, 필드, 지역변수 등)에 마킹해야 할 때 
	* 마커를 애너테이션을 활발히 활용하는 프레임워크에서 사용하고자 할 때
* marker interface 
	* 이 마킹이 된 객체를 매개변수 로 받는 메서드를 작성할 경우가 있을 때

### 정리
* 마커 인터페이스
	* 새로 추가하는 메서드 없이 단지 타입 정의가 목적이라면 마커 인터페이스 선택
* 마커 애너테이션
	* 클래스, 인터페이스 외의 프로그램 요소에 마킹
	* 애너테이션을 활발히 활용하는 프레임워크의 일부로 그 마커를 편입시키고자 할 때 
	* ElementType.TYPE인 마커 애너테이션을 작성하고 있다면, 잠시 여유를 갖고 정말 애 너테이션으로 구현하는 게 옳은지, 혹은 마커 인터페이스가 낫지는 않을지 곰곰이 생각 해보자