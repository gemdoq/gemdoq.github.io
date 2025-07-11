---
layout: single
title: "스프링 시큐리티 필터와 예외 처리"
date: 2025-08-03 17:30:00 +0900
categories: spring-security
tags: [Spring Security, Filters, Exception Handling]
typora-root-url: ../

---

# ✈️ 스프링 시큐리티 필터와 예외 처리

## 🎯 개요

Spring Security의 필터와 예외 처리로 보안 로직 강화
Spring Boot 프로젝트에서 인증과 인가를 커스텀해 사용자 경험 개선
이 포스트는 시큐리티 시리즈 2부로, 1부(아키텍처와 기본 개념)와 함께 보면 이해에 도움

## 📋 목록

- [ ]  필터와 예외 처리란?
- [ ]  주요 필터 개념
- [ ]  예외 처리 개념
- [ ]  인가 관련 개념
- [ ]  설정 예시
- [ ]  실무 팁과 주의사항

---

## ✏️ 필터와 예외 처리란?

### 필터 정의

Spring Security의 필터는 HTTP 요청마다 보안 로직 적용
DelegatingFilterProxy와 FilterChainProxy로 요청 처리와 필터 체인 관리

### 예외 처리 정의

인증과 인가 실패 시 적절한 응답 제공
ExceptionTranslationFilter로 에러 분리 및 사용자 친화적 페이지 표시

> 💡왜 필요하나?
>
> Spring Boot 프로젝트에서 인증 실패 시 로그인 페이지 이동, 인가 실패 시 에러 페이지 표시로 보안과 UX 개선
> Gradle 빌드로 배포 시 일관된 보안 설정 유지

---

## ✏️ 주요 필터 개념

### 1. DelegatingFilterProxy

서블릿 필터와 Spring Security 빈 연결
스프링 부트의 application.yml 설정과 유사한 통합 역할

### 2. FilterChainProxy

여러 SecurityFilterChain 관리
요청 URL별로 적절한 필터 체인 선택

### 3. SecurityFilterChain

URL 패턴에 적용되는 필터 목록
HttpSecurity로 보안 규칙 정의

### 4. AbstractAuthenticationProcessingFilter

인증 요청 처리 추상 클래스
UsernamePasswordAuthenticationFilter로 로그인 폼 데이터 처리

> 💡특이사항
>
> IntelliJ Ultimate로 디버깅 시 UsernamePasswordAuthenticationFilter 동작 확인
> FilterChainProxy 설정으로 URL별 보안 분리 가능

---

## ✏️ 예외 처리 개념

### 1. ExceptionTranslationFilter

인증과 인가 예외 처리
AuthenticationException은 로그인 페이지로, AccessDeniedException은 에러 페이지로 이동

### 2. AuthenticationEntryPoint

인증 실패 시 동작 정의
커스텀 로그인 페이지로 리다이렉트

### 3. AccessDeniedHandler

인가 실패 시 동작 정의
403 에러 페이지 표시

> 💡특이사항
>
> Spring Boot의 application.yml에서 에러 페이지 경로 설정 가능
> 커스텀 예외 처리로 사용자 친화적 응답 제공

---

## ✏️ 인가 관련 개념

### 1. AuthorizationManager

인가 결정 로직 구현
RequestMatcher로 URL 패턴 확인 후 접근 제어

### 2. Custom Authorization Managers

특정 조건 기반 인가 로직
IP 주소 제한 같은 커스텀 설정 추가

### 3. RoleHierarchyVoter

역할 계층 기반 인가
ROLE_ADMIN이 ROLE_USER 권한 포함

> 💡특이사항
>
> Gradle 빌드 시 spring-security-config 의존성 확인
> IntelliJ Ultimate로 역할 계층 디버깅 편리

---

## ✏️ 설정 예시

Spring Security 필터와 예외 처리 설정
Gradle 기반 Spring Boot 프로젝트에 적용

```groovy
// build.gradle
plugins {
    id 'org.springframework.boot' version '3.4.5'
    id 'io.spring.dependency-management' version '1.1.7'
    id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

```java
// SecurityConfig.java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .permitAll()
            )
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(new CustomAuthenticationEntryPoint())
                .accessDeniedHandler(new CustomAccessDeniedHandler())
            );
        return http.build();
    }

    @Component
    public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {
        @Override
        public void commence(HttpServletRequest request, HttpServletResponse response,
                             AuthenticationException authException) throws IOException {
            response.sendRedirect("/login?error=unauthenticated");
        }
    }

    @Component
    public class CustomAccessDeniedHandler implements AccessDeniedHandler {
        @Override
        public void handle(HttpServletRequest request, HttpServletResponse response,
                           AccessDeniedException accessDeniedException) throws IOException {
            response.sendRedirect("/access-denied");
        }
    }
}
```

> 💡특이사항
>
> JDK21 환경에서 Gradle 8.x 이상 추천
> application.yml로 로그인 페이지 경로 커스텀 가능

---

## ✏️ 실무 팁과 주의사항

### 필터와 예외 처리 팁

- 간단 시작: 기본 SecurityFilterChain부터 설정
- 디버깅: IntelliJ Ultimate로 필터 흐름 확인
- 공식 문서: Spring Security 6.x 설정 참고
- 커스텀 페이지: 로그인과 에러 페이지 디자인
- 에러 처리: 사용자 친화적 메시지 추가

### 주의사항

- 필터 순서 확인: 잘못된 순서로 보안 로직 실패 가능
- 의존성 호환: Gradle 빌드 시 spring-boot-starter-security 버전 점검
- 로그 확인: 인증 실패 시 콘솔 로그로 디버깅
- 테스트: 로컬 환경에서 /admin, /login 접속 확인

> 💡특이사항
>
> application.yml에 spring.security.enabled=false로 테스트 가능
> IntelliJ Ultimate 디버거로 ExceptionTranslationFilter 동작 점검
> 이 설정으로 Spring Boot 프로젝트의 보안 안정성 확보