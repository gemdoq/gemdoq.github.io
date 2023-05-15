---
layout: single
title: "Persistent, JDBC, JPA/Hibernate, Mybatis의 차이"
date: 2022-11-16 11:24:33 +0900
categories: java
tags: [영속성, Persistent, JDBC, JPA, Hibernate, Mybatis]
typora-root-url: ../
---


## Persistent, JDBC, JPA, Hibernate, Mybatis
> - Persistent
> - JDBC
> - JPA
> - Hibernate
> - Mybatis

<br>

![spring-jdbc-layer](/images/2022-11-16-differ-persistent-jdbc-jpa-mybatis/spring-jdbc-layer.png){: width="560"}

## 영속성(Persistent)

### 정의

데이터를 생성한 프로그램이 종료되더라도 사라지지 않는 데이터의 특성

영속성을 갖지 않는 데이터는 단지 메모리에서만 존재하기 때문에 

프로그램을 종료하면 모두 소멸

파일 시스템, 혹은 데이터베이스 활용으로 `데이터를 영구하게 저장하는 것`을 의미

### Persistence Framework

JDBC 프로그래밍의 복잡함이나 번거로움 없이 간단한 작업만으로 

데이터베이스와 연동되는 시스템을 빠르게 개발할 수 있으며 안정적

#### SQL Mapper

필드를 매핑시키는 것이 목적

(SQL을 직접 명시)

Mybatis, JdbcTempletes

#### ORM(Object Relational Mapping)

Persistant API

관계형 데이터베이스의 '관계'를 Object에 반영하자는 것이 목적

(데이터베이스 객체를 자바 객체로 매핑 → 객체 관계를 바탕으로 SQL을 자동 생성)

JPA, Hibernate


<br>

## JDBC

![spring-jdbc-architecture](/images/2022-11-16-differ-persistent-jdbc-jpa-mybatis/spring-jdbc-architecture.png){: width="560"}

### 정의

JDBC는 DB에 접근할 수 있도록 Java에서 제공하는 API

Java의 Data Access 기술의 근간

모든 Persistence Framework는 내부적으로 JDBC API를 이용

데이터베이스에서 자료를 쿼리하거나 업데이트하는 방법을 제공

<br>

## JPA(Java Persistent API)

![spring-jpa-architecture](/images/2022-11-16-differ-persistent-jdbc-jpa-mybatis/spring-jpa-architecture.png){: width="560"}

### 정의

Java에서 제공하는 ORM 표준 API

ORM을 사용하기 위한 표준 인터페이스를 모아둔 것

`Spring Data JPA`는 JPA 리포지토리 데이터 소스 액세스를 제공하여 애플리케이션 개발을 용이하게 지원

사용자가 원하는 JPA 구현체를 선택해서 사용

과거 EJB에서 제공되던 엔터티 빈(Entity Bean)을 대체하는 기술

### 구성 요소

1. 객체/관계 메타데이터
2. `javax.persistance` 패키지로 정의된 API 그 자체
3. JPQL(Java Persistence Query Language)

<br>

## Hibernate

![spring-hibernate-architecture](/images/2022-11-16-differ-persistent-jdbc-jpa-mybatis/spring-hibernate-architecture.png){: width="560"}

### 정의

JPA의 구현체 중 하나

HQL(Hibernate Query Language)이라 불리는 매우 강력한 쿼리 언어를 포함

- 객체지향적으로 데이터를 관리 → 비즈니스 로직 집중
- 테이블 생성, 변경, 관리가 쉬움
- 많은 내용이 감싸져 있기 때문에 이해하려면 알아야 할 것이 많음
- 이해하지 못하고 사용하면 데이터 손실 가능성이 있음

#### HQL(Hibernate Query Language)

- SQL과 매우 비슷하며 추가적인 컨벤션을 정의 가능
- 객체 지향적(상속, 다형성, 관계등의 객체지향의 강점) 
- HQL쿼리는 자바 클래스와 프로퍼티의 이름을 제외하고는 대소문자를 구분
- 쿼리 결과로 객체를 반환, 프로그래머에 의해 생성, 직접적으로 접근 가능
- SQL에서는 지원하지 않는 페이지네이션이나 동적 프로파일링과 같은 향상된 기능을 제공
- 여러 테이블을 작업할 때 명시적인 join을 요구하지 않음

<br>

## Mybatis

![spring-mybatis-architecture](/images/2022-11-16-differ-persistent-jdbc-jpa-mybatis/spring-mybatis-architecture.png){: width="560"}

### 정의

SQL, 저장 프로시저 그리고 몇 가지 고급 매핑을 지원하는 SQL Mapper

JDBC로 처리하는 상당 부분의 코드와 파라미터 설정 및 결과 매핑을 대신함

- 기존에 JDBC를 사용할 때는 DB와 관련된 여러 복잡한 설정(Connection)들을 다루어야 했지만 SQL Mapper는 자바 객체를 실제 SQL문에 연결함으로써, 빠른 개발과 편리한 테스트 환경을 제공
- SQL에 대한 세부적인 컨트롤을 하고자 할때 적합
- 데이터베이스 record에 원시 타입과 Map 인터페이스 그리고 자바 POJO를 설정해서 매핑하기 위해 xml과 Annotation을 사용
- MyBatis는 원래 Apache Foundation의 iBatis였으나, 생산성, 개발 프로세스, 커뮤니티 등의 이유로 Google Code로 이전되면서 MyBatis로 바뀜

<br>