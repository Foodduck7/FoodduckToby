## 6.7 어노테이션 트랜잭션 속성과 포인트컷

클래스나 메소드에 따라 각각 다른 트랜잭션 속성 적용할 떄<br>
-> 스프링 제공의 어노테이션 지정 방법

```<@Transactional>
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
```

사용되는 포인트컷: TransactionAttributePointcut

유연하다: 메소드 마다 다른 트랜잭션 속성을 설정할 수 있다<br>
-> 동일한 속성 정보의 어노테이션을 메소드마다 부여하는 중복 코드를 만들 수 있다

스프링의 4단계 대체(fallback) 정책

타깃 메소드 -> 타깃 클래스 -> 선언 메소드 -> 선언 타입

의 순서로  @Trasactional 적용 확인 가장 먼저 발견되는 속성 정보 사용

트랜잭션 전파 방식 디폴트: REQUIRED

트랜잭션은 선언적으로 적용할 수 있다<br>
선언적 트랜잭션(declarative transaction): AOP를 이용해 코드 외부에서 트랜잭션의 기능을 부여해주고 속성을 지정할 수 있게 함<br>
프로그램에 의한 트랜잭션(programmatic transaction): 개별 데이터 기술의 트랜잭션 API를 사용해 직접 코드 안에서 사용하는 방법

스프링은 자바 클래스 객체에도 선언적 트랜잭션을 적용할 수 있다<br>
또한 트랜잭션 추상화도 제공해 어떠한 기술이나 환경에 종속되지 않는다

트랜잭션 추상화의 핵심
1. 트랜잭션 매니저
2. 트랜잭션 동기화

트랜잭션 매니져로 트랜잭션 시작 후 제어<br>
트랜잭션 객체 만들기<br>
트랜잭션 매니져에 트랜잭션 객체 제공 -> 새로운 트랜잭션 요청

트랜잭션 속성 중 읽기전용과 제한시간 등<br>
트랜잭션 시작 시에만 적용 -> 이후 참여 메소드의 속성은 무시된다

롤백 테스트: 테스트를 진행하는 동안에 조작한 데이터를 모두 롤백하고 테스트를 시작하기 전 상태로 만들어준다<br>
항상 처음과 동일한 User 테이블의 테스트 데이터로 테스트를 수행할 수 있다<br>
-> @ContextConfiguration

테스트에도 @Transactional 사용가능<br>
@Rollback: 롤백 기능 제어

@TransactionConfiguration: 클래스 레벨에 부여<br>
모든 메소드에 트랜잭션을 적용해면서 모든 트래잭션이 롤백되지 않고 커밋되게 함

@NotTransactional<br>
클래스 레벨의 @Transactional 설정 무시<br>
= @Transactional(propagation=Propagation.NEVER)

7.6.2//개인적으로 읽기