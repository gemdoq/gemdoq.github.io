---
layout: single
title: "필터와 인터셉터의 개념과 역사"
date: 2022-12-07 11:15:21 +0900
categories: Java
tags: [Filter, Interceptor, DelegatingFilterProxy, SpringBoot]
typora-root-url: ../
---


## 필터와 인터셉터의 개념과 역사
> - 필터
> - 인터셉터
> - 필터와 인터셉터의 차이
> - DelegatingFilterProxy 등장
> - SpringBoot 등장

<br>

![filter](/images/2022-12-07-filter-vs-interceptor/filter.png){: width="560"}

## 필터란

### 정의

필터(Filter)는 Dispatcher Servlet에 요청이 전달되기 전/후에 URL 패턴에 맞는 모든 요청에 대한 부가작업을 처리할 수 있는 기능을 제공

필터를 추가하기 위해서는 javax.servlet의 Filter 인터페이스를 implements해야 사용 가능

### Filter의 메소드
```java
public interface Filter {

    public default void init(FilterConfig filterConfig) throws ServletException {}

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {}

    public default void destroy() {}
}
```

#### init메소드

필터 객체를 초기화하고 서비스에 추가

웹 컨테이너가 1회 호출하여 객체를 초기화하면 이후의 요청들은 doFilter를 통해 처리

#### doFilter메소드

URL 패턴에 맞는 모든 HTTP 요청이 dispatcher servlet으로 전달되기 전 웹 컨테이너에 의해 실행

파라미터 중 FilterChain의 doFilter를 통해 다음 대상으로 요청을 전달

#### destroy메소드

필터 객체를 서비스에서 제거하고 사용하는 자원을 반환하기 위한 메소드

웹 컨테이너가 1회 호출하면 이후의 요청들은 doFilter에 의해 처리되지 않음

### 용도

필터에서는 스프링과 무관하게 전역 작업들을 처리

대표적으로 보안 공통 작업, 이미지나 데이터의 압축, 문자열 인코딩과 같이 애플리케이션 전반에 사용되는 작업들 처리

<br>

## 인터셉터(Interceptor)

### 정의

필터와 다르게 Spring이 제공하는 기술

Dispatcher Servlet과 Controller 간 요청과 응답에 간섭하여 가공할 수 있는 기능 제공

필터는 Servlet Container에서 작동하는 것과 달리 인터셉터는 Spring Container에서 작동

### 작동 원리

Dispatcher Servlet은 handler mapping을 통해 적절한 Controller를 찾도록 요청하고 그 결과로 HandlerExecutionChain을 반환

1개 이상의 Interceptor가 등록되어 있으면 순차적으로 인터셉터들을 거쳐 컨트롤러가 실행

### Interceptor메소드

인터셉터를 추가하기 위해서는 org.springframework.web.servlet의 HandlerInterceptor 인터페이스를 implements 해야 사용 가능
```java
public interface HandlerInterceptor {

    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception { return true; }

    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {}

    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {}
}
```

#### preHandle메소드

preHandle메소드는 컨트롤러가 실행되기 전 실행되므로 컨트롤러 이전에 처리해야 하는 전처리 작업이나 요청 정보를 가공하거나 추가하는 경우에 사용

3번째 파라미터인 handler는 핸들러 매핑이 찾아준 컨트롤러 bean에 매핑되는 HandlerMethod라는 새로운 타입의 객체로써, @RequestMapping이 붙은 메소드의 정보를 추상화한 객체

preHandle의 반환 타입은 boolean인데 반환값이 true이면 다음 단계로 진행, false라면 다음 인터셉터 또는 컨트롤러는 작업이 중단되어 실행되지 않음

#### postHandle메소드

postHandle메소드는 컨트롤러가 호출된 뒤에 실행되므로 컨트롤러 이후에 처리해야 하는 후처리 작업시 사용

4번째 파라미터인 modelAndView는 컨트롤러가 반환하는 데이터를 담고 있는 객체

최근에는 Json 형태로 데이터를 제공하는 RestAPI 기반의 컨트롤러(@RestController)를 만들면서 자주 사용되지 않음

