---
layout: single
title: "Persistent Context에 대하여"
date: 2022-11-28 10:33:16 +0900
categories: knowledge
tags: [Persistent Context, 1차 캐시, 동일성 보장, 쓰기 지연, JDBC Batch option]
typora-root-url: ../
---


## Persistent Context에 대하여
> - Persistent Context란
> - 엔티티의 생명주기(Entity LifeCycle)
> - 사용 이유

<br>

## Persistent Context란

### 정의

영속성 컨텍스트(Persistence Context), JPA를 이해하는데 가장 중요한 용어

논리적인 개념, 눈에 보이지 않음

Entity를 영구 저장하는 환경

`EntityManager.persist(entity);`

실제로는 DB에 저장하는 것이 아니라, 영속성 컨텍스트를 통해서 Entity를 영속화

`persist()` 시점에는 Entity를 영속성 컨텍스트에 저장

EntityManager를 통해서 영속성 컨텍스트에 접근

EntityManager가 생성되면 1:1로 영속성 컨텍스트가 생성

<br>

## 엔티티의 생명주기(Entity LifeCycle)

Persistent Context에 따라 Entity가 다음의 단계를 거치는 것

### 비영속(new/transient)

객체를 생성만 하고, 영속성 컨텍스트와 전혀 관계가 없는 상태

```java
// 객체를 생성한 상태 (비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
```

### 영속(managed)

영속성 컨텍스트에 저장된 상태

Entity가 영속성 컨텍스트에 의해 관리되는 상태

```java
// 객체를 생성한 상태 (비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
EntityManager entityManager = entityManagerFactory.createEntityManager();
entityManager.getTransaction().begin();
// 객체를 저장한 상태 (영속)
entityManager.persist(member);
```
`EntityManager.persist(entity);`

영속 상태가 된다고 바로 DB에 쿼리가 날라가지 않음

`transaction.commit();`

트랜잭션의 commit 시점에 영속성 컨텍스트에 있는 정보들이 DB에 쿼리로 전송

### 준영속(detached)

영속성 컨텍스트에 저장되었다가 분리된 상태

영속성 컨텍스트에서 지운 상태 

```java
// 회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태
entityManager.detach(member);
```

### 삭제(removed)

실제 DB 삭제를 요청한 상태 
```java
// 객체를 삭제한 상태
entityManager.remove(member);
```

<br>

## 사용 이유

### 1차 캐시

Map<key, value> 형식의 영속성 컨텍스트

key는 @Id로 선언한 필드값

value는 해당 Entity

```java
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
/* 영속 상태 (Persistence Context 에 의해 Entity 가 관리되는 상태) */
// DB 저장 X, 1차 캐시에 저장됨
entityManager.persist(member); 
// 1차 캐시에서 조회
Member findMember = entityManager.find(Member.class, "Member1"); 
```

#### 예시

`entityManager.find()` 실행

- EntityManager는 Transaction단위로 생성
- 사용자의 요청에 대한 비지니스가 종료되어 해당 DB Transaction이 끝날 때 같이 삭제

1. DB보다 먼저, 1차 캐시를 조회
2. 해당 Entity가 존재하면 바로 반환
3. 1차 캐시에 조회하고자 하는 Entity가 없으면 DB에서 조회
4. 해당 Entity를 DB에서 꺼내와 1차 캐시에 저장 후 Entity 반환
5. 이후에 다시 해당 Entity를 조회하면 1차 캐시에 있는 Entity를 반환

### 동일성 보장(Identity) 

```java
Member a = entityManager.find(Member.class, "member1");
Member b = entityManager.find(Member.class, "member1");
System.out.println(a == b); // 동일성 비교 true
```
하나의 Transaction 안에서 같은 영속 Entity의 == 비교 시 true임을 보장

트랜잭션 격리 수준 중 `반복 가능한 읽기(REPEATABLE READ)` 수준을 1차 캐시로 DB가 아닌 아닌 애플리케이션 차원에서 제공

### 엔티티 등록 시 트랜잭션을 지원하는 쓰기 지연(Transactional Write-Behind)

```java
EntityManager entityManager = emf.createEntityManager();
EntityTransaction transaction = entityManager.getTransaction();
// EntityManager는 데이터 변경 시 트랜잭션을 시작해야 한다.
transaction.begin(); // Transaction 시작

entityManager.persist(memberA);
entityManager.persist(memberB);
// 이때까지 INSERT SQL을 DB에 보내지 않는다.

// 커밋하는 순간 DB에 INSERT SQL을 보낸다.
transaction.commit(); // Transaction 커밋 
```
`entityManager.persist()` : JPA가 insert SQL 쌓기를 시작

`transaction.commit()` : 커밋하는 시점에 insert SQL을 동시에 DB에 전송

- `flush()` : SQL 저장소에 쌓여있는 Query들을 DB에 반영(DB Sync) - 1차 캐시를 지우지는 않음
- `commit()` : 실제 DB Transaction 커밋

#### JDBC 일괄 처리 옵션(JDBC batch option)

Spring Boot 사용시, 어플리케이션 설정을 아래와 같이 하면, 

DB에 쿼리를 날리는 것에 대해 insert query를 해당 값만큼 모아서 처리 가능

`spring.jpa.properties.hibernate.jdbc.batch_size=10`

### 엔티티 "수정" 시 변경 감지(Dirty Checking) 

```java
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
transaction.begin(); // Transaction 시작

// 영속 엔티티 조회
Member memberA = em.find(Member.class, "memberA");

// 영속 엔티티 데이터 수정
memberA.setUsername("hi");
memberA.setAge(10);

transaction.commit(); // Transaction 커밋
```
Entity 데이터만 수정하고 commit 하면 알아서 DB에 반영

데이터를 set하면 해당 데이터의 변경을 감지하여 자동으로 UPDATE Query 전송

### 엔티티 삭제

```java
Member memberA = em.find(Member.class, "memberA");

em.remove(memberA); // 엔티티 삭제
```
위의 Entity 수정 매커니즘과 동일

Transaction의 commit 시점에 DELETE Query 전송

<br>