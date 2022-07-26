# CH02. JPA 시작

## 라이브러리와 프로젝트 구조
JPA구현체로 하이버네이트를 사용하기위한 핵심 라이브러리

- hibernate-core : 하이버네이트 라이브러리
- hibernate-entitymanager : 하이버네이트가 JPA 구현체로 동작하도록 JPA 표준을 구현한 라이브러리
- hibernate-jpa-2.1-api : JPA 2.1 표준 API를 모아둔 라이브러리

<br/>  

하이버네이트 구현체 사용을 위한 핵심 라이브러리

- JPA, 하이버네이트  
JPA 표준과 하이버네이트를 포함하는 라이브러리, hibernate-entitymanager를 라이브러리로 지정하면 다음 중요 라이브러리도 함께내려받는다.
    - hibernate-core.jar
    - hibernate-jpa-2.1-api.jar
- H2 데이터베이스  
H2 데이터베이스에 접속해야하므로 h2라이브러리도 지정

<br/>
<br/>

## 객체 매핑 시작
JPA 어노테이션 패키지 :  javax.persistence

<br/>

객체 매핑을 위한 어노테이션

- @Entity  
    이 클래스를 테이블과 매핑한다고 JPA에게 알려주는 어노테이션으로 이 클래스를 엔티티 클래스라 한다.
- @Table  
    엔티티 클래스에 매핑할 테이블 정보를 알려준다. 이 어노테이션 생략 시 클래스 이름을 테이블 이름으로 매핑한다.  
    

    - name 속성 : 테이블명을 작성하여 해당 테이블과 매핑을 한다.

- @Id  
    엔티티 클래스의 필드를 테이블의 기본키에 매핑한다. 이 필드를 식별자 필드라고 한다.
- @Column  
    필드를 컬럼에 매핑한다. 

    - name 속성 : 필드의 username을 테이블의 NAME컬럼에 매핑하려면 `@Column(name=NAME)` 을 작성해주면 된다.

- 매핑 정보가 없는 필드  
    매핑 어노테이션을 생략하면 필드명을 컬럼명으로 매핑한다. 

<br/> 
<br/> 

## persistence.xml 설정

META-INF/persistence.xml 경로에 위치해 있는 설정 파일로 이 위치에 있어야만 JPA가 별도의 설정 없이 인식 할 수 있다.

```
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">

<!-- 영속성 유닛으로 일반적으로 연결할 데이터베이스당 하나의 영속성 유닛을 등록한다. -->
    <persistence-unit name="jpabook">

        <properties>

            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <property name="hibernate.use_sql_comments" value="true" />
            <property name="hibernate.id.new_generator_mappings" value="true" />

            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>

</persistence>

```

<br/>
  
- JPA 표준 속성
    - javax.persistence.jdbc.driver: JDBC 드라이버
    - javax.persistence.jdbc.user : 데이터베이스 접속 아이디  
    - javax.persistence.jdbc.password : 데이터베이스 접속 비밀번호  
    - javax.persistence.jdbc.url : 데이터베이스 접속 URL  
- 하이버네이트 속성
    - hibernate.dialect: 데이터베이스 방언 설정

  
<br/>
<br/>
  

### 데이터베이스 방언

**데이터베이스 방언이란?**  
SQL 표준을 지키지 않거나 특정 데이터베이스만의 고유한 기능을 JPA에서는 방언이라 한다.

  <br/>

**데이터베이스마다의 차이점**

- 데이터 타입 : 가변 문자 타입으로 MySQL은 VARCHAR, 오라클은 VARCHAR2를 사용
- 다른 함수명 : 문자열을 자르는 함수로 SQL 표준은 SUBSTRING()을 사용하지만 오라클은 SUBSTR() 사용
- 페이징 처리 : MySQL은 LIMIT을 사용하지만 오라클은 ROWNUM을 사용

  
<br/>

애플리케이션이 특정 데이터베이스에 종속되는 기능을 많이 사용하면 나중에 데이터베이스를 교체하기 어렵다. 

**JPA는 특정 데이터베이스에 종속적이지 않은 기술이기 때문에 다른 데이터베이스로 손쉽게 교체가 가능하다.**

JPA 자체에는 데이터 베이스 방언을 설정하는 방법이 표준화 되어 있지 않지만, 하이버네이트를 포함한 대부분의 JPA 구현체들은 이런 문제를 해결하려고 다양한 데이터베이스 방언 클래스를 제공한다.

하이버네이트 속성 : `hibernate.dialect=`  

<br/>

하이버네이트에서 제공하는 대표적인 데이터베이스 방언
- H2 : org.hibernate.dialect.H2Dialect
- 오라클 10g : org.hibernate.dialect.Oracle10gDialect
- MYSQL : org.hibernate.dialect.MySQL5InnoDBDialect



개발자는 JPA가 제공하는 표준 문법에 맞추어 JPA를 사용하면 되고, 특정 데이터베이스에 의존적인 SQL은 데이터베이스 방언이 처리해준다.

  <br/>

