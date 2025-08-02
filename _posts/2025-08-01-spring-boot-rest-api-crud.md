---
layout: single
title: "Spring Boot로 REST API CRUD 구현"
date: 2025-08-01 17:34:00 +0900
categories: Spring Boot
tags: [Spring Boot, REST API, CRUD]
typora-root-url: ../

---
# ✈️ Spring Boot로 REST API CRUD 구현

## 🎯 개요

Spring Boot로 REST API 만들어 CRUD 기능 구현
JDK21과 Gradle 환경에서 JPA로 데이터베이스 연동
IntelliJ Ultimate로 빠른 개발과 Postman 테스트 가능

## 📋 목록

- [ ]  REST API란?
- [ ]  REST API 만드는 이유
- [ ]  준비 항목
- [ ]  구현 단계
- [ ]  실행과 테스트
- [ ]  실무 팁과 주의사항

---

## ✏️ REST API란?

### REST API 정의

HTTP 프로토콜 기반 클라이언트와 서버 통신 방식
CRUD 연산으로 자원 조작 인터페이스 제공

### 주요 구성 요소

- 엔티티: 데이터베이스 테이블 매핑 객체
- 리포지토리: 데이터베이스 상호작용 인터페이스
- 서비스: 비즈니스 로직 처리 계층
- 컨트롤러: HTTP 엔드포인트 정의

> 💡왜 필요하나?
>
> Spring Boot 프로젝트에서 MSA 적합 API 구현
> Gradle 빌드와 application.yml로 설정 간소화

---

## ✏️ REST API 만드는 이유

- 빠른 개발과 배포: Spring Boot 자동 설정 활용
- MSA 호환: 독립적인 서비스 구성
- 자바 생태계 연계: JPA와 통합 편리
- 커뮤니티 지원: 풍부한 문서와 예시
- 확장성 강화: 모듈화된 기능 추가

---

## ✏️ 준비 항목

- JDK21: java.net에서 다운로드
- Spring Initializr: start.spring.io로 프로젝트 생성
- IntelliJ Ultimate: 개발 환경 구성
- Gradle: 빌드 툴로 의존성 관리
- H2 데이터베이스: 테스트용 메모리 DB
- Postman: API 테스트 도구

> 💡특이사항
>
> Gradle 8.x 이상으로 JDK21 호환성 확보
> application.yml로 데이터소스 설정 관리

---

## ✏️ 구현 단계

### 1단계: 프로젝트 설정

start.spring.io로 Spring Web, Spring Data JPA, H2 선택 후 다운로드
build.gradle에 의존성 확인
```groovy
plugins {
    id 'org.springframework.boot' version '3.4.5'
    id 'io.spring.dependency-management' version '1.1.7'
    id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '21'
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
}
```

application.yml에 H2 설정 추가
```yaml
spring:
  application:
    name: rest-api-app
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password:
  h2:
    console:
      enabled: true
server:
  port: 8080
```

### 2단계: 엔티티 클래스 생성

사용자 정보 저장 엔티티
```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;

    public User() {}
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
}
```

### 3단계: 리포지토리 인터페이스 생성

데이터베이스 연동 인터페이스
```java
public interface UserRepository extends JpaRepository<User, Long> {
}
```

### 4단계: 서비스 레이어 구현

비즈니스 로직 처리
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    public User getUserById(Long id) {
        return userRepository.findById(id).orElse(null);
    }

    public User createUser(User user) {
        return userRepository.save(user);
    }

    public User updateUser(Long id, User userDetails) {
        User user = userRepository.findById(id).orElse(null);
        if (user != null) {
            user.setName(userDetails.getName());
            user.setEmail(userDetails.getEmail());
            return userRepository.save(user);
        }
        return null;
    }

    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

### 5단계: 컨트롤러 레이어 구현

HTTP 엔드포인트 정의
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @Autowired
    private UserService userService;

    @GetMapping
    public List<User> getAllUsers() {
        return userService.getAllUsers();
    }

    @GetMapping("/{id}")
    public User getUserById(@PathVariable Long id) {
        return userService.getUserById(id);
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.createUser(user);
    }

    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User userDetails) {
        return userService.updateUser(id, userDetails);
    }

    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
    }
}
```

> 💡특이사항
>
> IntelliJ Ultimate로 JPA 엔티티 디버깅 편리
> H2 콘솔로 데이터베이스 상태 확인 가능

---

## ✏️ 실행과 테스트

1. Gradle로 빌드 및 실행:
```bash
./gradlew bootRun
```
2. Postman으로 API 테스트:
   - GET /api/users: 모든 사용자 조회
   - GET /api/users/{id}: 특정 사용자 조회
   - POST /api/users: 사용자 생성
   - PUT /api/users/{id}: 사용자 수정
   - DELETE /api/users/{id}: 사용자 삭제
3. http://localhost:8080/h2-console로 DB 확인

---

## ✏️ 실무 팁과 주의사항

### REST API 구현 팁

- 엔티티 우선: 데이터 구조 정의로 기반 마련
- 리포지토리 다음: 데이터 상호작용 정의
- 서비스 구현: 로직 처리로 기능 완성
- 컨트롤러 마지막: 엔드포인트로 API 마무리
- 확장: 페이징이나 인증 추가

### 주의사항

- 의존성 호환: Gradle 빌드 시 spring-boot-starter-data-jpa 버전 확인
- 데이터베이스 설정: application.yml에서 H2 URL 정확히 입력
- 로그 확인: IntelliJ 콘솔로 에러 디버깅
- 테스트: Postman으로 CRUD 기능 모두 확인