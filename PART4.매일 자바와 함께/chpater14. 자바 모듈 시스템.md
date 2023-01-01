# chpater14. 자바 모듈 시스템

---

**chapter14의 핵심** ✨

모듈화를 이용했을 때 캡슐화, 독립성 등 아키텍처 측면에서 어떤 장점이 있는지 살펴보자!

---

## 소프트웨어 유추

이제까지 이해하고 유지보수하기 쉬운 코드를 구현하는 데 사용할 수 있는 새로운 언어 기능을 소개했다면 (ex: 람다, 스트림 등) 소프트웨어 아키텍처 기반에서 어떤 기능이 있는지 살펴보자. 🙋‍♀️

<br>

### 관심사 분리

**관심사 분리**는 컴퓨터 프로그램을 고유의 기능으로 나누는 동작을 권장하는 원칙이다. 위 원칙은 mvc 같은 아키텍처 관점, 복구 기법을 비즈니스 로직과 분리하는 등의 하위 수준 접근 등의 상황에 유용하다.

- 개별 기능을 따로 작업할 수 있으므로 팀이 쉽게 협업할 수 있다.
  
- 개별 부분을 재사용하기 쉽다.
  
- 전체 시스템을 쉽게 유지보수할 수 있다.
  

<br>

### 정보 은닉

**정보 은닉**은 세부 구현을 숨기도록 장려하는 원칙이다. 세부 구현은 숨겨서 프로그램의 어떤 부분을 바꾸더라도 다른 부분까지 영향을 미칠 가능성을 줄일 수 있다.

어디서 많이 들어봤는데...? 🙄 싶으면 맞다! 캡슐화가 바로 이것이다.

**캡슐화**란 특정 코드 조각이 애플리케이션의 다른 부분과 독립되어 있음을 의미한다.

<br>

나 궁금한 게 있는데 이게 자바 9의 새로운 기능이라고? 원래 패키지도 같은 일을 하지 않았나? 🤔 라는 생각이 들 수도 있다.

그렇지만 자바9 이전까지 클래스와 패키지가 의도된 대로 공개되었는지 컴파일러로 확인할 수 있는 기능도 없었고, protected, private 등의 접근 제한자와 패키지 수준 접근 권한 등은 아래와 같은 한계가 있었다.

1. 원하는대로 접근 권한을 설정하기 어려움.
  
2. 원하지 않는 메소드도 공개해야 함.
  

그렇지만 자바9에서 모듈 시스템을 구현하면서 위와 같은 한계를 극복할 수 있게 됐다!

<br>

## 자바 모듈 시스템을 설계한 이유

우선, 모듈화의 한계에 대해서 더 자세히 알아보자

1. 제한된 가시성 제어
  
2. 클래스 경로
  

제한된 가시성 제어에 대해선 이전에 언급했으니 클래스 경로에 대해서 자세히 설명하겠다.

<br>

### 클래스 경로

자바는 애플리케이션을 번들하고 실행하는 기능과 관련해서 태생적인 약점을 갖고 있다.

1. 클래스를 모두 컴파일 한다.
  
2. 그리고 한 개의 JAR 파일에 넣는다.
  
3. 클래스 경로에 JAR 파일을 추가해 사용한다.
  
4. JVM이 동적으로 클래스 경로에 정의된 클래스를 필요할 때 읽는다.
  

📌 참고!

