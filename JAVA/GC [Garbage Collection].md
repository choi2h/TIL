# GC \[Garbage Collection\]

- `동적으로 할당한 메모리 영역` 중 사용하지 않는 영역을탐지하여 해제하는 기능
- 개발자가 직접 메모리 할당과 해제를 컨트롤하지 않도록 도와준다.
- 즉, 자동으로 쓰레기 데이터를 정리해주는 역할을 한다.

<br/>

## Stack과 Heap

- Stack : 정적으로 할당된 메모리 영역

    - 원시 타입의 데이터가 값과 함께 할당
    - Heap영역에 생성된 Object 타입의 데이터의 참조 값 할당
- Heap : 동적으로 할당된 메모리 영역 
    - 모든 object 타입의 데이터 할당
    - Heap영역의 Object를 가리키는 참조 변수가 Stack에 할당

<br/>

## Reachability

- Reachable
    - 유효한 참조가 있는 객체
- Unreachable
    - 유요한 참조가 없는 객체
- `힙 객체들의 참조 방법`
    - 힘 내의 다른 객체에 의한 참조
    - Java 메서드 실행 시에 사용하는 지역 변수와 파라미터들에 의한 참조
    - 네이티브 스택 : JNI(Java Native Interface)에 의해 생성된 객체에 대한 참조
    - 메서드 영역의 정적 변수에 의한 참조

<br/>
<br/> 

## GC가 불필요한 Object를 선별하는 방법 : Mark&Sweep

- Mark : 살아남아야 할 오브젝트에 마킹(표시)하는 작업
    - reachable한 객체 표시
- Sweep : 마킹되지 않은 오브젝트를 지우는 작업
    - unreachable한 객체를 제거
- 동작방식
1. Stack의 모든 변수를 스캔하면서 각각 어떤 객체를 참조중인지 확인한다.
2. Reachable 오브젝트가 참조하고 있는 객체는 마킹한다.
3. 마킹되지 않은 객체는 Heap에서 제거한다.

<br/>
<br/> 

# JAVA의 Heap 메모리 영역

JVM 메모리 영역은 아래와 같이 두 부분으로 나눌 수 있다.

