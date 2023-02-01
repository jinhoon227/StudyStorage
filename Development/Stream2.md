# Stream 2편 중간연산

## 스트림 중간연산
스트림을 생성했다면 해당 스트림을 어떻게 가공할지 정해야합니다.
그래서 가공하는 여러가지 함수를 지원하는데 하나씩 소개해 드리겠습니다.
참고로 중간연산은 여러번 사용해도 문제가 없습니다!

```java
List<String> names = Arrays.asList("Eric", "Elena", "Java");
```
위의 List 를 기준으로 stream 예제를 보여드리도록 하겠습니다.


### Filtering
```java
Stream<T> filter(Predicate<? super T> predicate);
```
Predicate 는 boolean 형을 리턴해주는 함수형 인터페이스 입니다.
그래서 원하는 조건에 만족하는 값을 필터링할 수 있습니다.
```java
String
Stream<String> stream = names.stream()
        .filter(name -> name.contains("a")) // Elena, Java;
```
이름에 "a" 가 포함되는 모든 문자열을 필터링합니다.
만약 더 복잡한 조건을 원한다면 직접 커스텀할 수 있습니다.

```java
Stream<String> stream = names.stream()
                .filter(name -> {
                    if (name.contains("E") && !name.contains("r")) {
                        return true;
                    }
                    return false;
                }); // Elena
```
이름에 "E" 를 포함하고, "r" 를 포함하지 않는 이름으로 필터링을 해보았습니다.
해당 부분을 외부함수로 빼내면 좀 더 객체지향적으로 만들 수 있겠죠.


### Mapping
```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```
스트림의 값을 넣어서 다른 값으로 바꿔줍니다. 리턴값으로는 값을 변환한
새로운 스트림을 줍니다. 값을 변환하기위해서 람다를 인자로 받습니다.

```java
Stream<String> stream = 
  names.stream()
  .map(String::toUpperCase);
// [ERIC, ELENA, JAVA]
```
`map` 에 `String::toUpperCase` (람다)를 넣어주어 각각의 이름을
대문자로 변환시켜서 반환해줍니다.

그리고 좀 더 복잡한 `flatmap` 도 있습니다.
```java
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
```
mapper 에 값을 보시면 `? extends Stream<? extends R>>` 으로 
기존 `map` 과 다르게 `? extends Stream<>` 으로 한번 더 감싸주었다.
그런데 리턴값은 `map` 과 똑같이 `Stream<>` 입니다. 결과적으로 중첩된
구조를 한층 벗겨주는 역할을 합니다.

```java
List<List<String>> list =
        Arrays.asList(Arrays.asList("a", "1"), Arrays.asList("b", "2")); // [[a, 1], [b, 2]]

List<String> flatList = list.stream() // Stream<List<String>>
        .flatMap(s -> s.stream()) // Stream<String>
        .collect(Collectors.toList()); // [a, 1, b, 2] 
```
예제를 보면 이중리스트를 flatMap 을 이용해 하나의 리스트로 바꿔준것을
확인할 수 있습니다.

### Sorting
```java
Stream<T> sorted();
Stream<T> sorted(Comparator<? super T> comparator);
```
기본 `sorted` 함수는 오름차순으로 정렬해줍니다. 이 외의 정렬을
`comparator` 를 이용해서 정렬할 수 있습니다.

```java
names.stream()
  .sorted((s1, s2) -> s2.length() - s1.length())
  .collect(Collectors.toList());
//Elena, Eric, Java
```
`Comparator` 를 이용할 정렬 기준을 직접 작성해주시면 됩니다.

### Boxed
기본스트림을 다른 타입으로 변경해줍니다.
```java
IntStream.of(14, 11, 20, 39, 23) // IntStream
  .sorted() // IntStream
  .boxed() // Stream<Integer>
  .collect(Collectors.toList());
// [11, 14, 20, 23, 39]
```
`IntStream` 을 정렬을 해보았습니다. 그런데 `IntStream` 에는
`collect` 함수를 지원하지 않기때문에 `boxed` 를 이용해 `Stream<Integer>`
로 바꿔주어야 합니다. 이런 `IntSream, DoubleStream, LongStream` 형식
으로 지원하지 않는 함수가 있으면 `boxed` 를 사용해 타입을 바꾸면
사용할 수 있습니다.

### Peek
stream 의 각각의 연산을 대상으로 연산을 합니다.
결과값에 영향을 끼치지 않아 중간에 값을 확인할때 사용합니다.

하지만 결과값에 영향을 끼치지 않을뿐, 해당 함수를 이용해 
다른 리스트에 값을 추가하는 등 연산을 할 수 있습니다.
권장 드리는 방식은 아닙니다.
```java
int sum = IntStream.of(1, 3, 5, 7, 9)
  .peek(System.out::println)
  .sum();
```

### Distinct
요소내 중복값을 제거해줍니다.
```java
List<String> stringNumbers = Arrays.asList("1", "2", "3", "1");

List<String> distinctNumbers = stringNumbers.stream()
        .distinct()
        .collect(Collectors.toList()); // 1, 2, 3
```

### limit
결과값의 개수를 제한합니다.
```java
Stream<String> generatedStream =
        Stream.generate(() -> "g").limit(3); // [g, g, g]
```

### skip
첫번째 요소부터 설정값까지 건너띕니다.
```java
Stream<String> stream = names.stream()
        .skip(1); // "Elena", "Java"
```



출처
https://futurecreator.github.io/2018/08/26/java-8-streams/