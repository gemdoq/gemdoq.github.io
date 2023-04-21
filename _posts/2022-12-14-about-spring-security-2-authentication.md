---
layout: single
title: "Spring Security: (2)Authentication"
date: 2022-12-14 10:41:33 +0900
categories: java
tags: [Spring Security]
typora-root-url: ../
---


## Spring Security: (2)Authentication
> - 정의
> - 구조

<br>

## 정의

Authentication(인증)은 특정 리소스에 접근하려는 사용자의 신원을 확인하는 방법
일반적으로 사용자 이름과 암호를 입력하도록 요구하여 인증을 수행
인증이 수행되고 나면, 정체를 알고 Authorization(인가)를 수행 가능

## 구조

### 주요 구성요소

- SecurityContextHolder
- SecurityContext
- Authentication
- GrantedAuthority
- AuthenticationManager
- ProviderManager
- AuthenticationProvider
- AuthenticationEntryPoint
- AbstractAuthenticationProcessingFilter

### 구체적인 역할

#### SecurityContextHolder
![securitycontextholder](/images/2022-12-14-about-spring-security-2-authentication/securitycontextholder.png){: width="560"}

누가 인증되었는지에 대한 정보를 저장

스프링시큐리티는 어떻게 값이 채워지든 SecurityContextHolder에 값만 있으면 현재 인증된 사용자로 인식

사용자가 인증되었음을 나타내는 가장 간단한 방법은 SecurityContextHolder에 직접 값을 추가하는 방법

```java
SecurityContext context = SecurityContextHolder.createEmptyContext();
Authentication authentication =
    new TestingAuthenticationToken("username", "password", "ROLE_USER");
context.setAuthentication(authentication);

SecurityContextHolder.setContext(context);
```
1. SecurityContext instance 생성
2. 새 Authentication 객체 생성(일반적으로 UsernamePasswordAuthenticationToken 사용)
3. SecurityContextHolder에 SecurityContext 주입

인증된 주체에 대한 정보를 얻으려면 SecurityContextHolder에 접근

```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
String username = authentication.getName();
Object principal = authentication.getPrincipal();
Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
```
ThreadLocal을 사용하면, 필터체인프록시가 시큐리티컨텍스트를 항상 비워주기 때문에 주체의 요청이 처리되고 나서 쓰레드가 비워지는 것을 보장

일부 응용프로그램은 스레드를 사용하는 특수한 방식으로 인해 스레드로컬을 사용하는데 온전히 적합하지는 않음
예를 들어 Swing Client는 JVM의 모든 스레드를 동일한 시큐리티 컨텍스트를 사용하기 위해 요구
그래서 SecurityContextHolder에 컨텍스트가 어떻게 저장될지에 대한 전략 지정하여 설정 가능

#### SecurityContext

#### Authentication

#### GrantedAuthority

#### AuthenticationManager

#### ProviderManager

#### AuthenticationProvider

#### AuthenticationEntryPoint

#### AbstractAuthenticationProcessingFilter

