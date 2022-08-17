# ThreadLocal

### 일반적인 변수 필드 사용 시 문제점

- 여러 쓰레드가 같은 인스턴스의 필드에 값을 접근하면 처음 쓰레드가 보관한 데이터가 사라질 수 있다.  

<br/> 

## ThreadLocal이란

- 해당 쓰레드만 접근할 수 있는 특별한 저장소를 말한다.
- ex) 여러 사람이 같은 물건 보관 창구를 사용하더라도 창구 직원은 사용자를 인식해서 사용자별로 확실하게 물건을 구분해준다.

- 각 쓰레드마다 별도의 내부 저장소를 제공한다. 

- 같은 인스턴스의 쓰레드 로컬 필드에 접근해도 문제 없다.
- 쓰레드마다 다른 저장소를 사용하기 때문데 데이터 무결성이 보장된다.

<br/>
<br/>

## ThreadLocal 사용법

- 저장 : ThreadLoca.set(xxx);
- 조회 : ThreadLocal.get()
- 제거 : ThreadLocal.remove()

<br/>
<br/> 

## 사용해보기

### 문제코드

1. Runnable을 구현한 클래스를 하나 생성한다.  
      
    

```
@Slf4j 
public class PrintRun implements Runnable {

    private NameStorage nameStorage;
    private String threadName;

    public PrintRun(NameStorage nameStorage, String threadName) {
        this.nameStorage = nameStorage;
        this.threadName = threadName;
    }

    @Override
    public void run() {
        nameStorage.setMyThreadName(threadName);

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        nameStorage.printThreadName();
    }
}
```

  

2\. 이름저장소를 하나 생성한다.

```
public class NameStorage {

    private  String threadName;

    public void setMyThreadName(String newThreadName) {
        log.info("Change my thread {} to {}.", threadName.get(), newThreadName);
        this.threadName = newThreadName;
    }

    public void printThreadName() {
        log.info("thread name={}", threadName);
    }
}

```

  

3\. 각각의 Runnable 객체에 같은 nameStore와 다른 Thread\*라는 이름을 주입해준다. 

그 다음 쓰레드를 순차적으로 실행시켜준다.

```
public class TestRun {
    public static void main(String[] args) throws InterruptedException {
        NameStorage nameStorage = new NameStorage();

        Thread thread1 = new Thread(new PrintRun(nameStorage, "Thread1"));
        Thread thread2 = new Thread(new PrintRun(nameStorage, "Thread2"));

        thread1.start();
        Thread.sleep(1000);
        thread2.start();
    }
}

```

  
<br/>

- 예상되는 결과 : Thread1은 Therad1을 출력하고 Thread2는 Thread2를 출력한다.
- 실제 결과 : Thread1과 Thread2 모두 Thread2를 출력하였다.

```
15:12:42.860 [Thread-0] INFO ffs.test.TestThreadLocal - Change my thread null to Thread1.
15:12:43.861 [Thread-1] INFO ffs.test.TestThreadLocal - Change my thread Thread1 to Thread2.
15:12:43.870 [Thread-0] INFO ffs.test.PrintRun - thread name=Thread2
15:12:44.861 [Thread-1] INFO ffs.test.PrintRun - thread name=Thread2

```

- Thread1이 sleep에 걸린 사이에 Thread2가 접근하여 threadName을 변경했기 때문이다.
- 로그의 두번째줄을 보면 Thread1에서 Thread2로 이름을 바꾼 것을 확인할 수 있다.  
    
- 이 문제를 해결하기 위해 ThreadLocal을 사용해보자

  
<br/>
  

### ThreadLocal 적용 코드

- 공유해서 사용하는 NameStorage의 threadName 필드에 ThreadLocal을사용해보자.

```
@Slf4j
public class NameStorage {

    private  ThreadLocal<String> threadName;

    public NameStorage() {
        this.threadName = new ThreadLocal<>();
    }

    public void setMyThreadName(String newThreadName) {
        log.info("Change my thread {} to {}.", threadName.get(), newThreadName);
        this.threadName.set(newThreadName);
    }

    public void printThreadName() {
        log.info("thread name={}", threadName.get());
    }
}

```

  

```
public class TestRun {
    public static void main(String[] args) throws InterruptedException {
        NameStorage nameStorage = new NameStorage();

        Thread thread1 = new Thread(new PrintRun(nameStorage, "Thread1"));
        Thread thread2 = new Thread(new PrintRun(nameStorage, "Thread2"));

        thread1.start();
        Thread.sleep(1000);
        thread2.start();
    }
}

```

 <br/> 

- 예상되는 결과 : Thread1은 Therad1을 출력하고 Thread2는 Thread2를 출력한다.
- 실제 결과 : Thread1은 Therad1을 출력하고 Thread2는 Thread2를 출력한다.

```
15:15:00.123 [Thread-0] INFO ffs.test.TestThreadLocal - Change my thread null to Thread1.
15:15:01.124 [Thread-1] INFO ffs.test.TestThreadLocal - Change my thread null to Thread2.
15:15:01.132 [Thread-0] INFO ffs.test.PrintRun - thread name=Thread1
15:15:02.124 [Thread-1] INFO ffs.test.PrintRun - thread name=Thread2

```

- 예상 결과와 실제 결과가 같이 나온 것을 확인할 수 있다.
- 각 쓰레드마다 각각의 저장공간을 사용하기 때문에 서로의 필드값에 관여하지 않게 된다.
- 쓰레드 로컬 덕분에 쓰레드마다 각각 별도의 데이터 저장소를 가지게 되었으며 동시성 문제도 해결되었다.

  
<br/>
<br/>
  

## ThreadLocal 사용 시 주의사항

- **ThreadLocal.remove()** 
- 쓰레드 로컬을 모두 사용하고 나면 꼭 쓰레드 로컬에 저장된 값을 제거해주어야 한다.
- 쓰레드 로컬은 전용 보관소에 값을 보관하고 있기 때문에 메모리 문제가 발생 할 수 있다.
- 쓰레드 로컬의 값을 사용 후 제거하지 않으면 WAS(톰캣)처럼 쓰레드 풀을 사용하는 경우에 심각한 문제가 발생할 수 있다.
- 쓰레드를 생성하는 비용은 비싸기 때문에 쓰레드를 제거하지 않고, 보통 쓰레드 풀을 통해서 쓰레드를 재사용한다.
- 쓰레드 로컬의 값을 제거하지 않으면 쓰레드 풀을 통해 새로운 쓰레드를 실행 할 때, 이전 쓰레드가 사용했던 값이 조회되게 된다.
- 이런 문제를 예방하려면 쓰레드 사용이 끝날 때 쓰레드 로컬의 값을 ThreadLocal.remove()를 통해서 꼭 제거해야 한다.