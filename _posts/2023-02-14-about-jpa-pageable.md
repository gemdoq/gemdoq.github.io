---
layout: single
title: "JPA에서 Pageable에 대해"
date: 2023-02-14 10:19:22 +0900
categories: java
tags: [Pageable]
typora-root-url: ../
---

## JPA에서 Pageable에 대해
> - 페이징이란
> - Spring Data JPA에서의 페이징과 정렬
> - 쿼리 메서드에서 페이징 사용하기
> - 반환 타입에 따른 페이징 결과
> - Spring Web MVC에서 더 편하게 페이징하기

<br>

## 페이징이란

게시판이나 댓글, 블로그를 개발할 때 페이징은 중요한 역할

페이징은 수많은 게시글과 같은 정보들을 페이지로 나눠 효과적으로 정보를 제공

![google](/images/2023-02-14-about-jpa-pageable/google.png){: width="560"}

과거 페이징을 구현하기 위해서 page 관련 쿼리를 파라미터로 받아서 직접 처리하는 방법이 있었으나, Spring Data JPA에서 페이징을 효과적으로 처리할 수 있는 방법을 제공

<br>

## 

<br>
