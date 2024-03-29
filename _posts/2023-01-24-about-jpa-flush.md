---
layout: single
title: "JPA에서 Flush에 대하여"
date: 2023-01-24 10:19:37 +0900
categories: java
tags: [JPA, Flush]
typora-root-url: ../
---

## JPA에서 Flush에 대하여
> - 정의
> - 동작 과정
> - 사용 방법
> - 옵션 설정


<br>

## 정의

영속성 컨텍스트의 변경 내용을 DB 에 반영하는 것을 의미

Transaction commit 이 일어날 때 flush가 동작하는데, 이때 쓰기 지연 저장소에 쌓아 놨던 INSERT, UPDATE, DELETE SQL들이 DB에 전송(영속성 컨텍스트를 비우는 게 아님!)

영속성 컨텍스트의 변경 사항들과 DB의 상태를 동기화하는 작업

<br>

## 동작 과정

1. 변경을 감지한다. (Dirty Checking)
2. 수정된 Entity를 쓰기 지연 SQL 저장소에 등록
3. 쓰기 지연 SQL 저장소의 Query를 DB에 전송(등록, 수정, 삭제)
   - flush가 발생한다고 해서 commit이 이루어지는 것이 아니고 flush 다음에 실제 commit
   - 플러시가 동작할 수 있는 이유는 데이터베이스 트랜잭션 때문
   - 트랜잭션이 시작되고 해당 트랜잭션이 commit 되는 시점에 동기화

<br>

## 사용 방법

### em.flush()을 통한 직접 호출

```java
// 영속 상태 (Persistence Context 에 의해 Entity 가 관리되는 상태)
Member member = new Member(200L, "A");
entityManager.persist(member);

entityManager.flush(); // 강제 호출 (쿼리가 DB 에 반영됨)

System.out.println("DB INSERT Query 가 즉시 나감. -- flush() 호출 후 --  Transaction commit 됨.");
tx.commit(); // DB에 insert query 가 날라가는 시점 (Transaction commit)
```

플러시는 쓰기 지연 SQL 저장소에 있는 Query들만 DB에 전송되는 과정이므로 1차 캐시는 그대로 보존

### JPQL 쿼리 실행시 플러시 자동 호출

```java
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);

// 중간에 JPQL 실행
query = entityManager.createQuery("select m from Member m", Member.class);
List<Member> members = query.getResultList();
```

memberA, B, C는 영속성 컨텍스트에만 존재할 뿐, DB에 INSERT Query가 전송되지 않아 반영되지 않은 상태

JPQL Query 실행 시, flush()가 자동 호출되어 DB에 SQL로 번역된 INSERT Query가 전송, DB에 반영됨(동기화)

### 트랜잭션 커밋 시 플러시 자동 호출

<br>

## 옵션 설정

```java
em.setFlushMode(플러쉬 모드 타입 옵션);
```
1. FlushModeType.AUTO
   - 기본값
   - 트랜잭션을 커밋하거나 쿼리를 실행할 때 flush()를 먼저 수행
2. FlushModeType.COMMIT
   - 트랜잭션을 커밋할 때만 flush()를 먼저 수행
   - 쿼리를 실행할 때는 flush()를 먼저 수행하지 않음
   - persist한 것과 전혀 다른 테이블을 조회하는 경우

<br>
