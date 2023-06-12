## Day3 - GC 와 GC 알고리즘

원티드 백엔드 프리 온보딩 코스에서 김재녕님 강의를 들으면서 작성한 글이다.
이번 강의는 GC 와 GC 알고리즘에 대해 설명한다.

## GC

GC 는 Heap 영역을 관리한다. 프로그래머의 수동관리의 부담을 덜어준다.

단점 : 모든 객체에 대한 생성/삭제를 추적하므로 원래의 앱 사용 리소스보다 더 많은 성능 필요,
프로그래머가 객체 해제하는 CPU 스케줄링 불가, GC는 기능이 완벽하지 않아 런타임에 중지될가능성있음,
수동관리보다 비효율적(GC를 위한 연산과 메모리 필요)

- Reference : 객체가 다른 객체를 참조하는것
- Root Set : 특정한 객체를 포인팅한다.

Root Set 이 가리키는것과, 가리킨 객체가 참조하는 객체들을 마킹한다. 그 후 아무도 참조하지 않는
객체를 비운다. 그 후 이로 인해 생긴 메모리 단편화를 없앤다.

- Mark : 참조객체와 참조되지 않는 객체를 식별하는 과정
- Sweep : 마킹된 객체를
- Compact : 메모리 압축 작업, 메모리 단편화를 없애는 과정, 오래걸리는 작업이고 모든 작업을 전부 멈추고 해야될 수 도 있어서 하는 알고리즘도 있고
안하는 알고리즘도 있다.

기본적인 GC 단점
- 모든 객체를 스캔하고 마킹하고 메모리 압축하는 작업은 비효율적
- 시간이 지남에따라 더 많은 객체가 할당되면 시간이 오래걸림
=> Generations 방식 도입

### JVM Generations

Young Generation, Old Generation, Metaspace(과거 Permanent Generation) 이 있다. 

Young Generation : 모든 객체가 할당되는 영역이며 마이너 GC가 발생한다.
- Stop-the-world 이벤트 : 해당 작업이 완료될 때까지 모든 앱 스레드가 중지되는 행위(마이너 GC 는 항상 발생)
- Eden : 객체 인스턴스를 생성하면 할당된다.
- S0 : Eden 이 가득차면 참조객체는 S0 으로 이동된다.
- S1 : S0, Eden 이 가득차면 참조객체는 S1 으로 이동한다.
- 여기서 S1 이 가득차면 S0 으로 참조객체를 이동한다.

Old Generation : 메이저 GC 이다. 오랫동안 참조된 객체가 저장되는 영역, GC가 발생하면
큰 영역이기때문에 발생안시키는게 좋다. Young Generation 의 survivor 에서 aging 값이 임계값을 넘으면
오래쓸거라 판단하고 옮겨진다.

## Heap 을 제외한 GC 영역

정확히는 GC 가 처라하는게 아닌 영향을 끼치는 부분,
Heap 외에는 OS 와 JVM 이 처리한다.

- Metaspace : JDK 8이후, Metaspace 사용량이 최대가 되면 GC가 트리거가 되기도함.
- CodeCache : JIT 컴파일러가 Java 바이트코드를 컴파일한 네이티브 코드를 저장하는 메모리 영역,
Code Cache sweeper 스레드가 메모리를 관리한다.(위에 설명한 GC 와는 다르다)

## GC 알고리즘

- a parallel phase : GC가 멀티스레드실행가능
- a serial phase : GC가 싱글스레드실행
- a stop-the-world phase : GC가 앱 작업과 동시에 실행안됨
- a concurrent phase : GC가 백그라운드에서 실행되어 앱 작업과 동시에 실행될 수 있음
- an incremental phase : GC가 작업 완료전에 종료하고 나중에 이어서 작업 가능

### Serial GC

- 싱글스레드
- 서버 환경에는 적합하지 않음

### Parallel GC

- 멀티스레드 => STW 시간이 줄어듬

### CMS GC

기존 GC 과정을 쪼개서 최대한 STW 과정을 분리해서 작업한다.

Initial Mark 시 STW 발생, 그 후 앱이 실행되면서 지속적으로 Concurrent Mark(STW 발생안함),
이 후 Remark(STW 발생) 으로 마킹 마무리후 Concurrent Sweep(STW 발생안함) 으로 sweep 처리
- Initial Mark(STW) : GC 루트로부터 도달 가능한 객체를 마킹
- Concurrent Marking : Initial Marking 된 객체들에 그래프를 통해 추적
- Remark(STW) : 앞 단계에서 마킹이 누락된 객체 찾는 작업 수행
- Concurrent Sweep : 추적 불가능 객체 삭제

Compact 작업을 따로 수행하지 않아 메모리 단편화 문제 발생

### G1 GC

CMS GC 의 메모리 단편화 문제를 해결하기 위해나왔다. Heap 영역을 Region 크기(1MB ~ 32MB)로 2000개 정도로 쪼갠뒤
각 영역은 Eden, Survivor, Old Generation 이 존재한다.(가상메모리다)
G1 은 메모리 공간이 큰 멀티프로세스 머신에서 실행되는 앱을 기준으로 설계된 GC 이다.
많은 여유 공간을 생성할 수 있는 영역부터 처리한다고 하여 Garbage First 란 이름이 붙었다.

- Initial Mark(STW) : 일반적인 Young GC
- Root Region Scanning : 앱이 실행되는 동안 Old Generation 을 위해 survivor region 스캔
- Concurrent Marking : 앱이 실행되는 동안 전체 힙에서 Live 객체 찾음
- Remark(STW) : 
- Cleanup(STW) :
- Copying(STW) :

