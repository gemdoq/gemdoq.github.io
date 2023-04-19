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
- 스프링 부트 2.7 Dependency Version : [🔗 링크](https://docs.spring.io/spring-boot/docs/2.7.x/reference/html/dependency-versions.html#appendix.dependency-versions)
- 스프링 부트 3.0 Dependency Version : [🔗 링크](https://docs.spring.io/spring-boot/docs/3.0.x/reference/html/dependency-versions.html#appendix.dependency-versions)
### 스프링 시큐리티 마이그레이션
- 스프링 부트 3.0은 스프링 시큐리티 6.0을 사용
- 스프링 시큐리티의 이전 버전을 사용하고 있다면 시큐리티 5.8 버전으로 먼저 마이그레이션 후 시큐리티 6.0으로 업그레이드 할 것을 권장
- 관련 마이그레이션 가이드 : [🔗 링크](https://docs.spring.io/spring-security/reference/5.8/migration/index.html)
- 5.8 → 6.0 마이그레이션 가이드 : [🔗 링크](https://docs.spring.io/spring-security/reference/6.0/migration/index.html)

## 핵심 변경 사항

### Java 17 Baseline

### Java EE → Jakarta EE 10(Jakarta EE 9+)

패키지 명 변경 : Javax.* → Jakarta.*
스프링 부트 3.0에서 Jakarta 9+를 채택하면서 Javax 패키지를 사용하는 import 구문 변경 필요

### Tomcat 10.1 적용

### Hibernate ORM 6.1 적용

<br>

## 인프라 측면에서 변경 사항

### AOT(Ahead-Of-Time) 도입

JIT Compiler와 AOT Compiler 모두 기계어를 만든다는 역할은 동일

AOT Compiler는 실행하기 전에 코드에 대한 정적 코드 테스트를 진행하고 그것을 기반으로 기계어를 생성
JIT Compiler는 런타임에 기계어를 생성

#### AOT Compiler

- 실행 전에 무겁고 복잡한 분석 및 최적화가 수행
- 런타임에 실행하는 속도가 빠름

#### JIT Compiler

- 상황에 맞춘 최적화 코드를 생성 가능(할당받은 코어, OS, 커널 버전, CPU 등)
- 런타임에 오버헤드가 발생
- C1, C2 컴파일러로 구성

### GraalVM 네이티브 이미지(Native Image) 지원

#### GraalVM이란?

GraalVM은 Java로 구현된 HotSpot/OpenJDK 기반의 Java VM 및 JDK

#### 기존 JDK와 차이점

- GraalVM Compiler <> JIT Compiler
- GraalVM Native Image, AOT Compilation을 기반으로 한 JIT Compiler를 사용
- Truffle 언어 구현 프레임워크 및 GraalVM SDK 제공
  - 다른 프로그래밍 언어(JavaScript, Ruby, R 등)를 런타임으로 동작할 수 있게 지원
- Native 애플리케이션으로 확장

GraalVM은 OpenJDK 8 + Graal + other things으로 구성

#### Native Image

자바 코드를 독립형 네이티브 실행 파일 또는 네이티브 공유 라이브러리로 컴파일하는 기술
생성된 네이티브 실행 파일은 JVM이 설치되어 있지 않아도 OS와 머신 아키텍쳐에서 사용 가능
생성된 네이티브 이미지 안에는 SubstrateVM(SVM)이 포함

##### SVM

- GraalVM의 하위 프로젝트
- JVM의 기능을 모두 가지고 있으나 GC나 Thread Control 성능이 좋지 않음
- 메인 메소드가 실행되기 이전의 실행 준비 단계를 모두 처리하여 자체 이미지에 포함
- 적은 메모리로 시작 가능

### Project Loom(Java 19) 지원

- 기존의 Thread(실)는 자원 사용면에서 비효율적
- 작성하고 디버깅하기 어려운 비동기 개발 방식
- Fiber(섬유)라는 Virtual Thread를 통해 컨티뉴에이션 + 스케쥴러를 통한 블로킹

### Project CRaC 활용 가능

실행 중인 JVM 이미지를 저장
자바가 시작될 때(Warm-up) 필요한 기능의 수행에 필요한 준비 시간 단축

<br>