#### afterCompletion메소드

afterCompletion메소드는 이름 그대로 모든 뷰에서 최종 결과를 생성하는 일을 포함해 모든 작업이 완료된 후에 실행

요청 처리 중에 사용한 리소스를 반환할 때 사용하기 적합

postHandler와 달리 컨트롤러 하위 계층에서 작업이 진행되다 중간에 예외가 발생하더라도 afterCompletion은 반드시 호출

### 용도

인터셉터에서는 클라이언트의 요청과 관련되어 전역적으로 처리해야 하는 작업들을 처리

대표적으로 세부적으로 적용해야 하는 인증이나 인가와 같이 클라이언트의 요청과 관련된 작업, API 호출에 대한 IP나 요청 정보에 대한 기록 작업 등  

### 인터셉터(Interceptor)와 AOP

인터셉터 대신 컨트롤러에 적용하고 싶은 부가기능들을 어드바이스로 만들어 AOP(Aspect Oriented Programming, 관점 지향 프로그래밍)를 적용할 수 있으나, 아래와 같은 이유들로 컨트롤러의 호출 과정에 적용되는 부가기능들은 인터셉터를 사용하는 편
1. 컨트롤러는 타입과 실행 메소드가 모두 제각각 → pointCut(적용할 메소드 선별) 작성이 어려움
2. 컨트롤러는 파라미터나 리턴값이 일정하지 않음
3. AOP에서는 HttpServletRequest/Response 객체를 얻기 어렵지만 인터셉터는 파라미터로 받을 수 있음

<br>

## 필터와 인터셉터의 차이

### 관리되는 컨테이너

필터와 인터셉터는 관리되는 영역이 다름

필터는 스프링 이전의 서블릿 영역에서 관리되고, 인터셉터는 스프링 영역에서 관리

### 스프링의 예외 처리 여부

일반적으로 스프링을 사용한다면 ControllerAdvice와 ExceptionHandler를 이용한 예외 처리 기능을 주로 사용

예를 들어 원하는 멤버를 찾지 못하여 로직에서 MemberNotFoundException을 던졌다면 404 status로 응답을 받일 원하고, 이를 위해 다음과 같은 예외 처리기를 구현하여 활용

이를 통해 예외가 서블릿까지 전달되지 않고 처리

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(MemberNotFoundException.class)
    public ResponseEntity<Object> handleMyException(MemberNotFoundException e) {
        return ResponseEntity.notFound()
            .build();
    }
}
```

하지만 필터는 서블릿 영역에서 관리되므로, 스프링이 처리해주는 내용들(예외처리 등)을 적용받을 수 없음

그래서 만약 필터에서 MemberNotFoundException이 던져지면, 에러가 처리되지 않고 서블릿까지 전달

이에 서블릿은 핸들링되지 않은 예상치 못한 Exception을 만났으므로 내부에 문제가 있다고 판단하여 500 status로 응답을 반환

이를 해결하려면 필터에서 다음과 같이 Response 객체에 예외 Handling 필요

```java
public class MyFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletResponse servletResponse = (HttpServletResponse) response;
        servletResponse.setStatus(HttpServletResponse.SC_NOT_FOUND);
        servletResponse.getWriter().print("Member Not Found");
    }
}
```

### Request/Response 객체 조작 가능 여부

```java
public MyFilter implements Filter {

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        // 다른 Request와 Response를 넣어 넘겨줄 수 있음
        chain.doFilter(new MockHttpServletRequest(), new MockHttpServletResponse());
    }
}
```

필터는 Request와 Response를 조작할 수 있지만 인터셉터는 조작할 수 없음

여기서 조작한다는 것은 내부 상태를 변경하는 것이 아닌 다른 객체로 바꿔친다는 의미

필터가 다음 필터를 호출하기 위해서 filterChain을 거쳐야 하는데, 이때 전달되는 Request/Response에 우리가 원하는 객체를 넣어 넘겨줄 수 있음
(null이 들어가서 NPE이 발생할 수도 있지만) 

```java
public class MyInterceptor implements HandlerInterceptor {

    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // boolean만 반환되므로 Request나 Response를 교체할 수 없음
        return true;
    }
}
```

반면 인터셉터는 처리 과정이 필터와 다르게, Dispatcher Servlet이 여러 인터셉터 목록을 가지고 있고, for문으로 순차적으로 실행

그리고 true를 반환하면 다음 인터셉터가 실행되거나 컨트롤러로 요청이 전달, false를 반환하면 작업이 중단

<br>

## DelegatingFilterProxy 등장

지금까지의 내용은 Spring 1.2 이전까지의 필터와 인터셉터에 대한 개념

Spring 1.2 이전까지의 필터는 서블릿이 제공하는 기술이므로 서블릿 컨테이너에 의해 생성되며 서블릿 컨테이너에 등록되었지만 필터에서도 DI와 같은 스프링 기술이 필요한 경우가 발생

→ Spring 1.2부터 DelegatingFilterProxy가 나오면서 Servlet Filter 역시 스프링에서 관리가 가능

### 정의 

Servlet Container에서 관리되는 proxy filter로써 우리가 만든 필터를 가지고 있음

우리가 만든 필터는 Spring Container의 bean으로 등록되는데, 요청이 오면 DelegatingFilterProxy가 요청을 받아서 Spring Container에 등록된 bean(우리가 만든 필터)에 요청을 위임

### 동작 원리

#### Filter 구현체, 우리가 만든 필터 "MyFilter"
```java
@Component
public class MyFilter implements Filter {

