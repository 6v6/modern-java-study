# chapter5. 스트림 활용

**Stream API 가 지원하는 다양한 연산을 살펴보자**

<br>

## 스트림을 통해
- 외부 반복 to 내부 반복 (데이터 컬렉션 etc.)
- 데이터를 처리하는 구체적인 방식을 개발자가 아닌 stream api 가 관리되고 숨겨짐 == **추상화**
    - ex) 내부적으로 다양한 최적화 기법/병렬 실행 기법


<br>

## 필터링

### Predicate 필터링
- Predicate
    - 참거짓을 반환하는 함수

```java
IntStream.of(1,2,2,3,4).stream()
                .filter(num->num<3) //일치하는 모든 요소 스트림 반환
                .collect(Collectors.toList())
```

### 고유 요소 필터링

- 객체의 hashCode, equals 로 같은지 판단한 뒤 중복제거 후 고유 요소만 필터링

```java
IntStream.of(1,2,2,3,4).stream()
                .filter(num->num<3)
                .distinct()
                .collect(Collectors.toList())
```

<br>

## 슬라이싱

- 엄청 큰 컬렉션 이나 무한 stream 에서도 사용가능

### takeWhile()
- 정렬된 리스트에서 최초 predicate==false 요소가 나온 순간 반복 작업 중간에 break 할 수 있음

```java
IntStream.of(1,2,2,3,4).stream()
                .takeWhile(num->num<3)
                .collect(Collectors.toList())
```
- 그럼 정렬되지 않은 컬렉션이라면?

### dropWhile()

- takeWhile 로 떨어져나간 나머지 요소 가져오기

```java
IntStream.of(1,2,2,3,4).stream()
                .dropWhile(num->num<3)
                .collect(Collectors.toList())
```
- 그럼 정렬되지 않은 컬렉션이라면?

### limit()

- 앞에서부터 n개 선택

```java
IntStream.of(1,2,2,3,4).stream()
                .filter(num->num<3)
                .limit(3)
                .collect(Collectors.toList())
```

### skip()

- 앞에서부터 n개 pass
```java
IntStream.of(1,2,2,3,4).stream()
                .filter(num->num<3)
                .skip(2)
                .collect(Collectors.toList())
```

<br>

## 매핑

- 기존 요소를 함수를 적용한 새로운 return 요소들로 바꿔버림

```java
List<Integer> stringNums =IntStream.of(1,2,2,3,4)
                .mapToObj(num-> String.valueOf(num))
                .map(String::length)
                .collect(Collectors.toList())
```


### flatMap()
- 여러개의 컬렉션을 하나의 스트림으로 평면화
- 스트림의 각 값을 다른 스트림으로 만들어서, 모든 스트림을 하나로 연결

```java
Arrays.stream(new String[]{"Hi", "Hello"})
    .map(word->word.split("")) //Stream<String[]>
    .flatMap(Arrays::stream) //Stream<String>
    .collect(Collectors.toList())
```

## 검색

- 쇼트서킷 이용하므로 효율적 계산

### anyMatch()

- 프레디케이트에 적어도 한 요소가 일치하는지 여부

### allMatch()

- 모든 요소가 프레디케이트에 일치하는지 여부

### noneMatch()

- 모든 요소가 프레디케이트에 일치하지 않는지 여부

### findAny()

- 현재 스트림에서 임의 요소 반환

```java
menu.stream()
.filter(Dish::isVegetarian)
.findAny() //Optional<Dish>
.ifPresent(dish->System.out.println(dish.getName())) //null exception 으로 터지는 문제 랩핑
```

### findFirst()

- 병렬 스트림에서 안전하게 첫번째 요소 반환


<br>

## 리듀싱

- 여러 스트림 요소를 처리해서 폴딩(조합)하는 연산
- 내부에서 상태를 갖는다.
- 내부 반복이 추상화 되면서 **병렬**로 reduce를 실행할 수 있게됨 (parallelStream())


```java
int sum = numbers.stream().reduce(0, (a,b)->a+b); //초기값, operator

int sum = numbers.stream().reduce(0, Integer::sum); //Integer::min, Integer::max
```

- Map Reduce

```java
menu.stream().map(d->1).reduce(IntStrem::sum);
menu.stream().count();
```

<br>

## 숫자형 스트림

### primitive type 특화 스트림

- IntStream, DoubleStream, LongStream
- boxing 과정에서 일어나는 효율성을 위해 만들어짐
    - max(), sum(), min(), average()

```java
menu.stream()
.map(Dish::getCalories) //Stream<Integer>
.reduce(Integer::sum);

menu.stream()
.mapToInt(Dish::getCalories) //IntStream
.sum();

menu.stream()
.mapToInt(Dish::getCalories)
.boxed() //Stream<Integer>

IntStream.range(1,100);
IntStream.rangeClosed(1,100);
```

- OptionalInt
    - stream 의 요소가 null 인 경우에 대한 랩핑

```java
OptionalInt maxCalory = menu.stream().mapToInt(Dish::getCalories).max().orElse(0); //null 이면 0 
```


## 스트림 만들기

```java
Stream<String> stream = Stream.of("Hi", "Hello");

//빈 스트림
Stream<String> empty = Stream.empty();

//nullable 스트림
Stream<String> nullable = Stream.ofNullable(System.getProperty("lineSeparator"));

//array to stream
int sum = Arrays.stream(new int[]{1,2,3}).sum();

//무한 스트림 == unbound stream ==> collection 과의 차이점
IntStream.iterate(0, n->n+2)
.limit(100)

//종료 조건
IntStream.iterate(0, n->n<100, n->n+2)

IntStream.iterate(0, n->n+2)
.takeWhile(n->n<100)

IntStream.generate(Math::random) //supplier
.limit(10)
```

* Tip
    - 스트림은 자원을 자동으로 해제가능 AutoCloseable
    - 스트림을 병렬로 처리하려면 기존 상태를 바꾸지 않는 불변 상태여야한다.(값을 가지고 있지 않아야함)

