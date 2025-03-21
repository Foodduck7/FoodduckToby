#  Chapter 6 - 스프링 AOP & 트랜잭션

##  스프링의 ProxyFactoryBean

- 스프링은 프록시 오브젝트를 생성해주는 **ProxyFactoryBean**을 제공
- 프록시 생성만 담당하고, 부가기능(Advice)은 별도의 빈으로 관리
- 부가기능은 `MethodInterceptor` 인터페이스 구현으로 작성

###  주요 용어

| 용어 | 설명 |
|------|------|
| Advice | 타깃에 종속되지 않는 **순수한 부가기능** |
| Pointcut | 어떤 메소드에 부가기능을 적용할지 **선정 알고리즘** |
| Advisor | Advice + Pointcut을 묶은 조합 객체 |

>  Advisor는 "누구(Pointcut)에게 어떤 충고(Advice)를 해야 하는지" 알고 있음

###  프록시 동작 흐름

1. 프록시는 요청을 받으면 **Pointcut에 메소드 적용 여부를 확인**
2. 통과하면 **Advice(MethodInterceptor)** 호출
3. Advice는 내부에서 **`proceed()`** 로 타깃 메소드 실행

![프록시 흐름](https://github.com/user-attachments/assets/03484f6e-54e6-4c5d-af0d-b208492aa3cc)

---

##  DefaultAdvisorAutoProxyCreator

- **자동 프록시 생성기**, 스프링 빈 후처리기
- 빈 생성 시마다 포인트컷을 적용하고 **프록시로 감싸서 빈으로 등록**

>  빈 후처리기를 활용하면 원본 빈 대신 프록시를 등록할 수 있음

---

##  포인트컷 표현식 (AspectJ 기반)

- `AspectExpressionPointcut` 사용 시, **클래스 + 메소드 선정 기준**을 한 번에 지정
- `execution()` 사용 예:

```java
execution(* com.example.service.*Service.*(..))
