---
layout: single
title: "Spring Security: (2)Authentication"
date: 2022-12-14 10:41:33 +0900
categories: java
tags: [Spring Security]
typora-root-url: ../
---


## Spring Security: (2)Authentication
> - 정의
> - 구조

<br>

## 정의

Authentication(인증)은 특정 리소스에 접근하려는 사용자의 신원을 확인하는 방법

일반적으로 사용자 이름과 암호를 입력하도록 요구하여 인증을 수행

인증이 수행되고 나면, 정체를 알고 Authorization(인가)를 수행 가능

## 구조

### 주요 구성요소

- SecurityContextHolder
- SecurityContext
- Authentication
- GrantedAuthority
- AuthenticationManager
- ProviderManager
- AuthenticationProvider
- AuthenticationEntryPoint
- AbstractAuthenticationProcessingFilter

### 구체적인 역할

#### SecurityContextHolder
![securitycontextholder](/images/2022-12-14-about-spring-security-2-authentication/securitycontextholder.png){: width="560"}

누가 인증되었는지에 대한 정보를 저장

스프링 시큐리티는 어떻게 값이 채워지든 SecurityContextHolder에 값만 있으면 현재 인증된 사용자로 인식

사용자가 인증되었음을 나타내는 가장 간단한 방법은 SecurityContextHolder에 직접 값을 추가

```java
SecurityContext context = SecurityContextHolder.createEmptyContext();
Authentication authentication =
    new TestingAuthenticationToken("username", "password", "ROLE_USER");
context.setAuthentication(authentication);

SecurityContextHolder.setContext(context);
```
1. SecurityContext instance 생성
2. 새 Authentication 객체 생성(일반적으로 UsernamePasswordAuthenticationToken 사용)
3. SecurityContextHolder에 SecurityContext 주입

인증된 주체에 대한 정보를 얻으려면 SecurityContextHolder에 접근

```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
String username = authentication.getName();
Object principal = authentication.getPrincipal();
Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
```
ThreadLocal을 사용하면, 필터체인프록시가 시큐리티컨텍스트를 항상 비워주기 때문에 주체의 요청이 처리되고 나서 쓰레드가 비워지는 것을 보장

일부 응용프로그램은 스레드를 사용하는 특수한 방식으로 인해 ThreadLocal을 사용하는데 온전히 적합치 않음

예를 들어 Swing Client는 같은 시큐리티 컨텍스트를 사용하기 위해 JVM의 모든 스레드를 요구할 수도 있음

그래서 SecurityContextHolder에 컨텍스트가 어떻게 저장될지에 대한 전략 설정 가능

독립적인 어플리케이션에서는 SecurityContextHolder.MODE_GLOBAL전략을 사용하는 것이 좋음

그외 어플리케이션들은 SecurityContextHolder.MODE_INHERITABLETHREADLOCAL전략을 통해 보안 스레드에 의해 동일한 보안 정체성을 가지는 스레드들을 이용

SecurityContextHolder.MODE_THREADLOCAL의 모드는 두가지 방식 가능

첫번째는 시스템 속성을 세팅하는 방법, 두번째 방법은 SecurityContextHolder에서 정적 메소드를 호출하는 방법

#### SecurityContext

SecurityContext는 SecurityContextHolder에 포함되어 있으며, Authentication 객체를 가지고 있음

#### Authentication

스프링 시큐리티에서 Authentication 인터페이스는 두가지 쓰임
1. AuthenticationManager에 사용자가 인증을 위해 제공한 자격 증명을 입력
   - 이런 경우, isAuthenticated()는 false 반환
2. 현재 인증된 사용자를 나타내고, SecurityContext에서 Authentication를 획득 가능

##### 내부 요소

1. principal: 사용자 식별. 사용자 이름/비밀번호로 인증할 때 UserDetail의 인스턴스
2. credentials: 보통의 경우 암호를 의미. 대부분 인증된 후에는 이 값이 지워져 유출 방지
3. authorities: 사용자의 역할이나 범위 등 사용자에게 부여된 권한. GrantedAuthority 인스턴스

#### GrantedAuthority

사용자의 role이나 scope 등의 주체에게 부여된 권한을 의미

Authentication.getAuthorities() 메소드를 통해 GrantedAuthority Collection 획득

사용자명/암호 기반 인증 경우, UserDetailService에 의해 GrantedAuthority 호출

#### AuthenticationManager

스프링 시큐리티의 필터가 인증을 수행하는 방법을 정의하는 API

AuthenticationManager를 호출한 컨트롤러에 의해 Authentication이 SecurityContextHolder에 설정

스프링 시큐리티의 필터 인스턴스들을 통합하지 않는다면, SecurityContextHolder를 직접 세팅하고, AuthenticationManager를 사용하지 않을 수 있음

ProviderManager가 AuthenticationManager의 가장 일반적인 구현체

#### ProviderManager

AuthenticationProvider 인스턴스의 리스트를 위임하는 AuthenticationManager의 가장 일반적인 구현체

각각의 AuthenticationProvider는 인증이 성공하거나 실패했음을 나타내거나, 혹은 결정을 내릴 수 없음을 나타내고 AuthenticationProvider에게 결정하도록 허용할 수 있음

