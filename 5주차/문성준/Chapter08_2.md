# 🌱 Chapter 8. POJO 프로그래밍

## 🧭 스프링이 지향하는 핵심 철학: POJO

스프링은 **기술적인 복잡성으로부터 개발자를 해방**시키고,  
핵심 로직을 **객체지향적으로, 깔끔하게 구현**할 수 있도록 지원하는 프레임워크다.

> POJO(Plain Old Java Object)는 스프링이 추구하는 가장 중요한 개발 방식이며,  
> 스프링의 모든 기술은 **애플리케이션을 POJO로 개발할 수 있게 하는 수단**이다.

---

## 🧾 POJO란?

**Plain Old Java Object**  
→ 특별한 규약이나 환경, 프레임워크에 종속되지 않고 **순수한 자바 객체**로 설계된 클래스

### ✅ POJO의 조건

- **특정 프레임워크에 종속되지 않음**  
  (ex. `extends`나 `@FrameworkAnnotation` 강제 없음)
- **특정 실행 환경에 의존하지 않음**  
  (ex. 컨테이너가 없어도 테스트 가능)
- **객체지향 원리에 충실함**  
  (캡슐화, 책임 분리, 의존성 최소화)

### ✅ POJO의 장점

- **자유로운 객체지향 설계** 적용 가능
- **테스트 용이성** (JUnit으로 단독 테스트 가능)
- **재사용성/유지보수성** 우수
- **환경 독립성** 확보 (서블릿 컨테이너 없어도 동작 가능)

---

## 💡 스프링은 어떻게 POJO를 실현하는가?

스프링은 핵심 기술인 **DI/IoC**, **AOP**, **PSA**를 통해 POJO 스타일의 애플리케이션 구현을 가능하게 한다.

---

## 🧪 DI (의존성 주입) / IoC (제어의 역전)

> 객체 간의 **의존 관계를 외부에서 주입받도록 하여**,  
> **결합도를 낮추고 테스트/확장성을 향상**시키는 방식

### DI 활용 예시

- **핵심 기능의 변경**  
  → 구현체 교체 (ex. `SmsSender → KakaoSender`)
- **동적 기능 변경**  
  → 런타임 시점에 다른 구현 주입
- **부가기능의 추가**  
  → 프록시 패턴으로 부가기능 삽입
- **인터페이스 분리 & 전략 패턴 활용**
- **싱글톤, 스코프 관리**  
  → `@Scope`, `@Singleton`, `@RequestScope` 등

> ✅ 실무에서는 `@Autowired`, `@Bean`, `@Component`, `@Configuration` 등을 통해 구성

---

## 🌀 AOP (관점지향 프로그래밍)

> 핵심 로직과 관심사(부가기능)를 분리하여,  
> **비즈니스 로직을 깔끔하게 유지**할 수 있도록 지원하는 기술

### 대표 적용 사례

- 트랜잭션 처리 (`@Transactional`)
- 보안 처리 (`@PreAuthorize`)
- 로깅/모니터링
- 성능 측정

> ✅ 실무에서는 `@Aspect`, `@Around`, `@Before`, `@AfterReturning` 등을 통해 구현

---

## 🌐 PSA

> 다양한 기술/환경의 변화에도 코드 변경을 최소화하도록 하는 **서비스 접근 추상화 계층**

### 대표 PSA 예시

| 기능 | 추상화 API | 구현체 |
|------|------------|--------|
| 데이터 접근 | `JdbcTemplate`, `JpaRepository` | JDBC, Hibernate, MyBatis |
| 트랜잭션 | `PlatformTransactionManager` | JTA, JDBC, Hibernate TX |
| 메시징 | `MessageSource` | 파일/DB/클라우드 메시지 |
| 캐시 | `@Cacheable` + CacheManager | EhCache, Redis, Caffeine |

> PSA 덕분에 스프링은 환경 변화에 유연하며, 실무에서도 **플러그인 방식의 기술 교체**가 가능하다.

---

## 📌 실무 포인트 요약

| 기술 | 실무 적용 |
|------|-----------|
| DI/IoC | `@Autowired`, `@Qualifier`, `@Bean`으로 의존성 외부 주입 |
| AOP | 공통 관심사 분리 (로깅, 트랜잭션, 보안 등) |
| PSA | 구현체 교체 없이 API 변경 최소화 |
| POJO | 테스트 가능, 유지보수 쉬움, 프레임워크 독립 |

---

## 📚 참고

- 《토비의 스프링 3.1》 - 이일민 저
- Spring Boot 기준 버전: 3.x (Spring Framework 6.x)
- 공식 문서: [https://docs.spring.io/spring-framework/reference](https://docs.spring.io/spring-framework/reference)
