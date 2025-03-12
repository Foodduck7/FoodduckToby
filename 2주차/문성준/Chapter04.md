# Chapter 4,예외 처리

# 1. 예외 처리의 핵심 원칙

예외 처리는 다음 두 가지 원칙 중 하나를 반드시 충족해야 한다.

1. 모든 예외는 적절하게 복구되어야 한다.
2. 작업을 중단시키고 운영자 또는 개발자에게 분명하게 통보되어야 한다.

즉, 예외를 무조건 던져버리거나 무시하는 것은 바람직하지 않으며, 예외가 발생했을 때 해당 예외를 어떻게 처리할 것인지에 대한 전략이 필요하다.

# 2. 예외 처리 시 하면 안 되는 코드

## 2.1. 무조건적인 예외 무시

```
try {
// 예외 발생 가능 코드
} catch (Exception e) {
System.out.println(e); // 단순 출력
}
```

이런 방식은 예외를 단순히 출력하는 것에 그쳐 실제 원인을 파악하기 어렵게 만든다. 로그 파일에 기록하거나 적절한 조치를 취해야 한다.

---

## 2.2. 무책임한 throws 선언

```
public void method1() throws Exception {
// ...
}

public void method2() throws Exception {
// ...
}

public void method3() throws Exception {
// ...
}
```

모든 메서드에서 throws Exception을 선언하면, 어떤 예외가 발생할지 명확하지 않으며, 예외를 적절하게 처리할 기회를 박탈당한다.

---

# 3. 예외의 종류와 특징

## 3.1. Error

java.lang.Error 클래스의 하위 클래스

JVM에서 발생하는 비정상적인 시스템 오류

대표적인 예: OutOfMemoryError, StackOverflowError

일반적인 애플리케이션 코드에서 직접 처리할 필요는 없음

---

## 3.2. Exception과 Checked Exception

개발자가 작성한 코드에서 발생하는 예외

Exception은 Checked Exception과 Unchecked Exception으로 나뉜다.

Checked Exception: RuntimeException을 상속하지 않은 예외로 반드시 try-catch나 throws를 사용해야 한다.

Unchecked Exception (Runtime Exception): RuntimeException을 상속한 예외로, 명시적인 예외 처리를 강제하지 않음.

Checked Exception을 남용하면 코드가 불필요하게 복잡해질 수 있다.

---

## 3.3 RuntimeException과 Unchecked Exception

대표적인 예외: NullPointerException, IllegalArgumentException

프로그램 오류로 인해 발생하는 예외이며, 반드시 명시적으로 처리할 필요는 없음

---

# 4. 효과적인 예외 처리 방법

## 4.1 예외 복구 (Recovery)

예외 상황을 감지하고 문제를 해결한 후 정상적인 흐름을 유지하는 방법

예제: 네트워크 연결 실패 시 일정 횟수 재시도 후 복구
```
int maxRetry = 3;
while (maxRetry-- > 0) {
try {
// 네트워크 요청
break;
} catch (IOException e) {
if (maxRetry == 0) throw new RuntimeException("연결 실패", e);
}
}
```
---

##  4.2 예외 회피 (Avoidance)

예외를 직접 처리하지 않고 호출한 곳으로 넘김

무책임한 예외 회피는 바람직하지 않으며, 명확한 의도를 가지고 사용해야 함
```
public void process() throws IOException {
throw new IOException("IO 문제 발생");
}

```
---

##  4.3 예외 전환 (Exception Translation)

예외를 보다 의미 있는 예외로 변환하여 던지는 기법

예제: 내부적으로 발생한 예외를 보다 의미 있는 도메인 예외로 변경
```
try {
// DB 작업 수행
} catch (SQLException e) {
throw new DataAccessException("데이터베이스 접근 오류", e);
}

```
---


### 4.3.1. 중첩 예외 (Nested Exception)

getCause()를 이용하여 원래 예외를 추적 가능

```

catch (SQLException e) {
throw new CustomDatabaseException("DB 오류 발생").initCause(e);
}
```

---


### 4.3.2. 예외 Wrapping

Checked Exception을 Unchecked Exception으로 변환하여 사용 가능

```

catch (SQLException e) {
throw new RuntimeException("DB 오류", e);
}
```

---


# 5. 애플리케이션 예외 처리 전략

## 5.1. 비즈니스 로직 예외

애플리케이션 로직에서 예외를 의도적으로 발생시키는 경우

예제: 중복된 사용자 ID를 검증하고 예외 발생

```

if (userExists(userId)) {
throw new DuplicateUserException("이미 존재하는 사용자 ID입니다.");
}

```

---

## 5.2. SQLException 처리

SQLException은 DB에 따라 에러 코드가 달라질 수 있으므로 직접 처리하지 않고, 스프링의 DataAccessException으로 변환하여 사용

