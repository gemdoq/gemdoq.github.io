---
layout: single
title: "Spring Boot 3.0 버전 변경 사항 정리"
date: 2022-12-15 11:23:33 +0900
categories: java
tags: [Spring Boot]
typora-root-url: ../
---


## Spring Boot 3.0 버전 변경 사항 정리
> - 핵심 변경 사항
> - 인프라 측면에서 변경 사항

<br>
22년 11월 스프링 6.0이 공식적으로 릴리즈

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