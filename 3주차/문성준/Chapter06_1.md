# 토비의 스프링 - AOP
- 트랜잭션 코드의 분리, 고립된 단위 테스트 정리 <br>

- AOP(Aspect-Oriented Programming, 관점 지향 프로그래밍)는 스프링에서 핵심 기능과 부가적인 관심사를 분리하여 코드의 유지보수성을 높이고, 테스트의 독립성을 보장하는 중요한 개념이라고 생각합니다.<br> 특히 트랜잭션 관리와 같은 부가적인 관심사를 AOP로 분리하면 핵심 로직과 구분할 수 있어 <br>더욱 명확하고 모듈화된 코드를 작성할 수 있게 됩니다.

## 1. 트랜잭션 코드의 분리
   트랜잭션 관리는 데이터베이스 작업에서 ACID(Atomicity, Consistency, Isolation, Durability)를 보장하기 위한 필수적인 기능이다. 일반적으로 트랜잭션을 처리하려면 try-catch-finally 블록을 사용하거나, 특정 API를 직접 호출하는 방식으로 구현한다. 하지만 이 방식은 다음과 같은 문제를 야기할 수 있다.

### 1.1 트랜잭션 코드가 비즈니스 로직을 침범하는 문제
트랜잭션 관리는 핵심 비즈니스 로직과 직접적인 관련이 없지만, 기존 방식으로 구현하면 try-catch-finally 블록이 코드 곳곳에 중복되며, 코드의 가독성이 떨어지고 유지보수가 어려워진다.

(기존 방식) 트랜잭션 코드가 포함된 서비스 메서드

```java
// 예시 코드 
public void someBusinessLogic() {
Connection con = null;
try {
con = dataSource.getConnection();
con.setAutoCommit(false);

        // 핵심 비즈니스 로직
        doSomething();

        con.commit();
    } catch (Exception e) {
        if (con != null) {
            try {
                con.rollback();
            } catch (SQLException se) {
                throw new RuntimeException(se);
            }
        }
        throw new RuntimeException(e);
    } finally {
        if (con != null) {
            try {
                con.close();
            } catch (SQLException se) {
                throw new RuntimeException(se);
            }
        }
    }
}
```

위의 코드처럼 트랜잭션을 직접 관리하면 코드가 복잡해지고, 핵심 비즈니스 로직(doSomething())과 분리되지 않아 유지보수성이 떨어진다.

### 1.2 AOP를 이용한 트랜잭션 분리
AOP를 사용하면 트랜잭션 코드를 핵심 비즈니스 로직에서 완전히 분리할 수 있다.

(AOP 방식) 트랜잭션을 분리한 코드
```
@Transactional
public void someBusinessLogic() {
doSomething(); // 핵심 로직만 존재
}
```
@Transactional 어노테이션을 사용하면, AOP를 통해 트랜잭션을 자동으로 관리할 수 있다.

비즈니스 로직에서는 트랜잭션 코드에 대한 신경을 쓸 필요가 없다.

스프링이 자동으로 트랜잭션을 시작하고, 예외 발생 시 롤백을 수행한다.

## 2. 메서드 분리와 DI(의존성 주입)를 이용한 클래스 분리
   AOP를 적용하면서 중요한 원칙 중 하나는 단일 책임 원칙(SRP) 을 준수하여 코드의 응집도를 높이고, 의존성을 줄이는 것이다.

### 2.1 메서드 분리를 통한 역할 분리
트랜잭션과 관련된 코드를 별도의 클래스에 위임하면 더 깔끔한 구조를 만들 수 있다.


```
@Service
public class SomeService {
private final SomeRepository repository;

    public SomeService(SomeRepository repository) {
        this.repository = repository;
    }

    @Transactional
    public void process() {
        repository.saveData(); // 핵심 로직만 포함
    }
}```

트랜잭션과 관련된 처리는 스프링이 자동으로 수행하며, 핵심 로직만 남게 된다.

2.2 DI를 이용한 클래스 분리
Dao, 서비스 및 트랜잭션 관리를 분리하여 역할을 명확히 할 수 있다.

의존성 주입(DI)을 통해 각 객체가 독립적으로 테스트 가능하게 만들 수 있다.


