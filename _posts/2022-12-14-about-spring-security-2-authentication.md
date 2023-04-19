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

#### SecurityContext

#### Authentication

#### GrantedAuthority

#### AuthenticationManager

#### ProviderManager

#### AuthenticationProvider

#### AuthenticationEntryPoint

#### AbstractAuthenticationProcessingFilter
