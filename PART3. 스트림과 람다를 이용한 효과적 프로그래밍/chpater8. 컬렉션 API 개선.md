
# chapter 8. 컬렉션 API 개선

> 컬렉션 팩토리 사용하기  
> 리스트 및 집합과 사용할 새로운 관용패턴 배우기   
> 맵과 사용할 새로운 관용 패턴 배우기

## 리스트 팩토리
### Arrays.asList
~~~java
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");

/* Arrays.asList 사용으로 변경*/
List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut");
~~~
고정된 크기의 리스트를 만들었으로 새로운 요소를 추가하거나 요소를 삭제한다면 `UnsupportedOperationException`이 발생한다.  
요소를 갱신하는것은 정상적으로 작동한다.

### List.of
`List.of`팩토리 메서드를 활용하여 간단하게 리스트를 만들수 있게 되었다.
~~~java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
~~~
of 메소드를 사용하더라도 요소를 추가하거나 삭제하면 여전히`UnsupportedOperationException` 에러가 발생한다.

#### Arrays.asList와 of의 차이점 -> 가변인수를 사용하지 않는다
~~~java
static <E> List<E> of() {
    return ImmutableCollections.emptyList();
}

static <E> List<E> of(E e1) {
    return new List12(e1);
}
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9, E e10) {
    return new ListN(new Object[]{e1, e2, e3, e4, e5, e6, e7, e8, e9, e10});
}
~~~
* 가변인수를 사용한다면 그만큼의 배열을 리스트로 만들고 추후 객체 반환까지 해야하는 비용이 발생
* `List.of`를 활용해 불필요한 비용을 줄일수 있게 되었다.

## 집합 팩토리
### Set.of
~~~java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");

/* Key - Value 를 번갈아 가며 생성 */
Map<String, Integer> friendsMap = Map.of("Raphael", 30, "Olivia", 30, "Thibaut", 30);

/* Map의 인수가 많을경우 -> entry 사용 */
Map<String, Integer> friendsMapEntry = Map.ofEntries(entry("Raphael", 30),
entry("Olivia", 30),
entry("Thibaut", 30));
~~~

## 리스트와 집합 처리
* `removeIf` : `Predicate`를 만족하는 요소를 제거한다. `List`나 `Set`을 구현하거나 구현을 상속받은 클래스에서 이용할 수 있다
* `replaceAll` : 리스트에서 이용할수 있는 기능으로 `UnaryOperator`함수를 이용해 요소를 바꾼다.
* `sort` : `List`인터페이스에서 제공하는 기능으로 리스트를 정렬한다.

~~~java
/* 반복자의 상태는 컬렉션의 상태와 서로 동기화되지 않고 있음 
    => ConcurrentModificationException에러 발생 
 */
for (Transaction transaction : transactions) {
    if (True) {
        transactions.remove(transaction);
    }
}

/* 두 개의 개별 객체가 컬렉션을 관리함 */
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();) {
    Transaction transaction = iterator.next();
    if (true) {
        transactions.remove(transaction);
    }
}

/* Iterator 객체를 명시적으로 삭제하여 문제를 해결함 */
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();) {
    Transaction transaction = iterator.next();
        if (true) {
            iterator.remove();
    }
}
~~~
**removeIf를 사용하여 구현**
~~~java
transactions.removeIf(transaction -> true);
~~~
**replaceAll 메서드를 이용하여 리스트의 각 요소를 새로운 요소로 변경**
~~~java
// 기존 컬렉션은 그대로, 새로운 컬렉션을 생성O (stream api 이용)
referenceCodes.stream().map(Strings::toUpperCase)).collect(toList());
// 기존 컬렉션을 변경, 새로운 컬렉션 생성X
referenceCodes.replaceAll(code -> code.toUpperCase());
~~~

## 맵 처리
### forEach 메서드
* Java 8 부터 Map 인터페이스는 `BiConsumer(키와 값을 인수로 받음)`를 인수로 받는 `forEach`메서드가 추가되어 간단하게 구현 가능
~~~java
Map<String, Integer> ageOfFriends = new HashMap<>();
ageOfFriends.forEach((friend, age) -> print(friend + "is" + age)))
~~~
### 정렬 메서드
다음 두 개의 새로운 유틸리티를 활용하여 맵의 항목을 key 또는 Value를 기준으로 정렬할 수 있다.
* `Entry.comparingByValue`
* `Entry.comparingByKey`
~~~java
Map<String, String> favouriteMovies = Map.ofEntries(entry("Raphael", "Star Wars"), entry("Cristina", "Matrix"), entry("Olivia", "James Bond"));

favouriteMovies.entrySet()
        .stream()
        .sorted(Entry.comparingByKey());
~~~

### getOrDefault
맵에 존재하지 않는 키로 값을 찾으려할때 NPE가 발생하여 NULL 체크가 필요했는데, `getOrDefault`메서드를 통해 해결할 수 있다.
~~~java
Map<String, String> favouriteMovies = Map.ofEntries(entry("Raphael", "Star Wars"), entry("Cristina", "Matrix"), entry("Olivia", "James Bond"));
favouriteMovies.getOrDefault("Olivia", "Matrix"); // James Bond 출력
favouriteMovies.getOrDefault("Thibaut", "Matrix"); // Matrix 출력
~~~
키가 존재하더라도 값(value)이 Null인 경우에는 여전히 Null을 반환함.

