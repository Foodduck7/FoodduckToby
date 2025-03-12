# Chapter 3 - 스프링의 JdbcTemplate

## 1. JdbcTemplate 개요

### 1-1. JdbcTemplate의 역할
`JdbcTemplate`은 **JDBC API를 간편하게 사용할 수 있도록 도와주는 스프링의 핵심 클래스**다.  
기존의 JDBC는 **Connection, Statement, ResultSet** 등을 직접 관리해야 하고, `try-catch-finally` 블록을 사용하여 리소스를 직접 해제해야 하는 불편함이 있었다.

`JdbcTemplate`은 **반복적인 JDBC 코드(연결, 실행, 자원 해제)를 내부에서 처리하여, 개발자가 SQL에 집중할 수 있도록 도와준다.**

### 1-2. JdbcTemplate의 주요 특징
- **코드 간결화** : 반복되는 JDBC 관련 코드 제거
- **자원 관리 자동화** : `Connection`, `Statement`, `ResultSet`을 자동으로 닫아줌
- **예외 변환** : `SQLException`을 스프링의 `DataAccessException`으로 변환
- **명확한 API 제공** : `query()`, `update()`, `queryForObject()` 등의 직관적인 메서드 지원

---

## 2. JdbcTemplate 설정

### 2-1. JdbcTemplate을 사용한 DAO 구성

```
import org.springframework.jdbc.core.JdbcTemplate;
import javax.sql.DataSource;

public class UserDao {
    private JdbcTemplate jdbcTemplate;
    
    public UserDao(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }
}
```
DataSource를 주입받아 JdbcTemplate을 생성하는 방식으로 사용한다.

2-2. DataSource 설정 방법

(1) XML 기반 설정
```
<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
    <property name="url" value="jdbc:mysql://localhost:3306/mydb" />
    <property name="username" value="root" />
    <property name="password" value="password" />
</bean>

<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <constructor-arg ref="dataSource" />
</bean>

```

(2) Java 기반 설정 (Spring Boot 사용 시)
```java
@Configuration
public class DataSourceConfig {
    @Bean
    public DataSource dataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("root");
        dataSource.setPassword("password");
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```
Spring Boot에서는 application.properties에 DataSource 정보를 설정하면 자동으로 JdbcTemplate이 생성됨.

---

# 3. JdbcTemplate 주요 메서드
###  3-1. update()
- INSERT, UPDATE, DELETE 등 데이터 변경 작업에 사용됨.
- update() 메서드는 실행된 행(row)의 개수를 반환함.
사용 예시
```agsl
String sql = "UPDATE users SET name = ? WHERE id = ?";
int rowsAffected = jdbcTemplate.update(sql, "John", 1);
System.out.println("업데이트된 행 수: " + rowsAffected);
```
update()는 PreparedStatement의 ? 바인딩을 자동으로 처리해준다.

---

## 3-2. queryForObject()

- **단일 행(single row) 결과를 조회할 때 사용**
- 쿼리 결과가 **반드시 1개의 행**이어야 함 (여러 개일 경우 예외 발생)

### 사용 예시
```java
String sql = "SELECT name FROM users WHERE id = ?";
String name = jdbcTemplate.queryForObject(sql, String.class, 1);
System.out.println("이름: " + name);
```
결과가 한 행일 때만 사용해야 하며,
결과가 없거나 여러 개일 경우 예외 발생 (IncorrectResultSizeDataAccessException)

---

## 3-3. query()
여러 개의 행을 조회할 때 사용
RowMapper 인터페이스를 활용하여 결과를 객체로 변환 가능
```agsl
// 람다 써서 간단하게 RowMapper 구현
String sql = "SELECT * FROM users";
List<User> users = jdbcTemplate.query(sql, (rs, rowNum) ->
    new User(rs.getInt("id"), rs.getString("name"))
);
for (User user : users) {
    System.out.println(user.getName());
}
```
## RowMapper 인터페이스 활용
```
public class UserRowMapper implements RowMapper<User> {
    @Override
    public User mapRow(ResultSet rs, int rowNum) throws SQLException {
        return new User(rs.getInt("id"), rs.getString("name"));
    }
}
```
`List<User> users = jdbcTemplate.query("SELECT * FROM users", new UserRowMapper());`
RowMapper를 별도로 구현하여 재사용 가능하도록 구성할 수도 있음.

---

## 3-4. queryForList()
특정 컬럼의 값을 리스트 형태로 반환할 때 사용
```String sql = "SELECT name FROM users";
List<String> names = jdbcTemplate.queryForList(sql, String.class);
System.out.println(names);
```
특정 컬럼의 값만 필요한 경우 query()보다 간결하게 사용할 수 있음.

## 3-5. batchUpdate()
여러 개의 데이터를 한 번에 삽입/수정할 때 사용
대량의 데이터 변경 시 성능을 향상시킴.
```
String sql = "INSERT INTO users (id, name) VALUES (?, ?)";
List<Object[]> batchArgs = new ArrayList<>();
batchArgs.add(new Object[]{1, "Alice"});
batchArgs.add(new Object[]{2, "Bob"});
batchArgs.add(new Object[]{3, "Charlie"});

jdbcTemplate.batchUpdate(sql, batchArgs);

```
batchUpdate()를 활용하면 반복적인 insert/update 작업을 최적화할 수 있음.

---

## 4. JdbcTemplate 사용 시 주의할 점

###  4-1. queryForObject()의 단일 결과 제한
queryForObject()는 결과가 여러 개일 경우 예외 발생
결과가 없을 수도 있는 경우 query()를 사용하여 처리하는 것이 안전

---

### 4-2. 예외 처리
JDBC 예외(SQLException)는 스프링이 자동으로 변환하여 DataAccessException을 던짐
특정 예외를 처리하려면 try-catch로 감싸고 DataAccessException을 캐치하여 처리 가능

---

### 4-3. 트랜잭션 처리
JdbcTemplate은 기본적으로 AutoCommit 상태
여러 개의 update 작업을 하나의 트랜잭션으로 묶으려면 @Transactional을 활용
   ```
   @Service
   public class UserService {
   @Autowired
   private JdbcTemplate jdbcTemplate;

   @Transactional
   public void updateUserAndLog(int userId, String newName) {
   jdbcTemplate.update("UPDATE users SET name = ? WHERE id = ?", newName, userId);
   jdbcTemplate.update("INSERT INTO logs (message) VALUES (?)", "User updated: " + userId);
   }}
   ```
@Transactional을 붙이면 두 개의 update가 하나의 트랜잭션으로 처리됨.