---
title: "스프링부트로 MySQL 연동과 CRUD 구현"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - MySQL
  - JPA
last_modified_at: 2025-07-23
---

#### 📌 용어 한눈에
- MySQL: 관계형 DB, 데이터 영구 저장  
- CRUD: 생성, 조회, 수정, 삭제 작업  
- JPA: 객체와 DB 매핑 API  
- Hibernate: JPA 구현체, 매핑 처리  
- EntityManager: 엔티티 생명주기 관리  
- JDBC: 자바와 DB 연결 표준  

---
#### 📌 MySQL 연동과 CRUD란
MySQL은 데이터를 영구 저장하는 DB  
스프링부트와 연동해 안정적 관리  
CRUD는 데이터 생성, 조회, 수정, 삭제  

JPA와 Hibernate로 객체 중심 CRUD  
EntityManager가 MySQL과 엔티티 조율  

---
#### 📌 MySQL 쉽게 이해
외장 하드 비유  
하드(MySQL)는 데이터 영구 저장  
노트북(애플리케이션)은 하드에 기록/조회  
JPA는 하드 사용 규칙  
Hibernate는 저장/수정 로직(ORM)  
EntityManager는 파일 관리자, 데이터 정리  

사용자 정보 저장(Create) 요청  
사용자가 입력(엔티티)하면  
EntityManager가 정리 후 Hibernate 전달  
Hibernate가 하드(MySQL)에 저장  

MySQL은 안정적 저장소  
JPA, Hibernate, EntityManager 협력  

---
#### 📌 MySQL 설정

##### ✅ application.yml
```yaml
spring:
  datasource:
    url: ${MYSQL_URL:jdbc:mysql://localhost:3306/testdb}
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: ${MYSQL_USER:root}
    password: ${MYSQL_PASSWORD:your-password}
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
```

MySQL 서버 실행 후 DB 생성  
환경 변수로 URL, 사용자, 비밀번호 관리  

> 💡 팁  
> IntelliJ 환경 변수 설정으로 보안 강화  
> JPA 콘솔로 쿼리 확인  

---
#### 📌 CRUD 구현

##### ✅ 의존성 추가
build.gradle에 추가  
```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
runtimeOnly 'mysql:mysql-connector-j:8.0.33'
```

> 💡 팁  
> Gradle 빌드 후 IntelliJ로 의존성 확인  

##### ✅ 엔티티 클래스
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

@Entity: 하드에 파일 구조 정의  
Lombok으로 코드 간소화  

##### ✅ 리포지토리
CRUD 자동화  
```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
}
```

JpaRepository로 Hibernate 활용  
기본 메서드 제공  

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

    public User createUser(String username, String email) {
        User user = new User(username, email);
        return userRepository.save(user);
    }

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }

    public User updateUser(Long id, String username, String email) {
        Optional<User> userOptional = userRepository.findById(id);
        if (userOptional.isEmpty()) {
            throw new RuntimeException("User not found");
        }
        User user = userOptional.get();
        user.setUsername(username);
        user.setEmail(email);
        return userRepository.save(user);
    }

    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

createUser는 파일 생성  
updateUser는 내용 수정  
EntityManager 조정, Hibernate 저장  

> 💡 팁  
> @Transactional로 트랜잭션 관리  
> IntelliJ로 테스트 자동 생성  

##### ✅ 컨트롤러
REST API로 CRUD 테스트  
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

    @GetMapping
    public List<User> getAllUsers() {
        return userService.getAllUsers();
    }

    @GetMapping("/{id}")
    public Optional<User> getUserById(@PathVariable Long id) {
        return userService.getUserById(id);
    }

    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestParam String username, @RequestParam String email) {
        return userService.updateUser(id, username, email);
    }

    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
    }
}
```

> 💡 팁  
> Postman으로 API 테스트  
> IntelliJ HTTP 클라이언트로 요청 작성  

---
#### 📌 MySQL 장점
- 영구 저장으로 프로덕션 적합  
- JPA, Hibernate 통합 쉬움  
- 대량 데이터 처리 가능  
- 커뮤니티 지원 풍부  

---
#### 📌 주의점
- MySQL 서버, DB 생성 필수  
- JDBC URL, 인증 정보 확인  
- ddl-auto 프로덕션에서 validate 추천  
- 영속성 컨텍스트 관리 주의  
- 대량 데이터 시 쿼리 최적화  

---
#### 📌 MySQL과 CRUD의 가치
MySQL로 안정적 데이터 관리  
JPA, Hibernate로 코드 간소화  
EntityManager가 연결 조율  
핵심 로직에 집중 가능  

---
#### ✍ 느끼며
MySQL 연동 처음엔 설정 복잡했음  
외장 하드 비유로 역할 이해  
잘못된 설정 데이터 불일치 위험  
IntelliJ JPA 콘솔로 디버깅 편리  
application.yml로 설정 간소화  
MySQL과 JPA로 설계 집중 가능