```
@Repository
public class SomeRepository {
public void saveData() {
// DB 저장 로직
}
}
```

### 3. 고립된 단위 테스트

#### 3.1 복잡한 의존관계 속에서 테스트
   AOP를 적용한 코드의 단위 테스트는 복잡한 의존 관계를 고려해야 한다. 트랜잭션이 적용된 코드를 직접 테스트하면 데이터베이스와 연결이 필요하기 때문에 테스트가 어려울 수 있다.

#### 3.2 테스트 대상 오브젝트 고립시키기
단위 테스트(Unit Test)는 테스트 대상 객체를 고립시켜야 한다. 그러나 트랜잭션이 포함된 메서드를 테스트하면 데이터베이스가 필요할 수 있다. 이를 방지하려면 목(Mock) 객체 또는 Stub을 활용해야 한다.

(목 객체 사용) 데이터베이스를 사용하지 않는 단위 테스트

```
@ExtendWith(MockitoExtension.class)
class SomeServiceTest {

    @Mock
    private SomeRepository repository;

    @InjectMocks
    private SomeService service;

    @Test
    void testProcess() {
        // given
        doNothing().when(repository).saveData();

        // when
        service.process();

        // then
        verify(repository, times(1)).saveData();
    }
}```


@Mock을 사용하여 SomeRepository의 가짜 객체를 생성하고, 실제 데이터베이스를 사용하지 않는다.

verify() 메서드를 통해 saveData()가 한 번 호출되었는지 확인한다.

## 4. 단위 테스트와 통합 테스트

### 4.1 단위 테스트(Unit Test)
   단위 테스트는 하나의 메서드 또는 클래스 단위로 수행되며, 가능한 한 외부 의존성을 최소화해야 한다.

Mock 프레임워크를 사용하여 데이터베이스를 거치지 않고 테스트를 수행한다.

### 4.2 통합 테스트
실제 데이터베이스와 연동하여 전체적인 흐름을 검증하는 테스트이다.

스프링 컨텍스트와 트랜잭션이 정상적으로 동작하는지 확인하는 데 사용된다.

```
@SpringBootTest
@Transactional
class SomeServiceIntegrationTest {

    @Autowired
    private SomeService service;

    @Autowired
    private SomeRepository repository;

    @Test
    void testProcessWithDatabase() {
        service.process();
        assertNotNull(repository.findData());
    }
}
```
@SpringBootTest를 사용하여 스프링 컨텍스트를 로드한다.

@Transactional을 활용하여 테스트 후 자동으로 롤백하여 데이터 정합성을 유지한다.

## 5. 목(Mock) 프레임워크 활용
   
### 5.1 Mock 객체를 이용한 단위 테스트
   Mock 프레임워크는 실제 객체를 생성하지 않고도 테스트할 수 있도록 돕는다. 대표적으로 Mockito를 활용할 수 있다.
```
@Mock
private SomeRepository repository;
```
@Mock을 사용하면 실제 객체 대신 가짜 객체를 생성하여, 의존성을 제거한 상태에서 테스트할 수 있다.

### 5.2 스프링 컨텍스트 없이 실행 가능한 테스트
@MockBean을 사용하면 스프링 컨텍스트에서 특정 빈을 Mock으로 대체할 수 있다.

```java
@SpringBootTest
class SomeServiceTest {

    @MockBean
    private SomeRepository repository;
}
```
### 정리
- AOP를 활용하여 트랜잭션을 분리하면 핵심 로직과 부가적인 관심사를 분리할 수 있다.

- DI를 통해 클래스와 메서드를 분리하면 코드의 유지보수성이 높아지고, 테스트가 쉬워진다.

- 고립된 단위 테스트를 수행하려면 Mock 프레임워크를 활용하여 데이터베이스 없이 테스트해야 한다.

- 단위 테스트와 통합 테스트를 구분하여 실행하면 코드의 신뢰성을 높일 수 있다.

- Mockito와 같은 Mock 프레임워크를 활용하면 실제 의존성을 제거한 상태에서 테스트할 수 있다.

이러한 개념을 적용하면, 트랜잭션을 분리하면서도 유지보수성과 테스트 용이성을 극대화할 수 있다는 것을 배우게 되었다.