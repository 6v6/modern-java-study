# chapter2. 동작 파라미터화 코드 전달하기

***

**chapter2의 핵심** ✨

해야 할 일(method)을 파라미터로 넘길 수 있음!

***

  <br>

클라이언트의 가변적인 요구사항에 대해 최소한의 비용으로 쉽게 구현할 수 있어야 함

→ **동작 파라미터화**가 등장한 이유

  <br>

#### 엥... 동작 파라미터화가 뭔데? 🙄

**어떻게 실행할 것인지 결정하지 않은 코드 블록**을 의미하며, 나중에 프로그램에서 호출 및 실행될 코드 블록이라고 생각하면 된다.

<br>

## 변화하는 요구사항에 대응하기

이제는 변덕이 심한 농부의 요구사항에 대응하며 동작 파라미터화가 어떤 친구인지 알아보자! 👊

<br>

### 1. 녹색 사과 필터링

👩‍🌾 : 농장 재고 목록에서 녹색 사과만 필터링 해 주세요!

```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>(); // 사과 누적 리스트
    for (Apple apple : inventory) {
      if (apple.getColor() == Color.GREEN) { // 녹색 사과만 선택
        result.add(apple);
      }
    }
    return result;
  }
```

그런데 농부가 말을 바꿔서 빨간 사과도 같이 필터링 해 달라고 하면 어떨까?

<br>

### 2. 색을 파라미터화

👩‍🌾 : 음... 어쩌고색도 필터링 해 주세요!!

단순히 복붙하지 않고 코드를 추상화해서 색을 파라미터화하는 방법을 사용하자!

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
   List<Apple> result = new ArrayList<>();
   for (Apple apple : inventory) {
     if (apple.getColor() == color) { 
       result.add(apple);
     }
   }
   return result;
}
```

<br>

농부가 요구사항을 또 추가한다면 어떨까? 

👩‍🌾 : 색 말고도 무게로도 필터링 해 주세요~~



물론, 색을 필터링 했던 것처럼 파라미터에 weight 값을 추상화해서 넣을 수 있다.

```java
  public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (apple.getWeight() > weight) {
        result.add(apple);
      }
    }
    return result;
  }
```

그렇지만 이전 코드와 너무 비슷하고... 진짜 이래도 되나? 싶다... 🙃

그래서 머리를 굴려서 생각해 본 게 flag 방식! 필터링 기준을 flag 파라미터에 넣는 방식이다. 

<br>

### 3. 가능한 모든 속성으로 필터링

이러시면 안 됩니다...

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventoryr(){
		if((flag && apple.getColor().equals(color)) || (!flag && apple.getWeight() > weight)) {
				result.add(apple);	
		}
	}
    return result;	
}
```

아 내심 뿌듯하게 작성했는데 농부가 움... 녹색 사과 중에 무거운 사과를 또 필터링 하고 싶은데요? 이러면 이 코드는 답이 없어진다... ~~사실 원래도 답 없는 코드이다~~

<br>

이처럼 변화하는 요구사항에 좀 더 유연하게 코드를 작성할 필요가 있다. 다행히 우리는 그럴 때 활용할 수 있는 방법도 알고 있다. 바로 동작 파라미터화를 사용하면 된다! 😎

참 또는 거짓을 반환하는 함수인 **프레디케이트**를 활용해서 선택 조건을 결정하는 인터페이스를 정의해 보자.



사과의 색과 무게 같은 선택 조건을 캡슐화하면 이런 그림이 그려진다. (전략 디자인 패턴)

