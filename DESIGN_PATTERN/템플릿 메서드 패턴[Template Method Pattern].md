# 템플릿 메서드 패턴 \[Template Method Pattern\]
추상클래스를 사용해 알고리즘의 틀을 정의하며, 추상 클래스를 상속받은 서브 클래스가 핵심 로직을 재정의 하도록 하는 패턴.

- 알고리즘의 틀(골격)을 정의한다.
- 알고리즘의 구조는 그대로 유지하면서 알고리즘의 특정 단계를 서브클래스에서 재정의할 수도 있다.
-  템플릿 메소드는 알고리즘의 각 단계를 정의하며, 서브클래스에서 일부 단계를 구현할 수 있도록 유도한다.
- `추상 클래스를 통해 코드의 중복 해결`
    - 코드의 변경이 필요 할 때 한 곳에서만 수정하면 된다.(단일 책임의 원칙)
- 아무것도 하지 않는 구상 메서드를 정의 할 수 있다. : `후크(hook)`
    - 서브클래스에서 오버라이드 할 수 있지만, 필수는 아니다.
    - 별 내용 없는 기본 메서드를 구현하면 서브클래스에서 필요에 따라 원하는 기능을 넣을 수 있다.


<br/>


> **템플릿 메서드 패턴의 목적**
> 
> 작업에서 알고리즘의 골격을 정의하고 일부 단계를 하위 클래스로 연기합니다.   
> 템플릿 메서드를 사용하면 하위 클래스가 알고리즘의 구조를 변경하지 않고도 알고리즘의 특정 단계를 재정의 할 수 있습니다.  
> 
> \- GOF 디자인패턴의 정의

  
<br/>
<br/>


## 변하는 것과 변하지 않는 것을 분리

- 좋은 설계는 변하는 것과 변하지 않는 것을 분리하는 것이다.  
- 변하는 것 : 핵심 비즈니스 로직
- 변하지 않는 것 : 로그와 같은 부수적인 로직
- 우리는 이 둘을 분리해서 모듈화 해야한다.

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

결과

``` log
00:20:02.789 [Test worker] INFO hello.advanced.trace.template.TemplateMethodTest - 비즈니스 로직1 실행
00:20:02.801 [Test worker] INFO hello.advanced.trace.template.TemplateMethodTest - resultTime=13
00:20:02.805 [Test worker] INFO hello.advanced.trace.template.TemplateMethodTest - 비즈니스 로직2 실행
00:20:02.806 [Test worker] INFO hello.advanced.trace.template.TemplateMethodTest - resultTime=1
```

- 위 코드를 보면 비즈니스로직을 실행하는 부분을 제외하고는 코드가 같은 걸 볼 수 있다.  
- 정작 코드가 변하는 부분은 비즈니스 로직을 실행하는 부분 뿐이다.  
- 템플릿 메소드 패턴을 통해서 변하는 부분과 변하지 않는 부분을 분리해보자.  

<br/>

### 템플릿 메소드 패턴을 적용한 코드
- 추상클래스를 사용해 중복되는 코드를 미리 구현하였다.
- 비즈니스 로직은 해당 추상클래스를 상속한 서브클래스가 구현하도록 call() 이라는 추상메소드로 분리해주었다.
- execute()만 실행해주면 메소드 동작 시간을 확인할 수 있다.
- `중복되는 코드 없이 비즈니스 로직에만 집중할 수 있어졌다.`
- **단일 책임 원칙(SRP)**   
로그를 남기는 부분에 단일 책임 원칙을 지키게 되었다.


``` java
@Slf4j
public abstract class AbstractTemplate {

    // 템플릿 메소드
    public void execute() {
        long startTime = System.currentTimeMillis();

        //비즈니스 로직 실행
        call(); //상속
        //비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={}", resultTime);
    }

    protected abstract void call();
}
```

```java
@Slf4j
public class SubClassLogic1 extends AbstractTemplate{
    @Override
    protected void call() {
        log.info("비즈니스 로직1 실행");
    }
}

@Slf4j
public class SubClassLogic2 extends AbstractTemplate{
    @Override
    protected void call() {
        log.info("비즈니스 로직2 실행");
    }
}
```

결과 - 이전과 같은 결과가 나옴
``` log
00:27:28.634 [Test worker] INFO hello.advanced.trace.template.code.SubClassLogic1 - 비즈니스 로직1 실행
00:27:28.642 [Test worker] INFO hello.advanced.trace.template.code.AbstractTemplate - resultTime=9
00:27:28.646 [Test worker] INFO hello.advanced.trace.template.code.SubClassLogic2 - 비즈니스 로직2 실행
00:27:28.646 [Test worker] INFO hello.advanced.trace.template.code.AbstractTemplate - resultTime=0
```

<br/>

### 익명 내부 클래스 사용하기  
- 일일히 상속받은 서브 클래스를 만들어야 하는 일을 줄여보자. 
- 익명 내부 클래스를 사용하면 객체 인스턴스를 생성하면서 동시에 생성할 클래스를 상속 받은 자식 클래스를 정의할 수 있다.
-  이 클래스는 직접 이름을 지정할 일이 없고 클래스 내부에 선언되는 클래스여서 익명 내부 클래스라 한다.


``` java
@Test
    void templateMethodV2() {
        AbstractTemplate template1 = new AbstractTemplate() {
            @Override
            protected void call() {
                log.info("비즈니스 로직1 실행");
            }
        };

        log.info("클래스 이름1={}", template1.getClass()); //클래스 이름1=class hello.advanced.trace.template.TemplateMethodTest$1
        template1.execute();

        AbstractTemplate template2 = new AbstractTemplate() {
            @Override
            protected void call() {
                log.info("비즈니스 로직2 실행");
            }
        };

        log.info("클래스 이름2={}", template2.getClass()); //클래스 이름1=class hello.advanced.trace.template.TemplateMethodTest$2
        template2.execute();
    }
```

<br/>  
<br/>

## 좋은 설계란?
- 진정한 좋은 설계는 바로 “변경"이 일어날 때 자연스럽게 드러난다. 
- 변경에 쉽게 만드는게 좋은 설계라고 본다.

<br/>

### 템플릿 메서드 패턴의 장점
- `단일 책임 원칙(SRP)` 을 지킬 수 있게 된다.
- 자식 클래스가 알고리즘의 전체 구조를 변경하지 않고, 특정 부분만 재정의할 수 있다. 
- 상속과 오버라이딩을 통한 다형성으로 문제를 해결

<br/>
  
### 템플릿 메서드 패턴의 단점

- 상속의 단점을 그대로 가지게 된다.
    - 부모 클래스에 의존하는 것이다.
    - 부모 클래스를 수정하면, 자식 클래스에도 영향을 줄 수 있다.
- 자식 클래스 입장에서는 부모 클래스 기능을 전혀 사용하지 않지만 부모 클래스의 기능을 상속 받게 된다. 따라서 부모 클래스의 기능 사용 여부에 상관 없이 부모 클래스를 강하게 의존하게 된다. 

<br/>

### 추상 메서드를 써야할 때와 후크를 써야할 때의 구분점  
- 서브클래스가 해당 로직을 필수적으로 구현해야한다면 추상 메서드를 사용
- 해당 로직에 대해 선택적이라면 후크를 사용
- 후크는 오버라이드가 필수적이지 않다.

<br/>
<br/>

### 코드 출처 및 참고
- 김영한님의 스프링 핵심 원리[고급편]