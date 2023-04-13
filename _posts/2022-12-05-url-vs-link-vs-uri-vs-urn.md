---
layout: single
title: "URL, Link, URI, URN 개념"
date: 2022-12-05 10:22:16 +0900
categories: knowledge
tags: [URL, Link, URI, URN]
typora-root-url: ../
---


## URL, URI, URN 개념
> - URL
> - URI
> - URN

<br>

## URL

### 정의

URL(Uniform Resource Locator, 특정 자원 위치)

인터넷에서 특정 자원의 위치에 대한 모든 정보를 포함하는 문자열

### 구조

![anatomy-url](/images/2022-12-05-url-vs-link-vs-uri-vs-urn/anatomy-url.png){: width="560"}

Scheme 자원에 접속해야 하는 통신 규약

- 예시 : http, https, ftp, smtp

Domain 자원을 제공하는 서버 정보
- 예시 : 도메인 이름, IP 주소

Port 자원에 접속하기 위해 요청을 보내려는 통신 규약 포트번호
- 예시 : 80, 443, 일반적으로 기본 포트번호를 사용하므로 생략

Path 서버의 자원을 제공하는 출처 경로
- 예시 : 상위 경로/하위 경로

Parameters 자원을 제공하는 서버에 보낼 데이터 운용에 필요한 정보
- 예시 : ?key1=value1&key2=value2

Anchor 자원 내 특정부분을 대표하는 부분
- 예시 : #samples

<br>

## Link

브라우저의 URL에서 자원을 가져올 수 있는 HTML 요소

URL과 동의어는 아니지만 Link가 URL에 의존하므로 기술적으로는 동일

<br>

## URI

### 정의

URI(Uniform Resource Identifier, 특정 자원 식별자)는 자원을 특정할 수 있는 문자열

외관적으로 봤을 때 URL과 다른 점이 없음

URI는 자원을 가르키는 식별자일 뿐, 자원을 가져오기 위한 주소 아님

URI의 도메인명은 기존의 DNS 등록절차를 활용해 전역에서 특별한 이름

URL이 아닌 URI의 예시는 XML

### URL과 차이점

![url-vs-uri](/images/2022-12-05-url-vs-link-vs-uri-vs-urn/url-vs-uri.png){: width="560"}

URL은 식별자(Identifier)이면서, 동시에 접속할 수 있는 방향도 제공

<br>

## URN

### 정의

URN(Uniform Resource Name, 특정 자원 이름)

리소스가 존재하지 않는 경우에도 영구적인 방법으로 리소스 식별을 가능하게 만드는 문자열

URN은 인간 활동에 표준 식별자가 필요한 경우, 활용하기 위해 공공 표준 조직에서 발행한 식별자

<br>