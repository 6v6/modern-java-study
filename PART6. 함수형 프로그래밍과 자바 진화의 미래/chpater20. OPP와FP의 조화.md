# chpater20. OPP와 FP의 조화

<br/>

## 1 스칼라 소개

### 스칼라란?
* 객체지향과 함수형 프로그래밍의 혼합한 언어이다.
* JVM에서 동작하기에 자바의 클래스를 사용할 수 있다.
* 자바에서 라이브러리 추가만으로 스칼라 코드를 호출하여 사용할 수 있다. 
* 스칼라에서의 **모든 데이터 타입은 객체**로 취급한다.
* 스칼라에서는 모든 것이 객체다.
* 모든 변수의 형식은 컴파일 할 때 결정된다.

<br/>

### 명령형 스칼라
~~~java
object Beer {
    def main(args: Array[String]) {
        var n : Int = 2
        while (n <= 6) {
            println(s"hello ${n} bottles of beer")
            n += 1
        }
    }
}
~~~
* object로 클래스를 정의하고 동시에 **싱글턴 객체**를 만들었다.
* object 내부에 선언된 메서드는 **정적 메서드**로 간주되므로 **static을 명시할 필요 없다.**
* 스칼라의 **문자열 보간법** : 문자열에 접두어s를 붙이고, ${}안에 변수 위치
* 자동으로 반환형을 추론하여 반환형을 정의하지 않아도 된다.

<br/>

### 함수형 스칼라
~~~java
object Beer {
   // 2.to(6).foreach(n => println(s"hello ${n}")) 
   def main(args: Array[String]) { 
      2 to 6 foreach { n => println(s"hello ${n}")}
   }
}
~~~

<br/>



## 스칼라의 자료구조
> List, Set, Map, Stream, Tuple, Option...


### 컬렉션 만들기
~~~java
// Map
var authorsToAge = Map("R" -> 23, "M" -> 40,  "A" -> 54)
println(s"${authorsToAge.get("R")}")
        
// List 
val authors = List("R", "M", "A")
authors.foreach(name => print(s"${name} "))
        
// Set 선언 
val numbers = Set(1, 1, 2, 3, 5, 6)
numbers.foreach(number => print(s"${number} "))
}
~~~
* -> 를 이용하여 map을 선언할 수 있다.
* 스칼라변수 
  * var : 일반선언, 읽고 쓸 수 있는 변수
  * val : 상수 선언, 읽기전용
* 자동으로 변수형을 추론하여 `authorsToAge`의 형식을 지정하지 않아도 된다.

<br/>

### 불변과 가변
* 우의 컬렉션은 기본적으로 **불변**하다
* 갱신이 필요하면 새로운 컬렉션을 만들어야 함
* 암묵적인 데이터 의존성으 줄일 수 있다.


~~~java
val numbers = Set(2, 5, 3)
val newNumbers = numbers + "B" // +는 집합에 8을 더하는 메서드의 연산결과로 새로운 Set객체를 생성한다.
println(newNumbers) // (2, 5, 3, 8)
println(numbers) // (2, 5, 3)
~~~

<br/>


### 컬렉션 사용하기
~~~java
val fileLines = Source.fromFile("data.txt").getLines().toList
        val linesLongUpper = fileLines.filter(l => l.length() > 10)
        .map(l => l.toUpperCase())
~~~
* 스칼라의 컬렉션 동작은 스트림API와 비슷하다.
* filter : 길이가 10 이상인 행만 가져온 뒤
* Map : 대문자로 변경한다.

<br/>

~~~java
val linesLongUpper = fileLines.par filter (_.length() > 10) map(_.toUpperCase())
~~~
* 스트림의 `parallel`을 호출하여 파이프라인을 병렬로 실행한것처럼, `par`을 이용하면 된다.

<br/>

### 튜플
* 자바는 튜플을 지원하지 않는다. 필요하면 VO를 만들어써야한다.
* 자바 14부터 튜플식으로 쓸 수 있는 record 지원한다.
* **튜플 축약어** 간단한 문법으로 튜플을 만들 수 있다.
* _1, _2 의 접근자로 요소에 접근 가능하다.
~~~java
val raoul = ("Raoul", "+ 44 887007007")
println(raoul._1) 
~~~

