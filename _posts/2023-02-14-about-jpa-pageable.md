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

## Spring Data JPA에서의 페이징과 정렬

다음은 JpaRepository 인터페이스의 상속 다이어그램

![jparepo](/images/2023-02-14-about-jpa-pageable/jparepo.png){: width="560"}

JpaRepository의 부모 인터페이스인 PagingAndSortingRepository에서 페이징과 소팅이라는 기능을 제공

![pagingandsort](/images/2023-02-14-about-jpa-pageable/pagingandsort.png){: width="560"}

findAll() 메서드가 있고, 그 메서드의 반환 타입과 파라미터

- org.springframework.data.domain.Pageable : 페이징을 제공
- org.springframework.data.domain.Page : 페이징의 findAll()의 여러 반환 타입 중 하나

JpaRepository<>를 사용할 때, findAll() 메서드를 Pageable 인터페이스로 파라미터를 넘기면 페이징을 사용 가능



<br>