```

try {
// DB 작업 수행
} catch (SQLException e) {
throw new DataAccessException("데이터베이스 오류", e);
}
```

---

# 6. 스프링의 예외 전환 방법

## 6.1. DataAccessException

특정 기술(JDBC, JPA 등)에 의존적인 SQLException을 스프링이 DataAccessException으로 변환하여 일관된 예외 처리가 가능하도록 지원

## 6.2. 스프링 예외 계층 구조

```
RuntimeException
└ NestedRuntimeException
└ DataAccessException
├ TransientDataAccessException
├ ConcurrencyFailureException
├ OptimisticLockingFailureException
└ ObjectOptimisticLockingFailureException
```


이를 통해 개발자는 특정 DB 기술에 의존하지 않고 일관된 방식으로 예외를 처리할 수 있다.

---

# 7. 예외 처리 시 고려할 점

## 7.1. 예외 발생 시 서비스 중단 방지

예외가 발생하더라도 전체 서비스가 중단되지 않도록 개별 요청 수준에서 예외를 처리해야 함

## 7.2. 로그를 남길 때 주의할 점

예외 발생 시 반드시 로그를 남겨야 하지만, 민감한 정보가 노출되지 않도록 주의해야 함

## 7.3. 사용자 경험 고려

예외가 발생했을 때 사용자가 이해할 수 있는 메시지를 제공해야 함

예제: "시스템 오류가 발생했습니다." 대신 "현재 서비스가 원활하지 않습니다. 잠시 후 다시 시도해 주세요."

---

다시 말해,
- 예외 처리는 단순히 오류를 출력하는 것이 아니라, 적절한 복구, 회피, 전환을 통해 안정적인 애플리케이션 운영을 보장해야 한다.
- 스프링의 DataAccessException과 같은 추상화된 예외 계층을 활용하면, 기술에 종속되지 않는 예외 처리가 가능하다.
- 적절한 예외 처리 전략을 수립하여 개발자와 사용자 모두에게 신뢰성 있는 시스템을 제공해야 한다

---

# 질문
## 실무적으로 고려해본다면 ??

### 1. 예외 복구 전략
서비스에서 네트워크 오류가 발생했을 때, 몇 번의 재시도를 허용하는 것이 가장 적절할까?<br>
특정 예외 상황에서 즉시 복구를 시도할지, 아니면 로그만 남기고 중단하는 것이 더 나은 경우는 언제일까?

### 2. 예외 회피 전략
예외를 상위 계층으로 던지는 것이 적절한 경우는 어떤 경우일까?<br>
단순히 throws를 선언하는 것이 아니라, 보다 의미 있는 예외 처리를 위해 어떤 방식으로 로깅과 메시지를 구성해야 할까?

### 3. 예외 전환 전략
SQLException을 DataAccessException으로 변환하는 것이 왜 중요한가?<br>
특정 API나 프레임워크를 사용할 때, 예외 전환 패턴을 어떻게 적용하면 유지보수가 용이할까?

### 4. Checked Exception vs Unchecked Exception
Checked Exception을 사용하면 코드가 복잡해질 수 있는데, 실무에서 언제 사용해야 하는 것이 가장 적절할까?<br>
런타임 예외로 전환하는 것이 적절하지 않은 경우는 어떤 경우일까?

### 5. 로깅 및 사용자 경험

예외 로그를 남길 때, 개발자에게는 상세한 정보를 주되, 사용자에게는 적절한 메시지를 보여주려면 어떻게 구성하는 것이 좋을까?<br>
민감한 데이터를 포함한 예외 메시지가 로그에 기록되지 않도록 하는 방법은?
<hr>

## 이론적으로 더 고려해서 질문한 것들이 있다면 ?

### 1. 예외 처리 원칙
예외를 단순히 무시하는 것이 왜 위험한가?<br>
throws Exception을 무분별하게 사용하면 어떤 문제가 발생할 수 있는가?

### 2. 예외의 종류
Checked Exception과 Unchecked Exception의 차이점은?<br>
대표적인 RuntimeException의 예시와 이를 방지할 수 있는 코드 작성법은?

### 3. 효과적인 예외 처리 방법
예외 복구를 구현할 때 고려해야 할 요소는 무엇인가?<br>
예외 회피와 예외 전환을 비교하고, 각각의 장점과 단점은?

### 4. 스프링의 예외 처리
DataAccessException을 사용하면 어떤 이점이 있는가?<br>
스프링 예외 계층 구조를 이해하면 실무에서 어떤 이점이 있을까?

### 5. 트랜잭션과 예외 처리
스프링에서 예외가 발생했을 때 트랜잭션 처리는 어떻게 되는가?<br>
@Transactional을 사용할 때 예외 발생 시 주의해야 할 점은?
