
# Chapter2-2 스프링과 테스트

## 1. 스프링에서의 테스트 개념

### **1.1 스프링이 제공하는 가치: 객체지향과 테스트**

스프링 프레임워크는 **객체지향적인 설계**와 **테스트 가능성**을 핵심 가치로 삼는다.

현대의 애플리케이션은 점점 복잡해지고 있으며, 이러한 환경에서 **테스트가 가능한 코드**를 작성하는 것은 필수적인 요소가 되었다.

- 좋은 코드는 **테스트하기 쉬운 코드**이다.
- 코드가 유연하게 변경될 수 있도록 하려면 의존성 주입(DI)를 활용해야 한다.
- **테스트는 코드의 유지보수를 쉽게 하고, 새로운 기능 추가 시 리팩토링을 원활하게 해준다.**

---

## 2. 테스트 전략과 단위 테스트(Unit Test)

### **2.1 웹을 통한 테스트의 문제점**

웹 애플리케이션을 테스트할 때 **UI를 직접 조작해서 테스트하는 방법**은 다음과 같은 문제점을 가진다.

- **테스트의 범위가 너무 넓다.**
- **모든 레이어를 다 구현한 후에야 테스트할 수 있다.**
- **에러 발생 시 원인을 특정하기 어렵다.**
- **테스트 자동화가 어렵고 반복 실행하기 힘들다.**

**따라서, 보다 작은 단위에서 테스트하는 '단위 테스트(Unit Test)'가 필요하다.**

### **2.2 단위 테스트란?**

단위 테스트는 **하나의 기능(관심사)만을 독립적으로 테스트하는 방식**이다.

📌 단위 테스트의 특징

- 외부 환경(DB, 네트워크 등)에 영향을 받지 않음
- 특정 기능만 테스트하므로 빠르게 실행 가능
- 실패한 경우 원인을 쉽게 파악할 수 있음

```
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
}

@Test
void testAddition() {
    Calculator calculator = new Calculator();
    assertThat(calculator.add(2, 3), is(5));
}
```

**이처럼 독립적인 메서드를 검증하는 것이 단위 테스트의 핵심이다.**

---

## 3. 자동화된 테스트와 JUnit

### **3.1 단순 print문 테스트의 문제점**

```
public static void main(String[] args) {
    Calculator calculator = new Calculator();
    int result = calculator.add(2, 3);
    if (result == 5) {
        System.out.println("테스트 성공");
    } else {
        System.out.println("테스트 실패");
    }
}
```

- **책에서 누누이 강조하는, 개발자가 직접 결과를 눈으로 확인해야 한다.**
- **테스트 자동화가 불가능하다.**
- **수동 테스트는 반복 실행이 어렵다.**

이를 해결하기 위해 **JUnit 프레임워크**를 사용한다. (현재 Junit5)

### **3.2 JUnit을 활용한 테스트 자동화**

JUnit을 사용하면, 테스트 실행 결과를 자동으로 검증할 수 있다.

```
import static org.junit.Assert.*;
import org.junit.Test;

public class CalculatorTest {
    @Test
    public void testAddition() {
        Calculator calculator = new Calculator();
        assertEquals(5, calculator.add(2, 3));
    }
}
```

**JUnit을 활용하면 테스트를 자동화하고 반복 실행이 가능하다.**

---

## 4. 좋은 테스트의 특징

### **4.1 일관성 있는 테스트**

좋은 테스트는 **일관된 결과**를 보장해야 한다.

- 실행 환경에 따라 결과가 달라지면 안 된다.
- 데이터베이스 상태에 의존하지 않아야 한다.
- 실행 순서와 관계없이 동일한 결과를 보장해야 한다.

해결 방법

- **테스트 전후에 DB 초기화 (`deleteAll()`)**
- **Mock 객체를 활용하여 외부 의존성을 제거**
- **JUnit의 `@BeforeEach`와 `@AfterEach`를 활용하여 테스트 환경을 일정하게 유지**

```
@BeforeEach
public void setUp() {
    userRepository.deleteAll();
}
```

- **매 테스트 실행 전, DB 데이터를 초기화하여 일관성을 유지한다.**

### **4.2 포괄적인 테스트와 네거티브 테스트**

- **테스트는 다양한 상황을 고려해야 한다.**
- - **경계값, 예외 케이스 등을 포함해야 한다.**
- **예외 발생이 예상되는 경우 이를 검증하는 코드가 필요하다.**

```
@Test(expected = IllegalArgumentException.class)
public void testInvalidInput() {
    Calculator calculator = new Calculator();
    calculator.add(null, 5);
}
```

**잘못된 입력 값이 주어졌을 때, 예상된 예외가 발생하는지 검증한다.**

---

## 5. 테스트 주도 개발 (TDD)

### **5.1 TDD란?**

TDD(Test Driven Development)는 **테스트 코드를 먼저 작성한 후,**