만약 어떤 AuthenticationProvider도 인증할 수 없다면, ProviderManager가 '전달된 인증 형태를 지원하도록 구성되지 않았다'는 것을 나타내는 특수한 AuthenticationException 예외인, ProviderNotFoundException가 발생하며 실패

![providermanager](/images/2022-12-14-about-spring-security-2-authentication/providermanager-2513912.png){: width="560"}

실제로 각 AuthenticationProvider는 특정한 각 유형의 인증을 수행

예를 들어 한 AuthenticationProvider는 사용자명/암호를 인증할 수 있는데, 다른 건 SAML assertion을 인증할 수 있음

이를 통해 각 AuthenticationProvider는 하나의 AuthenticationManager bean으로 노출되면서도 다양한 여러 유형의 인증을 지원할 수 있음

또한 ProviderManager를 사용하여 상위 AuthenticationManager(선택 사항) 구성 가능
이는 AuthenticationProvider가 인증을 수행할 수 없는 경우 참조

상위 AuthenticationManager는 어떤 유형이든 될 수 있지만, 거의 ProviderManager의 인스턴스

![providermanager-parent](/images/2022-12-14-about-spring-security-2-authentication/providermanager-parent.png){: width="560"}

실제로 여러 ProviderManager 인스턴스는 동일한 AuthenticationManager를 공유할 수 있음

일반적으로 하나의 인증에 서로 다른 ProviderManager로 인해 서로 다른 인증 기제를 가지는 여러 개의 SecurityFilterChain 인스턴스가 필요

![providermanagers-parent](/images/2022-12-14-about-spring-security-2-authentication/providermanagers-parent.png){: width="560"}

일반적으로 ProviderManager는 성공적인 인증 요청에 의해 반환된 Authentication 객체로부터 민감 자격증명 정보를 제거하려 함

이렇게 하면 HttpSession에 필요 이상으로 암호와 같은 정보가 남아있는 것을 방지할 수 있음
이런 경우, stateless(비저장) 어플리케이션에서 더 나은 성능을 위해 사용자 객체의 캐쉬를 사용하는 경우 문제가 될 수 있음

만약 Authentication이 캐쉬에 UserDetails 인스턴스같은 객체에 대해 참조하고 자격증명이 제거되어 있는 경우, 캐쉬에 저장된 값에 대해 더 이상 인증할 수 없음

만약 캐쉬를 사용한다면, 캐쉬 구현체와 AuthenticationProvider 안에 객체의 복제본을 만들어서 Authentication 객체를 반환하도록 해야 함

대신 ProviderManager에서는 eraseCredentialsAfterAuthentication을 꺼야 함

#### AuthenticationProvider

ProviderManager에는 여러 AuthenticationProvider 인스턴스를 넣을 수 있음
각 AuthenticationProvider는 특정 유형의 인증을 수행함

예를 들어 DaoAuthenticationProvider는 사용자명/암호 기반의 인증을 수행하고, JwtAuthenticationProvider는 JWT 토큰 인증을 수행함

#### AuthenticationEntryPoint

AuthenticationEntryPoint는 HTTP 응답으로 클라이언트에게 자격증명에 대한 요청을 보내는 데 쓰임

경우에 따라 클라이언트는 사전에 자격증명을 포함해서 리소스 요청을 보냄

그런 경우 스프링 시큐리티는 이미 포함되어 있으므로 클라이언트에 자격증명을 요청하는 HTTP응답을 보낼 필요가 없음

그렇지 않으면 접속권한이 없는 상태의 자원에 대해 인증이 되지 않은 요청을 하기 때문에 AuthenticationEntryPoint에서 클라이언트에게 자격증명을 요구함

#### AbstractAuthenticationProcessingFilter

사용자 자격증명을 인증하는데 쓰이는 기본적인 필터

자격증명에 대해 인증되기 전에 스프링 시큐리티는 AuthenticationEntryPoint를 이용하여 자격증명 요청

다음으로 AbstractAuthenticationProcessingFilter가 제출된 모든 인증 요청에 대한 인증

![abstractauthenticationprocessingfilter](/images/2022-12-14-about-spring-security-2-authentication/abstractauthenticationprocessingfilter.png){: width="560"}

1. 사용자가 자신의 자격증명을 제출하면 AbstractAuthenticationProcessingFilter가 HttpServletRequest의 Authentication을 인증되게 만듦(예를 들어 UsernamePasswordAuthenticationFilter는 HttpServletRequest에서 사용자명과 암호로부터 UsernamePasswordAuthenticationToken을 생성함)
2. 인증을 위해 Authentication이 AuthenticationManager한테 전달됨
3. 만약 인증이 실패 시
   1. SecurityContextHolder가 지워지고, 
   2. (기억하기가 설정되어 있을 경우)RememberMeServices.loginFail가 호출되고,
   3. AuthenticationFailureHandler가 호출됨
4. 만약 인증이 성공 시
   1. SessionAuthenticationStrategy에 새로운 로그인을 알리고, 
   2. Authentication이 SecurityContextHolder에 설정되고, (만약 이후의 요청에 사용하기 위해 SecurityContext를 저장하려면) SecurityContextRepository#saveContext를 명시적으로 호출하며,
   3. RememberMeService.loginSuccess가 호출되며,
   4. ApplicationEventPublisher가 InteractiveAuthenticationSuccessEvent를 게시하고,
   5. AuthenticationSuccessHandler가 호출됨