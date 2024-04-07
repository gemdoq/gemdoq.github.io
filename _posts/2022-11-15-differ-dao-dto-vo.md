---
layout: single
title: "DAO, DTO, VO의 차이"
date: 2022-11-15 10:22:33 +0900
categories: java
tags: [DAO, DTO, VO]
typora-root-url: ../
---


## DAO, DTO, VO
> - DAO(Data Access Object)
> - DTO(Data Transfer Object)
> - VO(Value Object)

<br>

![spring-package-flow](/images/2022-11-15-differ-dao-dto-entity/spring-package-flow.png){: width="560"}

## DAO(Data Access Object)

- 객체지향적인 설계패턴에서 사용하는 데이터 저장소에 접근하기 위한 객체
- Persistence Layer(DB에 data를 CRUD하는 계층)
- DB와 비지니스 로직을 분리/연결하는 고리 역할

### JDBC template 사용 DAO 구현 예시

1. 데이터베이스 연결을 위한 **DataSource 설정** (스프링의 XML 또는 Java Config에 정의)

    ```java
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/mydatabase");
        dataSource.setUsername("username");
        dataSource.setPassword("password");
        return dataSource;
    }
    ```

<br>

2. JdbcTemplate을 사용하기 위한 Bean으로 정의

    ```java
    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
    ```

<br>

3. 데이터 액세스 작업을 위한 DAO인터페이스 작성

    ```java
    public interface UserDao {
        void addUser(User user);
        User getUserById(int userId);
        List<User> getAllUsers();
        void updateUser(User user);
        void deleteUser(int userId);
    }
    ```

<br>

4. DAO 인터페이스를 구현하는 클래스를 작성하고 JDBC Template를 사용하여 데이터베이스 액세스 작업을 구현

    ```java
    @Repository
    public class UserDaoImpl implements UserDao {
        
        @Autowired
        private JdbcTemplate jdbcTemplate;
        
        @Override
        public void addUser(User user) {
            String sql = "INSERT INTO users (username, email) VALUES (?, ?)";
            jdbcTemplate.update(sql, user.getUsername(), user.getEmail());    
        }
        
        @Override
        public User getUserById(int userId) {
            String sql = "SELECT * FROM users WHERE id=?";
            return jdbcTemplate.queryForObject(sql, new Object[]{userId}, new UserMapper());
        }
        
        @Override
        public List<User> getAllUsers() {
            String sql = "SELECT * FROM users";
            return jdbcTemplate.query(sql, new UserMapper());
        }
        
        @Override
        public void updateUser(User user) {
            String sql = "UPDATE users SET username=?, email=? WHERE id=?";
            jdbcTemplate.update(sql, user.getUsername(), user.getEmail(), user.getId());
        }
        
        @Override
        public void deleteUser(int userId) {
            String sql = "DELETE FROM users WHERE id=?";
            jdbcTemplate.update(sql, userId);
        }
    }
    ```

<br>

5. 데이터베이스 Record를 Java Object로 mapping하기 위한 RowMapper클래스를 작성

    ```java
    public class UserMapper implements RowMapper<User> {
        @Override
        public User mapRow(ResultSet rs, int rowNum) throws SQLException {
            User user = new User();
            user.setId(rs.getInt("id"));
            user.setUsername(rs.getString("username"));
            user.setEmail(rs.getString("email"));
            return user;
        }
    }
    ```

<br>

6. DAO를 사용하여 DB CRUD 수행

    ```java
    @Autowired
    private UserDao userDao;
    
    // 사용자 추가
    User user = new User();
    user.setUsername("John");
    user.setEmail("john@example.com");
    userDao.addUser(user);
    
    // 사용자 조회
    User fetchedUser = userDao.getUserById(1);
    
    // 모든 사용자 조회
    List<User> users = userDao.getAllUsers();
    
    // 사용자 업데이트
    fetchedUser.setUsername("Updated Name");
    userDao.updateUser(fetchedUser);
    
    // 사용자 삭제
    userDao.deleteUser(1);
    ```

<br>

위와 같이 JDBC Template을 사용하여 간단하고 효율적으로 DB 접속을 수행할 수 있으며, 반복적인 작업을 줄이고, 더 나은 유지보수성과 확장성을 제공할 수 있다.

<br>

## DTO(Data Transfer Object)

- 계층간 데이터 교환을 위한 객체(Java Beans)
- DB에서 데이터를 얻어 Service나 Controller 등으터 보낼 때 사용하는 객체
- DB의 데이터가 DTO형태로 Presentation Logic Tier로 전달
- 로직을 갖고 있지 않은 순수한 데이터 객체

### UserDto 예시

```java
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class UserDto {
  @NotBlank
  @Pattern(regexp = "^([\\w-]+(?:\\.[\\w-]+)*)@((?:[\\w-]+\\.)*\\w[\\w-]{0,66})\\.([a-z]{2,6}(?:\\.[a-z]{2})?)$")
  private String email;

  @JsonIgnore
  @NotBlank
  @Size(min = 4, max = 15)
  private String password;

  @NotBlank
  @Size(min = 6, max = 10)
  private String name;

  public User toEntity() {
    return new User(email, password, name);
  }

  public User toEntityWithPasswordEncode(PasswordEncoder bCryptPasswordEncoder) {
    return new User(email, bCryptPasswordEncoder.encode(password), name);
  }
}
```

<br>

## VO(Value Object)

- VO는 값 그 자체를 나타내는 객체
- DTO와 달리 로직을 포함할 수 있음
- 불변성의 보장을 사용하기 위해 생성자를 사용
- 서로 다른 이름을 갖는 VO 인스턴스라도 필드값이 같다면 두 인스턴스는 같은 객체
- 주소값이 아닌 값의 동등성을 가지도록 hashCode()와 equals()를 재정의
- setter 같은 변조가능성이 있는 메서드가 존재하면 안 됨

### VO 예시

```java
public class CarVO {
    private final String color;
    
    public CarVO(String color) {
        this.color = color;
    }
    
}
```

자동차의 color 필드를 가지는 VO에 대해 비교하는 테스트

```java
class CarVOTest {
    
    @Test
    void CarVOEqualTest() {
        final String color = "red";
        
        CarVO car1 = new CarVO(color);
        CarVO car2 = new CarVO(color);
        
        assertThat(car1).isEqualTo(car2);
        assertThat(car1).hasSameHashCodeAs(car2);
    }
    
}
```

<img src="/images/2022-11-15-differ-dao-dto-vo/FailedVOTest.png" alt="FailedVOTest" style="zoom:50%;" />

객체의 주소값을 비교하기 때문에 값이 같은 것과 상관없이 테스트를 통과하지 못한다.

이런 문제가 있기 때문에 값을 비교하기 위해서 equals()와 hashCode()를 Overriding해줘야 한다.

```java
public class CarVO {
    private final String color;
    
    public CarVO(String color) {
        this.color = color;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        CarVO carVO = (CarVO) o;
        return Objects.equals(color, carVO.color);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(color);
    }
}
```

<img src="/images/2022-11-15-differ-dao-dto-vo/PassedTest.png" alt="PassedTest" style="zoom:50%;" />

<br>

