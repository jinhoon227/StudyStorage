
## 작성계기

개발 중간에 단위테스트를 진행하고 있었다. 그런데 `Store` 라는 엔티티에는 수 많은 필드가 작성되어있고,
내가 테스트할려는 로직에는 그 수 많은 필드의 값을 다 채울 필요가 없었다. 그런데 `int` 자료형인 필드의 경우
DB 에 들어갈시 `null` 로 들어가면 에러가 발생한다. 그래서 기본값을 설정해주는 `@ColumnDefault`
를 필드에 붙여주면 해결이 될 줄 알았으나.. 실제로는 그렇지 않았다. 이를 해결하기 위해서 `@DynamicInsert`
를 사용하는데 이에 대해 자세히 서술할려고 한다.

## @ColumnDefault 에 대해

먼저 `@ColumnDefault` 에 대해 이야기하자면, DDL 처음 생성시 기본값을 채워주는 롬복이다.
```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class StoreTopItem {

    @Id @GeneratedValue
    @Column(name = "store_id")
    private Long id;
    
    private String firstGradeItemName;
    private String secondGradeItemName;
    private String thirdGradeItemName;

    @ColumnDefault("0")
    private int firstGradeItemNumber;
    @ColumnDefault("0")
    private int secondGradeItemNumber;
    @ColumnDefault("0")
    private int thirdGradeItemNumber;
    
    public StoreTopItem(String firstGradeItemName){
        this.firstGradeItemName = firstGradeItemName;
    }
}
```
해당 클래스는 가게에 많이 판매된 상품의 수를 저장하는 엔티티다.
상품의 수를 저장하는 필드 `firstGradeItemNumber` 의 경우 자료형이
`int` 이기 때문에 기본값 없이 생성하면 에러가 난다. 그래서 `@ColumnDefault("0")` 롬복으로 기본값은 0 으로 저장하도록 했다.

`StoreTopItem` 엔티티를 실제로 DB 에서 **생성**할때는 다음과 같은 쿼리가 날라간다.
```sql
CREATE TABLE store_top_item (
                                store_id BIGINT NOT NULL AUTO_INCREMENT,
                                first_grade_item_name VARCHAR(255),
                                second_grade_item_name VARCHAR(255),
                                third_grade_item_name VARCHAR(255),
                                first_grade_item_number INT DEFAULT 0,
                                second_grade_item_number INT DEFAULT 0,
                                third_grade_item_number INT DEFAULT 0,
                                PRIMARY KEY (store_id)
);
```
sql 쿼리에서 `first_grade_item_number INT DEFAULT 0` 로 기본값은 0으로 세팅되도록 된다.
그래서 기본값이 0 으로 알아서 세팅되니까 문제가 없을 줄 알았는데.. 그게 아니었다.

## JPA 에서 Insert 와 Update

JPA 에서는 Insert 와 Update 할 때 기본전략으로 **모든 엔티티의 필드를 업데이트** 한다.
예를 들어 `StoreTopItem` 에서 생성자로 `firstGradeItemName` 값만 넣어주고 객체를 만든 뒤 DB에 저장을 한다고 해보자.
저장을 할 때에는 DB에 Insert 쿼리가 날라가는데 다음과 같은 쿼리가 날라간다.
```sql
INSERT 
INTO
    store_top_item
    (id, first_grade_item_name, second_grade_item_name, third_grade_item_name, first_grade_item_number, second_grade_item_number, third_grade_item_number) 
VALUES
    (DEFAULT, "ThisIsfirstGradeItemNameValue", null, null, null, null, null) // 모든 칼럼에 대한 쿼리 생성
```

실제로 날리는 쿼리를 보면 실제로 값을 저장한 `firstGradeItemName` 필드에 대한 Insert 가 아닌 모든 필드에 대해 Insert 를 해준다.
당연히 `firstGradeItemName` 외에는 데이터를 넣어두지 않았기때문에, 다른 필드는 `null` 값으로 Insert 를 한다! 어쨋든 `null` 값으로
Insert 를 해버렸으니 `first_grade_item_number` 의 `DEFAULT 0` 이 작동하지 않고 `null` 값으로 저장된다. DB 에서 `int` 자료형은
`null` 값으로 저장이 가능하다. Update 는 구체적인 예제를 들지않았지만 마찬가지로 모든 필드에 대해 Update 를 한다.