![image](https://user-images.githubusercontent.com/62419307/202650508-1ff4ef1a-8ea1-471b-9207-f25af778855d.png)



이 프레디케이트를 어떻게 활용하면 될까? 이전에 만든 메서드 파라미터에 프레디케이트를 받도록 수정하면 된다.

→ method가 다양한 동작을 파라미터로 받아서 내부적으로 다양한 동작을 수행할 수 있다.

<br>

🙋‍♀️ 전략 디자인 패턴이 뭐죠...?

전략 디자인 패턴은 각 알고리즘(전략)을 캡슐화하여 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법이다. 말이 어려운데... 그냥 난 음료수 먹겠다고 한 다음에 실제 주문할 때 콜라 주세요 하는 기법이라고 생각하면 된다. 🥤

<br>

### 4. 추상적 조건으로 필터링

이제는 filterApples 내부에서 컬렉션을 반복하는 로직과 실제 필터링하는 동작이 분리가 된다. 

```java
 public static List<Apple> filter(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (p.test(apple)) { // 프레디케이트 객체로 검사 조건을 캡슐화
        result.add(apple);
      }
    }
    return result;
  }
```

이와 같이 어떤 필터링 조건이 들어와도 나는 나만의 길을 간다는 느낌!으로 추상화된 코드를 작성할 수 있다.

<br>

#### 코드/동작 전달하기

이제 농부가 어떤 요구조건을 제시해도 프레디케이트만 적절하게 수정하면 된다.

👩‍🌾 : 150g이 넘는 빨간 사과가 필요해요!



![image](https://user-images.githubusercontent.com/62419307/202656157-eea9ac41-6252-41fa-a067-3c85f015be60.png)

이미지와 같이 해야 할 일을 그대로 파라미터로 넘기면 간단하게 해결된다! 그렇지만 객체를 넘겨줘야 하기 때문에 인스턴스화 하는 부분이 불필요하게 느껴지기도 한다. 이 부분은 어떻게 해결해야 할까? 🤔

<br>

## 복잡한 과정 간소화

무게나 색에 대해서 각각 인스턴스를 만들어주는 건 번거로운 일이다. 그래서 익명 클래스 기법을 사용하는 것이 좋다.

<br>

#### 익명 클래스란?

클래스의 선언과 인스턴스화를 동시에 수행할 수 있는 기법으로, 자바의 지역 클래스와 비슷한 개념이다. 즉, 즉석에서 필요한 구현을 만들어서 사용할 수 있다.

<br>

### 5. 익명 클래스 사용

```java
List<Apple> redApples = filterApple(inventory, new ApplePredicate() {
	public boolean test(Apple apple) {
		return RED.quals(apple.getColor());
  }
});
```

안드로이드를 했을 때가 어렴풋이 기억난다... ⭐

지금이나 예전이나 익명 클래스 방식은 1. 많은 공간을 차지한다. 2. 많은 프로그래머가 익명 클래스 사용에 익숙지 않다 라는 단점이 있다. 내가 작성했지만 이...이게 뭐꼬..? 와 같은 상황이 펼쳐질 수 있다.

또한, 이렇게 코드를 장황하게 작성해 놓고는 결국 객체를 만들고 명시적으로 새로운 동작을 정의하는 메서드를 구현해야 한다는 점은 변하지 않는다. (익명 클래스: 😥) 

<br>

### 6. 람다 표현식 사용

그래서 더 간단한 방식을 사용해 본다! 람다 표현식은 3장에서 더 자세히 볼 테니 이런 방법이 있구나 하고 가볍게 알고 있으면 된다.

```java
List<Apple> result = filterApples(inventory, 
                          (Apple apple) -> RED.equals(apple.getColor()));
```

<br>

### 7. 리스트 형식으로 추상화

이제 형식 파라미터도 추가 되면서 유연성과 간결성을 동시에 잡을 수 있다. (멋져)

```java
public interface Predicate<T> {
	boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
// 형식 파라미터 T 등장
	List<T> result = new ArrayList<>();
	for(T e : list) {
		if(p.test(e)) {
			result.add(e);
		}
	}
	return result;
}

//실제 사용
List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
List<Integer> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);
```

<br>

## 실전 예제

지금까지 동작 파라미터화가 변화하는 요구사항에 쉽게 적응하는 유용한 패턴이라는 걸 알아봤다. 동작 파라미터화는 동작을 캡슐화한 다음에 메서드로 전달해서 동작을 파라미터화하는 방식이었다. 이제는 실전 예제를 통해 어떻게 활용할 수 있는지 살펴보자.

<br>

### Comparator로 정렬하기

자바로 정렬을 하는 건 파이썬보단 좀 번거롭다고 생각할 만 한데 Comparator를 사용하기 위해 객체를 생성해서 그 객체를 파라미터로 넣는 과정이 정말 귀찮기 때문이다... 😇

이 방식도 자바8을 사용하면 깔끔하게 작성이 가능하다.

```java
// java.util.Comparator
public interface Comparator<T> {
	int compare(T o1, T o2);
}

//익명 클래스
inventory.sort(new Comparator<Apple>() {
	public int compare(Apple a1, Apple a2) {
		return a1.getWeight().compareTo(a2.getWeight());
	}
}


// lambda
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

<br>

## 정리

1. 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다.

2. 동작 파라미터를 활용하면 가변적인 요구사항에 유연하게 대응할 수 있다.

3. 자바 API의 많은 메서드는 정렬, 스레드 등을 포함한 다양한 동작으로 파라미터화할 수 있다.


