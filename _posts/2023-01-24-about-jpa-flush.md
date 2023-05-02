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
> -


<br>

## 정의

영속성 컨텍스트의 변경 내용을 DB 에 반영하는 것을 의미

Transaction commit 이 일어날 때 flush가 동작하는데, 이때 쓰기 지연 저장소에 쌓아 놨던 INSERT, UPDATE, DELETE SQL들이 DB에 전송(영속성 컨텍스트를 비우는 게 아님!)

영속성 컨텍스트의 변경 사항들과 DB의 상태를 동기화하는 작업

<br>
