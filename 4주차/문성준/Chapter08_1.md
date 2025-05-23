# 📘 Chapter 8. 스프링이란 무엇인가

## 🌀 스프링이란?

> **자바 엔터프라이즈 개발을 효율적으로 하기 위한 경량급 애플리케이션 프레임워크**  
> 오픈소스로 개발되었으며, DI(의존성 주입), AOP(관심사 분리), 추상화 등을 통해  
> **복잡한 비즈니스 로직과 기술 구현을 깔끔하게 분리**함으로써 생산성과 유지보수성을 극대화한다.

---

## 🔍 스프링의 특징

### 1. 애플리케이션 프레임워크 

- 단순한 기술 프레임워크가 아닌, **비즈니스 로직, 기술, 계층, 영역을 아우르는 범용 개발 프레임워크**
- 일관된 프로그래밍 모델을 제공하여 **전 계층을 관통하는 구조적 통일성**을 지향

> 💡 실무 팁: Controller, Service, Repository 계층 모두 스프링의 일관된 방식(DI, 트랜잭션 관리 등)을 적용 가능

---

### 2. 경량

- 무조건 코드가 가볍다는 의미가 아닌, **서버와 개발 환경의 부담이 적고 필요한 기능만 선택적으로 사용 가능**
- Tomcat, Jetty와 같은 경량 서블릿 컨테이너에서도 완벽하게 구동

> ✅ Spring Boot는 자동 설정 및 내장 서버로 더욱 가볍고 빠른 개발 환경 제공

---

### 3. 오픈소스 

- Apache License 2.0 기반 → **자유롭게 사용/수정/배포 가능**
- 대규모 커뮤니티와 기업(Spring Team, VMware) 지원 → **안정성 확보**

---

## 🎯 스프링의 목적

> **엔터프라이즈 개발의 복잡함을 줄이고, 비즈니스 로직에 집중할 수 있도록 기술적인 복잡성을 추상화/분리**하는 것

### 🧩 왜 복잡한가?

1. **기술적 제약조건**
    - 멀티스레드, 트랜잭션, 보안 등 다양한 고려 요소 존재
2. **비즈니스 로직의 복잡성 증가**
    - 빠르게 변하는 도메인 요구사항 대응 필요

---

## 🛠️ 스프링의 해결 전략

###  비침투적 설계 

- 스프링 기술을 적용했는지 **코드에서 드러나지 않음**
- 예: `@Transactional`, `@Autowired`, `@Component` → 핵심 로직과 분리

---

### 🧩 문제 → 해결 전략

| 문제 | 스프링의 전략 |
|------|----------------|
| 기술 접근 방식이 일관되지 않음 | **추상화(Abstraction)** → 트랜잭션, 데이터 접근, 메시징 등 인터페이스 제공 |
| 기술 코드가 비즈니스 로직과 섞임 | **AOP (관점지향 프로그래밍)** → 보안, 로깅, 트랜잭션 등 별도 분리 가능 |

> 💡 `JdbcTemplate`, `TransactionTemplate`, `@Transactional`, `@Aspect` 등의 도구가 대표적

---

### 💡 비즈니스 로직은 애플리케이션 안에서 처리

- DB는 영속성과 복잡한 검색만 맡기고,  
  **로직 가공/분석은 애플리케이션에서 처리** → 테스트와 확장성에 유리

---

## 🔑 스프링의 핵심 원칙

### 객체지향 설계 기반

- 의존성 주입(DI)
- 관심사 분리(AOP)
- 인터페이스 기반 설계

> 테스트 용이성, 유지보수성, 유연성 확보

---

## 🧪 실무 관점 요약

| 관점 | 실무 적용 사례 |
|------|----------------|
| 일관된 프로그래밍 모델 | Controller → Service → Repository 계층에 공통 적용 |
| 추상화 기반 기술 설계 | DB, 트랜잭션, 메시징 등 특정 구현체 종속성 최소화 |
| 비침투적 설계 | 로깅, 트랜잭션, 인증 등 핵심 로직과 분리 |
| 경량 구조 | Spring Boot + Embedded Tomcat 으로 WAR 없이 실행 |
| 확장성 | WebFlux, Spring Data, Spring Batch 등 다양한 확장 모듈 제공 |

---

## 📚 참고

- 《토비의 스프링 3.1》 - 이일민 저
- 실무 기준: Spring Boot 3.x / Spring Framework 6.x
- 공식 문서: [https://spring.io/projects/spring-framework](https://spring.io/projects/spring-framework)
