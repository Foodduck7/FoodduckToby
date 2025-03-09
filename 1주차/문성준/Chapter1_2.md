
---

# 싱글톤 레지스트리와 오브젝트 스코프 & DI

## 1. 오브젝트의 동일성과 동등성

Java에서 "오브젝트가 같다"는 표현을 사용할 때는 주의해야 한다.

 이는 "동일성"과 "동등성" 두 가지 개념으로 나뉜다.

- **동일성**: 두 개의 참조 변수가 동일한 메모리 주소를 가리키는 경우 (`==` 연산자 사용)
- **동등성** : 두 개의 오브젝트가 같은 데이터를 포함하는 경우 (`equals()` 메서드 사용)

```
String a = new String("Hello");
String b = new String("Hello");

System.out.println(a == b);       // false (서로 다른 객체이므로 동일하지 않음)
System.out.println(a.equals(b));  // true (내용이 같으므로 동등함)
```

### 1-2. `equals()`와 `hashCode()`

자바에서 `equals()` 메서드를 오버라이딩할 때는 `hashCode()`도 같이 오버라이딩해야 한다. 
그렇지 않으면 `HashMap` 같은 컬렉션에서 예상치 못한 동작이 발생할 수 있다.

```
class User {
    private String name;

    public User(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        User user = (User) obj;
        return Objects.equals(name, user.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}
```

---

## 2. @Configuration을 이용한 싱글톤 빈 관리

### 직접 객체 생성과 `@Configuration`을 이용한 차이점

**[직접 객체 생성]**

```

Dao factory = new Dao();
UserDao dao1 = factory.userDto();
UserDao dao2 = factory.userDto();

System.out.println(dao1); // ex> x001
System.out.println(dao2); // ex> x002 (다른 객체)
```

**[`@Configuration`을 이용한 싱글톤 관리]**

```
@Configuration
public class DaoConfig {
    @Bean
    public UserDao userDao() {
        return new UserDao();
    }
}
ApplicationContext context = new AnnotationConfigApplicationContext(DaoConfig.class);
UserDao dao1 = context.getBean(UserDao.class);
UserDao dao2 = context.getBean(UserDao.class);

System.out.println(dao1 == dao2); // true (싱글톤 유지)
```

---

## 3. 싱글톤 패턴과 싱글톤 레지스트리

### 싱글톤 패턴

싱글톤 패턴은 애플리케이션에서 단 하나의 객체만 존재하도록 강제하는 디자인 패턴이다.

```
public class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return instance;
    }
}
```

**문제점 ?**

- 상속이 불가능함 (`private` 생성자로 인해)
- 테스트가 어렵다 (다른 객체로 대체 불가능)
- JVM이 여러 개 존재하는 서버 환경에서는 싱글톤이 보장되지 않음
- 전역 상태를 가지기 쉬워 유지보수성이 떨어짐

### 스프링의 싱글톤 레지스트리

스프링 컨테이너는 기본적으로 빈을 싱글톤으로 관리하며, `싱글톤 레지스트리` 역할을 수행한다. 개발자가 직접 싱글톤을 구현하지 않아도 스프링이 이를 자동으로 보장한다.

---

## 4. 멀티스레드 환경에서의 싱글톤 문제

### Race Condition 발생 가능성

싱글톤 빈을 여러 스레드가 동시에 사용하면 데이터 덮어쓰기 등의 문제가 발생할 수 있다.

```
@Component
public class Counter {
    private int count = 0;

    public void increment() {
        count++;
    }
}
```

위 코드는 멀티스레드 환경에서 `count` 값이 예상과 다르게 동작할 수 있다.

### 해결책: 무상태 (Stateless) 디자인

- **상태를 내부에 저장하지 않는다.** (필드를 공유하지 않고 메서드 파라미터로 전달)
- `ThreadLocal`을 활용하여 스레드별로 별도 저장 공간을 확보할 수도 있다.

---

## 5. 스프링 빈의 스코프

### 빈의 주요 스코프

| 스코프 | 설명 |
| --- | --- |
| singleton | 애플리케이션 전체에서 단 하나의 인스턴스 |
| prototype | 요청할 때마다 새로운 객체 생성 |
| request | HTTP 요청마다 새로운 객체 생성 |
| session | 웹 세션별로 하나의 인스턴스 유지 |

```
@Component
@Scope("prototype")
public class PrototypeBean {
    // 매번 새로운 객체 생성
}
```

---

## 6. 의존 관계 주입 - DI

DI는 **오브젝트 간의 의존성을 외부에서 주입해주는 패턴**으로, 객체의 생성과 의존성 결정을 컨테이너가 담당하도록 한다.

```
public class Dao {
    private final Service service;

    public Dao(Service service) {
        this.service = service;
    }
}
```

### **DI의 장점**

- 코드 재사용성이 높아짐
- 객체 간 결합도가 낮아져 테스트 용이
- 다양한 객체로 교체 가능하여 확장성이 증가

---

## 7. 의존 관계 주입의 방법

### 1) 생성자 주입 (권장하는 방법)

