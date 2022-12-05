# chapter6. 스트림으로 데이터 수집

---

**chapter6의 핵심** ✨

컬렉터가 무엇이고 어떻게 활용하는지 알아보자!

--- 

<br>

## 컬렉터? 🤔

함수형 프로그래밍은 명령형 프로그래밍과 달리 how보단 what 즉, '무엇'을 원하는지 직접 명시할 수 있다. 5장에서 toList를 배웠던 걸 기억해 보자! 😳 이 toList도 하늘에서 뚝 떨어진 게 아니라 Collector 인터페이스의 구현체이다. 

우리는 컬렉터의 장점을 활용해 앞으로 toList처럼 Collector 인터페이스의 구현체를 넘겨서 결과를 수집하는 과정을 간단하면서도 유연한 방식으로 정의할 수 있다.

🤦‍♀️ Collector와 Collectors... 뭐가 다른 거지?

책에서 자꾸 둘을 혼용해서 쓰길래 헷갈려서 추가한다.
+ Collector : Collector 인터페이스
https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collector.html

+ Collectors : Collector 인터페이스를 구현한 클래스
https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html

<br>

### 팩토리 메서드 🛠

groupingBy 같은 Collectors 클래스에서 제공하는 미리 정의된 컬렉터라고 생각하면 된다.

Collectors에서 제공하는 메서드의 기능은 세 가지로 구분할 수 있다.

+ 스트림 요소를 하나의 값으로 리듀스하고 요약

+ 요소 그룹화

+ 요소 분할

<br>

## 리듀싱과 요약

이전 장에서 컬렉터로 스트림의 모든 항목을 하나의 결과로 합칠 수 있다는 건 알고 있겠지요? ~~모르면 돌아가야 됩니다...~~

이제 메뉴 예제를 활용해서 Collector 팩토리 클래스로 만든 컬렉터 인스턴스로 어떤 일을 할 수 있는지 알아보자! 

<br>

### 1. 메뉴에서 요리 수 계산하기

👩‍🍳 : 우리 메뉴에서 요리 수가 몇 개였더라...? 



수를 셀 수 있는 **counting**이라는 팩토리 메서드를 활용할 수 있다.

```java
long howManyDishes = menu.stream().collect(Collectors.counting());
```

이미 Collctors 클래스의 정적 팩토리 메서드를 모두 import 했다고 가정하면 더 간단하게 구현할 수 있다.

```java
long howManyDishes = menu.stream().count();
```

<br>

### 2. 스트림에서 최댓값과 최솟값 검색

👩‍🍳 : 칼로리가 가장 높은 요리를 알고 싶어! 또는 가장 낮은 요리도!



**Collectors.maxBy, Collectors.minBy** 메서드를 이용해서 스트림의 최댓값과 최솟값을 계산할 수 있다.

```java
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);

Optional<Dish> mostCalorieDish = menu.stream().collect(maxBy(dishCaloriesComparator));
```

<br>

#### 🙋‍♀️ 엥 근데 Optional은 뭐죠...?

이 예제로 봤을 때 만약 menu가 비어있다면 어떤 요리도 반환되지 않을 것이다. 나중에 더 자세히 배울 예정이라 자바8이 값을 포함하거나 또는 포함하지 않을 수 있는 컨테이너 Optional을 제공한다고만 간단하게 알아두면 된다.

<br>

### 3. 요약 연산

스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산에도 리듀싱 기능이 자주 활용되는데 이러한 연산을 **요약**연산이라고 한다.



👩‍🍳 : 메뉴 리스트의 총 칼로리를 알고 싶어요!



**Collectors.summingInt**라는 요약 팩토리 메서드로 쉽게 구할 수 있다.

**summingLong, summingDouble**도 같은 방식으로 동작하되, 다루는 형식만 다르다고 이해하면 된다.

```java
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

<br>

이미지를 보며 어떻게 요약 연산이 작동하는지 알아보자. 

칼로리로 매핑된 각 요리의 값을 탐색하면서 초깃값으로 설정되어 있는 누적자에 칼로리를 더하고 있다.

![image](https://user-images.githubusercontent.com/62419307/205480888-2a2963a2-93f0-43b3-b3ae-3b318f811a2b.png)

<br>

평균값 계산이 필요할 때는 **averagingInt, averagingLong, averagingDouble** 등을 활용할 수 있다.

```java
double avgCalories = menu.stream().collect(averagingInt(Dish::getCalories));
```

<br>

평균값이나 합계 등 두 개 이상의 연산을 한 번에 수행하고 싶을 땐 **summarizingInt**를 활용할 수 있다. 마찬가지로 타입별로 활용할 수 있는 팩토리 메서드가 다양하다.

```java
IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));
```

<br>

### 4. 문자열 연결

👩‍🍳 : 메뉴의 모든 요리명들을 연결해 주세요~!



**joinging** 팩토리 메서드를 이용해 스트림의 각 객체의 toString 메서드를 호출해서 추출한 모든 문자열을 하나의 문자열로 연결할 수 있다.

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining());
```

