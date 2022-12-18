# chapter 10. 람다를 이용한 도메인 전용 언어

> DSL의 주요 기능은 개발자와 도메인 전문가 사이의 간격을 좁히는 것  
> DSL은 크게 내부적, 외부적 DSL로 나눌 수 있음    
> 최신자바는 자체 API에 작은 DSL을 제공    
> 자바로 DSL을 구현할 때 메서드 체인, 중첩 함수, 함수 시퀀싱 세 가지 패턴 사용

## DSL

도메인 전용 언어

dsl로 애플리케이션의 비즈니스 로직을 표현함으로 버그나 오해등을 미리 방지할 수 있음

dsl은 특정 도메인을 대상으로 만들어진 특수 프로그래밍 언어! = 특정 비즈니스 도메인을 인터페이스로 만든 api라고 볼 수 있음

메이븐, 앤트 등이 빌드 과정을 표현하는 dsl로 볼 수 있음

### 장점

- 간결함
- 가독성
- 유지보수
- 높은 수준의 추상화
- 집중
- 관심사분리

### 단점

- DSL 설계의 어려움
- 개발 비용
- 추가 우회 계층
- 새로 배워야 하는 언어
- 호스팅 언어 한계

### JVM에서 이용할 수 있는 다른 DSL 해결책

- 내부 DSL
    - 기존 자바 언어를 이용하면 외부 DSL에 비해 새로운 패턴과 기술을 배워 DSL을 구현하는 노력이 현저히 줄어들음
    - 순수 자바로 DSL을 구현하면 나머지 코드와 함께 DSL을 컴파일 할 수 있음. 따라서 다른 언어의 컴파일러를 이용하거나 외부 DSL을 만드는 도구를 사용할 필요가 없으므로 추가 비용이 들지 않음
    - 개발 팀이 새로운 언어를 배우거나 또는 익숙하지 않고 복잡한 외부 도구를 배울 필요가 없음
    - DSL 사용자는 기존의 자바 IDE를 이용해 자동 완성, 리팩터링과 같은 기능을 사용할 수 있음
    - 한개의 언어로 하나 또는 여러 도메인을 대응하지 못해 추가 DSL을 개발해야 하는 상황에서 자바를 이용하여 추가 DSL을 쉽게 합칠수 있음
- 다중 DSL
    - 새로운 프로그래밍 언어를 배우거나 또는 팀의 누군가가 해당 기술 보유해야 함
    - 두 개 이상의 언어가 혼재하므로 여러 컴파일러로 소스를 빌드하도록 빌드 과정의 개선 필요
    - 자바와의 호환성이 완벽하지 않을 수 있어 기존 컬렉션을 대상 언어의 api에 맞게 변환해야 함
- 외부 DSL
    - 자신만의 문법과 구문으로 새 언어를 설계해야 함.. → 매우 큰 작업
    - 외부 DSL이 제공하는 무한한 유연성이 가장 큰 장점

### 최신 자바 API의 작은 DSL

- 스트림 api는 컬렉션을 조작하는 dsl
- 데이터를 수집하는 dsl인 collectors

## DSL을 만드는 패턴과 기법

*예제 도메인(주식가격 Stock, 거래 Trade, 주문 Order)

### 메서드 체인

dsl에서 가장 흔한 방식 중 하나

한개의 메서드 호출로 계속해서 객체 자신이 호출되도록 → 플루언트 스타일

```java
Order order = forCustomer("BigBank")
				.buy(80)
				.stock("IBM")
				.on("NYSE")
				.at(125.00)
				.sell(50)
				.stock("GOOGLE")
				.on("NASDAQ")
				.at(375.00)
				.end();
```

```java
public class MethodChainingOrderBuilder {

    public final Order order = new Order();

    private MethodChainingOrderBuilder(String customer){
        order.setCustomer(customer);
    }
    
    public static MethodChainingOrderBuilder forCustomer(String customer){
        // 고객의 주문을 만드는 정적 팩토리 메서드
        return new MethodChainingOrderBuilder(customer);
    }
    
    public TradeBuilder buy(int quantity){
        // 주식을 사는 TradeBuilder 만들기
        return new TradeBuilder(this, Trade.Type.BUY, quantity);
    }

    public TradeBuilder sell(int quantity){
        // 주식을 파는 TradeBuilder 만들기
        return new TradeBuilder(this, Trade.Type.SELL, quantity);
    }
    
    public MethodChainingOrderBuilder addTrade(Trade trade){
        // 주문에 주식 추가
        order.addTrade(trade);
        // 유연하게 추가 주문을 만들어 추가할 수 있도록 주문 빌더 자체를 반환
        return this;
    }
    
    public Order end(){
        // 주문 만들기를 종료하고 반환
        return order;
    }
}
```

빌더를 구현해야 한다는 것이 단점

### 중첩된 함수 이용

다른 함수 안에 함수를 이용해 도메인 모델 생성

```java
Order order = order("BigBank",
									buy(80,
										stock("IBM", on("NYSE")), at(125.00)),
									sell(50,
										stock("GOOGLE", on("NASDAQ")), at(375.00))
								);
```

