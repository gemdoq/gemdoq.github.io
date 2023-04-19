---
layout: single
title: "Spring Boot 3.0 버전 변경 사항 정리"
date: 2022-12-15 11:23:33 +0900
categories: java
tags: [Spring Boot]
typora-root-url: ../
---


## Spring Boot 3.0 버전 변경 사항 정리
> - 요구사항
> - 업그레이드 마이그레이션
> - 핵심 변경 사항
> - 인프라 측면에서 변경 사항

<br>
22년 11월 스프링 6.0이 공식적으로 릴리즈

## 요구사항

- Java Version : Java 17+
- Gradle 7.5+, Maven 3.5+
- Hibernate 6.1
- Spring Framework 6
- Jakarta EE 9+

Kotlin 사용시 1.6+ 버전 사용 필요

<br>

## 업그레이드 마이그레이션

기본 스프링 부트 2.7 버전이 스프링에서 공식적으로 이야기하는 최신버전이므로 그 이전의 스프링 부트 버전은 우선 2.7로 업그레이드를 하여 사이드 이펙트를 줄이며 순차적으로 업그레이드 할 것을 권장

### 스프링 부트 버전 의존성을 체크
- 스프링 부트 2.7 Dependency Version : [링크](https://docs.spring.io/spring-boot/docs/2.7.x/reference/html/dependency-versions.html#appendix.dependency-versions)
- 스프링 부트 3.0 Dependency Version : [링크](https://docs.spring.io/spring-boot/docs/3.0.x/reference/html/dependency-versions.html#appendix.dependency-versions)
### 스프링 시큐리티 마이그레이션
- 스프링 부트 3.0은 스프링 시큐리티 6.0을 사용
- 이전 버전을 사용하고 있다면 5.8 버전으로 먼저 마이그레이션하고 6.0으로 업그레이드 할 것을 권장
- 관련 마이그레이션 가이드 : [링크](https://docs.spring.io/spring-security/reference/5.8/migration/index.html)
- 5.8 → 6.0 마이그레이션 가이드 : [링크](https://docs.spring.io/spring-security/reference/6.0/migration/index.html)

## 핵심 변경 사항

### Java 17 버전을 Baseline으로 결정
### Java EE → 최근 릴리즈 된 Jakarta EE 10 버전에 중점 변경(Jakarta EE 9+) (Jakarta EE 10 APIs such as Servlet 6.0 and JPA 3.1)
### Tomcat 10.1 적용
### Hibernate ORM 6.1 적용

<br>

## 인프라 측면에서 변경 사항

### AOT(Ahead-Of-Time) 도입
### GraalVM 네이티브 이미지(Native Image) 지원
### Project Loom(Java 19)이라는 가상스레드(Virtual Thread) 지원, Project CRaC 활용 가능

<br>