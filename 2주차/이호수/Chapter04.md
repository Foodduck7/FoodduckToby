## 4장 예외 처리

예외 처리의 핵심 원칙: 아래 2가지 중 한가지 상황이 무조건 발생해야한다<br>
1. 모든 예외는 적절하게 복구되어야 한다
2. 작업을 중단시키고 운영자 or 개발자에게 통보되어야한다

throws Exception 의 문제점
1. 다양한 종류의 예외를 무조건 Exception 으로 뭉뚱그린다
2. throws Exception를 선언한 메소드를 사용하는 메소드도 같은 방법을 사용해야한다
3. 적절한 처리를 통해 복구될 수 있는 예외 상황도 해결이 어려워진다

예외의 종류와 특징

1. Error
   java.lang.Error 클래스의 하위 클래스들<br>
   시스템에 비정상적인 상황이 발생한 경우 -> 주로 JVM에서 발생함<br>
   OOM(Out Of Memory Error)/메모리 누수, Thread Death/스레드 등의 에러<br>
   어플리케이션에서는 해당 에러에 대한 처리를 할 필요는 없다
<br>
2. Exception 과 checked Exception<br>
   개발자들이 만든 어플레케이션 코드의 작업 중에 예외 상황이 발생했을 경우<br>
   Exeception 은 checked Exception과 unchecked Exception으로 나뉜다<br>
   unchecked Exception은 RuntimeException을 상속한다<br>
   checked Exception의 경우 catch 문으로 잡거나 throws를 정의해서 메소드 밖으로 던져야 한다<br>
   예외 확인이 어려워지거나 의미없이 throws를 사용할 수있음
   <br>
3. RuntimeException과 unchecked Exception/runtime Exception<br>
   java.lang.RuntimeException을 상속한 예외들<br>
   unchecked라고 하는 이유 -> 명시적인 예외처리를 강제하지 않는 이유<br>
   명시적으로 에러를 처리해도 됨(try-catch, throws 등)<br>
   프로그램의 오류가 있을 때 발생하도록 의도된 것들<br>
   대표적인 예시: NPE(Null Pointer Exception), IllegalArgumentException 등

효과적안 예외처리 방벙

1. 예외 복구: 예외 상황 파악 -> 문제 해결<br>
   사용자에게 상황을 알림 -> 다른 작업 흐름으로 유도<br>
   예외가 처리된다면 어플리케이션에서는 설계된 흐름에 따라 진행되어야한다<br>
   어떤 식으로든 예외를 복구할 수 있을 때만 사용한다<br>
   int maxEntry 값으로 시도 가능 횟수를 정하고<br>
   ```while(manxEntry-- > 0) { 
    try { ... 
    } catch( Exception e ){
    ... 
    }
<br>   
   의 방법으로 사용할 수 있다

2. 예외처리 회피: 예외 처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것<br>
   throws 문 선언 or catch 문으로 예외 잡고 -> 로그 남김 -> 예외 또 던짐<br>
   긴밀하기 역할을 분담하는 관계에서가 아니라면 무책임한 책임회피가 될 수 있다<br>
   즉, 의도나 확신이 분명해야한다
   <br>
3. 예외 전환(exception translation): 예외를 적절한 예외로 전환해 던진다

+ 1. 내부에서 발생한 예외를 던지는 것이 적절한 의미를 부여하지 못할 떄 의미가 분명한 예외로 바꾼다<br>
   충분히 예상 가능하고 복구 가능한 예외 상황일 때<br>
   중첩 예외(nested exception): getCause() 메소드를 사용해 처음 발생한 예외를 확인할 수 있다<br>
   ```catch(QweException e) {
   throw new RtyException().initCause(e);
   }
<br>
  initCause() 메소드로 근본 원인이 되는 예외를 넣어줄 수 있다
<br>

+ 2. 예외를 wrapping 하는것<br>
   중첩 예외 이용, 원인이 되는 예외를 담는다<br>
   주로 예외처리를 통해 복구 가능하지 않아 예외 처리가 필요 없는 런타임 예외로 바꾸기 위해 사용한다<br>
   어플리케이션 로직 상에서 예외 조건이 발견되거나 예외 상황이 발생하는 것 -> 의도적으로 던진다: 체크 예외 사용해야한다<br>
   throws를 사용하지 않아서 다른 계층의 메소드 작성에서 불필요한 throws 선언을 막아준다<br>
   가장 좋은 방법: 런타임 예외로 포장 -> 던짐 -> 예외처리 서비스(자세한 로그) -> 관리자에게 메일, 사용자에게 메세지

예외 상황에서 무조건 프로그램이 종료된다 -> 사용자 만족도가 떨어진다
에러가 발생한 해당 작업만 중단하는 게 적절한 조치이다

JDO, JPA, Hibernate 등의 객체/엔티티 단위로 정보를 업데이트하는 경우<br>
-> 낙관적 락(Optimistic lock)이 발생할 수 있음

스프링의 예외 전환 방법: 기술에 상관없이 DataAccessException의 서브 클래스인 ObjectOptimisticLockingFailureException으로 통일 시킬 수 있음

Exception 계층
RuntimeException -> NestedRuntimeException -> DataAccessException ->TransientDAtaAccessException -> ConcurrencyFailureException -> OptimisticLockingFailureException -> ObjectOptimisticLockingFailureException






