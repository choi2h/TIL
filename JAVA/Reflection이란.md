# Reflection

구체적인 클래스 타입을 알지 못해도 그 클래스의 정보(메서드, 타입, 변수 등등)에 접근할 수 있게 해주는 자바 API

<br/>

### 사용하는 이유
자바는 정적언어로 부족한 부분이 있음   
→ 이 때, 동적인 문제를 해결하기 위해서 사용
런타임 시점에 지금 실행되고 있는 클래스를 가져와서 실행해야 하는 경우

<br/>

> 정적 언어? 동적 언어?  
> 정적 언어: 컴파일 시점에 타입을 결정  
>  ex) JAVA, C, C++ ,,,  
> 동적 언어: 런타임 시점에 타입을 결정   
> ex) Javascript, Python, Ruby...

<br/>

리플렉션은 애플리케이션 개발보다는 프레임워크 라이브러리에서 많이 사용된다.
프레임워크, 라이브러리를 사용하는 사람이 어떤 클래스를 만들지 모르기때문에  이를 동적으로 해결해주기 위해 리플렉션 사용하게 된다.  
ex) Spring DI, Proxy, ModelMapper,,,

<br/>

``` java
public class Book {
     private final String name;
     private int count;
     private int price;

     public Book(String name, int count, int price) {
         this.name = name;
         this.count = count;
         this.price = price;
     }

     public void reduceCount() {
          this.count--;
     }

     public String getName() {
         return name;
     }

    public int getCount() {
             return count;
         }

    public int getPrice() {
             return price;
         }
}

public static void main(String[] args) {
    Object obj = new Book("Java", 10, 20000);
}
```

모든 객체는 Object의 하위 클래스이므로 위와 같이 Book 객체 생성이 가능하다. 그러나 obj는 Book의 메서드는 사용이 불가능하다. 

**불가능한 이유 : 컴파일러가 있는 자바는 구체적인 클래스를 모르면 해당 클래스의 정보에 접근할 수 없다.**
- 자바는 컴파일러를 사용하여 컴파일 타임에 타입이 결정된다.
- obj라는 이름의 객체는 컴파일 타임에 Object라는 타입으로 결정되었기 때문에 Object 클래스의 인스턴스 변수와 메서드만 사용 가능하다.
- obj는 Book클래스라는 구체적인 타입은 모른다.


이런 문제를 가능하게 해주는 기능이 Reflection이다.

<br/>

## Reflection 접근 방법

1. Method 접근  
book 생성 시 10으로 초기화했던 Book의 count가 메서드 실행 후 9가 된 것을 확인할 수 있다.
    ```java
            Object obj = new Book("Java1", 10,  20000);
            Class bookClass = Book.class;

            Method reduceCount = bookClass.getMethod("reduceCount");

            //reduceCount 메서드 실행 -> invoke(객체, 매개변수) : 매개변수를 허용하지 않으면 null을 인수로 전달
            reduceCount.invoke(obj, null);

            Method getCount = bookClass.getMethod("getCount");
            int count = (int) getCount.invoke(obj, null);
            System.out.println(count); // 9
    ```



2. 생성자 접근
    ```java
    Class bookClass2 = Class.forName("test.web.socket.reflection.Book");
            Constructor<? extends Book> fullCarConstructor = bookClass2.getConstructor(String.class, int.class, int.class);
            Book book = fullCarConstructor.newInstance("Java2", 10, 3000);

            System.out.println("Book="+book.getName()+" bookCount="+book.getCount()); //Book=Java2 bookCount=10
    ```

3. 필드 접근
    ```java
            Class bookClass3 = Class.forName("test.web.socket.reflection.Book");
            Field[] fields = bookClass3.getDeclaredFields();
            for(Field f : fields) {
                System.out.println("Field: " + f);
            }
            /*
                Field: private final java.lang.String test.web.socket.reflection.Book.name
                Field: private int test.web.socket.reflection.Book.count
                Field: private int test.web.socket.reflection.Book.price
            */  
    ```

<br/>

### 어떻게 가능한가?
JAVA에서 JVM이 실행되면 사용자가 작성한 자바 코드가 컴파일러를 거쳐 바이트 코드로 변환되어 static 영역에 저장된다. Reflection은 이 정보를 활용하며 클래스 이름만 알고 있다면 언제든 static 영역을 뒤져 정보를 가져온다.

<br/>

### 활용 방안
우리가 코드를 작성하면서 리플렉션을 활용할 일은 거의 없다.
애플리케이션 개발보다는 사용자가 어떤 클래스를 만들지 예측할 수 없는 프레임워크나 라이브러리에 많이 사용된다.

<br/>

### 장점
- 확장성 기능  
정규화된 이름을 사용하여 확장성 개체의 인스턴스를 만들어 외부 사용자 정의 클래스를 사용할 수 있다.


### 단점
- 성능 오버헤드 발생  
컴파일 타임이 아닌 런타임에 동적으로 타입을 분석하고 정보를 가져오므로 JVM을 최적화 할 수 없다.  
- 내부 노출   
접근할 수 없는 private 인스턴스 변수, 메서드에 접근할 수 있기 때문에 추상화가 깨진다.

### 주의할점
- Reflection은 런타임 시 객체의 타입이 결정되기 때문에 컴파일 시 문제점을 체크할 수 없다. Reflection 사용 시 잘못된 파라미터를 전달한다면 런타임 시 에러를 발생시킬 것이다. 때문에 충분한 디버깅을 실행해봐야 한다.

<br/>

### 참고
https://tecoble.techcourse.co.kr/post/2020-07-16-reflection-api/  
https://www.geeksforgeeks.org/reflection-in-java/


