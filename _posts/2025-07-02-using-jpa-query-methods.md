---
title: "스프링부트 JPA Query Methods 활용"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - JPA
  - Query Methods
last_modified_at: 2025-07-24
---

#### 📌 용어 한눈에
- JPA Query Methods: 메서드 이름으로 쿼리 자동 생성  
- Spring Data JPA: JPA 간소화 스프링 모듈  
- Hibernate: JPA 구현체, 매핑 처리  
- EntityManager: 엔티티 생명주기 관리  
- Repository: 데이터 접근 계층, 쿼리 정의  

---
#### 📌 Query Methods란
메서드 이름을 규칙대로 지으면  
Spring Data JPA가 쿼리 자동 생성  
SQL 없이 데이터 조회/조작  

Hibernate가 쿼리 실행  
EntityManager가 DB와 엔티티 조율  

---
#### 📌 Query Methods 쉽게 이해
노트북 검색 비유  
노트북(DB)은 데이터 저장  
검색창(Repository)에 키워드(메서드) 입력  
JPA는 검색 규칙  
Hibernate는 검색 로직(ORM)  
EntityManager는 검색 관리자, 조건 정리  

예: "user" 시작 파일 검색  
`findByUsernameStartingWith` 입력  
EntityManager가 조건 정리, Hibernate가 검색  

Query Methods는 간단한 키워드로 데이터 접근  

---
#### 📌 설정

##### ✅ application.yml
H2 DB 설정  
```yaml
spring:
  datasource:
    url: ${H2_URL:jdbc:h2:mem:testdb}
    driver-class-name: org.h2.Driver
    username: sa
    password:
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
  h2:
    console:
      enabled: true
      path: /h2-console
```

H2 콘솔로 쿼리 확인  

> 💡 팁  
> IntelliJ H2 콘솔 접속  
> 환경 변수 `H2_URL`로 동적 설정  

---
#### 📌 Query Methods 구현

##### ✅ 의존성
build.gradle에 추가  
```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
runtimeOnly 'com.h2database:h2'
```

> 💡 팁  
> Gradle 빌드 후 IntelliJ 의존성 확인  

##### ✅ 엔티티
User 테이블 매핑  
```java
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Entity
@Table(name = "users")
@Getter
@Setter
@NoArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;

    private String email;

    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }
}
```

@Entity: 데이터 구조 정의  
Lombok으로 코드 간소화  

##### ✅ 리포지토리
쿼리 메서드 정의  
```java
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByUsernameStartingWith(String prefix);

    Optional<User> findByEmail(String email);

    List<User> findByUsernameAndEmail(String username, String email);

    List<User> findByOrderByUsernameAsc();
}
```

`findByUsernameStartingWith`은 `LIKE 'prefix%'` 생성  
EntityManager 조정, Hibernate 실행  

##### ✅ 서비스
```java
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import java.util.List;
import java.util.Optional;

@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;

    public List<User> findUsersByUsernamePrefix(String prefix) {
        return userRepository.findByUsernameStartingWith(prefix);
    }

    public Optional<User> findUserByEmail(String email) {
        return userRepository.findByEmail(email);
    }

    public List<User> findUsersByUsernameAndEmail(String username, String email) {
        return userRepository.findByUsernameAndEmail(username, email);
    }

    public List<User> findAllUsersSortedByUsername() {
        return userRepository.findByOrderByUsernameAsc();
    }

    public User createUser(String username, String email) {
        User user = new User(username, email);
        return userRepository.save(user);
    }
}
```

> 💡 팁  
> IntelliJ JPA 콘솔로 쿼리 확인  
> @Transactional로 트랜잭션 관리  

##### ✅ 컨트롤러
REST API로 테스트  
```java
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;
import java.util.List;
import java.util.Optional;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;

    @PostMapping
    public User createUser(@RequestParam String username, @RequestParam String email) {
        return userService.createUser(username, email);
    }

    @GetMapping("/prefix")
    public List<User> findUsersByUsernamePrefix(@RequestParam String prefix) {
        return userService.findUsersByUsernamePrefix(prefix);
    }

    @GetMapping("/email")
    public Optional<User> findUserByEmail(@RequestParam String email) {
        return userService.findUserByEmail(email);
    }

    @GetMapping("/search")
    public List<User> findUsersByUsernameAndEmail(@RequestParam String username, @RequestParam String email) {
        return userService.findUsersByUsernameAndEmail(username, email);
    }

    @GetMapping("/sorted")
    public List<User> findAllUsersSortedByUsername() {
        return userService.findAllUsersSortedByUsername();
    }
}
```

> 💡 팁  
> Postman으로 API 테스트  
> IntelliJ HTTP 클라이언트 활용  

---
#### 📌 Query Methods 장점
- SQL 없이 메서드 이름으로 쿼리  
- 가독성, 유지보수성 향상  
- JPA, Hibernate 자동화  
- 초보자도 빠르게 사용  

---
#### 📌 주의점
- 메서드 이름 JPA 규칙 준수  
- 복잡 쿼리는 `@Query` 고려  
- 영속성 컨텍스트 관리 주의  
- 쿼리 성능 테스트 필수  
- show-sql로 쿼리 확인  

---
#### 📌 Query Methods의 가치
데이터 조회 간소화  
JPA, Hibernate로 자동화  
EntityManager가 DB 연결 조율  
핵심 로직에 집중 가능  

---
#### ✍ 느끼며
Query Methods 처음엔 이름 짓기 어색했음  
검색 비유로 역할 이해  
잘못된 이름은 쿼리 오류  
IntelliJ JPA 콘솔로 디버깅 편리  
application.yml로 설정 간소화  
Query Methods로 설계 집중 가능