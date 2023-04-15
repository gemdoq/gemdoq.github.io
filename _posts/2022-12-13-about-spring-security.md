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
> - Spring Security 사용
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

## Spring Security 사용

### Dependency Update

build.gradle에 다음과 같이 의존성 추가 후 반영
```groovy
dependencies {
	compile "org.springframework.boot:spring-boot-starter-security"
}
```
스프링부트에서 자동으로 실행하는 것
1. SpringSecurityFilterChain이라는 이름의 서블릿 필터를 bean으로 등록(애플리케이션 URL 보호, 유저명과 암호 검증, 로그인 양식 리디렉션 등을 담당)
2. 콘솔에 기록하기 위해, user명과 임의로 생성된 암호로 UserDetailsService라는 이름의 bean을 등록
3. 모든 요청에 대한 Filter로써 SpringSecurityFilterChain를 등록











<br>