<br>

이때, Dish 클래스가 요리명을 반환하는 toString 메서드를 포함하고 있으면 더 간단하게 코드를 작성할 수 있다.

```java
String shortMenu = menu.stream().collect(joining());
```

<br>

👩‍🍳 : 엥 저기요 근데 메뉴가 너무 뭉텅이로 나오는데요...?



다행히! 구분 문자열을 넣을 수 있도록 오버로드된 joining 팩토리 메서드도 있다.

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));
```

<br>

#### 😵 오잉 근데 reduce로도 할 수 있지 않나?

넵... 사실 그렇다... 그렇지만 의미론적인 문제가 발생한다.

+ collect : 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드

+ reduce : 두 값을 하나로 도출하는 불변형 연산

더 자세한 내용은 7장에서 확인할 수 있다. 😂

<br>

## 그룹화

명령형 프로그래밍과 달리 자바 8의 함수형을 이용하면 간단하게 그룹화를 구현할 수 있다.

계속해서 메뉴 예제를 통해서 그룹화에 대해 알아보자.

다음은 요리 타입에 따라 메뉴를 그룹화한 코드이다.

```java
Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));
```

위의 코드에서 확인할 수 있듯이 groupingBy 메서드를 기준으로 스트림이 그룹화되므로 이를 **분류 함수**라고 한다.

<br>

![image](https://user-images.githubusercontent.com/62419307/205483052-d4ca5ec9-66b4-4ef1-9b4c-fac22453b6f7.png)

참고로, 단순한 속성 접근자 대신 더 복잡한 분류 기준이 필요한 경우에는 직접 구현을 해야 한다.

<br>

### 1. 그룹화된 요소 조작

👩‍🍳 : 메뉴 타입별로 그룹화한 뒤, 500 칼로리가 넘는 요리들만 알려줘!



그러면 음! 프레디케이트를 적용하는 방법을 사용하면 되겠다는 생각이 번쩍 든다.

```java
Map<Dish.Type, List<Dish>> caloricDishesByType = menu.stream().filter(dish -> dish.getCalories() > 500)
                                                              .collect(groupingBy(Dish::getType));
```

그렇지만 필터 프레디케이트를 만족하는 요리 타입이 없으면 아예 결과 맵에서 해당 키 자체가 사라지는 문제가 발생한다... 🙃

<br>

그래서 Collector 형식의 두 번째 인수를 갖도록 groupingBy 팩토리 메서드를 오버로드해서 문제를 해결해야 한다.

```java
menu.stream().collect(
        groupingBy(Dish::getType,
            filtering(dish -> dish.getCalories() > 500, toList())));
