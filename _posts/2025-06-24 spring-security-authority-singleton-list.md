---
layout: single
title: "스프링 시큐리티 권한과 싱글톤 리스트 활용"
date: 2025-06-24 14:12:23 +0900
categories: Spring Boot
tags: [GrantedAuthority, Authentication]
typora-root-url: ../

---
#### 📌 용어 한눈에
- GrantedAuthority: 스프링 시큐리티에서 권한/역할 정의 인터페이스  
- Authentication: 사용자 인증 정보, 권한 리스트 포함  
- Collections.singletonList: 단일 객체 불변 리스트 반환  
- ROLE: 권한 접두사 (예: ROLE_USER, ROLE_ADMIN)  
- 불변 리스트: 수정 불가 리스트, 안정성 보장  

---
#### 📌 스프링 시큐리티 권한이란
사용자가 할 수 있는 작업 결정  
List<GrantedAuthority>로 권한 관리  
ROLE_USER, ROLE_ADMIN 같은 역할 부여  

##### 실생활 비유
카페 직원의 역할 배지  
바리스타는 커피 만들기, 매니저는 재고 관리  
배지가 권한 나타냄  

---
#### 📌 Collections.singletonList란
자바 유틸리티 메서드  
단일 객체를 불변 리스트로 반환  
메모리 효율, 수정 방지  

##### 실생활 비유
직원에게 역할 하나만 부여  
큰 상자(ArrayList) 대신 배지 하나(singletonList) 전달  
공간 절약, 변경 불가  

---
#### 📌 singletonList 쓰는 이유
단일 권한 부여 시 간결  
ArrayList보다 메모리 적게 사용  
불변성으로 권한 안정성 확보  

##### 그림으로 보기
ArrayList로 권한  
```
[메모리]
[ArrayList]
   |
   v
[ROLE_USER] (150KB, 생성/추가 비용)
```

singletonList로 권한  
```
[메모리]
[singletonList]
   |
   v
[ROLE_USER] (100KB, 간단 생성)
```

> 💡 팁  
> IntelliJ로 메모리 사용량 디버깅 가능  

---
#### 📌 코드로 살펴보기

##### ✅ singletonList로 권한
```java
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import java.util.Collections;

public class UserConfig {
    public UserDetails createUser() {
        return new User(
                "user",
                "password",
                Collections.singletonList(new SimpleGrantedAuthority("ROLE_USER"))
        );
    }
}
```

- 단일 권한 ROLE_USER  
- 메모리 절약, 간결  

##### ✅ ArrayList로 권한
```java
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import java.util.ArrayList;
import java.util.List;

public class UserConfig {
    public UserDetails createUser() {
        List<SimpleGrantedAuthority> authorities = new ArrayList<>();
        authorities.add(new SimpleGrantedAuthority("ROLE_USER"));
        return new User("user", "password", authorities);
    }
}
```

- 동일 결과, 생성 비용 큼  

##### ✅ 스프링 시큐리티 설정
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
                        .requestMatchers("/api/admin/**").hasRole("ADMIN")
                        .anyRequest().authenticated()
                )
                .formLogin();
        return http.build();
    }
}
```

> 💡 팁  
> application.yml에 `spring.security`로 JWT, OAuth 설정 가능  
> IntelliJ로 `@PreAuthorize` 자동완성 활용  

---
#### 📌 singletonList 사용 시기
- 단일 권한 부여에 최적  
- 다중 권한은 `List.of` 또는 ArrayList  
- 불변성 필요 시 선호  

##### 실생활 비유
역할 하나(바리스타)면 배지 하나(singletonList)  
여러 역할(바리스타, 캐셔)면 상자(List.of)  

---
#### 📌 주의점
- singletonList는 불변, 추가/삭제 불가  
- 동적 권한 변경은 ArrayList 선택  
- 단일 권한 확정 시 singletonList 사용  
- IntelliJ로 권한 리스트 디버깅 추천  

---
#### ✍ 느끼며
스프링 시큐리티 권한 처음엔 복잡했음  
singletonList로 간결함과 효율성 깨달음  
카페 배지 비유로 메모리 절약 이해  
불변성으로 안정성 보장 실감  
대규모 앱에서 리소스 절약의 중요성 체감  
앞으로 단일 권한은 singletonList 적극 활용