# Comparable Comparator

코딩테스트 문제나 개발하면서 기본으로 제공되는 정렬로 해결이 안되는 경우가 많다.
그때 Comparable, Comparator 를 활용해 커스텀 정렬 기준을 만들어 사용할 수 있다.

참고로 Comparable, Comparator 는 인터페이스로 무조건 구현이 필요하다.

## Comparable
```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```
`Comparable` 클래스에는 `public int compareTo(T o)` 함수가 선언되어있어,
사용한다면 해당 함수를 구현해줘야 한다. 그리고 매개변수가 단 **한 개** 만`(T o)` 있다!
비교대상이 자기자신과 `(T o)` 이기 때문이다. 그래서 자기자신과 비교할 `o` 객체랑 **어떻게** 비교할지
`public int compareTo(T o)` 함수를 구현해주면 되는것이다.

```java
class Student implements Comparable<Student> {
    String name;
    int korean;
    int english;
    int math;

    public Student(String name, int korean, int english, int math) {
        this.name = name;
        this.korean = korean;
        this.english = english;
        this.math = math;
    }

    @Override
    public int compareTo(Student o) {
        if (this.korean != o.korean) {
            return o.korean - this.korean;
        }

        if (this.english != o.english) {
            return this.english - o.english;
        }

        if (this.math != o.math) {
            return o.math - this.math;
        }

        return this.name.compareTo(o.name);
    }

}
```
위의 `Student` 클래스를 보면 비교를 위해 `Comparable<Student>` 를 상속받아
`compareTo(Student o)` 를 구현하였다. 참고로 예제는
[백준 문제 10825](https://www.acmicpc.net/problem/10825) 의 정답을
가져왔다.


```java
Student student1 = new Student("a", 50, 60, 70);
Student student2 = new Student("b", 60, 60, 70);
Student student3 = new Student("c", 70, 60, 70);
int num = student1.compareTo(student2); // 10 을 반환한다.
```
여기서는 `student1` 국어점수가(`korean`) 50점으로 `student2` 대한 차이값 10 을 반환한다.
`student1` 을(본인) 기준으로 `student2`가 비교대상이 된다.

```java
if (this.korean != o.korean) {
            return o.korean - this.korean;
        }
```
참고로 `return o.korean - this.korean` 에서 `o.korean` 이 비교대상,
`this.korean` 이 본인이다.

### 주의사항
위의 예제처럼 단순히 본인과 비교대상의 차의 값을 리턴하면 코드로 간결하게
표현이 가능하지만, 리턴값이 `int` 자료형이기 때문에 이를 유의해야한다.
만약 본인 국어점수가 2,100,000,000 점, 비교 대상 점수가 -2,100,000,000점
이면 `this.korean - o.korean` 눈 4,200,000,000 가 결과값으로 나오면서
int 자료형을 초과해 오버플로우가 나면 음수로 값을 반환할 수 있다.

그러니 자료형과 값의범위를 꼭 확인하자!

## Comparator
```java
@FunctionalInterface
public interface Comparator<T> {

    int compare(T o1, T o2);
}
```
`Comparator` 의 경우 함수형인터페이스이며, `compare` 함수 외에도
여러 `default` `static` 함수들이 포함되어있다.

`Comparable` 와는 다르게 구현해야할 `int compare(T o1, T o2)` 의
매개변수가 2개 이다. `Comparable` 은 자기자신과 비교를 했지만, `Comparator`
의 경우 두 개의 객체를 넘겨주기에, 넘겨주는 두 개의 객체 서로가 비교대상이다.

```java
class Student implements Comparator<Student> {
    String name;
    int korean;
    int english;
    int math;

    public Student(String name, int korean, int english, int math) {
        this.name = name;
        this.korean = korean;
        this.english = english;
        this.math = math;
    }

    @Override
    public int compare(Student o1, Student o2) {
        if (o1.korean != o2.korean) {
            return o2.korean - o1.korean;
        }

        if (o1.english != o2.english) {
            return o1.english - o2.english;
        }

        if (o1.math != o2.math) {
            return o2.math - o1.math;
        }

        return o1.name.compareTo(o2.name);
    }
}
```
`Student` 클래스에서 `Comparator` 로 변경했다. `int compare(T o1, T o2)`
메소드는 본인을 기준으로 비교하는게 아닌 매개변수로 받은 두 객체끼리 비교를 해준다.
```java
Student student1 = new Student("a", 50, 60, 70);
Student student2 = new Student("b", 60, 60, 70);
Student student3 = new Student("c", 70, 60, 70);
int num = student3.compare(student1, student2); // 10 을 반환한다.
```
그래서 `student3.compare(student1, student2)` 에서는 `student1` 과
`student2` 를 비교해서 점수차이인 10 을 반환한다. 
`student3` 은 그냥 `Student` 클래스에 있는 `compareTo` 함수를 호출하기위해
사용했을뿐, 실제로 비교에는 아무런 영향을 끼치지 않는다.

근데 이러면 `student3` 는 사용할 필요도없는데 사용하지 않는가? 다른 사람이
보면 헷갈리게 할 수 있는 코드가 된다(쓰레기같은 코드다). 그래서 익명객체를 이용해서 구현해주자.

```java
class Test {
    public void test() {
        Comparator<Student> comparator = new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2) {
                if (o1.korean != o2.korean) {
                    return o2.korean - o1.korean;
                }

                if (o1.english != o2.english) {
                    return o1.english - o2.english;
                }

                if (o1.math != o2.math) {
                    return o2.math - o1.math;
                }

                return o1.name.compareTo(o2.name);
            }
        };
        
    }

    class Student {
        String name;
        int korean;
        int english;
        int math;

        public Student(String name, int korean, int english, int math) {
            this.name = name;
            this.korean = korean;
            this.english = english;
            this.math = math;
        }
    }
}
```
이런식으로 `Student` 클래스에 `Comparator` 구현하는게 아닌
익명 객체로 구현해두고 필요할때 써먹을 수 있게 만들 수 있다.

`Comparable` 과 다르게 `Student` 클래스에 구현하지 않아도 된다!
라는건 생각해보면 많은 이점이 있다. `Comparable` 은 `Student` 에 
단 하나의 기준만 세울 수 있지만, `Comparator` 를 활용하면 여러개의
익명객체를 만듬으로써 여러가지 기준에서 필요한것을 골라서 사용할 수 있다. 

```java
Student student1 = new Student("a", 50, 60, 70);
Student student2 = new Student("b", 60, 60, 70);
Student student3 = new Student("c", 70, 60, 70);
Student student = comparator.compare(student1, student2); // student1 을 반환한다.
```
`compare` 함수를 사용할때도 무의미한 `Student` 객체를 사용해서 호출하는게아닌
`Comparator` 익명 객체를 활용해 비교를 진행할 수 있다.

## 정렬

자바의 경우 기본적으로 오름차순으로 정렬을 해주지만, 특정 객체끼리 정렬을 하면 커스텀이 필요하다.
그래서 `Comparable` 의 `int compareTo(T o)` 함수와, `Comparator` 의 `int compare(T o1, T o2)` 함수를 이용해
정렬기준을 세우는데 해당 함수 반환값이 둘 다 `int` 이다.

리턴값이

**0, 음수 일 때** 교환하지 않는다.  
**양수 일 때** 교환한다.

`Comparable` 기준으로 예를 들면

`this.korean - o.korean` 은 오름차순으로 정렬이 된다.  
예를 들어 본인 국어점수가 50점, 비교 대상이 60점이면 결과가 -10 이므로 변경이
일어나지 않는다. 그러니 이런식으로 각각 비교하면 결국 50 60 70 ... 이런 순으로
오름차순으로 정렬이 된다.

`o.korean - this.korean` 은 내림차순으로 정렬이 된다.  
예를 들어 본인 국어점수가 50점, 비교 대상이 60점이면 10 이므로 변경이 생긴다.
그리고 본인 국어점수가 50점, 비교 대상이 40점이면 -10 이므로 변경이 생기지 않는다.
그러니 이런식으로 각각 비교하면 결국 70 60 50 으로 내림차순으로 정렬이 된다.
 
```java
List<Student> students = new ArrayList<>();
students.add(new Student("a", 50, 60, 70));
students.add(new Student("b", 60, 60, 70));
students.add(new Student("c", 70, 60, 70));

// Comparable 사용시
Collections.sort(students);

// Comparator 익명객체 사용시
Collections.sort(students, comparator);
students.sort(comparatro);
```

컬렉션에서  
`Comparable` 의 경우 `Collections.sort(students)` 를 사용해 정렬할 수 있다.
당연히 `Student` 클래스에 `Comparable` 을 구현하지않으면 에러가 난다.

`Comparator` 의 경우 `Collections.sort(students)` 을 해도 되지만
익명객체를 이용한다면 익명객체를 함께 넣어줘야한다. 그리고
`Comparable` 과 다르게 `Student` 클래스 리스트에서 `sort` 정렬을 익명객체를 넣어서도
사용할 수 있다.

배열의 경우에는
`Collections` 에서 `Arrays` 로 바꿔주면 된다.

## 마무리

마지막으로 해당 문서에 이용한 백준에서 풀었던 예제 코드를 올린다.
나는 `Comparable` 을 이용하여 풀었다. `Comparator` 가 객체지향 관점에서는 더 좋지만,
푸는 시간이 중요한 코딩테스트 같은경우 `Comparable` 이 더 간단하게 구현할 수 있다.

```java
import java.util.Collections;

public class Baekjoon10825 {

    public void solution() {
        int N = 0;
        List<Student> students = new ArrayList<>();
        try {
            BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
            StringTokenizer st;

            N = Integer.parseInt(br.readLine());
            for (int i = 0; i < N; i++) {
                st = new StringTokenizer(br.readLine());
                students.add(new Student(st.nextToken(),
                        Integer.parseInt(st.nextToken()),
                        Integer.parseInt(st.nextToken()),
                        Integer.parseInt(st.nextToken())));
            }

        } catch (Exception e) {

        }

        Collections.sort(students);
        StringBuilder sb = new StringBuilder();
        students.forEach(s -> sb.append(s.name).append(System.getProperty("line.separator")));
        System.out.println(sb);
    }
}

class Student implements Comparable<Student> {
    String name;
    int korean;
    int english;
    int math;

    public Student(String name, int korean, int english, int math) {
        this.name = name;
        this.korean = korean;
        this.english = english;
        this.math = math;
    }

    @Override
    public int compareTo(Student o2) {
        if (korean != o2.korean) { // 국어점수가 같지않으면
            return o2.korean - korean; // 국어 점수를 내림차순으로
        }

        if (english != o2.english) { // 국어점수가 같고 영어점수가 같지않으면
            return english - o2.english; // 영어 점수를 오름차순으로
        }

        if (math != o2.math) { // 국어점수가 같고 영어점수가 같고 수학점수가 같지않으면
            return o2.math - math; // 수학 점수를 내림차순으로
        }

        // 국어점수 같고 영어점수 같고 수학점수 같으면
        return name.compareTo(o2.name); // 이름 사전순으로 정렬
        // string 의 경우 기본적으로 compareTo 함수를 이용해 정렬할 수 있다.
    }
}
```

참고  
https://st-lab.tistory.com/243