### @ColumnDefault("0") 이 안됬던 이유

나는 `@ColumnDefault("0")` 를 사용해 `first_grade_item_number` 에는 `0` 값이 들어가있는줄 알았다. 하지만 Insert 할 때 `null` 로 넣어져서 실제로 DB 에서는 
`null` 값이 저장되어 있었다. `null` 값이 저장된것 까지는 괜찮다. 하지만 테스트를 하기위해서 `StoreTopItem` 엔티티를 다시 가져오면 DB 에 있는 `first_grade_item_number` 의 `null` 값이
JAVA 에서 `first_grade_item_number` 은 `int` 자료형에 맵핑이 되지 않아서 오류가 생긴다. JAVA 의 `int` 는 `null` 저장이 불가능
하기 때문이다.

### JPA 의 기본전략

어쨋든 문제는 JPA 의 하이버네이트가 모든 필드를 업데이트 하기때문에 생긴 문제다. 그래서 왜 모든 필드를 업데이트 하는데? 라는 의문이 들 수 있다.
모든 필드를 업데이트하면 위와같은 오해? 가 생길 수도 있고 무엇보다 모든 필드를 보낸다는것은 그만큼 더 많은 데이터를 주고받는 비효율적인 상황이다.

그렇지만 나보다 뛰어난 분들이 만든 JPA 시스템에 그런것도 고려하지 않았을리가 없으니 찾아보니 이유가 있었다.
1. 모든 필드를 사용하면 바인딩 되는 데이터만 다를 뿐 등록, 수정 쿼리가 항상 같다. 따라서 애플리케이션 로딩 시점에 쿼리를 미리 생성해두고 재사용할 수 있다.
2. 데이터베이스에 동일한 쿼리를 보내면 데이터베이스는 이전에 한 번 파싱된 쿼리를 재사용할 수 있다.

정리하자면 Insert 와 Update 에서 동일한 구조의 쿼리를 보낸다. 따라서 JPA hibernate 와 DB 에서 동일한 구조를 재사용함으로써
최적화가 가능하다.

## 해결법 첫번째 : @DynamicInsert, @DynamicUpdate

첫번째 해결방법으로느 JPA 의 기본전략을 적용시키지 않는것이다. `@DynamicInsert`, `@DynamicUpdate` 를 사용하면 
모든 필드로 업데이트하지 않고 필요한 부분만 업데이트 한다. 예제와 함께 설명을 하겠다.

```java
@Entity
@Getter
@DynamicInsert
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class StoreTopItem {

    @Id @GeneratedValue
    @Column(name = "store_id")
    private Long id;
    
    private String firstGradeItemName;
    private String secondGradeItemName;
    private String thirdGradeItemName;

    @ColumnDefault("0")
    private int firstGradeItemNumber;
    @ColumnDefault("0")
    private int secondGradeItemNumber;
    @ColumnDefault("0")
    private int thirdGradeItemNumber;
    
    public StoreTopItem(String firstGradeItemName){
        this.firstGradeItemName = firstGradeItemName;
    }
}
```

적용방법은 간다하다. 그냥 엔티티에 `@DynamicInsert` 롬복을 달아주면 된다. 업데이트에 적용하고 싶으면 `@DynamicUpdate` 를 추가하면 된다.

이전에 예시를 들었던것처럼 `StoreTopItem` 에서 생성자로 `firstGradeItemName` 값만 넣어주고 객체를 만든 뒤 DB에 저장을한다.
엔티티에 `@DynamicInsert` 가 있으면 저장을 할 때 DB에 Insert 쿼리가 다음과 같이 날라가게된다.
```sql
INSERT 
INTO
    store_top_item
    (id, first_grade_item_name) 
VALUES
    (DEFAULT, "ThisIsfirstGradeItemNameValue")
```

필요한 값만 Insert 쿼리를 날려준다. 그러면 `first_grade_item_number` 에 대한 값은 Insert 하지 않았으므로 기본값을 0 으로 설정해둔것이
정상적으로 적용이 된다. 그래서 DB 에는 `0` 으로 저장이 된다. 그러면 `@ColumnDefault("0")` 가 의도한대로 적용이 되었다.

