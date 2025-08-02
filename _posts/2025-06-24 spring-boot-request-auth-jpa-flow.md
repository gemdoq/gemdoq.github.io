---
layout: single
title: "스프링부트 요청 인증 JPA 흐름 파악하기"
date: 2025-06-24 14:12:23 +0900
categories: knowledge
tags: [SecurityConfig, UserDetailsService]
typora-root-url: ../

---
#### 📌 용어 한눈에
- HTTP 요청: 클라이언트가 서버에 보내는 메시지 (GET, POST 등)  
- 인증: 사용자 신원 확인 (로그인 여부)  
- 인가: 사용자 권한 체크 (작업 가능 여부)  
- 컨트롤러: 요청 매핑, 서비스 호출  
- 서비스: 비즈니스 로직 처리  
- JPA 리포지토리: DB 작업, 데이터 관리  

---
#### 📌 스프링부트 요청 흐름이란
클라이언트 요청이 서버로 들어와 처리되는 과정  
인증, 인가, 매핑, 로직, DB, 응답까지  

##### 실생활 비유
카페에서 커피 주문  
문지기(시큐리티)가 신분 확인, 카운터(컨트롤러)가 주문 접수  
바리스타(서비스)가 커피 만들고, 창고(리포지토리)에서 재료 관리  
손님에게 커피 전달  

---
#### 📌 단계별 흐름

##### ✅ 1. 클라이언트 요청
프론트엔드에서 HTTP 요청 전송  
예: `/api/user/profile`로 프로필 조회  

```javascript
// JavaScript (프론트엔드)
fetch('http://localhost:8080/api/user/profile', {
    method: 'GET',
    headers: {
        'Authorization': 'Bearer <JWT_TOKEN>'
    }
})
.then(response => response.json())
.then(data => console.log(data));
```

> 💡 팁  
> Postman으로 요청 테스트 추천  
> IntelliJ HTTP 클라이언트로 요청 작성 가능  

##### ✅ 2. 인증/인가
스프링 시큐리티가 사용자 확인  
로그인 여부, ROLE_USER 권한 체크  

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import java.util.Collections;

@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public UserDetailsService userDetailsService() {
        User.UserBuilder builder = User.withUsername("user")
                .password("{noop}password")
                .authorities(Collections.singletonList(new SimpleGrantedAuthority("ROLE_USER")));
        return new InMemoryUserDetailsManager(builder.build());
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/api/user/**").hasRole("USER")
                        .anyRequest().authenticated()
                )
                .formLogin();
        return http.build();
    }
}
```

> 💡 팁  
> application.yml에 `spring.security`로 JWT 설정 추가  
> IntelliJ로 Authentication 디버깅  

##### ✅ 3. 요청 매핑
컨트롤러가 URL 매핑  
서비스 호출  

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.security.core.context.SecurityContextHolder;

@RestController
@RequestMapping("/api/user")
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/profile")
    public UserProfileResponse getProfile() {
        String username = SecurityContextHolder.getContext().getAuthentication().getName();
        return userService.getUserProfile(username);
    }
}
```

> 💡 팁  
> IntelliJ로 `@RequestMapping` 자동완성  
> application.yml에 `server.servlet.context-path`로 경로 설정  

##### ✅ 4. 서비스 로직
비즈니스 로직 처리  
리포지토리 호출  

```java
import org.springframework.stereotype.Service;

@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public UserProfileResponse getUserProfile(String username) {
        UserEntity user = userRepository.findByUsername(username)
                .orElseThrow(() -> new RuntimeException("User not found"));
        return new UserProfileResponse(user.getUsername(), user.getEmail());
    }
}

public record UserProfileResponse(String username, String email) {}
```

> 💡 팁  
> IntelliJ로 서비스 메서드 테스트 자동 생성  
> `@Transactional`로 트랜잭션 관리  

##### ✅ 5. JPA 리포지토리
DB 데이터 조회/저장  
엔티티 관리  

```java
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;
import lombok.Getter;
import lombok.Setter;

@Entity
@Table(name = "users")
@Getter
@Setter
public class UserEntity {
    @Id
    private String username;
    private String email;
}

import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserRepository extends JpaRepository<UserEntity, String> {
    Optional<UserEntity> findByUsername(String username);
}
```

> 💡 팁  
> application.yml에 `spring.jpa`로 Hibernate 설정  
> IntelliJ JPA 콘솔로 쿼리 확인  

##### ✅ 6. 응답 반환
컨트롤러가 JSON 응답 전송  
클라이언트에 결과 제공  

```json
{
    "username": "user",
    "email": "user@example.com"
}
```

---
#### 📌 흐름 그림
```
[클라이언트] ---- GET /api/user/profile ----> [서버]
                          |
                          v
                [시큐리티] 인증/인가
                          |
                          v
                [컨트롤러] 매핑
                          |
                          v
                [서비스] 로직
                          |
                          v
                [리포지토리] DB
                          |
                          v
                [응답] JSON --> [클라이언트]
```

---
#### 📌 주니어 팁
- 시큐리티는 기본 설정 먼저 익히기  
- 컨트롤러는 URL 매핑과 서비스 호출 집중  
- 서비스는 로직, 리포지토리는 DB 분리  
- JPA는 JpaRepository로 쿼리 최소화  
- `@Slf4j`로 로깅 추가, 디버깅 편리  
- application.yml로 설정 통합 관리  

---
#### ✍ 결론
스프링부트 흐름 처음엔 어지러웠음  
시큐리티의 인증/인가 중요성 깨달음  
컨트롤러, 서비스, 리포지토리 분리로 설계 감 잡힘  
JPA로 DB 작업 간소화 놀라움  
전체 그림 보이기 시작  