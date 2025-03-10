# Chapter3 스프링의 JDBCTEMPLATE

JdbcTemplate은 생성자의 파라미터로 DataSource를 주입한다.

```java
public class Dao {
    private JdbcTemplate jdbcTemplate;
    private DataSource dataSource;

    public void setJdbcTemplate(DataSource dataSource) {
        this.jdbcTemplate = new jdbcTemplate(dataSource);
        this.dataSource = dataSource;
    }
}
```

<br>

## JdbcTemplate의 기능

**1. update()**
- 변경 쿼리를 실행할 때 사용한다. 

```java 
String sql = "UPDATE users SET name = ? WHERE id = ?";
jdbcTemplate.update(sql, "John", 1);
```

**2. queryForInt()**
- Deprecated 메소드로 사용을 권장하지 않는다. 

**3. queryForObject()**

- 쿼리의 <span style="color:orange;">**결과가 Row 하나**</span>일 경우 사용

```java 
String sql = "SELECT name FROM users WHERE id = ?";
String name = jdbcTemplate.queryForObject(sql, String.class, 1);
```

**4. query()**
- 쿼리의 <span style="color:orange;">**결과가 여러개의 Row**</span>일 경우 사용 (`return : List<T> 형태`)

```java
String sql = "SELECT * FROM users";
List<User> users = jdbcTemplate.query(sql, (rs, rowNum) -> 
    new User(rs.getInt("id"), rs.getString("name"))
);
```

