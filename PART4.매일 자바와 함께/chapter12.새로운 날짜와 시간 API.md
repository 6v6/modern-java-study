# chapter12. 새로운 날짜와 시간 API

---
* Java 8에서 새로운 날짜와 시간 라이브러리를 제공하는 이유  
* 사람이나 기계가 이해할 수 있는 날짜와 시간 표현 방법  
* 시간의 양 정의하기  
* 날짜 조작, 포매팅, 파싱  
*시간대와 캘린더 다루기  
---

## LocalDate, LocalTime, Instant, Duration, Period 클래스

### LocalDate
* LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 **불변객체**이다. 
* LocalDate 객체는 어떤 시간대 정보도 포함하지 않는다. 
* 정적 팩토리 메소드 of를 활용하여 LocalDate 인스턴스를 만들 수 있다.
* 팩토리 메소드 now()는 시스템 시계의 정보를 이용해 현재 날짜 정보를 얻는다.
~~~java
LocalDate date = LocalDate.of(2017, 9, 21);
date.getYear();
~~~

### LocalTime(시간대를 포함한 클래스)
~~~java
LocalTime time = LocalTime.of(13, 45, 20); // 13:45:20
time.getHour();
~~~

**날짜와 시간 문자열 -> LocalDate/LocalTime 생성**
~~~java
// parse 메서드 사용
LocalDate date = LocalDate.parse("2017-09-21")
LocalTime time = LocalTime.parse("13:45:20")
~~~

### LocalDateTime(날짜와 시간 조합)
* LocalDateTime은 LocalDate와 LocalTime을 갖는다.
* LocalDateTime은 날짜와 시간을 모두 표현할 수 있다.
~~~java
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
LocalDateTime dt2 = LocalDateTime.of(date, time);
~~~

### Instant 클래스(기계의 날짜와 시간)
* Instant 클래스는 유닉스 에포크 시간을 기준으로 특정 지점까지의 시간을 초로 표현한다.
* LocalDateTime은 사람이 사용하도록, Instant는 기계가 사용하도록 만들어졌다.
* 팩토리 메서드 ofEpochSecond에 초를 넘겨줘서 Instant 클래스 인스턴스를 만들 수 있다.
* Instant 클래스는 나노초의 정밀도를 제공한다.
~~~java
Instant.ofEpochSecond(3);
Instant.ofEpochSecond(2, 1_000_000_000); // 2초 이후의 1억 나노초(1초)
~~~

### Duration과 Period 정의
* 지금까지 살펴본 모든 클래스는 Temporal인터페이스를 구현한다.
* Temporal 인터스페이스는 특정 시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의한다.

+ Duration클래스는 between으로 두시간 객체 사이의 지속시간을 만들수 있다.
~~~java
Duration d1 = Duration.between(time1, time2);
Duration d1 = Duration.between(dateTime1, dateTime2);
Duration d1 = Duration.between(instant1, instant2);
~~~

* Period 클래스의 팩토리 메서드 between을 이용하면 두 LocalDate의 차이를 확인할 수 있다.
~~~java
Period tenDays = Period.between(LocalDate.of(2017, 9, 11), LocalDate.of(2017, 9, 21));
~~~
* Duration과 Period 클래스는 자신의 인스턴스를 만들 수 있도록 다양한 팩토리 메서드를 제공한다.
~~~java
Duration threeMinutes = Duration.ofMinutes(3);
Duration threeMinutes = Duration.of(3, ChronoUnit.MINUTES);
Period tenDays = Period.ofDays(10);
Period threeWeeks = Period.ofWeeks(3);
~~~

---
* 지금까지 살펴본 모든 클래스는 불변이다.  
+ 불변 클래스는 함수형 프로그래밍과 스레드 세이프하여 도메인 모델의 일관성을 유지하는데 좋은 특성이다.
+ 새로운 날짜와 시간 API에서는 **변경된 객체 버전**을 만들 수 있는 메서드를 제공한다.
--- 
## 날짜 조정, 파싱, 포메팅
* withAttribte 메서드로 기존의 LocalDate를 바꾼 버전을 직접 간단히 만들 수 있다.
* 모든 메서드는 기존 객체를 변경하지 않는다.

**절대적인 방식으로 LocalDate의 속성 바꾸기**
~~~java
LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date2 = date1.withYear(2011); // 2011-09-21
LocalDate date3 = date2.withDayOfMonth(25); // 2011-09-25

// 첫 번째 인수로 TemporalField를 갖는 메서드를 사용하면 조금더 범용적으로 날짜를 변경할 수 있다.
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 2); // 2011-02-25
~~~
  
**상대적인 방식으로 LocalDate 속성 바꾸기**
~~~java
LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date2 = date1.plusWeek(1); // 2017-09-28
LocalDate date3 = date2.minusYear(6); // 2011-09-28
LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS); // 2012-03-28
~~~

### TemporalAdjusters 사용하기
+ Java는 다음주 일요일, 돌아오는 평일등 복잡한 날짜 조정기능을 지원한다.
+ 날짜와 시간 API는 다양한 상황에서 사용할 수 있도록 **TemporalAdjuster** 제공함
~~~java
LocalDate date1 = LocalDate.of(2014, 3, 18);
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY));
LocalDate date3 = date2.with(lastDayOfMonth());
~~~


**커스텀 TemporalAdjuster 구현**

~~~java
/* TemporalAdjuster 인터페이스 */
// 함수형 인터페이스로 되어 있으므로 adjustInto만 구현하면 된다.
@FunctionalInterface
public interface TemporalAdjuster {
    Temporal adjustInto(Temporal temporal);
}

