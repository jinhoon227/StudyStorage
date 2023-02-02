# Stream 3편 최종연산
중간연산에서 어느 정도 값을 원하는 방식으로 바꾸고
최종적으로 결과값을 반환해주는 연산입니다. 중간연산과 다른점이 있다면
최종연산은 마지막 단 한번만 사용할 수 있습니다.

```java
List<String> names = Arrays.asList("Eric", "Elena", "Java");

List<Product> productList =
        Arrays.asList(new Product(23, "potatoes"),
        new Product(14, "orange"),
        new Product(13, "lemon"),
        new Product(23, "bread"),
        new Product(13, "sugar"));

public class Product {

    int amount;
    String name;

    public Product(int amount, String name) {
        this.amount = amount;
        this.name = name;
    }

    public int getAmount() {
        return amount;
    }

    public String getName() {
        return name;
    }
}
```
위의 List 를 기준으로 stream 예제를 보여드리도록 하겠습니다.

### Calculating
기본형타입에는 여러가지 연산함수들이 있습니다. 
대부분 Long 반환인것을 주의해주세요.
```java
long count = IntStream.of(1, 3, 5, 7, 9).count(); // 5
long sum = LongStream.of(1, 3, 5, 7, 9).sum(); .// 25
```

평균, 최소, 최대의 경우 `stream` 이 비어있을때는 표현할 수 없으므로
`Optional` 로 리턴해줍니다.
```java
OptionalInt min = IntStream.of(1, 3, 5, 7, 9).min();
OptionalInt max = IntStream.of(1, 3, 5, 7, 9).max();
```
또는 `ifPresent` 를 이용해 `Optional` 로 값을 안받고 바로 처리해
줄 수 있습니다.
```java
DoubleStream.of(1.1, 2.2, 3.3, 4.4, 5.5)
  .average()
  .ifPresent(System.out::println); // 3.3
```

### Reduction
기본타입이 아닌 경우 `reduce` 이용해 값을 만들 수 있습니다.
* accumulator : 각 요소를 처리하는 계산 로직. 각 요소가 올 때마다 중간 결과를 생성하는 로직.
* identity : 계산을 위한 초기값으로 스트림이 비어서 계산할 내용이 없더라도 이 값은 리턴.
* combiner : 병렬(parallel) 스트림에서 나눠 계산한 결과를 하나로 합치는 동작하는 로직.

```java
// 1개 (accumulator)
Optional<T> reduce(BinaryOperator<T> accumulator);

// 2개 (identity)
T reduce(T identity, BinaryOperator<T> accumulator);

// 3개 (combiner)
<U> U reduce(U identity,
  BiFunction<U, ? super T, U> accumulator,
  BinaryOperator<U> combiner);
```

1개의 인자만 넘겨올경우 `accumulator` 만 사용하는데 해당 자료형이
`BinaryOperator<T>` 인데 해당 함수형 인터페이스는 같은 타입의 2개의
매개변수를 받아 하나의 매개변수로 반환해 줍니다.

```java
Optional reduced = names.stream()
                        .reduce((a, b) -> {
                            return a+b;
                        }); //Optional[EricElenaJava]
```
람다식으로 조금 더 짧게 표현이 가능하지만 `(a,b) -> a+b` 이해가 더 잘되도록
길게 표현했습니다.

두 개의 인자를 넘겨주면 초기값 설정이 가능합니다.
```java
        String reduced =names.stream()
                        .reduce("Start",
                                (a, b) -> {
                            return a+b;
                        }); //StartEricElenaJava
```
그리고 초기값을 넘겨줌으로써 빈값이 생길 수 없기에 해당 자료형으로
반환을 해줍니다.

마지막 3개의 인자를 넘겨줄 경우 각자 다른 쓰레드에서 연산을 수행 후
마지막에 합쳐줍니다. 그래서 병렬스트림에서만 동작을 합니다.
```java
String paralleReduced = names
                .parallelStream()
                .reduce("Start",
                        (a, b) -> {
                            return a + b;
                        },
                        (a, b) -> {
                            return a + b;
                        }); // StartEricStartElenaStartJava
```
병렬로 Start+Eric, Start+Elena, Start+Java 로 각각 수행후
마지막으로 StartEric + StartElena + StartJava 연산을 수행했습니다.

### Collecting
Collectors.toList()
```java
/**
 * 리스트로 값 가져오기
 */
List<String> collectorCollection =
  productList.stream()
    .map(Product::getName)
    .collect(Collectors.toList());
// [potatoes, orange, lemon, bread, sugar]
```

Collectors.joining()
```java
/**
 * 스트링 값 연결하기
 */
String listToString = productList.stream()
      .map(Product::getName)
      .collect(Collectors.joining());
// potatoesorangelemonbreadsugar

/**
 * 인자를 3개를 넘겨줄시 prefix, sufix 를 설정할 수 있다.
 */
String listToString = productList.stream()
        .map(Product::getName)
        .collect(Collectors.joining("|", "<", ">"));
// <potatoes|orange|lemon|bread|sugar>
```