![](https://blog.kakaocdn.net/dn/o0D2Z/btqBFL63e4b/s4nEA6WkbmjUp73nvBup6k/img.png)  

출처 : [https://www.waitingforcode.com/off-heap/on-heap-off-heap-storage/read](https://www.waitingforcode.com/off-heap/on-heap-off-heap-storage/read)

<br/>
<br/>  

## Heap 메모리 - Young Generation

- 생긴지 얼마 안된 객체가 있는 곳
- Eden과 두개의 Survival(Survival0 /Survival1) 총 세 부분으로 나눌 수 있다.
- 둘 중 하나의 Survival은 항상 비워있어야 한다.
- 두 Survival간에 우선순위는 없으며 랜덤으로 선택된다.

<br/>  

### Minor GC 동작과정

1. 새로 생성된 객체는 Eden에 존재하게 된다.
2. Eden 공간이 객체로 다 채워지게 되면 `Minor GC` 가 작동한다.
    - Eden영역이 다 차면 Reachable객체는 S1으로 옮긴다.
    - Eden영역의 Unrechable객체는 메모리에서 제거한다.
3. S1 공간이 객체라 다 채워지게 되면 똑같이 Minor GC가 작동한다.
    - S1영역의 Reachable객체는 S0으로 이동된다. → 이때 Age값이 증가한다.
    - S1영역의 Unrechable객체는 메모리에서 제거된다.  
    
4. 위 과정을 반복하다가 Age 값이 특정 값 이상이 되면 Old Generation으로 옮겨진다. : `Promotion` 

<br/>
<br/>  

## Heap 메모리 - Old Generation

- MinorGC 후에 오래 남은 객체가 존재하게 된다.
- Old Generation이 가득 차면 MajorGC가 발생한다.

 <br/>

### Major GC와 Full GC

- Major GC
    - Old generation이 가득 찼을 때 발생
- Full GC (MinorGC + MajorGC)
    - young generation과 old generation이 다 찼을 때 발생
- 단편화를 줄이기 위해 모든 변수들을 앞당기게 된다 : `compact` 

 <br/>
 <br/> 

## Perm (JDK8 부터 제거)

- 자바 애플리케이션을 실행할 때 클래스의 메타데이터를 저장하는 영역
- JDK8부터 제거되었으며 Metaspace 영역이 추가되었다.
- Permanaent Generaton이 제거되면서 생긴 이동
    - Symbols - Native Heap
    - Interned String(중복된 문자를 상수화) - Java Heap
    - Class statics - Java Heap

 <br/> 

### MetaSpace 영역

- Native 메모리 영역으로 취급 - OS레벨에서관리됨  
    - 메타데이터를 native 메모리에 저장 
- 메모리가 부족할 경우 이를 자동으로 늘려줌
- Java의 Classloader가 로드한 class들의 metadata가저장되는 공간
- \-XX:MetaspaceSize : 메타스페이스 영역의 최소/초기 메모리 양 설정
    - 시스템에서 기본으로 제공되는 것보다 더 많은 양을 사용하기 위해 사용
- \-XX:MaxMetaspaceSize : 메타스페이스 영역의 최대 메모리 양 설정
    -  메모리 누수 방지를 위한 메모리 영역 조절에 사용
- \-XX:CompressedClassSpaceSize : Compressed Class Size의 가상 사이즈 지정
    - 기본값 : 1G
    - 최대값 : 3G

<br/>  

### 변경된 메모리 구조

![](https://miro.medium.com/max/1026/0*rKZvTnuUkEc5LoXW.jpg)  
  
<br/>
<br/>
  

## GC 전제조건

- 대부분의 객체는 금방 접근 불가능한 상태가 된다.
- 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다. 

<br/>
<br/>   

## GC 특징

- GC가 작동하면 GC를 제외한 애플리케이션의 모든 쓰레드가 중단된다. : `Stop-the-world` 
- GC 튜닝이란 이 `Stop-the-world` 를 제어하기 위함이다.
- MinorGC는 수명이 짧은 객체만을 검사하기 때문에 애플리케이션에 영향이 없다.
- MajorGC는 살아있는 모든 객체를 확인하기 때문에 시간이 오래걸린다.
    - MajorGC가 발생하지 않도록 해야한다.

  
<br/>
<br/>
  

## GC 종류

### Serial GC (-XX:+UseSerialGC)

- 싱글쓰레드로 동작한다. (GC를 처리하는 쓰레드가 1개)

- 다른 GC에 비해 stop-the-world 시간이 길다.

- 메모리와 CPU 코어 개수가 적을 때 적합
- Old영역에서는 Mark-Sweep-Compact 알고리즘을 사용한다.

    - Mark : Old 영역에 참조된 객체를 식별
    - Sweep : Heap의 앞부분부터 unreachable 객체 제거
    - Compaction : 객체가 연속되게 쌓이도록 Heap의 가장 앞부분부터 채운다.

<br/>


### Parallel GC (-XX:+UseParallelGC)

- Java8의 기본 GC
-  멀티쓰레드로 수행한다.
- 메모리가 충분하고 코어의 개수가 많을 때 유리하다.
- SerialGC에 비해 stop-the-world 시간이 감소한다.

<br/>  

### Parallel Old GC

- Parallel GC를 개선한 GC
- Old 영역에서도 멀티 쓰레드 방식의 GC를 수행한다.
- Mark-Summary-Compact 알고리즘 사용
    - sweep : 단일 쓰레드가 old 영역 전체를 훑는다.
    - summary : 멀티 쓰레드가 old 영역을 분리해서 훑는다.

 <br/> 

### CMS(Concurrent Mark Sweep) GC

- stop-the-world 시간을 줄이기 위해 고안된 GC이다.
- 동작과정
    - Initial Mark  : 클래스 로더에서 가장 가까운 객체 중 살아있는 객체 찾음
    - Concurrent Mark : 살아있다고 확인한 객체에서 참조하고 있는 객체들을 따라가면서 확인
    - \-------– 이때부터 Stop-the-world 동작 ------------
    - Remark : Concurrent Mark에서 새로 추가되거나 참조가 끊긴 객체 확인
    - Concurrent Seep : 쓰레기 객체 제거
- 다른 GC방식보다 메모리와 CPU를 더 많이 사용한다.
- Compaction 단계가 없다.
- 조각난 메모리가 많아 Compaction작업을 실행하게 되면 다른 GC보다  Stop-the-world 시간이 길어진다.  
    

<br/>  

### G1(Garbage First) GC

- Java9부터 기본 GC
- CMS GC의 메모리 단편화 개선을 위해 고안된 방법
- 기존의 Young / Old 방식과 구조가 다르다.
- Heap을 일정한 크기의 Region으로 나눈다. (바둑판 형식)
- 해당 Region이 꽉 차면 GC를 실행하고 다른 Region에 객체를 할당한다.
- 어느 GC보다도 가장 빠르다.
  
<br/>
<br/>
  

## 참고

[[우아한Tech] 던의 JVMdml Garbage Collector](https://www.youtube.com/watch?v=vZRmCbl871I&t=590s)    
[[우아한Tech] 엘리의 GC](https://www.youtube.com/watch?v=Fe3TVCEJhzo)    
[Java (JVM) Memory Model - Memory Management in Java](https://www.digitalocean.com/community/tutorials/java-jvm-memory-model-memory-management-in-java)  
[[JVM] Garbage Collection Algorithms](https://medium.com/@joongwon/jvm-garbage-collection-algorithms-3869b7b0aa6f)  
[[NAVER D2] Java Reference와 GC](https://d2.naver.com/helloworld/329631)  
[[NAVER D2] Java Garbage Collection](https://d2.naver.com/helloworld/1329)    
[[Java] 자바 메타스페이스(Metaspace)에 대해 알아보자.](https://jaemunbro.medium.com/java-metaspace%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-ac363816d35e)    
[[자바봄]자바 Garbage Collection](https://javabom.tistory.com/7)