**이를 통과하는 실제 코드를 작성하는 개발 방법론**이다.

📌 **TDD의 과정**

1.   실패하는 테스트를 먼저 작성한다.

1. 테스트를 통과할 수 있도록 최소한의 코드를 작성한다.
2. 테스트가 성공하면 리팩토링을 진행한다.

```
@Test
public void testMultiplication() {
    Calculator calculator = new Calculator();
    assertEquals(6, calculator.multiply(2, 3));
}
```

**처음에는 `multiply()` 메서드가 없으므로 실패한다.**

```
public int multiply(int a, int b) {
    return a * b;
}
```

**이제 테스트를 통과한다.**

**TDD를 활용하면 코드 품질을 높이고, 불필요한 코드 작성을 방지할 수 있다.**

---

## 6. 스프링 테스트와 컨텍스트 공유

스프링 애플리케이션에서는 `ApplicationContext`를 공유하는 방식으로 테스트를 최적화할 수 있다.

```
@ExtendWith(SpringExtension.class)
@ContextConfiguration(locations="/spring/applicationContext.xml")
public class UserDaoTest {
    @Autowired ApplicationContext applicationContext;
    UserDao userDao;

    @BeforeEach
    public void setUp() {
        this.userDao = applicationContext.getBean("userDao", UserDao.class);
    }
}
```

**스프링 컨텍스트를 공유하여 불필요한 초기화를 방지하고, 테스트 속도를 향상시킨다.**

---

## 7. 결론

- **테스트는 코드의 유지보수를 돕고, 리팩토링 시 신뢰성을 높인다.**
- **JUnit을 활용하면 테스트 자동화가 가능하다.**
- **TDD를 사용하면 코드 품질이 향상된다.**
- **스프링 컨텍스트를 공유하면 테스트 성능을 최적화할 수 있다.**

! 중요 ! **테스트를 꾸준히 작성하면, 더욱 안정적인 애플리케이션을 개발할 수 있다.**

---

# 실무기준으로 생각한다면 어떤 것들이 있을까?

3회차와 이론이 연결되는 내용이라 질문이 비슷하거나 동일한 것이 있을 수 있습니다.

그만큼 한 번 더 생각하게 되는 시간이라 질의란에 기재했습니다.

## **1. 단위 테스트 vs 통합 테스트**

**❓단위 테스트와 통합 테스트의 적절한 비율은 어느 정도가 이상적일까?**

- 단위 테스트(빠르고 독립적) vs 통합 테스트(신뢰성 높지만 느림)의 균형을 어떻게 맞출까?
- 실제 실무에서는 어떤 테스트가 더 많이 필요할까?
- **CI/CD 파이프라인에서 실행할 테스트의 종류를 어떻게 선정하는 것이 좋을까?**

❓ **서비스 레이어(Service Layer)까지 단위 테스트하는 것이 좋은가, 아니면 통합 테스트에서 검증해야 하는가?**

- `MockMvc`를 이용해 서비스 레이어까지 단위 테스트하는 것이 좋은가?
- 아니면 `@SpringBootTest`와 함께 실제 DB와 연동하여 검증하는 것이 좋은가?
- Service 레이어의 단위 테스트를 작성할 때, **DB 연동이 필요한 경우 어떻게 할까?**

❓ **스프링 컨트롤러(Controller) 테스트는 단위 테스트로 해야 할까, 아니면 통합 테스트로 해야 할까?**

- `@WebMvcTest`를 이용한 컨트롤러 단위 테스트 vs 실제 애플리케이션 구동 후 `MockMvc`로 검증하는 방식의 차이점?

---

## **2. Mock 객체와 테스트 데이터 관리**

❓ **Mock 객체를 사용하는 것이 무조건 좋은 방법일까?**

- Mockito와 `@MockBean`, `@SpyBean`의 차이점과 활용 사례는?
- Mock을 과도하게 사용하면 오히려 실제 로직과 다른 동작을 검증하게 되는 문제가 발생할 수 있는데, 이를 방지하는 방법은?

❓ **테스트 데이터는 어떻게 관리하는 것이 좋을까?**

- `@Sql`을 활용해서 미리 데이터 삽입 vs `TestContainer`를 이용한 방법 중 어떤 것이 더 적절할까?
- 테스트 실행 후 데이터 정리를 자동화하는 방법은?
- Fake 데이터 생성을 위한 라이브러리 (`java-faker`, `EasyRandom`)을 활용하면 장점과 단점은?

❓ **테스트 코드가 오래될수록, 테스트 데이터가 많아질수록 유지보수하기 어려워지는데, 이를 어떻게 관리할까?**

---

## **3. DB와 트랜잭션을 활용한 테스트 최적화**

❓ **`@Transactional`을 활용한 테스트가 자동으로 롤백되는 이유는?**

