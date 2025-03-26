## 1. 트랜잭션 속성 테스트
### 트랜잭션 속성이란?
트랜잭션의 동작 방식을 제어하기 위한 설정값.<br> 
주로 다음의 속성이 있음

- 전파
- 격리 수준
- 읽기 전용
- 타임아웃
- 롤백 조건(rollbackFor, noRollbackFor)

### 테스트 목적 ?
- 선언된 트랜잭션 속성이 실제로 적용되는지 확인
- 예외 발생 시 롤백 여부를 확인
- readOnly가 적용된 경우 실제로 쓰기 작업이 차단되는지 확인

##  2. 애노테이션 트랜잭션 속성과 포인트컷
### @Transactional을 붙인 메서드 → 트랜잭션이 적용되는 포인트컷
- 스프링 AOP 기반으로 동작 → 설정된 트랜잭션 속성은 Advice로 적용됨
- 메서드 단위로 세밀한 트랜잭션 속성 적용 가능

## 3. 트랜잭션 어노테이션
### @Transactional
```java
@Transactional(
  propagation = Propagation.REQUIRED,
  isolation = Isolation.READ_COMMITTED,
  readOnly = false,
  rollbackFor = SomeException.class
)
```
- 클래스/메서드에 적용 가능
- 구체적인 트랜잭션 속성 부여 가능

## 4. 트랜잭션 속성을 이용하는 포인트컷
- @Transactional이 붙은 메서드가 트랜잭션 적용 대상이 됨
- Spring AOP에서 포인트컷은 @Transactional 어노테이션을 기준으로 트랜잭션 Advice를 적용

## 5. 대체 정책 
- 트랜잭션 적용 대상을 XML로 지정할 수도 있음 (너~~~~무 예~~~~~~~~전 방식)
- AspectJ 등을 이용한 고급 포인트컷 설정도 가능

##  6. 트랜잭션 어노테이션 적용
- 선언적 방식으로 트랜잭션을 설정할 수 있음 (@Transactional)
- 설정이 간결하고 유지보수가 쉬움
- Service 계층 중심으로 적용하는 것이 일반적입니다.

## 7. 트랜잭션 지원 테스트
- @Transactional을 테스트 메서드에 적용하면, 테스트 후 자동 롤백됨
- 테스트의 독립성을 보장
- 스프링의 테스트 지원을 통해 실제 DB를 대상으로 트랜잭션 테스트 가능

## 8. 선언적 트랜잭션과 트랜잭션 전파 속성
### 전파 속성 예시
- REQUIRED (default): 트랜잭션이 있으면 참여, 없으면 새로 생성
- REQUIRES_NEW: 항상 새 트랜잭션 시작
- NESTED: 중첩 트랜잭션
- NEVER: 트랜잭션이 있으면 예외 발생

##  9. 트랜잭션 동기화와 테스트
- 트랜잭션 동기화: 트랜잭션이 활성화된 상태에서 Connection이나 EntityManager를 공유하도록 해주는 기능
- 스프링은 ThreadLocal을 이용하여 같은 스레드에서 같은 커넥션을 공유하게 해줌

## 10. 트랜잭션 매니저와 트랜잭션 동기화
- PlatformTransactionManager 인터페이스를 통해 트랜잭션을 제어
- 테스트나 직접 제어가 필요한 경우 TransactionTemplate 사용 가능

## 11. 트랜잭션 매니저를 이용한 테스트용 트랜잭션 제어
```java
TransactionTemplate template = new TransactionTemplate(transactionManager);
template.execute(status -> {
   // 트랜잭션 내 작업
   return null;
});
```
명시적으로 트랜잭션을 시작하고 제어할 수 있음

## 12. 트랜잭션 동기화 검증
- 테스트 코드에서 커넥션이 같은지, 트랜잭션 범위 내에 있는지 검증
- 트랜잭션이 공유되고 있는지 확인할 수 있음

## 13. 롤백 테스트
#### 예외 발생 시 롤백 여부 확인
- @Transactional + 예외 상황 유발 → 롤백 되는지 확인

## 14. 테스트를 위한 트랜잭션 어노테이션
- 테스트 클래스나 메서드에 @Transactional 사용 → 테스트 후 자동 롤백
- @Rollback(true) 명시 가능

## 15. @Rollback, @TransactionConfiguration, NotTransactional, Propagation.NEVER
- @Rollback(false) → 실제 커밋 (디버깅용)
- @TransactionConfiguration(defaultRollback=true) (과거 사용, 현재는 거의 안 씀)
- NotTransactional: 트랜잭션 사용 안 함
- Propagation.NEVER: 트랜잭션이 있으면 예외 발생

## 16. 효과적인 DB 테스트
- 테스트 간의 데이터 간섭 최소화
- @Transactional로 테스트를 독립적으로 실행
- 실제 DB 사용 권장, H2 등으로 환경 유사하게 구성
- SQL 로그, 쿼리 성능도 확인할 수 있는 구조가 이상적


## 궁금증
중간
> "트랜잭션 어노테이션에서 @Transactional이 타입 레벨이든 메서드 레벨이든 상관없이 부여된 빈 오브젝트를 모두 찾아서 포인트 컷의 선정 결과로 돌려준다."

어떻게 돌려준다는거지? 

#### 이를 이해하려면  AOP와 포인트컷 동작 방식을 이해하면 됨.
위의 문구가 저는 오히려 '스프링 AOP가 트랜잭션 적용 대상을 판별하는 방식에 대한 설명'으로 생각했습니다.

그렇다면, 
### 포인트컷의 선정 결과로 돌려준다는 것은 어떻게 ???
이를 저는
"스프링 AOP는 @Transactional이 붙은 대상(메서드 or 클래스)을 스캔하니까,<br>
그 결과로 "이 객체는 트랜잭션 Advice를 적용해야한다"는 판단이 서게 되면,<br> 
해당 객체(빈)에 대해 프록시 객체를 생성해서, 트랜잭션을 관리하게 하는 것이다." 라고 이해했습니다.

>💡 "포인트컷의 선정 결과"란 → AOP에서 트랜잭션을 적용할지 말지 결정된 대상 목록을 의미 <br>
💡 "돌려준다"는 → 트랜잭션 적용 대상으로 선정된 빈에 트랜잭션 프록시를 적용한다는 의미

### 예시
```java
@Service
@Transactional // 타입 레벨
public class OrderService {
    public void createOrder() {
        // 트랜잭션 적용 대상
    }

    public void cancelOrder() {
        // 이것도 트랜잭션 대상
    }}
```
1. OrderService는 스프링이 빈으로 등록할 때 @Transactional을 보고,
2. "얘는 트랜잭션 Advice를 걸어야겠네"라고 판단하게 됨. → 포인트컷에 일치
3. 그래서 OrderService는 내부적으로 프록시 객체로 래핑됨.
4. 트랜잭션이 시작되고 커밋/롤백을 관리해주는 동작이 자동으로 들어감.

>다시 말해,<br>
@Transactional이 붙은 클래스나 메서드는 포인트컷 후보이며, <br>
스프링은 그런 빈들을 스캔하고, 트랜잭션 Advice가 필요한 대상들을 포인트컷의 선정 결과로 간주한다는 것이다.<br>
해당 빈들을 트랜잭션 프록시로 감싸서 반환함 → 그래서 "돌려준다"는 표현을 쓴 것으로 생각된다.