/* Working Day 만 구하는 커스텀 TemporalAdjuster */
@Override
public Temporal adjustInto(Temporal temporal) {
    DayOfWeek today = DayOfWeek.of(temporal.get(ChronoField.DAY_OF_WEEK)); // 현재 날짜 읽기
    int dayToAdd = 1;
    
    if (today == DayOfWeek.FRIDAY) {
        dayToAdd = 3;   
    } else if (today == DayOfWeek.SATURDAY) {
        dayToAdd = 2;
    }
    
    return temporal.plus(dayToAdd, ChronoUnit.DAYS);
}
~~~

## 날짜와 시간 객체 출력과 파싱
* DateTimeFormmatter 클래스
  * BASIC_ISO_DATE와 ISO_LOCAL_DATE등의 상수를 미리 정의하고 있음
  * DateTimeFormmatter를 활용하여 날짜나 시간을 특정 형식의 문자열로 만들 수 있다.
  * 스레드 세이프 한 클래스이며 특정 패턴으로 포매터를 만들 수 있는 정적 팩토리 메서드도 제공한다.
~~~java
LocalDate date = LocalDate.of(2014, 3, 18);
date.format(DateTimeFormatter.BASIC_ISO_DATE); // 20140318
date.format(DateTimeFormatter.ISO_LOCAL_DATE); // 2014-03-18

// 날짜나 시간을 표현하는 문자열을 파싱하여 날짜 객체를 다시 만들 수 있다.
LocalDate parse = LocalDate.parse("20140318", DateTimeFormatter.BASIC_ISO_DATE);
LocalDate parse2 = LocalDate.parse("2014-03-18", DateTimeFormatter.ISO_LOCAL_DATE);

// 정적 팩토리 메서드
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate localDate = LocalDate.of(2014, 3, 18);
String formattedDate = localDate.format(formatter);
LocalDate parse1 = LocalDate.parse(formattedDate, formatter);
~~~


**지역화 된 DateTimeFormatter도 만들기**  
조금더 복합적인 포매터를 만들기 위해 Builter를 사용할 수 있다.
~~~java
DateTimeFormatter italianFormatter = new DateTimeFormatterBuilder()
    .appendText(ChronoField.DAY_OF_MONTH)
    .appendLiteral(". ")
    .appendText(ChronoField.MONTH_OF_YEAR)
    .appendLiteral(" ")
    .appendText(ChronoField.YEAR)
    .parseCaseInsensitive() // 정해진 형식과 정확하게 일치하지 않아도 해석가능
    .toFormatter(Locale.ITALIAN);
~~~

## 다양한 시간대와 캘린더 활용 방법
* 새로운 날짜와 시간 API의 편리함 중 하나는 시간대를 간단하게 처리할 수 있다는 점이다.
* 기존 TimeZone을 대체할 수 있는 ZoneId 클래스가 새롭게 등장했다.
* 새로운 클래스를 이용하면 서머타임같은 복잡한 사항이 자동으로 처리된다. 
### 시간대 이용하기
* 표준 시간이 같은 지역을 묶어 시간대 규칙 집합을 정의한다.
* ZoneRules 클래스에는 약 40개 정도의 시간대가 있다.
* ZoneId의 getRules()를 이용해 해당 시간대의 규정을 획득할 수 있다.  
`ZoneId romeZone = ZoneId.of("Europe/Rome");`  
+ 지역 Id는 지역/도시형식으로 이루어지며 IANA Time Zone Database에서 제공하는 지역 집합 정보를 사용한다.  
`ZoneId zoneId = TimeZone.getDefault().toZoneId();`  

### UTC/Greenwich 기준의 고정 오프셋
* UTC / GMT 를 기준으로 시간대를 표현하기도 한다.
`ZoneOffset.of("-5:00"); // 현재 타임존 보다 5시간 느린 곳 정의`
* 위 예제에서 정의한 ZoneOffset으로는 서머타임을 정확히 처리할 수 없으므로 권장하지 않는다. 
* ZoneOffset은 ZoneId이므로 ZoneOffset을 사용할 수 있다. 
* ISO-8601 캘린더 시스템에서 정의하는 UTC/GMT와 오프셋으로 날짜와 시간을 표현하는 OffsetDateTime을 만들수 있다.

---
* Java 8 이전에서 제공하는 기존 Date 클래스와 관련 클래스는 결함이 존재했다.
* 새로운 날짜와 시간 API에서 날짜와 시간 객체는 모두 불변클래스이다.
* 새로운 API는 각각 사람과 기계가 편리하게 날짜와 시간 정보를 관리할 수 있으며 기존 인스턴스를 변환하지 않도록 처리 결과로 새로운 인스턴스가 생성된다.
* TemporalAdjuster를 이용하면 단순히 값을 바꾸는 것 이상의 복잡한 동작을 수행할수 있으며 자신만의 커스텀 날짜 변환 기능을 정의할 수 있다.
* 날짜와 시간 객체를 특정 포맷으로 출력하고 파싱하는 포매터를 정의 할 수 있다. 패턴을 이용할 수 있고 포매터는 스레드 세이프하다.
* 특정 지역/장소에 상대적인 시간대 또는 UTC/GMT 기준 오프셋을 이용해 시간대를 정의할 수 있다.
* ISO-8601 표준 시스템을 준수하지 않는 캘린더 시스템도 이용할 수 있다.
---
