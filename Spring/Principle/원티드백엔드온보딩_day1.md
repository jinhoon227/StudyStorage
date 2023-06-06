## Day1 - Java

원티드 백엔드 프리 온보딩 코스에서 김재녕님 강의를 들으면서 작성한 글이다.
JVM 과 메모리 구조, 클래스 로딩, Java 바이트코드와 JIT 컴파일러, GC 알고리즘, 스레드 동기화
에 대해 학습한다. JDK 17 기준으로 설명 한다.

## Java

- WORA : Write once, run anywhere => 좋은 이식성
- High-Level Language
- OOP ? => 순수 OOP 는 아니다. 자바는 원시타입(OOP 는 객체로써야함), 
정적메서드, 래퍼클래스(내부적으로 원시타입 사용함) 을 사용한다. 일부 절차적인 요소가 있다.

## Java Architecture

- JAVA 플랫폼 : SE, EE, ME 등 JDK 를 구현한 제품, 일반적으로 JDK 보다 넓은 의미로 쓰임
- JDK : Java Development Kit(JRE + Development Tools) 
- JRE : Java Runtime Environment(JVM + Library) : JDK 11 이후부터 JDK 는 JRE 를 포함해 JRE 가 따로 없다.
- JVM : 자바 가상 머신 - Java 바이트코드를 기계어로 변환하고 실행

## JVM

대표 기능 : 클래스로딩, GC 와 메모리관리, 스레드 관리, 예외처리를 한다.

클래스로더가 클래스파일 운영체제에 할당 받은 메모리(Runtime Data Areas) 에 적재시킨다.
메모리가 올라간것을 실행하는 Execution Engine 이 있다.
실행엔진은 Interpreter, JIT Compiler, Garbage Collector 로 구성되어있다.
Native Method Interface 는 Native Method Libraries 를 쓸 때 사용된다.

- Class Loaders(Class Loader Subsystem) : 바이트코드 로딩, 검증, 링킹 수행
- Runtime Data Areas : JVM 메모리 영역
- Execution Engine : 메모리 영역에 있는 데이터를 가져와 해당하는 작업 수행
- JNI(Java Native Interface) : JVM 과 네이티브라이브러리간 이진 호환성을 위한 인터페이스(로우 레벨 언어를 위함)
- Native Libraries : 네이티브 메서드 구현체를 포함한 플랫폼별 라이브러리

## Class Loaders

Java 파일이 Class 파일(자바바이트코드) 가 들어오면 수행된다.
자바 바이트코드를 동적으로 메모리에 로딩하는게 목적이다. 한 번에 모든 클래스가
메모리에 로드되지 않고 필요할 때마다 로드한다.

Loading -> Linking -> Initialization 순으로 이루어진다.

### Loading 

- Loading : Bootstrap Class Loader -> Execution(After Java 9 : Platform) Class Loader -> Application(System) Class Loader
- Linking : Verify -> Prepare -> Resolve
- Initialization : Initialization

- Bootstrap Class Loader : Java8 에서는 .jar 를 로드했지만 Java9 부터는 .base 를 로드 한다.
최상위 클래스 로더이자 네이티브 코드로 작성된 클래스 로더로 **base 모듈 로딩**, C/C++ 로 구현되어있어서 자바에서는 null 로 찍힌다.
- Platform Class Loader : Java SE platform 의 모든 API/클래스 로딩
- System Class Loader : Java 앱 레벨(우리가 구현한것들)의 클래스로딩(클래스패스, 모듈패스에 있는 클래스 로딩 -classpath, -cp 등 환경 변수 옵션 포함)

위임 : 로딩을 하기위해 찾기전에 먼저 상위클래스 로더에게 위임
System Class Loader 에서 못찾으면 Platform Class Loader 에서 찾고 못찾으면 
Bootstrap Class Loader 에서 찾는다. 찾으면 반환해준다. 못찾을시 `ClassNotFoundException` 발생

유일성 : 상위 클래스로더가 로딩한 클래스를 하위 클래스 로더가 다시 로딩하는것을 방지한다.

가시성 : 하위클래스 로더는 상위클래스 로더가 로딩한 클래스를 볼 수 있으나 반대는 안된다.

작동방식
1. 최하위 클래스 로더부터 클래스 찾는다.(System ClassLoader -> Platform classLoader -> Bootstrap classLoader) FQCN(Fully Qualified Class Name) 기반으로
로딩된 클래스(클래스명 : 패키지명 + 클래스명)인지 찾는다.
2. `java.lang.ClassLoader` 의 loadClass 메서드를 통해 클래스 로딩수행, 로딩된 클래스가있으면 반환 못찾으면 상위클래스로더에서찾음.
3. 상위 클래스가 해당클래스를 못찾을경우 하위클래스를 찾아서 로딩한다.
`ClassNotFoundException` 는 클래스를 못찾을때 발생, 
`NoClassDefFoundError` 는 클래스는 있는데 객체로드시 에러가 나면 발생한다.

### Linking

클래스를 클래스를 실행하기 위해서 결합하는 프로세스이다.

- Verification : 로딩된 바이트코드의 유효성 검증
- Preparation : 선언된 스태틱 필드를 초기화(초기값으로 초기화, 예를 들어 int 는 0, 객체 참조는 null), 필요한 메모리 할당
Run-Time Constant Pool 할당
- Resolution : 심볼릭 레퍼런스를 실제 참조(실제 주소값) 변환하는 작업

링킹은 새로운 자료구조를 메모리에 할당하기에 메모리 부족하면 Out of Memory 발생