    @Override
    public void doFilter(final ServletRequest request, final ServletResponse response, final FilterChain chain) throws ServletException, IOException {
        chain.doFilter(request, response);
    }

    @Override
    public void init(final FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
    }

    @Override
    public void destroy {
        Filter.super.destroy();
    }
}
```

#### addFilter로 우리가 만든 필터 "MyFilter" ServletContext에 등록
```java
public class MyWebApplicationInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    
    public void onStartup(ServletContext servletContext) throws ServletException {
        super.onStartup(servletContext);
        servletContext.addFilter("myFilter", DelegatingFilterProxy.class);
    }
}
```

#### 순서

1. Filter 구현체인 MyFilter가 Spring Container에 bean으로 등록
2. ServletContext가 Filter 구현체를 가지는 DelegatingFilterProxy를 생성
3. ServletContext가 DelegatingFilterProxy를 Servlet Container에 필터로 등록
4. 요청이 오면 DelegatingFilterProxy는 Filter 구현체인 MyFilter에게 요청을 위임하여 처리

<br>

## SpringBoot 등장

SpringBoot 등장 이후 내장 웹서버를 지원하기 때문에 Tomcat과 같은 Servlet Container까지 제어가 가능

SpringBoot가 Servlet Filter의 구현체 bean을 찾으면 DelegatingFilterProxy 없이 바로 Filter Chain에 등록

### Spring에서 doFilter 런타임 예외

```console
java.lang.RuntimeException
    at com.test.MyFilter.doFilter(MyFilter.java:17)
    at org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:358)
    at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:271)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
```

### SpringBoot에서 doFilter 런타임 예외
```console
at com.test.MyFilter.doFilter(MyFilter.java:17) [main/:na]
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) [tomcat-embed-core-9.0.56.jar:9.0.56]
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-core-9.0.56.jar:9.0.56]
    at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100) [spring-web-5.3.15.jar:5.3.15]
    at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) [spring-web-5.3.15.jar:5.3.15]
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) [tomcat-embed-core-9.0.56.jar:9.0.56]
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) [tomcat-embed-core-9.0.56.jar:9.0.56]
```

위의 Spring에서 doFilter 런타임 예외를 발생시킨 로그를 보면 DelegatingFilterProxy가 doFilter로 요청을 받고, 이를 실제 구현체인 MyFilter로 위임(invokeDelegate)

반면 SpringBoot에서 doFilter 런타임 예외를 발생시킨 로그를 보면 DelegatingFilterProxy 관련 내용이 없음

→ SpringBoot에서는 DelegatingFilterProxy 등록 안함

<br>