```

이렇게 코드를 작성하면 아래와 같이 500 칼로리가 넘는 요리가 존재하지 않더라도 해당 타입의 키는 그대로 살릴 수 있다.

```
{OTHER = [french fries, pizza], MEAT = [pork, beef], FISH = []}
```

<br>

항목을 조작하는 또 다른 방법으로 mapping 메서드를 사용할 수 있다.

예를 들어 이 함수를 이용해 그룹의 각 요리를 관련 이름 목록으로 반환이 가능하다.

```java
Map<Dish.Type, List<String>> dishNameByType = menu.stream()
                                                  .collect(groupingBy(Dish::getType, mapping(Dish::getName, toList()));
```

<br>

### 2. 다수준 그룹화

두 인수를 받는 팩토리 메서드 Collectors.groupingBy를 이용해서 항목을 다수준으로 그룹화할 수 있다.

👩‍🍳 : 메뉴 타입별로 그룹화한 뒤, 조건에 맞는 요리를 또 필터링해줘!

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = 
        menu.stream()
            .collect(
                    groupingBy(Dish::getType, // 첫 번째 수준의 분류 함수
                            groupingBy(dish -> {  // 두 번째 수준의 분류 함수
                                if(dish.getCalories() <= 400)
                                    return CaloricLevel.DIET;
                                else if (dish.getCalories() <= 700)
                                    return CaloricLevel.NORMAL;
                                else
                                    return CaloricLevel.FAT;
                            })
                    )
};	
```

그러면 아래와 같은 결과가 도출된다.

```
MEAT={DIET=[chicken], NORMAL=[beef], FAT=[pork]},
FISH={DIET=[prawns], NORMAL=[salmon]},
OTHER={DIET=[rice, seasonal fruit], NORMAL=[french fries, pizza]}
```



![image](https://user-images.githubusercontent.com/62419307/205484283-de4203b2-8a80-4151-9b42-b8dd600cf87c.png)

<br>

### 3. 서브 그룹으로 데이터 수집

groupingBy로 넘겨주는 컬렉터의 형식은 제한이 없기 때문에 서브 그룹으로도 그룹화할 수 있다. 

👩‍🍳 : 메뉴에서 요리의 수를 종류별로 계산해줘!

```java
Map<Dish.Type, Long> typesCount = menu.stream().collect(groupingBy(Dish::getType, counting()));
```

```
{MEAT=3, FISH=2, OTHER=4}
```

<br>

### 4. 컬렉터 결과를 다른 형식에 적용하기

collectingAndThen으로 컬렉터가 반환한 결과를 다른 형식으로 활용할 수 있다.

```java
Map<Dish.Type, Dish> mostCaloricByType =
            menu.stream()
                    .collect(groupingBy(Dish::getType, // 분류 함수
                            collectingAndThen(
                                    maxBy(comparingInt(Dish::getCalories)), // 감싸져있는 컬렉터
                                    Optional::get))); // 변환 함수
```

![image](https://user-images.githubusercontent.com/62419307/205484816-75c1d007-cda5-4132-9cc4-8273cfc70aaf.png)

<br>

## 분할

분할은 **분할 함수**라 불리는 프레디케이트를 분류 함수로 사용하는 특수한 그룹화 기능이다. 

분할 함수는 boolean을 반환하므로 결과적으로 그룹화 맵은 최대 두 개의 그룹으로 분류된다.

분할 함수가 반환하는 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지할 수 있다는 장점이 있다.

<br>

👩 : 채식주의 친구를 초대하고 싶어! 모든 요리를 채식 요리와 일반 요리로 분류해줘!

```java
Map<Boolean, List<Dsh>> partitionedMenu =
            menu.strem().collect(paritioningBy(Dish::isVegetarian));


List<Dish> vegetDishes =
        menu.stream().filter(Dish::isVegetarian).collect(toList());
```

```
{false=[pork, beef, chicken, prawns, salmon],
true=[french fries, rice, season fruit, pizza]}
```

<br>

## Collector 인터페이스

Collector 인터페이스는 리듀싱 연산(컬렉터)을 어떻게 구현할지 제공하는 메서드 집합으로 구성된다. Collector 인터페이스를 직접 구현해서 커스텀 컬렉터를 만들 수도 있다.



### 1. supplier 메서드 : 새로운 결과 컨테이너 만들기

supplier는 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수로, 빈 결과로 이루어진 Supplier를 반환해야 한다.

```java
public Supplier<List<T>> supplier() {
    return () -> new ArrayList<T>();
}
```

<br>

### 2. accumulator 메서드 : 결과 컨테이너에 요소 추가하기

accumulator 메서드는 리듀싱 연산을 수행하는 함수를 반환한다.

```java
public BiConsumer<List<T>, T> accumulator() {
    return (list, item) -> list.add(item);
}
```

<br>

### 3. finisher 메서드 : 최종 변환값을 결과 컨테이너로 적용하기

finisher 메서드는 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환하면서 누적 과정을 끝낼 때 호출할 함수를 반환해야 한다. 누적자 객체가 이미 최종 결과일 경우 항등 함수를 반환한다.

```java
public Function<List<T>, List<T>> finisher() {
    return i -> i;
}
```

<br>

### 4. combiner 메서드 : 두 결과 컨테이너 병합

combiner는 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할지 정의한다.

```java
public BinaryOperator<List<T>> combiner() {
    return (list1, list2) -> {
      list1.addAll(list2);
      return list1;
    };
}
```

<br>

### 5. Characteristics 메서드

Characteristics는 위 4개의 메서드와 달리 collect 메서드가 어떤 최적화를 이용해서 리듀싱 연산을 수행할 것인지 결정하도록 돕는 힌트 특성 집합을 제공한다.

+ UNORDERED : 리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않는다.

+ CONCURRENT : 다중 스레드에서 accumulator 함수를 동시에 호출할 수 있으며 이 컬렉터는 스트림의 병렬 리듀싱을 수행할 수 있다.

+ IDENTITY_FINISH : 리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용할 수 있다.

<br>

## 정리

1. collect는 스트림의 요소를 요약 결과로 누적하는 다양한 방법(컬렉터)을 인수로 갖는 최종 연산이다.

2. Collector 인터페이스는 최댓값, 최솟값, 합 등 다양한 팩토리 메서드를 제공한다.

3. 컬렉터는 다수준의 그룹화, 분할, 리듀싱 연산에 적합하게 설계되어 있다.