Symbolic Reference ? 바이트코드에서 클래스/인터페이스/필드 등
참조하는 다른 요소를 표현 방식. 클래스가 로딩 후 링킹 되는 시점에서
심볼릭 레퍼런스가 실제 주소값으로 대체됨.(Resolution)

Run-Time Constant Pool ? Method Area 에 할당되는 자료구조, 스태틱 상수와 심볼릭 레퍼런스를 포함한다.
Method Area 에 저장하므로 메모리 초과시 OutOfMemory 발생한다.

### Initialization

Special Method 중 Class Initialization Methods 와 정적필드에 대한 초기화

Special Method : 인스턴스초기화메서드(생성자), 클래스 초기화 메서드, Signature Polymorphic Methods

1. 클래스/인터페이스 초기화 잠금 흭득 시도
2. 다른 스레드가 해당 클래스/인터페이스를 초기화하고 있다면 초기화잠금을 해제하고
호기화 완료될때까지 해당 스레드 차단 반복
3. 현재 스레드에서 초기화가 진행되고 있다면 초기화에 대해 재귀요청이어야함
이 경우 초기화잠금을 해제하고 정상적으로 완료해야함
4. 이미 초기화가 완료된 상태면 그대로 초기화잠금해제
5. .

## 바이트코드

Java 바이트코드는 Object Code 이다. Binary code 이지만 Machine code 는 아닌것을 말한다.

Java 바이트코드 - invokedynamic : invokedynamic -> Bootstrap Method -> CallSite, Method Handle
순으로 진행됨(invokedyanmic 은 주로 람다를 사용하면 발생한다.)

Bootstrap Method : JVM 메서드, MethodHandle 객체의 생성과 초기화를 담당. CallSite 를 링키하는데
사용되는 메서드

메타 팩터리 메서드 (LambdaMetafactory.metafactory) : 모든 람다의 부투스트랩메서드이며
CallSite 객체를 생성

CallSite : Bootstrap 메서드 호출 결과로 반환되는 객체이며 Method Handle 를 담아두는홀더

메서드 핸들 : 메서드를 동적으로 찾거나, 적용, 호출하기위한 저수준 메커니즘이며 해당
연산의 참조를 타입화한것. Java 컴파일러가 추가한 스태틱메서드(Method Area 에 위치하는
람다로 전달되는 메서드)를 참조

람다는 어떤 함수가 들어올지모르므로 Method Handle 로 담을 수 있는것을 만들어둬야한다.

### Java 코드캐시 

JIT 컴파일러가 쓰는영역, JAVA 바이트코드를 컴파일한
네이티브코드를 저장하는 메모리 영역

Java 코드캐시 세그먼트 : non-profiled, profiled

### AOT 컴파일러

JIT Compiler 중에 하나였다. Java 17 에서 없어졌다. GraalVM 에서 사용가능 하다.

run-time 전에 네이티브로 컴파일 해둔다.

부팅시간이적고, 메모리 사용량 최소화, 현대 API 서버아키텍처(MSA) 와 잘어울린다.

### JIT 컴파일 방식

run-time 중에 Hot Method 추적하여 컴파일한다. Hot Method 를 분석하는
Profiler 가 있다.
C1 과 C2 컴파일러로 이루어져있다.
C1 - 런타임에 바이트코드를 기계어로 컴파일
C2 - 

장점 : 런타임최적화

## Runtime Data Area

- Method Area : 클래스메타 데이터(클래스의 구조나 정보)
- Heap Area : 인스턴스화 된 객체데이터
- Stack : 쓰레드마다 생성된다. 쓰레드간 공유하지 않으며, 메소드를 호출할때마다
Stack Frame 저장공간을 만든다.
- PC Register : 쓰레드별로 만든다. 실행중인 명령(오프셋)을 저장
- Native Method Stack : 쓰레드별로 만든다.

### Execution Engine

JVM 메모리영역에 있는 바이트코드를 읽어 네이티브코드로 변환하고 실행
- Interpreter : 메모리에 로드된 바이트코드를 한줄 씩 해석/실행
- JIT Compiler : 자주 호출되는 메소드의 바이트코드를
- GC

컴파일러와 인터프리터
컴파일러 : 프로그래밍 언어로 작성된 코드를 타겟언어로 변환하는 프로그램
high-level 언어를 low-level 언어로 변환

인터프리터 : 읽은 코드 및 해당 명령을 직접 분석/실행한다.
코드 구문 분석, 동작을 직접 수행(즉시 실행), 컴파일러에 의해 생성된 바이트코드를 명시적으로 진행

JAVA 는 인터프리터와 컴파일러를 혼합해서쓴다. javac 로 소스코드를 바이트코드로
변환한다. 변환된 바이트코드를 JVM 인터프리터가 분석하고 실행한다.

## 과정

1. java 파일을 컴파일러(javac) 가 class 파일로변환
2. JVM 실행시 바이트코드(class) 에 필요한걸 클래스 로더가 로딩
3. 로딩된 클래스의 바이트코드를 JVM 의 실행엔진이 해석및 실행
4. 실행 준비가 완료되면 JVM 은 메인메서드(Entry Point) 호출
5. 호출된 메인메서드를 실행할 메인스레드가 생성되며 메인메서드의 JVM stacks 생성
6. 그 후 생성된 메인 스레드 JVM stacks 에 메인메서드 스택프레임이 생성
7. 앱이 실행되면 필요한시점마다 처리하며 메모리 확보및데이터저장(클래스
, 메서드 정보를 Method Area 에 저장 등)

## 질문

프로파일러 : 실행에 필요한 정보를 분석하는것을 프로파일을 저장하는것을 확인하는것
GC 튜닝 - 왠만하면 하지마라.. 튜닝하기전에 안하고도 해결이 가능한방법을 찾는게 좋다