- `@Rollback(false)` 옵션을 사용할 경우 어떤 상황에서 문제가 발생할 수 있을까?
- **데이터가 실제로 DB에 반영되는지 확인하고 싶다면 어떤 전략이 필요할까?**

❓ **`@DataJpaTest`를 사용할 때, 실제 DB 대신 H2 인메모리 DB를 사용하는 것이 항상 좋은가?**

- H2 vs 실제 PostgreSQL/MySQL 테스트 환경을 구성할 때 각각의 장점과 단점은?
- Flyway나 Liquibase를 적용한 프로젝트에서 테스트 환경을 구축할 때 주의할 점은?

❓ **Spring Boot 테스트에서 TestContainers를 활용하는 것이 실무적으로 적절한가?**

- TestContainers는 실제 DB 환경과 동일한 상태를 만들 수 있지만, 테스트 실행 시간이 길어지는 단점이 있음.
- **로컬에서 빠르게 실행할 테스트와 CI/CD에서 운영 DB 환경을 고려한 테스트를 어떻게 분리할까?**

---

## **4. Spring Boot 환경에서 테스트 실행 속도 최적화**

❓ **테스트 실행 속도가 점점 느려지는 경우 어떻게 해결할 수 있을까?**

- `@SpringBootTest`는 무겁고 실행 시간이 길어지는데, 이를 최적화하려면?
- `@DirtiesContext`를 남발하면 애플리케이션 컨텍스트가 자주 재생성되는데, 이를 방지하려면?
- 테스트 실행 속도를 높이기 위해 Spring Boot의 `@TestConfiguration`을 활용하는 방법은?

❓ **ApplicationContext를 공유하는 방식과 성능 최적화 방법**

- `@SpringBootTest`를 사용하면 ApplicationContext를 매번 새로 생성하는데, 이를 방지하려면?
- `@TestConfiguration`을 활용하여 테스트 전용 빈을 주입하는 방법은?

❓ **"모든 테스트를 @SpringBootTest로 실행하면 안 되는 이유"는 무엇인가?**

- `@SpringBootTest`는 실제 애플리케이션과 동일한 환경을 구성하지만, 불필요한 설정까지 로딩할 수도 있음.
- 대신 `@WebMvcTest`, `@DataJpaTest`, `@MockBean` 등을 활용하여 필요한 부분만 테스트하는 것이 더 효율적임.
- **그러면, 언제 `@SpringBootTest`를 사용해야 하는가?**

---

## **5. 보안과 인증이 포함된 API 테스트**

❓ **Spring Security가 적용된 API를 테스트하려면 어떻게 해야 할까?**

- 인증이 필요한 컨트롤러를 MockMvc로 테스트하려면?
- Security Filter를 우회하는 테스트 설정을 어떻게 할까?
- `@WithMockUser`를 활용한 테스트와 `OAuth2` 기반 인증이 적용된 API 테스트의 차이점은?

❓ **JWT 기반 인증을 적용한 API 테스트를 어떻게 작성할까?**

- `@TestPropertySource`를 이용하여 테스트용 시크릿 키를 설정하는 방법?
- JWT 토큰을 발급하는 과정도 함께 테스트해야 할까, 아니면 단순히 토큰을 Mocking해서 사용해야 할까?

---

## **6. API 테스트 자동화 전략**

**❓API의 요청과 응답을 자동화하는 가장 좋은 방법은?**

- `RestAssured` vs `MockMvc`의 차이점과 선택 기준은?
- 대규모 프로젝트에서 API 테스트를 지속적으로 유지보수하려면?

❓ **API 테스트에서 JSON Schema Validation을 적용하는 것이 유용한가?**

- JSON 응답 구조가 바뀌었을 때, 테스트가 자동으로 실패하도록 만들려면?
- `JSONAssert`와 같은 라이브러리를 활용하여 응답 데이터를 검증하는 방법은?

❓ **클라이언트-서버 간의 API 스펙이 자주 변경될 경우, 테스트 코드가 유지보수하기 어려워질 수 있는데 이를 어떻게 해결할까?**

---

## **7. 마이크로서비스 환경에서 테스트 전략**

❓ **마이크로서비스 환경에서는 단일 서비스만 테스트하면 충분할까?**

- 여러 개의 서비스가 연동되는 경우, 통합 테스트를 어떻게 구성해야 할까?
- E2E(End-to-End) 테스트를 수행하는 것이 현실적으로 가능한가?

❓ **Kafka, RabbitMQ 같은 메시지 브로커를 활용한 테스트는 어떻게 해야 할까?**

- 비동기 이벤트 기반 시스템에서 단위 테스트를 작성하는 방법은?
- `EmbeddedKafka`를 활용한 테스트 사례는?

❓ **컨테이너 기반 테스트 (Docker, Kubernetes)**

- CI/CD 파이프라인에서 테스트 실행 속도를 높이려면?
- 운영 환경과 동일한 설정을 유지하면서 테스트하려면?