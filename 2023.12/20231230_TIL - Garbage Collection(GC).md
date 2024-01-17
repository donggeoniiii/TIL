# 20231230_TIL - Garbage Collection(GC)


## Java 쓰레드에 대해

공부해보니까 운영체제와 프로세스/쓰레드 개념이 부족하다 보니까 이해가 잘 되지 않는다. 아무래도 이 공부가 선수되어야 할 것 같아서 이후에 더 자세하게 정리하도록 하자.

## Java 가상 머신(Java Virtual Machine, JVM)

Java의 언어적 특징 중 중요한 부분인 메모리 관리 및 최적화를 담당하고, 다양한 실행 환경에서 동일하게 Java 어플리케이션이 실행되는 환경을 제공한다.

### JVM의 메모리 관리 방법

stack과 heap 영역에 대해 메모리를 관리한다. stack 영역은 메소드 종료 이후 바로 할당한 메모리를 해제하고, heap 영역은 GC가 관리한다.

### 프로그램이 실행되기까지

1. JVM이 운영체제(OS)로부터 프로그램이 필요한 메모리 할당 받아오기
2. Java 컴파일러가 소스 코드를 읽고 바이트 코드로 변환
3. 바이트 코드로 변경된 클래스 파일을 클래스 로더를 통해 로딩 해옴(클래스 로딩 과정은 생략)
4. 로딩된 클래스 파일을 실행 엔진을 통해 실행
5. 해석된 바이트 코드는 처음에 할당받아 온 JVM 메모리 영역에 올라가서 실행
    1. 이 과정에서 쓰레드 동기화, GC의 메모리관리 등이 일어남

## 가비지 컬렉션

기존 C/C++ 프로그래밍 시 사용하지 않는 메모리의 해제를 명시적으로 해줘야 했다. Java는 이를 JVM의 구성요소 중 하나인 가비지 컬렉션(GC)가 이를 대신한다. 

그렇지만 GC를 사용하더라도 사용한 메모리 영역이 없다면 `OutOfMemoryError`가 발생할 수 있다. 이 경우 WAS가 다운되므로 GC에 대한 이해는 중요하다.

### stop-the-world

GC가 실행되기 위해 JVM이 어플리케이션 실행을 멈추는 것. 필연적으로 발생하는 딜레이이고 GC 튜닝은 이 `stop-the-world` 시간을 줄이는 것을 의미한다.

### GC가 지우는 대상

- 객체가 null인 경우
- 메소드 블럭 실행 종료 후 메소드 안에서 생성된 객체(heap 영역)
- 부모 객체가 null일 때 포함하는 자식 객체
- 

### GC의 메모리 해제 과정(Mark & Compact)

1. marking
    
    heap 영역에서 참조되지 않는 객체의 메모리 영역을 마킹한다. 말 그대로이다. 모든 오브젝트를 스캔해야 하기 때문에 많은 시간이 소모된다.
    
2. normal deletion
    
    참조되지 않는 객체를 제거하고 메모리를 반환한다. 비어진 블록 위치는 메모리 할당자(allocator)가 빈 참조 위치를 저장해 두었다가 새로운 객체가 선언되면 할당시킨다.
    
3. compacting
    
    남은 참조 객체를 묶어서 공간을 효율적으로 비워놓는다. 이후 할당을 더 쉽고 빠르게 할 수 있게 된다.
    

그렇지만 정말 이런 과정으로 모든 객체를 마킹 후 컴팩트 하는 방식은 비효율적이다.

### Weak Generational Hypothesis와 JVM의 메모리 할당

> 신규로 생성한 객체 대부분은 금방 사용하지 않는 상태가 되고, 오래된 객체에서 새로운 객체로의 참조는 매우 적게 존재할 것이다.
> 

이 가설에 기반해, JVM은 `Young` 영역과 `Old` 영역으로 메모리를 분할한다.

- `Young`: 새롭게 생성된 객체 대부분이 여기에 위치한다. 대부분의 객체가 여기서 생성되고 사라진다. 여기서 객체가 사라지면 `Minor GC`가 발생한다고 한다.
- `Old`: 충분히 오래 살아있던 객체가 여기로 복사된다. 대부분 `Young` 영역보다 크게 할당된다. 이 영역에서 객체가 사라지면 `Major GC`가 발생한다고 한다.
- `Permanent`: 메소드 영역이다. JVM이 클래스와 메소드에 대한 메타 데이터들을 보관하고 있다.

### General GC가 작동하는 과정

1. 새로운 객체가 생성, `Young` 영역에 속하는 `Eden Space`에 보관한다
2. `Eden Space`가 꽉 차면 `Minor GC`가 발생한다
3. 참조되는 애들은 첫 번째 `Survivor(S0)`에 이동되고, 비참조 객체는 `Eden`이 clear되면서 메모리가 반환된다.
4. 다시 한 번 `Eden Space`가 꽉 차면 두 번째 `Minor GC`가 발생한다. 이번에는 `S1`에 저장된다.
    
    4-1. 여기서 `S0`에 저장된 객체들도 age가 증가한 상태로 `S1`에 이동된다. 
    
    4-2. `Eden`과 `S0`이 clear된다.
    
5. 다음 `Minor GC`에도 이게 반복된다. 이번엔 `S0`에 저장된다.
    
    5-1. `S1`에 있던 객체들은 age가 증가한 상태로 `S0`으로 이동된다.
    
    5-2. `Eden`과 `S1`이 clear된다.
    
6. 이 과정 중에서 `age threshold`를 넘어가는 객체들은 `Old` 영역으로 이동된다. 이를 Promotion이라고 한다.
7. 이 과정을 반복해서 `Old`에는 오랫동안 참조되는 객체가 남고, `Young` 영역은 계속해서 최적화가 일어나게 된다.