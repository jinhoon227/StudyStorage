## Day2 - JVM 메모리구조

원티드 백엔드 프리 온보딩 코스에서 김재녕님 강의를 들으면서 작성한 글이다.
JVM 구조에 대해 설명한다.

## Run Time Data Areas

Run Time Data Area 는 Method Area, Heap Area, Stack, PC Register, Native Method Stack 으로 구성되어있다.

생성시점
- JVM 실행시 : Heap, Method Area
- 스레드 실행시 : Pc register, Java Virtual Machine Stacks, Native Method Stacks(필요시)
- 클래스/인터페이스 생성시 : Run-time constant pool

main 메서드가 실행이되면 main 메서드에 대해 Call Stack 에 쌓인다.
Stack Memory 에 지역변수(원시값), 참조값이 저장된다. 
Heap Space 에 객체값이 참조하는 실제 값들이 저장된다.


### Method Area

JVM 실행시 생성되어 모든 스레드에게 공유되며 클래스별 구조와 정보(메타데이터)
저장하는 영역(클래스, 인터페이스, 인스턴스초기화에 사용되는 스페셜메소드, 런타임 상수 풀, 필드, 메서드데이터, 생성자코드, 메서드코드)

메스드 영역의 크기는 상황에 따라 고정, 확장, 축소가 가능하며
공간이 부족할시 OutOfMemoryError 발생

#### Run-Time Constant Pool

메소드영역에 할당된다. 일반 상수풀의 데이터기반으로 생성된다.(스태틱 상수, 심볼릭 레퍼런스)
런타임 시 상수만 포함되고 컴파일시 상수는 포함안된다.

### Heap

모든 객체 인스턴스, 배열에 대한 메모리에 할당되는 데이터 영역이다.
JVM 실행시 생성되고 JVM 스레드에 공유되는 영역이다. GC 의 처리대상 영역이다.

메스드 영역의 크기는 상황에 따라 고정, 확장, 축소가 가능하며
공간이 부족할시 OutOfMemoryError 발생

### Native Method Stacks

JAVA 외에 언어로 작성된 메서드(일반적으로 C/C++)

메스드 영역의 크기는 상황에 따라 고정, 확장, 축소가 가능하며
더 많은 스택이 필요시 StackOverFlowError 발생한다.
메모리가 부족하면 OutOfMemoryError 발생한다.

## 메모리 모델

멀티스레드 환경에서 데이터를 공유하기 위해 메모리를 사용하는 방법에 대한 명세

필수로 알아야될 것 : Shared Multiprocessor Architecture
=> Core1 의 L2 캐시에서 데이터를 조작했는데 RAM 에 저장된 데이터가
최신화가 안되서 다른 Core2 에서 RAM 을 읽어서 최신화가
유지안된 데이터를 읽어버리는 경우가 있다.

메모리 일관성 : 멀티스레드에서 하나의 데이터에 접근할 때, 일관적인
상태를 보장하는것

메모리 가시성 : 멀티스레드 환경에서 한 스레드에서 변경한 값을
다른 스레드에서 언제 보게 될지 정의한것

final : 불변성을 가지는 상수이기에 스레드에서 안전한다.
추가로 L2 캐시에 가져와서 사용하고 메모리를 업데이트 해줄필요가 없어서 좋다.

volatile : 레지스터(캐시)가 아닌 메인 메모리로부터 값을 읽고 쓰도록한다.
하지만 멀티쓰레드에서 원자성을 보장해 주지않는다.

메모리 누수 : 필요하지 않는 메모리를 계속 참조하여 해제를 하지 못해서
발생하는 누수(리소스 낭비) => 시스템 성능 저하시키는 치명적인 에러!

GC 를 통해 메모리 관리는 되지만 완벽하지 않다.
참조중이지만 사용하지 않는 객체의 경우 GC 가 제거할 수 없다.

누수 예시
- 스태틱 필드 객체 : 사용하지 않아도 참조되기에 누수 발생할 수 있다.
=> 정말 필요한 스태픽 필드만 사용하자, 싱글톤 객체 등을 구현할때는 lazy loading 기법으로 구현하자.
- 사용 리소스 해제 하지않아서 : 리스소 할당해놓고 사용않으면 해제하자
- equals, hashCode 메서드 올바르게 구현하지 않을시 : hashCode 를
활용하는 HashSet, HashMap 등에서 불필요하게 참조하게된다.
- 아우터클래스를 참조하는 이너클래스(non-static) 로 발생 : 이너클래스를
사용하기 위해 항상 아우터 클래스를 참조하기 때문에 아우터 클래스에 대한
누수수가 발생할 수 있다. 그러니 아우터클래스를 참조하지 않는 경우
이너클래스를 static 클래스로 구현할것
- ThreadLocal 클래스 : 명시적으로 값을 제거하지 않으면 참조가
유지되어 누수가 발생, 값 비울때 `ThreadLoacal.set(null)` 사용하지말것(실제로 메모리 해제되는게 아님)


메모리누수 모니터링 : VisualVM, YourKit 프로파일러 활용,
참조 객체를 통해 메모리누수방지(java.lang.ref 패키지에 참조 객체 활용),
벤치마크 활용

메모리 최적화 TIP
- 컬렉션 사용시 : 가능하면 초기 사이즈 지정해 초기화,
컬렉션별 CRUD 시간 복잡도 확인해서 의도와 맞게 사용,
HashMap 사용시 리사이징 일어나면서 해시값 재조정 하니 이를 고려할것
- 가능하면 특정 객체 재사용(메모리 사용량, GC 빈도 등) :
불변 객체나 캐싱 형태의 구현을 활용해 볼 것(스레드세이프 이기에 
함수형이나 멀티스레딩환경에 좋음) 불변 객체를 새로 만드는
비용이 더 크지않냐? 하지만 멀티스레드에서는 이게 더 낫음
- 박싱/언박싱 피할것
- Stream API 사용시 메모리 더 사용하거나 성능이 느릴 수 있으니 주의할 것

### Thread Dump

자바 프로세의 모든 스레드 상태에 대한 스냅샷

스레드덤프 방법
- jstack : 쉽고빠름
- JMC : 프로파일링에 따라 붙는 성능 오버헤드를 최소화한 프로파일링 도구
- jvisualvm
- jcmd
- jconsole

API 통신하면 상용 툴 쓸것

### Heap dump

JVM 메모리 상에 있는 모든 객체의 스냅샷

방법
- jamp
- VisualVM
- OutOfMemoryError 발생 시 자동 생성

관리
- Heap 크기 지정
- Metaspace 크기 지정(JDK 8부터 안됨)
- GC 알고리즘선택

APM VS Profiling : Profiling 은 개발자가 성능문제
최적화하는데 사용, APM 은 더 높은 수준