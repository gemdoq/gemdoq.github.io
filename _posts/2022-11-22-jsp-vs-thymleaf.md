---
layout: single
title: "JSP와 Thymleaf"
date: 2022-11-22 10:37:24 +0900
categories: knowledge
tags: [Template Engine, JSP, Thymleaf]
typora-root-url: ../
---


## Thymleaf와 JSP
> - Template Engine
> - JSP
> - Thymleaf

<br>

## Template Engine

![thymleaf](/images/2022-11-22-jsp-vs-thymleaf/thymleaf.png){: width="560"}

### 정의

템플릿 양식에 따른 입력 자료로 결과를 구현해주는 코드 요소

Rendering(구현) site에 따라, 서버 사이드와 클라이언트 사이드로 구분

### 특징

- 기존의 HTML에 비해서 간단한 문법
- 재사용성 높은 코드
- 유지 보수 용이

<br>

## JSP

일반적인 Java MVC에서 View Template Engine

Servlet 형태로 변환되어 실행

JSP 프로젝트의 경우 War로 빌드(배포 불편)

<br>

## Thymleaf

- Spring에서 일반적인 View Template Engine

- HTML 파일을 parsing 하고 정해진 위치에 구현
  
  → 필요한 자료를 엘리먼트에 속성으로 넣어 브라우저에서 바로 확인
  
- 웹(HTML, CSS, JS, XML) 및 독립형 환경에서 사용 가능

- Spring boot 프로젝트를 빌드하면 Jar로 빌드(배포 용이)

<br>

## JSP와 Thymleaf 비교

Thymleaf는 순수 HTML 형식을 유지

→ WAS없이 HTML Element에 속성으로 필요한 Data를 넣어 바로 View 화면을 확인

반면, JSP는 Servlet형태로 변환

→ 서버를 통해서 Rendering 후 View 화면을 확인

### JSP와 Thymleaf 동시 사용 불가능

![jsp](/images/2022-11-22-jsp-vs-thymleaf/jsp.png){: width="560"}

- JSP는 실행 가능한 jar를 사용할 때 지원되지 않습니다. 따라서 war 패키징을 해야 합니다.
- Spring boot 공식 지원 내장 WAS인 Undertow는 JSP를 지원하지 않습니다.
- 사용자 지정 error.jsp를 지원하지 않습니다.

<br>
