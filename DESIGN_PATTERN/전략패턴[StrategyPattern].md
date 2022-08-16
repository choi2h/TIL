# 전략패턴 \[Strategy Pattern\]

인터페이스 위임을 통해 핵심 로직을 구현하며, 필요에 따라 구현한 클래스를 주입하여 사용한다.

- 컨텍스트의 코드 변경 없이 새로운 전략을 취할 수 있다.
- 코드 수정 시 뼈대가 되는 코드를 변경하지 않아도 된다.
    - 전략 코드만 변경해주면 된다. (개방 폐쇄 원칙을 지킬 수 있다.)
- 클래스에 필요한 전략을 주입(조립)해준 후, 실행한다. : `선 조립, 후 실행`


<br/>

> 전략패턴의 의도  
> 알고리즘 제품군을 정의하고 각각을 캡슐화하여 상호 교환 가능하게 만들자.  
> 전략을 사용하면 알고리즘을 사용하는 클라이언트와 독립적으로 알고리즘을 변경할 수 있다.  
> \- GOF 디자인패턴의 정의

<br/>
<br/>

## 변하는 것과 변하지 않는 것을 분리

- 아래 코드를 보면 비즈니스로직을 실행하는 부분을 제외하고는 코드가 같은 걸 볼 수 있다.  
- 정작 코드가 변하는 부분은 비즈니스 로직을 실행하는 부분 뿐이다.  
- 전략 패턴을 통해서 필요한 상황에 따른 비즈니스 로직을 선택해보자.

### 기존 코드
``` java
    void templateMethod() {
        logic1();
        logic2();
    }

    private void logic1() {
        long startTime = System.currentTimeMillis();

        //비즈니스 로직 실행
        log.info("비즈니스 로직1 실행");
        //비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={}", resultTime);
    }

    private void logic2() {
        long startTime = System.currentTimeMillis();

        //비즈니스 로직 실행
        log.info("비즈니스 로직2 실행");
        //비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={}", resultTime);
    }
```

<br/>
 
### 전략 패턴을 이용한 코드
- 공통되는 로직은 클래스에 작성한다.
- 이 클래스는 Strategy라는 interface에만 의존하고 있다.
- 필요에 따라 인터페이스를 위임받아 구현한다.
- 클래스 사용 시 구현한 클래스를 주입해주면 된다.
    - 생성자 주입 방식
        - 클래스 생성 시 사용할 Strategy를 구현한 클래스만 잘 주입해주면 된다.
        - 조립한 이후에는 전략을 변경하기 어렵다.
    - 파라미터 주입 방식
        - execute() 실행 시 Strategy를 구현한 클래스만 잘 주입해주면 된다.
        - 실행시마다 전략을 변경할 수 있다.


```java
// 생성자 주입 방식
public class ContextV1 {

    private Strategy strategy;

    public ContextV1(Strategy strategy) {
        this.strategy = strategy;
    }

    public void execute() {
        long startTime = System.currentTimeMillis();

        //비즈니스 로직 실행
        strategy.call();
        //비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={}", resultTime);
    }
}
```

```java
// 파라미터 주입 방식
public class Context {

    public void execute(Strategy strategy) {
        long startTime = System.currentTimeMillis();

        //비즈니스 로직 실행
        strategy.call();
        //비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={}", resultTime);
    }
}
```


<br/>
인터페이스

```java
public interface Strategy {

    void call();
}
```

인터페이스를 구현한 클래스들

```java
public class StrategyLogic1 implements Strategy{

    @Override
    public void call() {
        log.info("비즈니스 로직1 실행");
    }
}

public class StrategyLogic2 implements Strategy{
    
    @Override
    public void call() {
        log.info("비즈니스 로직2 실행");
    }
}
```

<br/>

전략패턴의 사용
```java
//생성자 주입 방법
void strategyV1() {
        StrategyLogic1 strategyLogic1 = new StrategyLogic1();
        ContextV1 context1 = new ContextV1(strategyLogic1);
        context1.execute();

        StrategyLogic2 strategyLogic2 = new StrategyLogic2();
        ContextV1 context2 = new ContextV1(strategyLogic2);
        context2.execute();
}

//파라미터 주입 방법
void strategyV2() {
    ContextV2 contextV2 = new ContextV2();
    contextV2.execute(new StrategyLogic1());
    contextV2.execute(new StrategyLogic2());


    // 전략패턴도 마찬가지로 익명 클래스를 사용하여 클래스 생성을 줄일 수 있다.
    contextV2.execute(new Strategy() {
                @Override
                public void call() {
                    log.info("비즈니스 로직3 실행");
                }
    });

    // 람다를 사용하면 더 간결하게 표현할 수 있다.
    contextV2.execute(() -> log.info("비즈니스 로직4 실행"));
}
```

<br/>
<br/>

## 전략 패턴의 장단점

### 전략 패턴의 장점
- 런타임에 전략을 변경할 수 있다.
- 확장에 유리한 코드를 작성할 수 있다.
    - 상속 대신 인터페이스의 사용
- 새로운 전략을 추가하더라도 기존 코드가 변경되지 않는다.

<br/>

### 전략 패턴의 단점
- 어플리케이션에 들어가는 모든 전략을 파악하고 있어야 한다.
- 알고리즘이 늘어날 수록 객체도 무한정 늘어나게 된다.