# 20240101_TIL - Error & Exception

## Error와 Exception의 정의와 차이

프로그램이 실행 중 오작동을 하거나 비정상 종료되는 것을 `프로그램 오류`라고 한다. 여기서 프로그래머가 코드를 통해 처리할 수 없는 `에러(Error)`와 처리할 수 있는 `예외(Exception)`가 있다.

### Error의 종류

컴파일 시에 발생하는 문법적인 오류, 혹은 런타임 시 null 참조 등 프로세스에 심각한 문제를 초래하는 오류를 말한다.

대표적으로 빨간 줄로 뜨는 문법 오류나, 스택 오버플로우, 메모리 초과 등이 있다. 

요즘은 IDE가 사전에 검토를 해주기 때문에 개발 시 크게 고려할 필요는 없어 보인다.

### Exception의 종류

컴퓨터 시스템의 동작 중 예기치 않았던 `이상 상태`가 발생하여 수행 중인 프로그램이 영향을 받는 오류를 말한다.

대표적으로 잘못된 연산(`ArithmeticException`), 입출력 예외(`IOException`) 등이 있다. 이렇게 되는 데는 사용자의 잘못된 데이터 입력도 있겠지만, 개발자의 잘못된 코드나 하드웨어, 네트워크 오작동으로도 일어날 수 있다. 

프로그래머의 적절한 처리로 막을 수 있는 것인만큼, Error를 사전에 방지하고자 예외처리를 하기도 한다.

## Throwable 클래스

Java는 모든 것이 객체로 이루어진 언어답게 Error와 Exception도 객체로 처리한다.  그 중 `Throwable` 클래스는 예외처리를 가능하게 만드는 최상위 클래스로, `Exception`과 `Error`는 이 클래스를 상속한다.

`Error`는 시스템 레벨에서 발생하여 개발자가 조치하지 못하는 수준의 오류들을 모아놓았다.

- VMError
    - OutOfMemoryError
    - StackOverflowError
    - UnknownError
- LinkageError
    - NoClassDefFoundError
    - ClassFormatError

`Exception`은 개발자가 구현한 로직에서 발생하는 것들을 모아두었다. 개발자가 다른 방식으로 처리가 가능하다.

- ClassNotFoundException
- InstantiationException
- NoSuchField/MethodException
- RuntimeException
    - NullPointerException
    - IndexOutOfBoundException
    - ArithmeticException
    

### Exception의 2가지 종류

처리하지 않으면 컴파일되지 않는 `CheckedException`과 `UncheckedException`으로 나뉜다. 뒤에 (컴파일)이라는 말이 생략되었다고 보면 좋을 것 같다.

`CheckedException`에는 JVM 외부와 통신할 떄 주로 쓰이고, `RuntimeException` 외 모든 예외가 해당한다.

ex) IOException, SQLException 등

`UncheckedException`은 컴파일 때는 모르고 돌아가는 와중에 알게 되는 것들이므로, `RuntimeException` 하위에 들어가는 모든 예외가 해당한다.

ex) NullPointerException, IndexOutOfBoundExcetion 등

### 주요 메소드

`printStackTrace()`: 발생한 예외의 출처를 메모리상에서 추적

`getMessage()`: 한 줄로 요약된 메세지를 String으로 반환

`getStackTrace()`: `printStackTrace()`를 보완한 메소드. `StackTraceElement`라는 문자열 클래스의 배열로 변경해서 출력하고 저장한다.

### Exception Handling

Java에서 예외처리는 두 가지 방법이 있다. 처리하느냐, 던지느냐.

예외를 직접 처리하는 경우는 try-catch 문을 통해 처리하는 것으로, catch문 내에서 예외 발생 시 코드를 입력한다.

던지는 경우는 말 그대로 `throws` 키워드를 통해서 예외를 해당 메소드의 상위 메소드에서 처리하도록 넘긴다. 다른 메소드의 하위 메소드인 경우엔 던지는 것이 보통이다. 

참고로 `throws`를 사용하면 `return`보다 강력하므로 예외처리시 반환값이 발생하지 않는다.

### 사용자 예외 처리

개발자가 원하는 방식으로 예외처리도 가능하다. `Exception` 클래스를 상속받는 클래스를 만들고 정의하면 된다.
