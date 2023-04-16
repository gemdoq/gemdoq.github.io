---
layout: single
title: "Spring Security에 대해"
date: 2022-12-13 10:17:29 +0900
categories: java
tags: [Spring Security]
typora-root-url: ../
---


## Spring Security에 대해
> - Spring Security란
> - 사용
> - 구조
> - 주요 모듈
> - SecurityConfig

<br>

## Spring Security란

### 정의

Spring 기반의 애플리케이션의 보안(인증과 권한, 인가 등)을 제공하고, 일반적인 공격에 대한 방어를 제공하는 프레임워크

'인증'과 '인가'에 대한 부분을 Filter 흐름에 따라 처리

### 인증과 인가

- 인증(Authentication): 해당 사용자가 본인이 맞는지를 확인
- 인가(Authorization): 인증된 사용자가 요청한 자원에 접근 권한이 있는지 확인

Principal을 아이디로, Credential을 비밀번호로 사용하는 Credential 기반의 인증 방식

- Principal(접근 주체): 보호받는 Resource에 접근하는 대상
- Credential(비밀번호): Resource에 접근하는 대상의 비밀번호

### 계속되는 최신화

Spring Security는 지속적으로 발전되고 새로운 기능이 추가되고 있기 때문에 현재 Spring Security 공식문서의 최신 6.0.2 버전을 기준으로 내용을 정리했다.

<br>

## 사용

### Dependency Update

build.gradle에 다음과 같이 의존성 추가 후 반영
```groovy
dependencies {
	compile "org.springframework.boot:spring-boot-starter-security"
}
```

### SpringBoot Auto Configuration

스프링부트에서 자동으로 실행하는 것들은 다음과 같다.

1. SpringSecurityFilterChain이라는 이름의 서블릿 필터를 bean으로 등록(애플리케이션 URL 보호, 유저명과 암호 검증, 로그인 양식 리디렉션 등을 담당)
2. 콘솔에 기록하기 위해, user명과 임의로 생성된 암호로 UserDetailsService라는 이름의 bean을 등록
3. 모든 요청에 대한 Filter로써 SpringSecurityFilterChain를 등록

### Spring Security function

스프링 시큐리티가 동작하는 기능은 다음과 같다.

- 애플리케이션과 상호 작용하기 위해 유저에게 인증을 요구
- 기본 로그인 양식을 생성
- 콘솔에 기록되어 있는 사용자 이름과 암호가 있는 사용자에게 양식 기반 인증을 허용
- 암호 저장을 BCrypt로 보호
- 사용자가 로그아웃하도록 허용
- CSRF 공격 방지
- 세션 고정 보호
- 보안 헤더 통합
- 서블릿 API 메소드들 통합

## 구조

![filterchain](/images/2022-12-13-about-spring-security/filterchain.png){: width="560"}

클라이언트는 애플리케이션에 요청을 보내고, 컨테이너는 FilterChain을 만든다.
만들어진 FilterChain은 요청된 URI 경로를 기반으로 만들어진 HttpServletRequest를 처리해야 하는 Filter 인스턴스와 Servlet을 포함한다.

Spring MVC 어플리케이션에서 Servlet은 DispatcherServlet의 인스턴스며, 기껏해야 하나의 HttpServletRequest와 HttpServletResponse를 다룰 수 있다.

하지만 서블릿 위에 한개 이상의 필터를 겹쳐서 쌓아두고 HttpServletRequest와 HttpServletResponse를 처리하게 하는 FilterChain 덕분에 수많은 것들을 할 수 있다.

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
	// do something before the rest of the application
    chain.doFilter(request, response); // invoke the rest of the application
    // do something after the rest of the application
}
```

### DelegatingFilterProxy

![delegatingfilterproxy](/images/2022-12-13-about-spring-security/delegatingfilterproxy.png){: width="560"}

Spring은 Filter를 구현한 DelegatingFilterProxy에게 Servlet Container에서의 Filter 인스턴스를 생성하여 Spring에서 정의한 bean을 등록
→ DelegatingFilterProxy 사용으로 bean 인스턴스 조회 지연 가능

### FilterChainProxy

![filterchainproxy](/images/2022-12-13-about-spring-security/filterchainproxy.png){: width="560"}

스프링 시큐리티에서 서블릿에 대한 지원을 하는 특수 필터로 SecurityFilterChain을 통해 많은 Filter인스턴스에 위임 가능

### SecurityFilterChain

![securityfilterchain](/images/2022-12-13-about-spring-security/securityfilterchain.png){: width="560"}

스프링 시큐리티에서 요청에 대하여 어떤 Filter인스턴스를 호출해야 할지 결정할 때 사용
SecurityFilterChain의 보안필터들은 DelegatingFilterProxy 대신 FilterChainProxy에 등록되는 것으로 많은 이점을 가짐

1. 스프링 시큐리티에 문제 발생시 디버그를 FilterChainProxy에 찍기 좋음
2. FilterChainProxy은 스프링 시큐리티의 중추이기 때문에 중요한 작업들이 수행됨
3. Servlet Container에서의 Filter 인스턴스 호출은 URL에 의해 강제적으로 발생되지만, FilterChainProxy은 RequestMatcher 인터페이스에 의해 유연한 호출 설정이 가능

![multi-securityfilterchain](/images/2022-12-13-about-spring-security/multi-securityfilterchain.png){: width="560"}

### 보안 예외 처리

ExceptionTranslationFilter는 FilterChainProxy에 보안필터 중 하나로써 삽입되어 AccessDeniedException와 AuthenticationException을 HTTP 응답으로 변환 가능

![exceptiontranslationfilter](/images/2022-12-13-about-spring-security/exceptiontranslationfilter.png){: width="560"}

<br>