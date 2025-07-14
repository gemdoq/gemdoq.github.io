---
layout: single
title: "스프링 시큐리티로 로그인 기능 구현"
date: 2025-08-05 17:30:00 +0900
categories: spring-security
tags: [Spring Security, Login Implementation, Authentication]
typora-root-url: ../

---

# ✈️ 스프링 시큐리티로 로그인 기능 구현

## 🎯 개요

Spring Security로 Spring Boot 프로젝트에 로그인 기능 구현
JDK21과 Gradle 환경에서 DB 연동과 커스텀 로그인 페이지로 보안 강화
이 포스트는 시큐리티 시리즈 3부로, 1부(아키텍처와 기본 개념)와 2부(필터와 예외 처리)와 함께 보면 이해에 도움

## 📋 목록

- [ ]  로그인 기능이란?
- [ ]  준비 항목
- [ ]  로그인 구현 단계
- [ ]  실행과 테스트
- [ ]  실무 팁과 주의사항

---

## ✏️ 로그인 기능이란?

### 로그인 기능 정의

Spring Security로 사용자 인증 처리
UserDetailsService와 PasswordEncoder로 DB에서 사용자 정보 조회 및 비밀번호 암호화

### 주요 구성 요소

- UserDetailsService: DB에서 사용자 정보 로드
- PasswordEncoder: 비밀번호 암호화로 보안 강화
- SecurityContextHolder: 인증 정보 저장
- HttpSecurity: 로그인과 URL 접근 제어 설정

> 💡왜 필요하나?
>
> Spring Boot 프로젝트에서 사용자 인증으로 보안 강화
> Gradle 빌드와 application.yml로 간편한 설정 가능

---

## ✏️ 준비 항목

- Spring Boot 프로젝트: Spring Initializr로 생성
- 의존성: spring-boot-starter-security, spring-boot-starter-web, spring-boot-starter-data-jpa, h2-database, spring-boot-starter-thymeleaf
- H2 데이터베이스: 메모리 DB로 테스트
- IntelliJ Ultimate: 개발 환경 구성
- application.yml: 설정 파일 준비

> 💡특이사항
>
> Gradle 8.x 이상으로 JDK21 호환성 확보
> Thymeleaf로 커스텀 로그인 페이지 구현

---

## ✏️ 로그인 구현 단계

### 1단계: 프로젝트 설정

build.gradle에 의존성 추가
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
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
}
```

application.yml에 H2 설정 추가
```yaml
spring:
  application:
    name: security-app
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

### 2단계: 사용자 엔티티와 리포지토리

사용자 정보 저장 엔티티와 JPA 리포지토리
```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
    private String role;

    public User() {}
    public User(String username, String password, String role) {
        this.username = username;
        this.password = password;
        this.role = role;
    }
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}
```

### 3단계: UserDetailsService 구현

DB에서 사용자 정보 조회
```java
@Service
public class CustomUserDetailsService implements UserDetailsService {
    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
        return new org.springframework.security.core.userdetails.User(
            user.getUsername(),
            user.getPassword(),
            Collections.singletonList(new SimpleGrantedAuthority(user.getRole()))
        );
    }
}
```

### 4단계: 스프링 시큐리티 설정

로그인과 URL 접근 제어
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Autowired
    private CustomUserDetailsService userDetailsService;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/login", "/css/**").permitAll()
                .requestMatchers("/user/**").hasRole("USER")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .defaultSuccessUrl("/user/dashboard")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login?logout")
                .permitAll()
            );
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

### 5단계: 로그인 페이지 HTML

Thymeleaf로 커스텀 로그인 페이지
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Login</title>
    <link rel="stylesheet" href="/css/style.css">
</head>
<body>
    <h2>Login</h2>
    <form method="post" action="/login">
        <label>Username: <input type="text" name="username"></label><br>
        <label>Password: <input type="password" name="password"></label><br>
        <button type="submit">Login</button>
    </form>
    <p th:if="${param.error != null}" th:text="'Invalid username or password'"></p>
    <p th:if="${param.logout != null}" th:text="'Logged out successfully'"></p>
</body>
</html>
```

### 6단계: 테스트 데이터 추가

애플리케이션 시작 시 사용자 데이터 삽입
```java
@Component
public class DataInitializer implements CommandLineRunner {
    @Autowired
    private UserRepository userRepository;
    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public void run(String... args) throws Exception {
        userRepository.save(new User("user", passwordEncoder.encode("password"), "ROLE_USER"));
        userRepository.save(new User("admin", passwordEncoder.encode("admin"), "ROLE_ADMIN"));
    }
}
```

### 7단계: 컨트롤러 추가

사용자 대시보드 페이지
```java
@Controller
public class HomeController {
    @GetMapping("/login")
    public String login() {
        return "login";
    }

    @GetMapping("/user/dashboard")
    public String userDashboard() {
        return "dashboard";
    }
}
```

> 💡특이사항
>
> IntelliJ Ultimate로 Thymeleaf 템플릿 디버깅 편리
> application.yml에서 H2 콘솔 활성화로 DB 확인

---

## ✏️ 실행과 테스트

1. Gradle로 빌드 및 실행:
```bash
./gradlew bootRun
```
2. http://localhost:8080/login 접속
3. username: user, password: password로 로그인
4. 로그인 성공 시 /user/dashboard 이동
5. /logout으로 로그아웃 테스트
6. http://localhost:8080/h2-console로 DB 확인

---

## ✏️ 실무 팁과 주의사항

### 로그인 구현 팁

- 간단 시작: H2 메모리 DB로 테스트
- 디버깅: IntelliJ Ultimate로 UsernamePasswordAuthenticationFilter 동작 확인
- 공식 문서: Spring Security 6.x 설정 가이드 참고
- 커스텀 페이지: 로그인 폼 디자인 조정
- 에러 처리: 사용자 친화적 메시지 추가

### 주의사항

- 의존성 호환: Gradle 빌드 시 spring-boot-starter-security 버전 확인
- 비밀번호 암호화: BCryptPasswordEncoder 필수
- 로그 확인: IntelliJ 콘솔로 인증 에러 디버깅
- 테스트: 로컬 환경에서 로그인과 로그아웃 동작 점검

> 💡특이사항
>
> JDK21 환경에서 Gradle 8.x 이상 추천
> application.yml로 DB와 포트 설정 중앙화
> 이 설정으로 Spring Boot 프로젝트의 로그인 보안 강화