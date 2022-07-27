# ORM (Object Relational Mapping, 객체 - 관계 매핑)

여기서 말하는 객체란 우리가 흔히 알고있는 OOP의 객체를 말하며 ORM은 이 객체와 관계형 데이터베이스와의 데이터를 자동으로 매핑해주는 것이다.

<br/>

## 기존 SQL방식의 문제점

- 관계형 데이터베이스와 객체지향은 지향하는 목적이 서로 다르다.
    - 객체는 참조를 통해서 다른 객체와 연관관계를 갖지만 데이터베이스는 외래키를 사용하여 다른 테이블과 연관관계를 갖는다.  
    SQL방식을 사용한다면 설계 시 객체지향 방식이 아닌 데이터베이스의 설계에 따라서 객체를 설계하게 된다.

        ```java
        public class Member{
            private String name;
            private int age;
            private int teamId; //외래키 참조
        }

        public class Member{
            private String name;
            private int age;
            private Team team; // 객체 참조
        }
        ```
   
    - 객체는 상속이라는 개념이 존재하나 데이터베이스에는 상속이라는 개념이 존재하지 않는다.  
    상속 관계의 객체를 저장 및 조회하기 어려워지며 개발자가 중간에서 직접 가공해주어야 한다.

<br/>

## ORM 사용의 장점

- 객체지향적인 설계를 하여 더 직관적인 코드 작성이 가능해진다.
    - 객체 간의 관계를 바탕으로 SQL을 자동으로 생성하기 때문에 RDBMS의 데이터 구조와 Java의 객체지향 모델 사이의 간격을 좁힐 수 있다.
- SQL QUERY를 일일이 다 작성하지 않고 직관적인 코드로 데이터 조작이 가능해지며 비즈니스 로직에 더 집중할 수 있게 도와준다.
- DBMS에 대한 종속성이 줄어든다.
    - DBMS마다 SQL표준을 지키지 않은 고유한 기능이 존재한다. 기존 SQL Mapping방식을 사용하면 DBMS변경 시 일일이 수정해야만 했다.  
    그러나 ORM은 어느 DB에도 종속적이지 않으며 데이터베이스 변경 시 손쉽게 교체가 가능하다.
- 재사용 및 유지보수 편리성이 증가한다.
    직접 SQL을 일일이 확인하지 않아도 되며 매핑정보가 명확하여 ERD를 보는 것에 대한 의존도를 낮출 수 있다.
- 자바에서 가공할 경우 equals, hashCode의 오버라이드 같은 자바 기능이 사용 가능하여 간결하고 빠른 가공이 가능하다.

<br/>

>  DBMS(Database Management System)란?  
>  사용자와 데이터베이스 사이에서 사용자의 요구에 따라 정보생성 및 데이터베이스 관리를 해주는 소프트웨어  
> 
>  RDBMS(Relational Database Management System)란?  
>  관계형 데이터베이스 관리 시스템을 의미하며 관계형 데이터 모델을 기초로 모든 데이터를 2차원 테이블 형태로 표현하는 데이터베이스

<br/>

## ORM 사용의 단점
- 완벽히 ORM만으로는 구현하기가 어렵다.
    프로젝트의 복잡성이 높아질 수록 ORM을 다루는 난이도가 올라갈 수 있으며 잘못 구현될 경우 심한 속도 저하 및 일관성이 무너지는 문제점이 발생할 수 있다.
- 프로시저가 많은 시스템에선 ORM의 객체지향적인 장점을 활용하기 어렵다.

> [프로시저란?](./프로시저란.md)

<br/>
<br/>

### 참고
http://www.incodom.kr/ORM
https://gmlwjd9405.github.io/2018/12/25/difference-jdbc-jpa-mybatis.html
https://velog.io/@dnjscksdn98/Database-ORM이란