[JAR이 뭐였더라...?](https://github.com/hjyeon-n/BE_TIL/blob/master/AWS/AWS%20%EB%B0%B0%ED%8F%AC%20%EA%B3%BC%EC%A0%95%20%EC%A4%91%20%EC%95%8C%EA%B2%8C%EB%90%9C%20%EA%B2%83%EB%93%A4.md)

<br>

위 방법은 몇 가지 문제점이 있다.

- 클래스 경로에는 같은 클래스를 구분하는 버전 개념이 없다.
  
- 클래스 경로는 명시적인 의존성을 지원하지 않는다.
  

명시적인 의존성을 지원하지 않는 상황에서는 클래스 경로 때문에 어떤 일이 일어나는지 파악하기 어렵다. 이 문제점은 maven이나 gradle 빌드 도구가 해결하는 데 도움을 준다.

이렇게만 말하면 아항 그렇구나 하고 흘려들을 수 있는데...

자바9 이전에는 명시적인 의존성을 지원하지 않았기 때문에 클래스 경로를 찾을 수 없다는 에러가 떴을 때 하나하나... 클래스 파일을 넣어보고 이게 문젠가 저게 문젠가 꼭 찍먹을 해야 문제를 해결할 수 있다는 점에서 자바9의 모듈 시스템은 짱이라고 할 수 있다... 😎

📌 참고!

[maven, gradle 자세히 알려줘!](https://github.com/hjyeon-n/BE_TIL/blob/master/Spring%EC%9D%98%20%EC%9D%B4%EA%B2%83%EC%A0%80%EA%B2%83/Spring%EC%9D%98%20%EC%9D%B4%EA%B2%83%EC%A0%80%EA%B2%83.md)

<br>

## 자바 모듈

chapter14의 경우, 프로젝트 디렉터리 구조라든가 pom.xml 의 내용이 많아서 정리가 어려운 관계로 챕터 별로 정리를 하지 않고 하나로 통합해서 정리하려고 한다.

<br>

### 자바 모듈

모듈은 module이라는 새 키워드에 이름과 바디를 추가해서 정의하며, 새로운 자바 프로그램 구조 단위를 제공한다.

모듈 디스크립터는 module-info.java라는 특별한 파일에 저장된다. 이 파일은 모듈의 의존성, 어떤 기능을 외부로 노출할지를 정의한다.

![image](https://user-images.githubusercontent.com/62419307/210042130-38a07e52-5955-4ff4-96ac-774466416da4.png)

<br>

![image](https://user-images.githubusercontent.com/62419307/210042160-3090aa5d-4832-447c-b206-37c6b6ab4dda.png)

<br>

### 세부적인 모듈화와 거친 모듈화

- 세부적인 모듈화 : 모든 패키지가 자신의 모듈을 가짐
  
- 거친 모듈화 : 한 모듈이 시스템의 모든 패키지를 포함함 → 모듈화의 모든 장점을 잃음
  

<br>

## 모듈 정의와 구문들

### 1. requires

컴파일 타임과 런타임에 한 모듈이 다른 모듈에 의존함을 정의한다.

```java
module com.iteratrlearning.application {
    requires com.iteratrlearning.ui;
}
```

com.iteratrlearning.application은 com.iteratrlearning.ui 모듈에 의존한다.

<br>

### 2. exports

지정한 패키지를 다른 모듈에서 이용할 수 있도록 공개 형식으로 만든다. 아무 패키지도 공개하지 않는 것이 기본 설정이며 명시적으로 어떤 패키지를 공개할 것인지 지정할 수 있다.

```java
module com.iteratrlearning.ui {
    requires com.iteratrlearning.core;

    exports com.iteratrlearning.ui.panels;
    exports com.ireratrlearning.ui.widgets;
}
```

예제에서는 com.iteratrlearning.ui.panel, com.ireratrlearning.ui.widgets을 공개했다.

<br>

### 3. requires transitive

다른 모듈이 제공하는 공개 형식을 한 모듈에서 사용할 수 있다고 지정할 수 있다.

```java
module com.iteratrlearning.ui {
    requires transitive com.iteratrlearning.core;

    exports com.iteratrlearning.ui.panels;
    exports com.ireratrlearning.ui.widgets;
}

module com.iteratrlearning.application {
    requires com.iteratrlearning.ui;
}
```

com.iteratrlearning.application 모듈은 com.iteratrlearning.core에서 노출한 공개 형식에 접근할 수 있다.

<br>

### 4. exports to

사용자에게 공개할 기능을 제한함으로 가시성을 좀 더 정교하게 제어할 수 있다.

```java
module com.iteratrlearning.ui {
    requires com.iteratrlearning.core;

    exports com.iteratrlearnig.ui.panels;
    exports com.iteratrlearnig.ui.widgets to com.iteratrlearnig.ui.widgetuser;
}
```

to com.iteratrlearnig.ui.widgets의 접근 권한을 가진 사용자의 권한을 com.iteratrlearnig.ui.widgetuser로 제한할 수 있다.

<br>

### 5. open, opens

모듈 선언에 open 한정자를 이용하면 모든 패키지를 다른 모듈에 반사적으로 접근을 허용할 수 있다.

<br>

### 6. uses와 provides

자바 모듈 시스템은 provides 구문으로 서비스 제공자를, uses 구문으로 서비스 소비자를 지정할 수 있다.

<br>

## 정리

1. 관심사 분리와 정보 은닉은 추론하기 쉬운 소프트웨어를 만드는 중요한 두 가지 원칙이다.
  
2. 자바9 이전에는 각각의 기능을 담당하는 패키지, 클래스, 인터페이스로 모듈화를 구현했는데 효과적인 캡슐화를 달성하기에는 역부족이었다.
  
3. 자바9에서는 새로운 모듈 시스템을 제공하는데 module-info.java 파일은 모듈의 이름을 지정하며 필요한 의존성(requires)과 공개 API(exports)를 정의한다.
  
4. requires와 exports를 제외하고도 다양한 모듈 구문이 존재한다.