하이버네이트 전용 속성

- hibernate.show\_sql : 하이버네이트가 실행한 SQL을 출력
- hibernate.format\_sql : 하이버네이트가 실행한 SQL을 출력할 때 보기 쉽게 정렬
- hibernate.use\_sql\_comments: 쿼리를 출력할 때 주석도 함께 출력
- hibernate.id.new\_generator\_mappings : JPA 표준에 맞춘 새로운 키 생성 전략을 사용

  <br/>

> JPA 구현체들은 보통 엔티티 클래스를 자동으로 인식하지만 환경에 따라 인식하지 못할 때도 있다. 그 때는 다음과 같이 persistence.xml에 <class>를 사용해서 JPA에서 사용할 엔티티 클래스를 지정하면 된다.  
> <persistence-unit name=”jpabook”>  
>   <class>jpabook.start.Member</class>  
>   <properties>  
> ...

 <br/>
 <br/> 

## 애플리케이션 개발
 

코드는 크게 3부분으로 나뉘어진다.
- 엔티티 매니저 설정
- 트랜잭션 관리
- 비즈니스 로직

<br/>

### 엔티티 매니저 설정

1. 엔티티 매니저 팩토리 생성  
    persistence.xml의 설정정보를 사용해서 엔티티 매니저 팩토리를 생성한다.   
    `EntityManagerFactory emf = Persistence.createEntityManagerFactory(”jpabook”);`   
    이렇게 하면 persistence.xml에서 jpabook인 영속성 유닛을 찾아 엔티티 매니저 팩토리를 생성한다.   
    
- 엔티티 매니저 팩토리

    - JPA를 동작시키기 위한 기반 객체를 만듦
    - JPA 구현체에 따라서는 데이터베이스 커넥션 풀도 생성
    - 생성 비용이 아주 크며 애플리케이션 전체에서 딱 한번만 생성하고 공유해서 사용해야 한다.

2. 엔티티 매니저 생성  
    `EntityManager em = emf.createEntityManager();` 
- JPA의 기능 대부분은 이 엔티티 매니저가 제공하며 엔티티를 데이터베이스에 등록/수정/삭제/조회 할 수 있다.
- 내부에 데이터 소스(DB Connection)을 유지하면서 데이터베이스와 통신한다.    
- 데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드간에 공유하거나 재사용하면 안된다.

3. 종료  
- 사용이 끝난 엔티티 매니저와 엔티티 매니저 팩토리는 반드시 종료 해야 한다.  
    `em.close();`   `emf.close();`  


<br/>

### 트랜잭션 관리

JPA를 사용하려면 항상 트랜잭션 안에서 데이터를 변경해야 하며 트랜잭션 없이 데이터를 변경하면 예외가 발생한다. 트랜잭션을 시작하려면 엔티티 매니저에서 트랜잭션 API를 받아와야 한다.

 <br/> 

### 비즈니스 로직

- 등록 - `em.persist(member);`   
    엔티티를 저장하려면 엔티티 매니저의 persist() 메소드에 저장할 엔티티를 넘겨주면 INSERT SQL을 생성해서 데이터베이스에 등록한다.
- 수정 - `member.setAge(20);`   
    JPA는 어떤 엔티티가 변경되었는지 추적하는 기능을 갖추고 있다. 따라서 다른 메소드 없이 엔티티의 값만 변경한다면 UPDATE SQL을 생성해서 데이터베이스 값을 변경한다.
- 삭제 - `em.remove(member);`   
    remove()메소드에 삭제하려는 엔티티를 넘겨주면 DELETE SQL을 생성해서 실행한다.
- 한 건 조회 - `Member findMember = em.find(Member.class, id);`   
    find()메소드에 조회할 엔티티 타입과 @Id로 데이터베이스의 기본 키와 매핑한 식별자 값으로 엔티티 하나를 조회하는 가장 단순한 조회 메소드이다. 이 메소드 호출 시 SELECT SQL을 생성하여 결과를 조회하며 결과 값으로 엔티티를 생성해서 반환한다.

  
<br/>
<br/>
  

### JPQL

JPA를 사용하면 애플리케이션 개발자는 엔티티 객체를 중심으로 개발하고 데이터베이스에 대한 처리는 JPA에 맡겨야 한다.  그런데 테이블이 아닌 엔티티 객체를 대상으로 검색하려면 데이터베이스의 모든 데이터를 애플리케이션으로 불러와서 객체로 변경한 다음 검색해야 하는데 이는 사실상 불가능하다.

JPA는 JPQL이라는 쿼리 언어로 이런 문제를 해결한다.

<br/>

JPQL과 SQL 문법의 가장 큰 차이점
- JPQL은 엔티티 객체르 대상으로 쿼리한다. 쉽게 이야기해서 클래스와 필드를 대상으로 쿼리한다.
- SQL은 데이터베이스 테이블을 대상으로 쿼리한다.
 

JPQL은 데이터베이스 테이블을 전혀 알지 못하며 대소문자를 명확하게 구분한다.