<br/>

### 스트림
* 스칼라의 스트림은 자바의 스트림보다 다양한 기능을 제공한다. 
* 이전 요소가 접근할 수 있도록 기존 계산 값을 기억한다. 
* 인덱스를 제공하기 때문에 리스트처럼 인덱스로 스트림 요소에 접근할 수 있다. 
* 인덱스를 사용하여 요소에 참조하려면 캐시해야하기 때문에 자바의 스트림에 비해 메모리 효율성이 떨어진다. 

<br/>

### 옵션
* Optional과 같은 기능을 제공한다.
~~~java
def getCarInsuranceName(person: Option[Person], minAge: Int) =
   person.filter(_.getAge >= minAge)
   .flatMap(_.getCar)
   .flatMap(_.getInsurance)
   .map(_.getName)
   .getOrElse("Unknown")
~~~

<br/>

## 2 함수
> 스칼라의 함수는 어떤 작업을 수행하는 명령어 그룹이다.

<br/>

1. 함수 형식 : 함수형 인터페이스에 선언된 추상 메서드의 시그니처를 표현하는 개념
2. 익명 함수 : 자바의 람다 표현식과 달리 비지역 변수 기록에 제한을 받지 않는다.
3. 커링 지원 : 여러 인수를 받는 함수를 일부 인수를 받는 여러 함수로 분리

<br/>

### 스칼라의 일급 함수
* Predicate함수를 저장해뒀다가 filter함수에 인수로 전달할 수 있다.

~~~java
def isJavaMentioned(tweet: String) : Boolean = tweet.contains("Java")
val tweets = List("Java", "Scala")
tweets.filter(isJavaMentioned).foreach(println)
~~~

<br/>

### 익명 함수
* scala.Function1형식의 익명 클래스를 축약한 다음 Function1의 apply호출
~~~java
val isLong : String => Boolean = (tweet : String) => tweet.length > 60
var result = isLong.apply("short tweet")
println(result)
~~~

<br/>

### 클로저
* 함수의 **비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스**.
* 자바의 람다에서는 람다가 정의된 지역 변수를 final로 취급해서 고칠 수 없는 제약이 있다.
~~~ java
 var count = 0
 val inc = () => count+=1
 inc() // count를 캡처하고 증가시키는 클로저
 println(count)
 inc()
 println(count)
~~~

<br/>


### 커링
* 스칼라에서는 커리된 함수를 직접 제공할 필요가 없다.
* 커링은 x와 y라는 두 인수를 받는 함수 f를 한 개의 인수를 받는 g라는 함수로 대체하는 기법이다.
* 함수가 여러 커리된 인수 리스트를 포함하고 있음을 가리키는 함수 정의 문법을 제공하기 때문이다.
~~~java
def multiply(x : Int, y : Int) = x * y
val r = multiply(2, 10);


def multiplyCurry(x: Int)(y: Int) = x * y // 커리된 함수 정의
val r1 = multiplyCurry(2)(10) // result = 20
val multiplyByTwo : Int => Int = multiplyCurry(2) // 부분 적용된 함수라 부른다.
val r2 = multiplyByTwo(10) // result = 20
~~~

<br/>


### 클래스
* 스칼라는 완전한 객체지향 언어이므로 클래스를 만들고 객체로 인스턴스화 할 수 있다.
* 생성자, 게터, 세터가 암시적으로 생성된다.
~~~java
class Student(var name: String, var id: Int) // 클래스 선언
val s = new Student("Raoul", 1)
println(s.name)
~~~

<br/>

### 트레이트
* 추상화 기능으로 자바의 인터페이스와 유사하다
* 추상 메서드와 기본 구현을 가진 메서드 모두 정의할 수 있다
* 다중 상속 지원
* 인스턴스화 과정에서도 조합할 수 있다
~~~java
trait Sized {
  var size : Int = 0 // size 필드
  def isEmpty() = size == 0 //isEmpty 메서드
}

class Box
val b1 = new Box() with Sized // 객체를 인스턴스화 할 때 트레이트를 조합함
println(b1.isEmpty()) // true
val b2 = new Box()
b2.isEmpty() // 컴파일 에러: Box 클래스 선언이 Sized를 상속하지 않음
~~~

