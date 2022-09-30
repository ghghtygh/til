예외

### Error

java.lang.Error 클래스의 서브클래스

주로 JVM에서 발생한 시스템의 비정상적인 상황 발생시 사용

대응 방법이 없기에 시스템 레벨의 작업이 있지 않는 이상 애플리케이션에서는 에러처리 X

### Exception

java.lang.Exception 클래스와 그 서브클래스

애플리케이션 코드의 작업 중 발생한 예외상황에서 사용

일반적으로 체크예외를 말하며, 체크예외가 발생할 수 있는 메서드 사용시 반드시 예외 처리코드 작성 필요

체크예외 : Exception 클래스의 서브클래스이면서 RuntimeException 상속 X

언체크예외 : Exception 클래스의 서브클래스이면서 RuntimeException 상속

### RuntimeException

java.lang.RuntimeException 클래스를 상속한 예외는 명시적인 예외처리 강제 X

주로 프로그램의 오류가 있을 때 발생하도록 의도된 예외 (개발자의 부주의)

NullPointException : 오브젝트를 할당하지 않은 레퍼런스 변수를 사용하려고 시도

IllegalArgumentException : 허용되지 않는 값을 사용하여 메서드를 호출

에러는 시스템에 비정상적인 상황

예외는 프로그램 실행 중에 개발자의 실수로 예기치 않은 상황이 발생

### 체크 예외(Checked Exception)

Exception 클래스의 하위 클래스

반드시 예외처리를 해야함 (try/catch 또는 throws 선언)

ex) FileNotFoundException, ClassNotFoundException

### 언체크 예외(Unchecked Exception)

RuntimeException의 하위 클래스

예외처리를 강제하지 않음 

런타임시 발생할 수 있는 예외

ex) ArrayIndexOutOfBoundsException, NullPointException
 
---

## 예외처리 방법

### 예외 복구

예외상황 파악 → 문제 해결 → 정상상태

사용자 요청 파일이 없는 경우 → IOException 발생 → 사용자에게 다른 파일 이용하도록 안내 → 예외상황 해결

불안정한 네트워크로 DB 접속 실패 → SQLException 발생 → 대기 후 재시도(최대 횟수 지정) → 정상 접속
→ 예외상황 해결

### 예외처리 회피

예외 처리를 자신이 담당하지 않고 호출한 쪽으로 던지는 것 (throws 문 또는 catch 문 이후 rethrow)

JdbcContext나 jdbcTemplate에서의 콜백 오브젝트에서는 SQLException에 대한 예외를 회피하고 템플릿 레벨에서 처리하도록 던져줌

### 예외 전환

예외 복구가 불가능할 경우 예외를 적절한 에외로 전환해서 메서드 밖으로 던지는 것 

1. 내부에서 발생한 예외를 예외 상황에 대한 적절한 의미가 부여된 예외로 변경하기 위해 사용
    
    : 회원가입시 아이디가 중복되어 SQLException 발생시 서비스에서 예외 복구를 위해 DAO에서 의미가 분명한 예외로 전환
    
2. 예외를 처리하기 쉽고 단순하게 포장하기 위해 사용
    
    : 주로 예외처리가 강제된 체크예외를 언체크 예외인 런타임 예외로 바꾸는 경우에 사용
    비즈니스 로직에 의미가 없거나 복구가 불가능한 체크예외를 넘기는건 의미가 없기 때문에 시스템 Exception으로 인식하여 트랜잭션이 롤백되도록 런타입 예외로 포장하여 전달
    

### 예외 무시

catch 이후 아무일도 하지 않는 것