## 해결법 두번째 : @PrePersist

```sql

@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class StoreTopItem {

    @Id @GeneratedValue
    @Column(name = "store_id")
    private Long id;
    
    private String firstGradeItemName;
    private String secondGradeItemName;
    private String thirdGradeItemName;


    private Intger firstGradeItemNumber;

    private Intger secondGradeItemNumber;

    private Intger thirdGradeItemNumber;
    
    @PrePersist
    public StoreTopItem(String firstGradeItemName){
        this.firstGradeItemNumber = this.firstGradeItemNumber == null ? 0 : this.firstGradeItemNumber;
        this.secondGradeItemNumber = this.secondGradeItemNumber == null ? 0 : this.secondGradeItemNumber;
        this.thirdGradeItemNumber = this.thirdGradeItemNumber == null ? 0 : this.thirdGradeItemNumber;
    }
}
```

`@DynamicInsert` 를 사용하지 않고 `@PrePersist` 를 사용하면 `StoreTopItem` 엔티티가 `persist` 되기 전에 실행된다.
이렇게하면 Insert 되기전에 `firstGradeItemNumber` 값이 `null` 이면 `0` 으로 저장한다. 그리고 `firstGradeItemNumber` 의
자료형이 `int` 에서 `Integer` 로 바뀌었다. `int` 는 `null` 값을 저장하지 못하기 때문이다.

## @DynamicInsert VS @PrePersist

`@DynamicInsert` 와 `@PrePersist` 어떤것이 좋을까? 

둘 다 장단점이 있다. 전자의 경우 필요한 데이터한 삽입, 수정을 하기때문에 전송량이 적다. 그래서 엔티티의 필드가 엄청많은데 한 두개만
삽입, 수정을 한다면 성능측면에서 이점을 볼 수 있다.

후자의 경우 JPA 의 기본전략을 그대로 가져갈 수 있다. 쿼리 구조가 같다는 이점인데 일반적인 상황에서는 성능이 더 좋다고 생각된다.

나같은 경우에는 `@DynamicInsert` 를 채택했는데 그 이유는 다음과 같다.
- 예제로는 `StoreTopItem` 을 들었지만, 실제로 적용할 엔티티는 필드가 많기에 일부 필드만 업데이트할시 `@DynamicInsert` 가 성능상 좋다고 판단했다.
- 실제로 적용할 엔티티에서, `@ColumnDefault` 를 적용할 필드는 엔티티에 있는 필드말고도 값타입(@Embeddable) 에 있는 필드도 적용해야되는데 값타입에 `@PrePersist` 를 적용 할려면 매우 번거롭다.
- `int` 자료형에서 `@PrePersist` 를 적용할려면 `null` 값을 넣을 수 없으므로 `Integer` 자료형을 사용해야되는데 `Integer` 는 성능 저하를 유발한다.
- `@DynamicInsert` 를 사용하는편이 매우 간편하다. 그리고 간편함이라는것은 가독성 또한 좋다는것을 뜻한다.

## 후기

JPA 의 기본전략으로 전체필드를 업데이트한다는건 일반적으로 예측하기 어려웠다. 그리고 JPA 의 기본전략이 항상 성능이 좋은것도 아니었다.
JPA 를 다루면서 좋은 성능을 내고싶다면 이런 기본전략부터 시작해서 여러가지를 고려해봐야 겠다고 느겼다. 결론은 JPA 공부를 하자.. 아는만큼 보인다.



## Reference

https://velog.io/@choidongkuen/JPA-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-DynamicInsert-DynamicUpdate-%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B4%85%EC%8B%9C%EB%8B%A4  
https://dotoridev.tistory.com/6  
https://gaemi606.tistory.com/entry/JPA-DynamicInsert-DynamicUpdate-%EC%97%94%ED%8B%B0%ED%8B%B0-%EB%A6%AC%EC%8A%A4%EB%84%88  
https://mangchhe.github.io/jpa/2021/09/06/EntityDynamicQuery/  

