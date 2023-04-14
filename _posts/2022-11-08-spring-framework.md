---
layout: single
title: "Spring Framework란"
date: 2022-11-08 10:16:24 +0900
categories: java
tags: [Spring, Framework, Library]
typora-root-url: ../
---


## Spring Framework
> - 라이브러리 vs 프레임워크
> - 스프링 프레임워크란

<br>

## 라이브러리 vs 프레임워크
### 라이브러리

라이브러리는 자주 사용되는 로직을 

재사용하기 편리하도록 잘 정리한 일련의 코드 집합

### 프레임워크

소프트웨어의 구체적인 부분에 해당하는 설계와 구현을 

협업화된 형태로 클래스들을 제공하는 것

### 프레임워크의 장점

- 생산성
  - 개발할 때,비지니스 로직에만 집중
- 코드 품질
  - 코드의 재사용 및 유지 보수 용이
  - 확장성을 가진 코드 설계 가능

<br>

## 스프링 혹은 스프링 프레임워크(Spring Framework)

### 정의(Def)

#### 스프링(Spring)

자바 엔터프라이즈 개발을 편하게 하게금 도와주는 경량 오픈소스

POJO 기반의 Enterprise급 Application의 수월하게 개발하게끔

필요한 하부구조(Infrastructure)를 포괄적으로 제공하고,

스프링이 복잡하고 까다로운 하부구조를 대신 처리하기 때문에

개발자는 비지니스 로직 개발에만 온전히 집중 가능

#### 스프링 프레임워크(Spring Framework)

스프링을 프레임워크 관점에서 바라본 용어(=스프링)

#### POJO(Plain Old Java Object)

> 상속, 인터페이스가 필요없고, 
> 원하는 business logic만 넣은, 
> 아주 단순하고 가벼운 객체

### 주요 특징

1. DI(Dependency Injection)
  - 의존 관계 주입
  - 각각의 계층이나 서비스들 간에 의존성이 존재할 경우 Spring이 서로 연결
  - POJO 객체들 사이의 의존 관계를 Spring이 알아서 연관성을 찾아 결합
2. AOP(Aspect Orientated Programming)
  - 관점 중심 프로그래밍
  - Spring은 핵심적인 비즈니스 로직과 관련이 없으나 여러 곳에서 공통적으로 쓰이는 기능들을 분리(공통 관심사를 분리)하여 개발하고 실행 시에 서로 조합할 수 있는 AOP를 지원
  - 이를 통해 코드를 단순하고 깔끔하게 작성 가능
  - 횡단 관심을 수행하는 코드(Logging, Security, Transaction 등)는 aspect라는 특별한 객체로 모듈화하여 핵심 기능에 끼워넣을 수 있음(weaving)
3. Portable Service Abstraction
  - 이식 가능한 서비스 추상화
  - Spring은 완성도가 높은 라이브러리와 연결할 수 있는 인터페이스를 제공
  - 다른 프레임워크들과의 통합을 지원

### 구성 요소
- Core Container 중 Bean Container는 POJO 객체를 관리
- Spring에서 제공하는 다양한 기능 중 필요한 것을 선택적으로 사용

<br>