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

### 

<br>
