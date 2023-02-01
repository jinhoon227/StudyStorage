# Stream 1편 생성하기

## 1. 스트림이란?
자바8 에서 지원하는 기능이며 람다와 함께 사용이 가능합니다. 스트림을 활용하면
더욱더 객체지향적인 코드를 작성할 수 있고, 코드가 간결해집니다.

하지만 성능적인 측면에서는 오히려 안좋을 수 있습니다. 실제로 코딩테스트
에서도 스트림을 사용하면 시간초과가 나는 경우가 있습니다. 또한
디버그하기가 매우 힘듭니다. 
그러니 목적에 맞게 잘 사용하는것이 좋습니다.


스트림은 크게 3가지 과정으로 진행됩니다.

1. 생성하기 : 스트림 인스턴스 생성
2. 중간연산 : 원하는 결과값으로 만들기 위한연산(filter 등)
3. 최종연산 : 원하는 형태의 결과값을 반환하기 위한 연산(인트형 반환, 리스트로 반환, 맵으로 반환 등)

## 2. 생성하기

### 배열 스트림
```java
String[] arr = new String[]{"a", "b", "c"};
Stream<String> stream1 = Arrays.stream(arr);
Stream<String> stream2 = Stream.of("a", "b", "c"); // 가변인자 사용
```

### 컬렉션 스트림
```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
```

### Stream.generate
```java
/**
 * generate 사용시 Suplier<T> 함수형 인터페이스를 사용합니다
 * 간단히 소개하자면 인자는 없으나 리턴값을 반환하는 함수입니다.
 * 주의사항으로 generate 는 값을 무한히 만들므로 limit 연산을 이용해 개수를 제한해줘야 합니다.
 */
Stream<String> generatedStream =
        Stream.generate(() -> "g").limit(3); // [g, g, g]
```

### Stream.iterate()
```java
/**
 * iterate 사용시 특정 규칙이 있는 반복되는 값을 편하게 만들 수 있습니다.
 * 주의사항으로 generate 와 같이 값을 무한히 만들므로 limit 연산을 이용해 개수를 제한해줘야 합니다.
 */
Stream<Integer> iteratedStream =
         Stream.iterate(30, n -> n + 2).limit(3); // 30 32 34
```

### 기본타입 스트림
```java
/**
 * 제네릭 타입을 사용하지 않고 직접 해당타입(int, long, double) 스트림을 생성할 수 있습니다.
 * 대부분 범위 함수가 그렇듯 range(Inclusive, Exclusive) 입니다.
 * 하지만 rangeClosed(Inclusive, Inclusive) 입니다.
 */
IntStream intStream = IntStream.range(1, 5); // [1, 2, 3, 4]
LongStream longStream = LongStream.rangeClosed(1, 5); // [1, 2, 3, 4, 5]
```

### 문자열 스트림
```java
IntStream charsStream = 
  "Stream".chars(); // [83, 116, 114, 101, 97, 109]

IntStream charsStream =
    "Stream".codePoints(); // [83, 116, 114, 101, 97, 109]
```

### 병렬 스트림
```java
/**
 *  Fork/Join framework 를 사용하여 쓰레드를 처리하는 병렬스트림 입니다.
 */
Stream<Product> parallelStream = productList.parallelStream(); // 컬렉션
Arrays.stream(arr).parallel(); // 배열
IntStream intStream = IntStream.range(1, 150).parallel(); // 기본타입

// 되돌리기
IntStream intStream = intStream.sequential();
```

### 스트림 연결하기
```java
Stream<String> stream1 = Stream.of("Java", "Scala", "Groovy");
Stream<String> stream2 = Stream.of("Python", "Go", "Swift");
Stream<String> concat = Stream.concat(stream1, stream2);
// [Java, Scala, Groovy, Python, Go, Swift]
```

출처  
https://futurecreator.github.io/2018/08/26/java-8-streams/