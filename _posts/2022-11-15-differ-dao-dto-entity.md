---
layout: single
title: "DAO, DTO, Entity Class의 차이"
date: 2022-11-15 10:22:33 +0900
categories: java
tags: [DAO, DTO, Entity]
typora-root-url: ../
---


## DAO, DTO, Entity
> - DAO(Data Access Object)
> - DTO(Data Transfer Object)
> - Entity Class

<br>

![spring-package-flow](/images/2022-11-15-differ-dao-dto-entity/spring-package-flow.png){: width="560"}

## DAO(Data Access Object)

- 실제로 DB에 접근하는 객체
- Persistence Layer(DB에 data를 CRUD하는 계층)
- Service와 DB를 연결하는 고리의 역할
- SQL를 사용(개발자가 직접 코딩)하여 DB에 접근한 후 적절한 CRUD API를 제공
- JPA 대부분의 기본적인 CRUD method를 제공
- `extends JpaRepository<User, Long>`

### JPA 사용 예시

```java
public interface QuestionRepository extends
  CrudRepository<Question, Long> {
}
```

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

### VO(Value Object)

- VO는 DTO와 동일한 개념이지만 읽기전용(read only)
- VO는 특정한 비즈니스 값을 담는 객체
- DTO는 Layer간의 통신 용도로 오고가는 객체

<br>

## Entity Class

- 실제 DB의 테이블과 매칭될 클래스
- 영속성 어노테이션인 `@Entity`, `@Column`, `@Id`등 이용
- 최대한 외부에서 Entity 클래스의 getter method를 사용하지 않도록 해당 클래스 안에서 필요한 로직 method 구현
- 여기서 구현한 method는 주로 Service Layer에서 사용

### Entity 클래스와 DTO 클래스를 분리하는 이유
- View Layer와 DB Layer의 역할 분리
- 테이블과 매핑되는 Entity 클래스가 변경되면 여러 클래스에 영향을 끼치게 되는 반면 View와 통신하는 DTO 클래스(Request / Response 클래스)는 자주 변경되므로 분리

### User Entity Class 예시

```java
@Entity
@Getter
@AllArgsConstructor
@NoArgsConstructor
@EqualsAndHashCode
@ToString
public class User implements Serializable {
  private static final long serialVersionUID = 7342736640368461848L;

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  @JsonProperty
  private Long id;

  @Column(nullable = false)
  @JsonProperty
  private String email;

  @Column(nullable = false)
  @JsonIgnore
  private String password;
}
```

<br>