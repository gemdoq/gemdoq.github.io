---
layout: single
title: "스프링부트 개발 흔한 실수와 해결책"
date: 2025-06-30 14:12:23 +0900
categories: Spring Boot
tags: [N+1, Transaction]
typora-root-url: ../

---
#### 📌 용어 한눈에
- DI: 의존성 주입, 객체 의존 관계 자동 관리  
- JPA: 객체와 DB 매핑 API  
- Hibernate: JPA 구현체, 객체-테이블 매핑  
- EntityManager: 엔티티 생명주기 관리  
- Transaction: DB 작업 논리적 단위  
- EntityGraph: 연관 데이터 명시적 로드  
- N+1 문제: 연관 엔티티 조회 시 쿼리 과다 실행  

---
#### 📌 실수와 해결책이란
스프링부트 개발에서 흔한 실수  
설정 오류, JPA 오용, 트랜잭션 관리 부족  
성능 저하, 데이터 불일치 유발  

JPA와 Hibernate로 데이터 조작  
EntityManager가 DB와 엔티티 조율  
올바른 사용으로 문제 예방  

---
#### 📌 쉽게 이해
노트북 파일 관리 비유  
노트북(애플리케이션)은 파일(데이터) 관리  
JPA는 관리 규칙  
Hibernate는 저장/수정 로직(ORM)  
EntityManager는 파일 관리자, 작업 정리  

실수는 파일 잘못 저장/삭제  
EntityManager가 작업 조정, Hibernate 실행  

---
#### 📌 흔한 실수와 해결

##### ✅ 설정: application.yml
H2 DB 예시  
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

> 💡 팁  
> IntelliJ H2 콘솔로 데이터 확인  
> 환경 변수로 URL 관리  

##### ✅ 의존성
build.gradle 설정  
```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
implementation 'org.springframework.boot:spring-boot-starter-web'
runtimeOnly 'com.h2database:h2'
```

> 💡 팁  
> Gradle 빌드 후 IntelliJ 의존성 확인  

##### ✅ 실수 1: 트랜잭션 누락
@Transactional 없으면 EntityManager 작업 비일관  
데이터 불일치 위험  

**해결**  
서비스 메서드에 @Transactional  
```java
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;

    @Transactional
    public User updateUser(Long id, String username) {
        User user = userRepository.findById(id).orElseThrow(() -> new RuntimeException("User not found"));
        user.setUsername(username);
        return user;
    }
}
```

> 💡 팁  
> IntelliJ로 트랜잭션 로그 확인  
> application.yml에 `spring.jpa.show-sql: true` 설정  

##### ✅ 실수 2: N+1 문제
연관 엔티티 조회 시 쿼리 과다 실행  
예: 사용자 10명, 팀 조회 → 1+10 쿼리  

**원인**  
@ManyToOne FetchType.EAGER로 즉시 로드  
EntityManager가 연관 데이터별 쿼리 요청  

**해결**  
FetchType.LAZY, EntityGraph 사용  
```java
import jakarta.persistence.Entity;
import jakarta.persistence.FetchType;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.ManyToOne;
import jakarta.persistence.NamedEntityGraph;
import jakarta.persistence.NamedAttributeNode;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@NamedEntityGraph(name = "User.withTeam", attributeNodes = @NamedAttributeNode("team"))
@Entity
@Getter
@Setter
@NoArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;

    @ManyToOne(fetch = FetchType.LAZY)
    private Team team;
}

@Entity
@Getter
@Setter
@NoArgsConstructor
public class Team {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
}

import org.springframework.data.jpa.repository.EntityGraph;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
    @EntityGraph(value = "User.withTeam")
    List<User> findAll();
}
```

> 💡 팁  
> IntelliJ JPA 콘솔로 쿼리 수 확인  
> LAZY로 불필요 로드 방지  

##### ✅ 실수 3: 프로덕션 ddl-auto
ddl-auto: create로 데이터 손실  
프로덕션에서 치명적  

**해결**  
ddl-auto: none 또는 validate  
```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: validate
```

> 💡 팁  
> application.yml 환경 변수로 설정 분리  

##### ✅ 실수 4: 잘못된 DI
@Autowired 필드 주입, 테스트 어려움  

**해결**  
생성자 주입  
```java
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;
}
```

> 💡 팁  
> IntelliJ로 의존성 그래프 확인  

##### ✅ 실수 5: 엔티티 노출
컨트롤러에서 엔티티 반환  
Hibernate 프록시로 JSON 오류  

**해결**  
DTO로 변환  
```java
public record UserDto(Long id, String username) {
    public UserDto(User user) {
        this(user.getId(), user.getUsername());
    }
}

import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;

    @GetMapping("/{id}")
    public UserDto getUser(@PathVariable Long id) {
        User user = userService.findUser(id);
        return new UserDto(user);
    }
}
```

> 💡 팁  
> Postman으로 API 테스트  
> IntelliJ HTTP 클라이언트 활용  

---
#### 📌 실수 예방의 가치
올바른 설정으로 안정성 향상  
JPA, Hibernate로 데이터 관리 간소화  
EntityManager가 일관성 보장  
개발 효율성 증가  