### Shenandoah GC

G1 처럼 Heap 영역을 분리하여 저장한다.

- Init Mark(STW) : 마킹 진행, 루트 셋 스캔
- Concurrent Marking : 앱 실행과 동시에 힙 내에 객체를 추적
- Final Mark(STW) : 중단되었던 마킹/업데이트 큐를 비우고
루트셋을 다시 스캔해 Concurrent Marking 작업 완료
- Concurrent Cleanup : 마킹작업후 Live 객체 없는 region 회수
- Concurrent Evacuation : 컬렉션 셋을 다른 region 으로
동시에 이동(복사) 하는 단계(다른 GC 와 차이점)
- Init Update Refs(STW) : 객체들의 참조업데이트 초기화하고 모든 GC와
앱 스레드가 제거 작업 완료했는지 확인
- Concurrent Update References : concurrent evacuation 작업 중 이동된
객체에 대한 참조 업데이트 단계(다른 GC 와의 차이점)
- Final Update Refs(STW) : 기존에 루트 셋을 업데이트하여 참조 업데이트 단계를 완료하는 단계
- Concurrent Cleanup : 이 작업은 참조가 없는 컬렉션 셋 region 회수

G1 과 같이 region 으로 영역을 나눴으나 generation 으로 나누지 않았다.
앱과 동시에 더 많은 GC 작업을 수행해 일시 중지 시간을 줄였다.
Compact 과정을 G1 GC 에서는 STW 를 발생해서 했으나
앱이 실행중에 Compact 과정을 수행하기위해 Brooks Pointer 를 추가하여 참조 재배치를 하였다.

### ZGC

64비트 이상에서 사용이 가능하다.

특징 Relocation : 메모리 단편화로 인해 재배치가 필요한데 힙이 클수록 재배치가 느리다.
실행 중에 재배치를 하기위해 로드배리어를 활용한다.(STW 시간을 줄인다.)

Marking : 마킹시 marked0, marked1 등 메타데이터 비트를 사용
1. 루트 참조 객체(지역변수, 스태틱필드) 마킹(STW)
2. 루트로부터 참조되는 객체마킹, 로드배리어에서 감지한 마킹되지않은 참조들도 마킹
3. weak 참조 경우 등 처리(STW), Strong 참조 등이 없는지 확인해서 마킹처리
4. 미참조 클래스 언로딩 재배치 셋 선택

Reference Coloring : 참조란 가상 메모리의 바이트위치를 표시한 것을 의미

Relocation
1. 블록을 찾으면서 재배치 셋에 넣으며 재배치 처리
2. 재배치 셋의 모든 루트 참조를 재배치하고 해당 참조의 상태 업데이트(STW)
3. 남아있는 재배치셋의 참조를 모두 재배치, 이전/새 주소 매핑을 포워딩 테이블에저장
4. 재배치가 되어있으나 마킹이 되지않은 참조는 다음 마킹 작업때 새위치로 업데이트됨

Remapping & Load Barriers : 앱에서 해당 참조된 객체를 로딩할 때 로드 배리어가 트리거 되어 올바른 참조를 반환
이를 통해 객체에 액세스할 때마다 최신 상태의 참조를 반환하도록함 이로 인해 참조를 로딩할 때마다 성능저하 유발할 수 있으나 일시 중지 시간을 줄이기위한 트레이드오프
1. remap 비트가 1이면 최신값이기에 안전하게 반환가능
2. 그 후 참조된 객체가 재배치 셋에 포함되어있지 않으면 remap 비트 1로 설정, 업데이트된 참조반환
3. 해당 참조가 재배치 대상이기 때문에 실제로 재배치 이뤄졌는지 확인


낮은 지역속도를 가져 대기 시간이 짧은 앱에 적합하다. 컬러포인터를 지닌
로드배리어를 통해 스레드가 실행시 동시작업을 한다.
shenandoah GC 는 참조객체를 브룩스포인터, ZGC 는 참조 컬러를 사용한다.
ZGC 가 더 큰 앱에서 최적화되어있다.

### Epsilon GC

메모리를 할당하지만 실제로 회수하지 않는 GC 다. 사용가능한 메모리를 모두 사용하면 프로그램이 종료된다.
=> 테스팅을 할 때 사용한다.

## GC 선택

- SerialGC : 단일프로레서, 앱의 일시 중지 시간이 크게 상관없는경우
- ParallelGC : 일시 중이 허용 시간이 1초 이상의 경우
- G1GC : 응답시간이 전체 처리량보다 중요하며 GC로 인한 일시중지 시간을 짧에 유지

## GC 고려 사항

- Heap Size : 클수록 GC 의 시간이 길어지고 작을수록 GC 가 자주일어난다.
- Application Data set size : 핸들링하는 데이터 크기
- Number of CPUs
- Pause Time
- Throughput
- Memory Footprint 
- Promptness
- Java Version
- Latency : 지연시간이 얼마까지 가능한가

## 덤프 확인툴

클라우드 사용시 : 클라우드콘솔워치(돈으로 때려박아..)

스카우터, 데이터록, 유렐릭?

서버가 먹통되거나 느려지거나 데드락걸리거나 이럴때 덤프를 찍어보자.

## Reference

아래 단계로 내려갈수록 GC의 대상될 확률이 높다. 참조수준을 다루는 클래스
- StrongReference
- SoftReference : OutOfMemoryError 발생 전에 GC에 의해 제거됨
- WeakReference 
- PhantomReference