```
@Component
public class UserService {
    private final UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

- **불변성 유지** 가능 (final 키워드 사용)
- 순환 의존성을 방지할 수 있음

### 2) 수정자(Setter) 주입

```
@Component
public class UserService {
    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

- 의존성이 필수가 아닐 때 사용 가능
- 하지만 `final` 사용 불가 → 불변성 약화

### 3) 필드 주입 (지양해야한다.)

```
@Component
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

- DI 프레임워크가 없으면 테스트하기 어려움
- @Autowired는 지양하자.

---

<정리>

이 글을 통해 싱글톤 패턴, 스프링 빈의 스코프, 의존 관계 주입에 대해 정리했다.

DI는 객체지향 설계를 유연하게 만들며, 유지보수성을 향상시킨다.

---

### <질문>

해당 회차를 읽고 의문 및 질문점이 많이 생겼던 것 같다.

아래의 질문들에 대한 답들을 이제 공부해 나갈 것이다.

### **1. 오브젝트의 동일성과 동등성**

- Java에서 `equals()`를 오버라이딩하지 않으면 기본적으로 `==`과 같은 동작을 하는데, 이를 방지하기 위한 방법은?
- `hashCode()`를 오버라이딩할 때 주의해야 할 점은?
- `equals()`와 `==`을 혼용하면 발생할 수 있는 문제점은?

### **2. @Configuration과 싱글톤**

- `@Bean`을 선언한 메서드를 여러 번 호출해도 같은 인스턴스를 반환하는 이유는?
- `@Configuration`을 사용하지 않고 직접 객체를 생성하는 것이 왜 문제가 될 수 있는가?
- `@Component`와 `@Bean`의 차이점은?

### **3. 싱글톤 패턴과 싱글톤 레지스트리**

- Java의 싱글톤 패턴과 Spring의 싱글톤 레지스트리는 어떤 차이가 있는가?
- 싱글톤 패턴의 단점을 보완하기 위해 Spring이 제공하는 기능은 무엇인가?
- Spring의 빈이 싱글톤으로 관리된다고 해도, 여러 개의 JVM에서 동일한 인스턴스를 공유할 수 없는 이유는?

### **4. 멀티스레드 환경에서 싱글톤의 문제**

- 멀티스레드 환경에서 싱글톤이 Race Condition을 유발할 수 있는 이유는?
- 무상태(stateless) 객체와 상태(stateful) 객체의 차이는?
- 멀티스레드 환경에서 상태를 관리하기 위한 해결책은?

### **5. 스프링 빈의 스코프**

- 프로토타입 스코프를 사용할 때 주의해야 할 점은?
- 요청(Request) 스코프와 세션(Session) 스코프는 어떻게 동작하는가?
- `@Scope("prototype")`을 사용하면 어떤 장점과 단점이 있는가?

### **6. 의존 관계 주입 (Dependency Injection, DI)**

- 의존 관계 주입(DI)이 필요한 이유는?
- `생성자 주입`, `수정자 주입`, `필드 주입`의 차이점과 각각의 장점과 단점은?
- `ApplicationContext`를 직접 호출해서 빈을 찾는 방법과 `@Autowired`를 이용한 방법의 차이는?
- DI를 활용하면 테스트 코드가 용이해지는 이유는?

### **7. 의존 관계 주입의 응용**

- 스프링에서 DI를 활용해 개발 환경과 운영 환경을 쉽게 전환하는 방법은?
- 인터페이스를 사용한 DI의 장점과 단점은?
- 스프링에서 인터페이스 없이 직접 클래스를 주입하면 발생할 수 있는 문제점은?

---

## 실무적으로 생각 한다면?

### **1. 싱글톤과 실무에서의 빈 관리**

- `@Configuration`을 사용한 싱글톤 관리는 일반적인 프로젝트에서 어떻게 활용되는가?→ 예: **서비스 레이어에서 빈을 싱글톤으로 관리하면 어떤 이점이 있을까?**
- **멀티스레드 환경**에서 `@Service` 빈을 사용하면 발생할 수 있는 문제는?→ 예: **`@Transactional`을 붙인 서비스 메서드가 여러 스레드에서 호출될 때 동시성 문제가 발생할까?**
- **Spring의 빈 라이프사이클에서 싱글톤 빈의 생성과 소멸 시점을 정확히 관리하는 방법은?**→ 예: **`@PreDestroy`, `@PostConstruct`, `DisposableBean`, `InitializingBean` 등을 실무에서 언제 활용할까?**
- **싱글톤으로 관리되는 서비스 클래스에서 상태를 저장할 경우 생길 수 있는 문제점과 해결 방법은?**→ 예: **인스턴스 변수를 사용하지 않고 요청별 데이터를 관리하려면 어떤 방식을 사용해야 할까?**
    - 해결책: `ThreadLocal`, `RequestScope`, `PrototypeScope`, `TransactionSynchronizationManager` 활용 가능

---

### **2. 빈의 스코프와 실무 적용**

- `prototype` 스코프 빈을 사용해야 하는 경우는?→ 예: **매 요청마다 새로운 객체가 필요할 때 (예: 비밀번호 암호화, JWT 토큰 생성기)**
- `request`와 `session` 스코프 빈을 실무에서 언제 사용해야 하는가?→ 예: **API 요청별로 인증 정보를 저장하는 객체(UserContext)를 `request` 스코프 빈으로 만들면 어떤 이점이 있을까?**
- `singleton` 빈 안에서 `prototype` 빈을 주입하면 생기는 문제점과 해결 방법은?→ **해결 방법: `ObjectFactory`, `Provider` 또는 `lookup-method` 사용**
- 빈의 스코프를 잘못 사용해서 발생할 수 있는 대표적인 버그는?→ 예: **싱글톤 빈이 프로토타입 빈을 주입받으면 최초 생성된 객체만 계속 사용됨.**

---

### **3. 의존 관계 주입(DI)과 실무에서의 모범 사례**

- **스프링 DI(의존 관계 주입)를 잘못 적용하면 어떤 문제가 발생할 수 있는가?**→ 예: **필드 주입 vs 생성자 주입, 어느 것이 더 나은가?**
    - 생성자 주입을 사용하면 **테스트 가능성이 높아지고, 순환 의존성을 방지할 수 있음**.
- **DI를 활용하면 프로젝트 유지보수가 용이해지는 이유는?**→ 예: **인터페이스 기반 개발을 하면 어떤 장점이 있을까?**
    - 다양한 구현체를 쉽게 교체 가능 (예: `MySQLRepository` → `PostgresRepository` 변경 시 코드 수정 최소화)
- **실무에서 DI를 적용할 때 가장 자주 겪는 문제는 무엇인가?**→ 예: **순환 의존성(Circular Dependency)이 발생했을 때 어떻게 해결할 수 있을까?**
    - 해결책: **`@Lazy`, `@DependsOn`, `BeanFactory`, `ApplicationContextAware` 사용**

---

### **4. Spring의 의존성 관리와 확장성**

- `@Autowired` vs `@Inject` vs `@Resource`의 차이점과 실무에서 어떤 것을 사용하는 것이 좋은가?
- **DI를 활용해 테스트를 쉽게 만들기 위한 실무적인 방법은?**→ 예: **Mockito, @MockBean, @TestConfiguration을 어떻게 활용하면 좋을까?**
- **설정 클래스(@Configuration)를 여러 개로 나누는 실무적인 기준은?**→ 예: `DatasourceConfig`, `SecurityConfig`, `ServiceConfig` 등의 설정을 분리하면 어떤 이점이 있을까?
- **DI를 통해 객체를 동적으로 생성해야 할 때 어떤 방식이 좋은가?**→ 예: **팩토리 패턴, `@Conditional`, `BeanFactory`, `ApplicationContext.getBean()` 중 언제 무엇을 선택해야 할까?**

---

### **5. 트랜잭션과 싱글톤의 관계**

- **트랜잭션 관리(`@Transactional`)와 싱글톤 빈이 충돌할 가능성이 있는가?**→ 예: **같은 빈에서 여러 트랜잭션을 관리할 때, 새로운 트랜잭션이 열리면서 문제가 발생할 수 있는가?**
- **Spring에서 트랜잭션을 활용할 때 주의해야 할 싱글톤 이슈는?**→ 예: **서비스 레이어에서 상태를 변경하는 로직을 싱글톤 빈에서 수행할 경우 트랜잭션 롤백이 문제를 일으킬 가능성이 있는가?**
- **`@Transactional`을 사용했을 때 같은 스레드에서 여러 개의 요청이 발생하면 트랜잭션이 공유될 가능성이 있는가?**
    - 해결책: `Propagation.REQUIRES_NEW`, `TransactionTemplate` 사용

---

### **6. DI와 성능 최적화**

- **의존성 주입 방식이 성능에 미치는 영향은?**→ 예: **지연 주입(`@Lazy`)을 사용할 때 어떤 성능상의 이점이 있는가?**
- **빈 초기화 지연(Lazy Initialization)이 필요한 경우는?**→ 예: **리소스가 많이 드는 객체 (DB 커넥션, 대형 캐시)**
- **싱글톤 빈을 잘못 사용했을 때 발생할 수 있는 성능 이슈는?**→ 예: **싱글톤 빈에서 DB Connection을 직접 들고 있으면 어떤 문제가 생길까?**
    - 해결책: **DB Connection Pool (HikariCP) 활용**

---

### **7. Spring DI와 대규모 프로젝트에서의 유지보수**

- **DI를 적용할 때 인터페이스와 구현 클래스를 어떻게 나누는 것이 좋은가?**→ 예: **인터페이스를 통해 기능을 정의하고, 구현체를 나누면 어떤 장점이 있는가?**
- **마이크로서비스 아키텍처에서 DI를 활용하면 서비스 간 통신을 어떻게 유연하게 할 수 있는가?**→ 예: **FeignClient, RestTemplate, WebClient를 활용할 때 DI를 어떻게 적용해야 할까?**
- **DI를 활용한 모듈화 전략은 무엇인가?**→ 예: **각 모듈별로 독립적인 설정을 갖게 하려면 어떻게 해야 할까?**
    - 해결책: `@Import`, `@ComponentScan(basePackages = "...")` 활용