### 계산 패턴
맵에 키가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장해야 하는 상황이 필요할 때가 있다.
* `computeIfAbsent` : 제공된 키에 해당하는 값이 존재하지 않으면(값이 없거나 Null) 제공된 키로 새로운 Value를 맵에 추가한다.
* `computeIfPresent` : 제공된 키가 존재하면 새로운 Value로 맵에 추가한다.
* `compute` : 제공된 키로 새 Value를 추가한다.

### 삭제 패턴
특정 key에 특정 value일때만 제거 할 수 있도록 오버라이드된 메소드
=> map에 특정 key가 존재하고, 그 key값의 value가 특정 value인지를 체크함
~~~java
sampleMap.remove(key, value);
~~~

### 교체 패턴
* `replaceAll` : `BiFunction`을 적용한 결과로 각 항목의 값을 교체한다. 이전에 알려준 `List.replaceAll`과 비슷한 기능이다.
* `replace` : 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을때만 변경하도록 하는 오버로드된 버전도 있다.
~~~java
Map<String, String> favouriteMovies = Map.ofEntries(entry("Raphael", "Star Wars"), entry("Cristina", "Matrix"), entry("Olivia", "James Bond"));
favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
~~~

### 합침(Merge)
두 그룹의 연락처를 포함하는 두 맵을 합친다고 가정 -> `putAll`사용
~~~java
Map<String, String> family = Map.ofEntries(entry("Raphael", "Star Wars"), entry("Cristina", "Matrix"));
Map<String, String> friends = Map.ofEntries(entry("Olivia", "James Bond"));
Map<String, String> everyone = new HashMap<>(family);
everyone.putAll(friends);
~~~
<br>

중복된 키가 존재하고 값을 유연하게 다루기 위해 `merge`메서드를 활용할 수 있다.  
`merge`메서드는 중복된 메서드를 어떤 방식으로 합칠지 결정하는 `BiFunction`을 인수로 받는다.
~~~java
Map<String, String> family = Map.ofEntries(entry("Raphael", "Star Wars"), entry("Cristina", "Matrix"));
Map<String, String> friends = Map.ofEntries(entry("Olivia", "James Bond"), entry("Cristina", "James bond"));

Map<String, String> everyone = new HashMap<>(family);
everyone.merge(k, v, (movie1, movie2) -> movie1 + movie2)); 
~~~

~~~java
moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);
~~~
키와 연관된 기존 값에 합쳐질 널이 아닌 값 또는 갑싱 없거나 키에 널 값이 연관되어 있다면 두번쨰 인수값을 키와 연결한다.  
처음 키 값이 입력될땐 초기값으로 1L을 사용함  
그 다음부턴 값이 1로 초기화되어 있으므로 세번째 인수인 BiFunction을 적용해 값을 증가시킨다.


## ConcurrentHashMap
> `ConcurrentHashMap`: 동시성 친화적이며 최신기술을 반영한 HashMap 버전

### 리듀스와 검색
* `forEach` : 각 (key, value) 쌍에 주어진 액션을 실행
* `reduce` : 모든 (key, value) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
* `search` : 널이 아닌 값을 반환할 때 까지 각 (key, value)쌍에 함수를 적용

다음처럼 키에 함수 받기, 값, Map.Entry, (key, value)인수를 이용한 네 가지 연산 형태를 지원
* key, value로 연산 (forEach, reduce, search)
* key로 연산(forEachKey, reduceKeys, searchKeys)
* value로 연산(forEachValue, reduceValues, searchValues)
* Map.Entry 객체로 연산(forEachEntry, reduceEntries, searchEntries)

<br>

* 이들 연산은 `ConcurrentHashMap`의 **상태를 잠그지 않고 연산을 수행**한다.  
따라서 이들 연산에 제공한 함수는 계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 않아야 한다.
* **병렬성 기준값**(threshold)을 지정해야한다.  
맵의 크기가 주어진 기준값보다 작으면 순차적으로 연산을 실행한다.
~~~java
ConcurrentHashMap<Stirng, Long> map = new ConcurrentHashMap<>();
long parallelismThreshold = 1;
// reduceValue메서드를 사용해서 맵의 최댓값 찾기
Optional<Integer> maxValue = Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
~~~
### 계수
* `mappingCount`: 맵의 매핑 개수를 확인하는 메서드   
기존 size 메서드 대신 새 코드에서는 int를 반환하는 mappingCount 메서드를 사용하는것이 좋다.  
### 집합뷰
* `keySet`: ConcurrentHashMap을 집합 뷰로 반환함
맵을 바꾸면 집합도 바뀌고 반대로 집합을 바꾸면 맵도 영향을 받는다.  
* `newKeySet`: ConcurrentHashMap으로 유지되는 집합을 만들 수 있음

**keySet output ex)**
~~~
Initial Mappings are: {20=Geeks, 25=Welcomes, 10=Geeks, 30=You, 15=4}
The set is: [20, 25, 10, 30, 15]
~~~