Collectors.averagingInt()
```java
Double averageAmount = productList.stream()
  .collect(Collectors.averagingInt(Product::getAmount)); // 17.2
```

Collectors.summingInt()
```java
Integer summingAmount = productList.stream()
  .collect(Collectors.summingInt(Product::getAmount)); // 86

/**
 * mapToInt 를 사용하면 IntStream 으로 바뀌어 해당 함수를 이용할 수 있습니다.
 */
Integer summingAmount = productList.stream()
        .mapToInt(Product::getAmount)
        .sum(); // 86
```

Collectors.summarizingInt()
```java
IntSummaryStatistics statistics = productList.stream()
  .collect(Collectors.summarizingInt(Product::getAmount))

// IntSummaryStatistics {count=5, sum=86, min=13, average=17.200000, max=23}
```

Collectors.groupingBy()
```java
Map<Integer, List<Product>> collectorMapOfLists = productList.stream()
  .collect(Collectors.groupingBy(Product::getAmount));

/**
 *  amount 를 기준으로 값을 그룹핑(같은 리스트로 묶어서) 맵형식으로 반환해줍니다.
 */
{
    23=[Product{amount=23, name='potatoes'}, Product{amount=23, name='bread'}],
    13=[Product{amount=13, name='lemon'}, Product{amount=13, name='sugar'}],
    14=[Product{amount=14, name='orange'}]
}
```

Collectors.partitioning()
```java
Map<Boolean, List<Product>> mapPartitioned = productList.stream()
  .collect(Collectors.partitioningBy(el -> el.getAmount() > 15));

/**
 *  groupingBy 와는 다르게 조건을 만족하는지 true, false 로 구분해서 반환합니다.
 */
{
    false=[Product{amount=14, name='orange'}, Product{amount=13, name='lemon'}, Product{amount=13, name='sugar'}],
    true=[Product{amount=23, name='potatoes'},Product{amount=23, name='bread'}]
}
```

Collectors.collectingAndThen()
```java
/**
 * collect 를 한 후 추가로 할 작업에 있을 때 사용합니다.
 * 아래 예제는 Set 로 바꾸고, 수정불가능하게 만들어 주었습니다.
 */
Set<Product> unmodifiableSet = productList.stream()
        .collect(Collectors.collectingAndThen(Collectors.toSet(),
        Collections::unmodifiableSet));
```

Collector.of()  
만족스로운 Collector 함수가 없다면 직접 만들 수 도 있습니다.
```java
public static<T, R> Collector<T, R, R> of(
  Supplier<R> supplier, // new collector 생성
  BiConsumer<R, T> accumulator, // 두 값을 가지고 계산
  BinaryOperator<R> combiner, // 계산한 결과를 수집하는 함수.
  Characteristics... characteristics) { ... }
```

```java
Collector<Product, ?, LinkedList<Product>> toLinkedList =
                Collector.of(LinkedList::new, // Collector 를 생성하기위해 LinkedList 생성자를 넘겨줍니다.
                        LinkedList::add, // 생성된 리스트에 요소를 추가합니다. 병렬로 진행되기 때문에 5개의 연결리스트가 만들어집니다.
                        (first, second) -> { // 마지막연산으로 각각의 연결리스트를 합쳐 하나의 연결리스트가 만들어집니다.
                            first.addAll(second);
                            return first; 
                        });
```

```java
LinkedList<Product> linkedListOfPersons = productList.stream()
                .collect(toLinkedList);

// 결과적으로 하나의 연결리스트가 반환됩니다.
Product{amount=23, name='potatoes'}
Product{amount=14, name='orange'}
Product{amount=13, name='lemon'}
Product{amount=23, name='bread'}
Product{amount=13, name='sugar'}
```

### Matching
```java
// 하나라도 만족하면 true
boolean anyMatch(Predicate<? super T> predicate);

// 모두 만족해야 true
boolean allMatch(Predicate<? super T> predicate);

// 모두 만족하지 않아야 true
boolean noneMatch(Predicate<? super T> predicate);
```

```java
List<String> names = Arrays.asList("Eric", "Elena", "Java");

boolean anyMatch = names.stream()
  .anyMatch(name -> name.contains("a"));
boolean allMatch = names.stream()
  .allMatch(name -> name.length() > 3);
boolean noneMatch = names.stream()
  .noneMatch(name -> name.endsWith("s"));

// 결과는 전부 true 입니다.
```

### Iterating
```java
names.stream().forEach(System.out::println);
```

### find
```java
// findFirst 는 첫번째 요소를 반환합니다.
Optional find1 = names.stream()
        .filter(name -> name.contains("E"))
        .findFirst(); // Eric
        
// findAny 는 병렬로 작업 처리시 먼저 탐색되는것을 반환하기때문에 결과값이 바뀔 수 있습니다.
Optional find2 = names.stream().parallel()
        .filter(name -> name.contains("E"))
        .findAny(); // Eric 또는 Elena 가 출력됩니다.
```

출처
https://futurecreator.github.io/2018/08/26/java-8-streams/