```java
public class NestedFunctionOrderBuilder {

    public static Order order(String customer, Trade... trades){
        // 해당 고객의 주문 만들기
        Order order = new Order();
        order.setCustomer(customer);
        // 주문에 모든 거래 추가
        Stream.of(trades).forEach(order::addTrade);
        return order;
    }
    
    public static Trade buy(int quantity, Stock stock, double price){
        // 주식 매수 거래 만들기
        return buildTrade(quantity, stock, price, Trade.Type.BUY);
    }

    public static Trade sell(int quantity, Stock stock, double price){
        // 주식 매도 거래 만들기
        return buildTrade(quantity, stock, price, Trade.Type.SELL);
    }
    
    private static Trade buildTrade(int quantity, Stock stock, double price, Trade.Type buy){
        Trade trade = new Trade();
        trade.setQuantity(quantity);
        trade.setType(buy);
        trade.setStock(stock);
        trade.setPrice(price);
        return trade;
    }
    
    // 거래된 주식의 단가를 정의하는 더미 메서드
    public static double at(double price){
        return price;
    }
    
    public static Stock stock(String symbol, String market){
        // 거래된 주식 만들기
        Stock stock = new Stock();
        stock.setSymbol(symbol);
        stock.setMarket(market);
        return stock;
    }
    
    // 주식이 거래된 시장을 정의하는 더미 메서드 정의
    public static String on(String market){
        return market;
    }
}
```

결과 DSL에 수많은 괄호,, 인수 목록을 정적 메서드에 넘겨줘야 한다는 제약도 있음

### 람다 표현식을 이용한 함수 시퀀싱

```java
Order order = order(o -> {
		o.forCustomer("BigBank");
		o.buy(t -> {
				t.quantity(80);
				t.price(125.00);
				t.stock(s -> {
						s.symbol("IBM");
						s.market("NYSE");
				});
		o.sell(t -> {
				t.quantity(50);
				t.price(375.00);
				t.stock(s -> {
						s.symbol("GOOGLE");
						s.market("NASDAQ");
				});
		});
});
```

```java
public class LambdaOrderBuilder {

    // 빌더로 주문을 감쌈
    private Order order = new Order();
    
    public static Order order(Consumer<LambdaOrderBuilder> consumer){
        LambdaOrderBuilder builder = new LambdaOrderBuilder();
        // 주문 빌더로 전달된 람다 표현식 실행
        consumer.accept(builder);
        // OrderBuilder의 Consumer를 실행해 만들어진 주문을 반환
        return builder.order;
    }
    
    public void forCustomer(String customer){
        // 주문을 요청한 고객 설정
        order.setCustomer(customer);
    }
    
    public void buy(Consumer<TradeBuilder> consumer){
        // 주식 매수 주문을 만들도록 TradeBuilder 소비
        trade(consumer, Trade.Type.BUY);
    }

    public void sell(Consumer<TradeBuilder> consumer){
        // 주식 매도 주문을 만들도록 TradeBuilder 소비
        trade(consumer, Trade.Type.SELL);
    }
    
    private void trade(Consumer<TradeBuilder> consumer, Trade.Type type){
        TradeBuilder builder = new TradeBuilder();
        builder.trade.setType(type);
        // TradeBuilder로 전달할 람다 표현식 실행
        consumer.accept(builder);
        // TradeBuilder의 Consumer를 실행해 만든 거래를 주문에 추가
        order.addTrade(builder.trade);
    }
}
```

이전 두가지 방식 장점+

플루언트 방식으로 거래 주문 정의 가능&다양한 람다 표현식의 중첩 수준과 비슷하게 도메인 객체의 계층 구조 유지

### 실생활의 자바8 DSL

| 이름 | 장점 | 단점 |
| --- | --- | --- |
| 메서드 체인 | - 메서드 이름이 키워드 인수 역할을 함
- 선택형 파라미터와 잘 동작
- DSL 사용자가 정해진 순서로 메서드를 호출하도록 강제 가능
- 정적 메서드 최소화 또는 없앰
- 문법적 잡음 최소화 | - 구현이 장황
- 빌드를 연결하는 접착 코드 필요
- 들여쓰기 규칙으로만 도메인 객체 계층 정의 |
| 중첩 함수 | - 구현의 장황함 줄임
- 함수 중첩으로 도메인 객체 계층 반환 | - 정적 메서드의 사용이 빈번
- 이름이 아닌 위치로 인수 정의
- 선택형 파라미터를 처리할 메서드 오버로딩 필요 |
| 람다를 이용한 함수 시퀀싱 | - 선택형 파라미터와 잘 동작
- 정적 메서드를 최소화 또는 없앰
- 람다 중첩으로 도메인 객체 계층 반환
- 빌더의 접착 코드 없음 | - 구현이 장황
- 람다 표현식으로 인한 문법적 잡음 존재 |

자바 라이브러리에서 …

SQL 매핑도구(JOOQ), 동작주도개발 프레임워크(큐컴버), 엔터프라이즈 통합 패턴(스프링 통합)

---