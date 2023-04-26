---
layout: single
title: "Spring Security: (3)Authorization"
date: 2022-12-17 11:15:33 +0900
categories: java
tags: [Spring Security]
typora-root-url: ../
---


## Spring Security: (3)Authorization
> - 정의
> - 사전 호출 처리
> - 계층적 역할
> - 레거시 인증 구성 요소

<br>

## 정의

Authorization(인가)은 특정 리소스에 접근하려는 사용자의 권한을 확인하는 방법

스프링 시큐리티에서는 GrantedAuthority 객체를 통해 보안 주체에게 부여된 권한을 확인

## 사전 호출 처리

스프링 시큐리티는 웹 요청이나 메소드 호출과 같은 보안 객체에 대한 접근 통제를 할 수 있는 인터셉터를 AccessDecisoinManager에 의해 제공

### AuthorizationManager

AuthorizationManager는 AccessDecisoinManager과 AccessDecisionVoter 모두를 대체

AccessDecisoinManager이나 AccessDecisionVoter를 사용자 정의하는 어플리케이션들은 AuthorizationManager를 사용

AuthorizationManager는 최종 접속 권한 결정을 책임지는 AuthorizationFilter에 의해 호출되며, 두가지 메소드를 포함
```java
AuthorizationDecision check(Supplier<Authentication> authentication, Object secureObject);

default AuthorizationDecision verify(Supplier<Authentication> authentication, Object secureObject) throws AccessDeniedException {}
```
check 메소드는 인증 결정을 내리기 위해 필요한 모든 관련 정보를 전달
특히 보안 객체를 전달하여 실제 보안 개체 호출에 포함된 인수를 검사

verify 메소드는 check 메소드를 호출 후 AuthorizationDecision이 부정일 경우, AccessDeniedException 던짐

### 대리자 기반 AuthorizationManager 구현

사용자는 자신의 AuthorizationManager를 구현하여 권한 부여의 모든 측면을 제어할 수 있지만, 스프링 보안은 개별 AuthorizationManager와 협업할 수 있는 AuthorizationManager 위임 제공

RequestMatcherDelegatingAuthorizationManager는 요청을 가장 적절한 AuthorizationManager와 매칭

메소드 보안을 위해 AuthorizationManagerBeforeMethodInterceptor와 AuthorizationManagerAfterMethodInterceptor 사용 가능

![authorizationhierarchy](/images/2022-12-17-about-spring-security-3-authorization/authorizationhierarchy.png){: width="560"}

이 접근 방식을 통해 AuthorizationManager 구현의 구성을 인가 결정에 반영

#### AuthorityAuthorizationManager

스프링 시큐리티에서 제공하는 가장 일반적인 AuthorizationManager이며, 현재 Authentication의 권한들로 구성

#### AuthenticatedAuthorizationManager

익명이거나 완전히 인증되었거나 기억된 사용자를 구별하는 데 사용되는 AuthorizationManager

많은 사이트에서 remember-me설정에 따라 특정 제한된 접속을 허용하지만 전체 접속을 하기 위해 사용자는 자신의 신원을 확인하기 위해 로그인 필요

#### Custom Authorization Managers

어플리케이션에 따라 자체 인증 데이터베이스나 오픈 정책을 따로 사용하여 특정한 보안 관리 로직을 구현할 수 있는 AuthorizationManager

<br>

## 계층적 역할

일반적으로 어플리케이션에서 특정 역할은 자동으로 다른 역할을 포함하기 때문에 스프링 시큐리티에서는 RoleVoter에서 확장된, RoleHierarchyVoter라는 기능을 제공하여 사용자에게 도달할 수 있는 권한을 할당함으로써 역할 계층 구분
```java
@Bean
AccessDecisionVoter hierarchyVoter() {
    RoleHierarchy hierarchy = new RoleHierarchyImpl();
    hierarchy.setHierarchy("ROLE_ADMIN > ROLE_STAFF\n" +
            "ROLE_STAFF > ROLE_USER\n" +
            "ROLE_USER > ROLE_GUEST");
    return new RoleHierarchyVoter(hierarchy);
}
```
역할 계층 순서는 ROLE_ADMIN → ROLE_STAFF → ROLE_USER → ROLE_GUEST

화살표 기호(→)는 포함을 의미

ROLE_ADMIN으로 인증된 사용자는 네 역할을 모두 권한을 모두 소유

<br>

## 레거시 인증 구성 요소

### AccessDecisionManager
### 투표 기반 AccessDecisionManager 구현

<br>
