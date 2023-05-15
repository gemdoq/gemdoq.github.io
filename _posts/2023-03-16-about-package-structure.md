---
layout: single
title: "패키지 구조에 대하여"
date: 2023-03-16 11:27:20 +0900
categories: java
tags: [계층형 패키지 구조, 도메인형 패키지 구조]
typora-root-url: ../
---

## 패키지 구조에 대하여
> - 패키지 구조란
> - 계층형 패키지 구조
> - 도메인형 패키지 구조

<br>

## 패키지 구조란

Spring boot를 활용한 API 개발 시, 어떤 패키지 구조로 설계하느냐에 따라 유의미한 설계 및 개발 상의 특성이 발현

다양한 기준이 있겠지만 본 글에서는 개인적인 기준에 따라 패키지 구조를 분류 및 정리

최초 스프링을 학습하는 과정에서는 웹 계층형 패키지 구조를 기반으로 학습했으나, 

이후에 프로젝트를 만드는 과정에서 기존 웹 계층형 패키지 구조의 단점과 개선점들이 보였음

이에 다른 방식의 패키지 구조를 알아보다 도메인형 패키지 구조에 대해 알게 되어 프로젝트에 적용해보니 기존 웹 계층형 패키지 구조와 몇 가지의 차이가 있음

<br>

## 계층형 패키지 구조

![web-hierarchy](/images/2023-03-16-about-package-structure/web-hierarchy.png)

웹 계층형 패키지 구조라고도 하며, 앞서 설명했던 것처럼 이러한 방식은 다음과 같은 대표적인 웹 계층적 역할 기반으로 구분되어 패키징

- Web Layer : 사용자의 요청과 이에 대한 응답 반환의 전반적인 처리가 일어나는 영역을 의미
- Service Layer : Web Layer와 Repository Layer 사이에서 실질적인 어플리케이션 비즈니스 로직이 일어나는 영역
- Repository Layer : DB에 접근 및 통신하는 영역

이러한 각 계층들에는 Controller, Service, Repository 등과 같이 그 계층들을 대표하는 역할 클래스가 존재

이들을 기반으로 패키지 구조가 설계되어 개발하는 방식이 계층형 패키지 구조

계층형 구조는 전체적인 구조를 빠르게 파악할 수 있는 장점이 있지만, 각각의 패키지 디렉터리에 클래스들이 너무 많이 모이게 된다는 단점이 존재

스프링 학습 시 프로젝트에서 설계한 계층형 패키지 구조의 예시

![hierarchy](/images/2023-03-16-about-package-structure/hierarchy.png)

특히 SRP원칙을 준수하기 위해 Service 패키지에는 특정 역할을 하는 전용 구현체 클래스들을 많이 생성하기 때문에 여러 도메인의 클래스들이 혼재

<br>

## 도메인형 패키지 구조

![domain](/images/2023-03-16-about-package-structure/domain.png)

이번 프로젝트를 진행하면서 설계·개발했던 도메인 패키지 구조

해당 방식은 스프링 웹 계층에 주목하기보다는 도메인에 주목

이를 통해서 각각의 도메인 별로 패키지 분리가 가능

각각의 도메인들은 서로를 의존하는 코드가 없기 때문에 협업에 휴먼에러의 발생 가능성이 낮으며, 코드 재활용성이 향상

또한 핵심 Entity를 기반으로 패키징하기 때문에 OOP 관점 지향점과도 일치하며 ORM 기법과도 맥락이 같음

### 최상위 레벨의 패키지

최상위 레벨에서는 domain과 global로 구분
domain 패키지에서는 프로젝트와 DB 구조에서 핵심 역할을 하는 domian entitiy를 기준으로 하위 패키지 구성
global 패키지에서는 프로젝트 전방위적으로 사용할 수 있는 클래스들로 구성

### Domain 하위 패키지

앞서 설명한 것처럼 user, video와 같이 핵심 domain entity 별로 패키지 구성
각각의 domian 하위 패키지는 api, application, dao, domain, dto, exception으로 구성

- api : controller 클래스
- application : service, handler 클래스
- dao : dao, repository 클래스
- domain : entity 클래스
- dto : dto 클래스
- exception : exception 클래스

### Global 하위 패키지

해당 패키지에는 특정 domain에 종속되지 않고, 프로젝트 전방위적으로 사용할 수 있는 클래스
global 패키지는 auth, common, config, error, infra, util로 구성

- auth : 인증, 인가와 관련된 클래스
- common : 공통 클래스 혹은 공통 value 클래스
- config : 각종 configuration 클래스
- error : 공통 exception, error와 관련된 클래스
- infra : 외부 모듈, api 등을 사용하는 클래스
- util : 공통 util성 클래스

<br>
