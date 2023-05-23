---
layout: single
title: "Thymeleaf에 대해"
date: 2023-04-08 10:52:44 +0900
categories: java
tags: [Thymeleaf, SSR, Template]
typora-root-url: ../
---

## Thymeleaf에 대해
> - 정의
> - 사용법
> - 문법
> - 결론

<br>

## 정의

HTML, XML, JavaScript, CSS 등 모든 웹 템플릿을 지원하며 모델(데이터)을 뷰(화면)에 렌더링하는 Java 템플릿 엔진

Java 백엔드 어플리케이션에서 Spring 프레임워크와 함께 사용하기 가장 적합하며, 서버사이드 렌더링(SSR) 용도

깔끔한 순수 HTML 코드를 작성할 수 있도록 도와주며, 지역화(I18N)와 메시징 등 다양한 기능

<br>

## 사용법

타임리프를 사용하기 위해 다음과 같이 선언

build.gradle
```xml
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
}
```

html
```html
<html xmlns:th="http://www.thymeleaf.org">
```

<br>

## 문법

### 텍스트 : th:text, th:utext

#### 세팅

```java
public String textBasic(Model model) {
    model.addAttribute("data", "Hello Spring!");
    return "basic/text-basic";
}
```

#### 사용

HTML의 콘텐츠(content)에 데이터를 출력 `<span th:text="${data}"></span>`

HTML 속성이 아니라 HTML 콘텐츠 영역안에서 직접 데이터를 출력 `<>[[${data}]]</>`

타임리프는 기본적으로 이스케이프를 제공하므로, HTML 엔티티로 변경하지 않기 위해서는 다음 두 기능을 사용

- th:text → th:utext
- [[...]] → [(...)]

### 변수 : SpringEL

변수를 사용할 때는 변수 표현식을 사용

변수 표현식 : ${...}

#### 세팅

```java
public String variable(Model model) {
    User userA = new User("userA", 10);
    User userB = new User("userB", 20);
    List<User> list = new ArrayList<>();
    list.add(userA);
    list.add(userB);
    Map<String, User> map = new HashMap<>();
    map.put("userA", userA);
    map.put("userB", userB);
    model.addAttribute("user", userA);
    model.addAttribute("users", list);
    model.addAttribute("userMap", map);
    return "basic/variable";
}
@Data
static class User {
    private String username;
    private int age;
    public User(String username, int age) {
        this.username = username;
        this.age = age;
    } 
}
```

#### 사용

```html
${user.username}
${user['username']}
${user.getUsername()}
```

#### 지역 변수

th:with 를 사용하면 지역 변수를 선언해서 선언한 태그 안에서만 사용 가능

```html
<div th:with="first=${users[0]}">
<p>처음 사람의 이름은 <span th:text="${first.username}"></span></p> </div>
```

### 기본 객체 / 편의 객체

HTTP 요청 파라미터 접근: param - ${param.paramData}

HTTP 세션 접근: session - ${session.sessionData}

스프링 빈 접근: @ - ${@helloBean.hello('Spring!')}

#### 세팅

```java
public String basicObjects(HttpSession session) {
      session.setAttribute("sessionData", "Hello Session");
      return "basic/basic-objects";
  }
  @Component("helloBean")
  static class HelloBean {
      public String hello(String data) {
          return "Hello " + data;
} }
```

#### 사용

```html
<h1>기본 객체</h1>
<ul>
    <li>request = <span th:text="${#request}"></span></li>
    <li>response = <span th:text="${#response}"></span></li>
    <li>session = <span th:text="${#session}"></span></li>
    <li>servletContext = <span th:text="${#servletContext}"></span></li>
    <li>locale = <span th:text="${#locale}"></span></li>
</ul>
<h1>편의 객체</h1>
<ul>
    <li>Request Parameter = <span th:text="${param.paramData}"></span></li>
    <li>session = <span th:text="${session.sessionData}"></span></li>
    <li>spring bean = <span th:text="${@helloBean.hello('Spring!')}"></span></li>
</ul>
```

### 유틸리티 객체와 날짜

타임리프는 문자, 숫자, 날짜, URI등을 편리하게 다루는 다양한 유틸리티 객체들을 제공

- #message : 메시지, 국제화 처리
- #uris : URI 이스케이프 지원
- #dates : java.util.Date 서식 지원
- #calendars : java.util.Calendar 서식 지원
- #temporals : 자바8 날짜 서식 지원
- #numbers : 숫자 서식 지원
- #strings : 문자 관련 편의 기능
- #objects : 객체 관련 기능 제공
- #bools : boolean 관련 기능 제공
- #arrays : 배열 관련 기능 제공
- #lists , #sets , #maps : 컬렉션 관련 기능 제공
- #ids : 아이디 처리 관련 기능 제공, 뒤에서 설명

이 중 자바8 날짜 서식에 관련된 temporals를 예시로 사용

나머지 유틸리티 객체는 다음 링크 참조

[🔗 타임리프 유틸리티 객체 링크](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#expression-utility-objects){:target="_blank"}

[🔗 유틸리티 객체 예시 링크](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#appendix-b-expression-utility-objects){:target="_blank"}

#### 세팅

```java
public String date(Model model) {
      model.addAttribute("localDateTime", LocalDateTime.now());
      return "basic/date";
  }
```

#### 사용

```html
<h1>LocalDateTime - Utils</h1>
  <ul>
      <li>${#temporals.day(localDateTime)} = <span th:text="$
  {#temporals.day(localDateTime)}"></span></li>
    <li>${#temporals.month(localDateTime)} = <span th:text="$
   {#temporals.month(localDateTime)}"></span></li>
      <li>${#temporals.monthName(localDateTime)} = <span th:text="$
  {#temporals.monthName(localDateTime)}"></span></li>
      <li>${#temporals.monthNameShort(localDateTime)} = <span th:text="$
  {#temporals.monthNameShort(localDateTime)}"></span></li>
      <li>${#temporals.year(localDateTime)} = <span th:text="$
  {#temporals.year(localDateTime)}"></span></li>
      <li>${#temporals.dayOfWeek(localDateTime)} = <span th:text="$
  {#temporals.dayOfWeek(localDateTime)}"></span></li>
      <li>${#temporals.dayOfWeekName(localDateTime)} = <span th:text="$
  {#temporals.dayOfWeekName(localDateTime)}"></span></li>
      <li>${#temporals.dayOfWeekNameShort(localDateTime)} = <span th:text="$
  {#temporals.dayOfWeekNameShort(localDateTime)}"></span></li>
      <li>${#temporals.hour(localDateTime)} = <span th:text="$
  {#temporals.hour(localDateTime)}"></span></li>
      <li>${#temporals.minute(localDateTime)} = <span th:text="$
  {#temporals.minute(localDateTime)}"></span></li>
      <li>${#temporals.second(localDateTime)} = <span th:text="$
  {#temporals.second(localDateTime)}"></span></li>
      <li>${#temporals.nanosecond(localDateTime)} = <span th:text="$
  {#temporals.nanosecond(localDateTime)}"></span></li>
  </ul>
```

### URL 링크

타임리프에서 URL을 생성할 때는 @{...} 문법을 사용

#### 세팅

```java
public String link(Model model) {
    model.addAttribute("param1", "data1");
    model.addAttribute("param2", "data2");
    return "basic/link";
}
```

#### 사용

```html
<ul>
    <li><a th:href="@{/hello}">basic url</a></li>
    <li><a th:href="@{/hello(param1=${param1}, param2=${param2})}">hello query param</a></li>
    <li><a th:href="@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}">path variable</a></li>
    <li><a th:href="@{/hello/{param1}(param1=${param1}, param2=${param2})}">path variable + query parameter</a></li>
</ul>
```

단순한 URL
- @{/hello} → /hello
쿼리 파라미터
- @{/hello(param1=${param1}, param2=${param2})}
  - → /hello?param1=data1&param2=data2
  - ()에 있는 부분은 `쿼리 파라미터`로 처리
경로 변수
- @{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}
  - → /hello/data1/data2
  - URL 경로상에 변수가 있으면 ()부분은 `경로 변수`로 처리
경로 변수 + 쿼리 파라미터
- @{/hello/{param1}(param1=${param1}, param2=${param2})}
  - → /hello/data1?param2=data2
  - `경로 변수`와 `쿼리 파라미터`를 함께 사용 가능
  - 상대경로, 절대경로, 프로토콜 기준을 표현할 수도 있음
- /hello : 절대 경로
- hello : 상대 경로

### 리터럴

리터럴은 소스 코드상에 고정된 값을 말하는 용어

예를 들어서 다음 코드에서 "Hello" 는 문자 리터럴, 10 , 20 는 숫자 리터럴

```java
String a = "Hello"
int a = 10 * 20
```

타임리프에서 문자 리터럴은 항상 '(작은 따옴표)로 감싸야 함
<span th:text="'hello'">
          
그런데 문자를 항상 '로 감싸는 것은 너무 귀찮기에, 공백없이 쭉 이어진다면 하나의 의미 있는 토큰으로 인지해서 다음과 같이 작은 따옴표를 생략 가능

룰: A-Z, a-z, 0-9, [], ., -, _

<span th:text="hello">

- 오류

<span th:text="hello world!"></span>

문자 리터럴은 원칙상 '로 감싸야 함

중간에 공백이 있어서 하나의 의미있는 토큰으로도 인식되지 않음 

- 수정

<span th:text="'hello world!'"></span>

이렇게 '로 감싸면 정상 동작

#### 세팅

```java
public String literal(Model model) {
    model.addAttribute("data", "Spring!");
    return "basic/literal";
}
```

#### 사용

```html
<ul>
    <!--주의! 다음 주석을 풀면 예외 발생-->
    <!--    <li>"hello world!" = <span th:text="hello world!"></span></li>-->  
    <li>'hello' + ' world!' = <span th:text="'hello' + ' world!'"></span></li>
    <li>'hello world!' = <span th:text="'hello world!'"></span></li>
    <li>'hello ' + ${data} = <span th:text="'hello ' + ${data}"></span></li>
    <li>리터럴 대체 |hello ${data}| = <span th:text="|hello ${data}|"></span></li>
</ul>
```

- 리터럴 대체(Literal substitutions)

<span th:text="|hello ${data}|">

마지막의 리터럴 대체 문법을 사용하면 마치 템플릿을 사용하는 것 처럼 편리

### 연산

타임리프 연산은 자바와 크게 다르지 않기 때문에 HTML 엔티티를 사용하는 부분만 주의



#### 세팅

```java
public String operation(Model model) {
    model.addAttribute("nullData", null);
    model.addAttribute("data", "Spring!");
    return "basic/operation";
}
```

#### 사용

```html
<ul>
    <li>산술 연산
        <ul>
            <li>10 + 2 = <span th:text="10 + 2"></span></li>
            <li>10 % 2 == 0 = <span th:text="10 % 2 == 0"></span></li>
        </ul>
    </li> 
    <li>비교 연산
        <ul>
            <li>1 > 10 = <span th:text="1 &gt; 10"></span></li>
            <li>1 gt 10 = <span th:text="1 gt 10"></span></li>
            <li>1 >= 10 = <span th:text="1 >= 10"></span></li>
            <li>1 ge 10 = <span th:text="1 ge 10"></span></li>
            <li>1 == 10 = <span th:text="1 == 10"></span></li>
            <li>1 != 10 = <span th:text="1 != 10"></span></li>
        </ul>
    </li>
    <li>조건식
        <ul>
            <li>(10 % 2 == 0)? '짝수':'홀수' = <span th:text="(10 % 2 == 0)? '짝수':'홀수'"></span></li>
        </ul>
    </li>
    <li>Elvis 연산자
        <ul>
            <li>${data}?: '데이터가 없습니다.' = <span th:text="${data}?: '데이터가 없습니다.'"></span></li>
            <li>${nullData}?: '데이터가 없습니다.' = <span th:text="${nullData}?: '데이터가 없습니다.'"></span></li>
        </ul>
    </li>
    <li>No-Operation
        <ul>
            <li>${data}?: _ = <span th:text="${data}?: _">데이터가 없습니다.</span></li>
            <li>${nullData}?: _ = <span th:text="${nullData}?: _">데이터가 없습니다.</span></li>
        </ul>
    </li>
</ul>
```

- 비교연산: HTML 엔티티를 사용해야 하는 부분을 주의
  -  ```> (gt), < (lt), >= (ge), <= (le), ! (not), == (eq), != (neq, ne)```
- 조건식: 자바의 조건식과 유사
- Elvis 연산자: 조건식의 결과가 null인 경우, 대체값이 사용되고, 변수가 null이 아닌 경우에는 변수의 값이 사용
- No-Operation: 
  - _ 기호는 무슨 값이든 상관 없음을 뜻하는 기호
  - Elvis 연산자 뒤에 _가 오면 해당 조건식의 결과가 null인 경우 마치 타임리프가 실행되지 않는 것처럼 동작
  - 이것을 잘 사용하면 HTML 내용 그대로 활용 가능

### 속성 값 설정

타임리프는 주로 HTML 태그에 th:* 속성을 지정하는 방식으로 동작

th:* 로 속성을 적용하면 기존 속성을 대체하고, 기존 속성이 없으면 새로 생성

#### 세팅

```java
public String attribute() {
    return "basic/attribute";
}
```

#### 사용

```html
<h1>속성 설정</h1>
  <input type="text" name="mock" th:name="userA" />
<h1>속성 추가</h1>
- th:attrappend = <input type="text" class="text" th:attrappend="class=' large'" /><br/>
- th:attrprepend = <input type="text" class="text" th:attrprepend="class='large '" /><br/>
- th:classappend = <input type="text" class="text" th:classappend="large" / ><br/>
<h1>checked 처리</h1>
- checked o <input type="checkbox" name="active" th:checked="true" /><br/>
- checked x <input type="checkbox" name="active" th:checked="false" /><br/> 
- checked=false <input type="checkbox" name="active" checked="false" /><br/>
```

- 속성 설정
  - th:*속성을 지정하면 타임리프는 기존 속성을 th:*로 지정한 속성으로 대체, 기존 속성이 없다면 새로 생성
  - ```html
    <input type="text" name="mock" th:name="userA" />
    ```
  - → 타임리프 렌더링 후 
  - ```html
    <input type="text" name="userA" />
    ```

- 속성 추가
  - th:attrappend : 속성 값의 뒤에 값을 추가
  - th:attrprepend : 속성 값의 앞에 값을 추가
  - th:classappend : class 속성에 자연스럽게 추가

- checked 처리
  - ```html
  <input type="checkbox" name="active" checked="false" />
  ```
  - HTML에서 checked속성은 checked속성값과 상관없이 checked 속성만 있어도 체크
  - 타임리프의 th:checked는 값이 false인 경우 checked속성 자체를 제거
  - ```html
    <input type="checkbox" name="active" th:checked="false" />
    ```
  - → 타임리프 렌더링 후: 
  - ```html
    <input type="checkbox" name="active" />
    ```

### 반복

타임리프에서 반복은 th:each 를 사용

추가로 반복에서 사용할 수 있는 여러 상태 값을 지원

#### 세팅

```java
public String each(Model model) {
    addUsers(model);
    return "basic/each";
}
private void addUsers(Model model) {
    List<User> list = new ArrayList<>();
    list.add(new User("userA", 10));
    list.add(new User("userB", 20));
    list.add(new User("userC", 30));
    model.addAttribute("users", list);
}
```

#### 사용

```html
<h1>기본 테이블</h1>
<table border="1">
    <tr>
        <th>username</th>
        <th>age</th>
    </tr>
    <tr th:each="user : ${users}">
        <td th:text="${user.username}">username</td>
        <td th:text="${user.age}">0</td>
    </tr>
</table>
<h1>반복 상태 유지</h1>
<table border="1">
    <tr>
        <th>count</th>
        <th>username</th>
        <th>age</th>
        <th>etc</th>
    </tr>
    <tr th:each="user, userStat : ${users}">
        <td th:text="${userStat.count}">username</td>
        <td th:text="${user.username}">username</td>
        <td th:text="${user.age}">0</td>
        <td>
            index = <span th:text="${userStat.index}"></span>
            count = <span th:text="${userStat.count}"></span>
            size = <span th:text="${userStat.size}"></span>
            even? = <span th:text="${userStat.even}"></span>
            odd? = <span th:text="${userStat.odd}"></span>
            first? = <span th:text="${userStat.first}"></span>
            last? = <span th:text="${userStat.last}"></span>
            current = <span th:text="${userStat.current}"></span>
        </td>
    </tr>
</table>
```

- 반복 기능
  - ```html
    <tr th:each="user : ${users}">
    ```
  - 반복시 오른쪽 컬렉션( ${users} )의 값을 하나씩 꺼내서 왼쪽 변수( user )에 담아서 태그를 반복 실행
  - th:each는 List뿐만 아니라 배열, java.util.Iterable, java.util.Enumeration을 구현한 모든 객체를 반복에 사용할 수 있고, Map도 사용할 수 있는데 이 경우 변수에 담기는 값은 Map.Entry

- 반복 상태 유지
  - ```html
    <tr th:each="user, userStat : ${users}">
    ```
  - 반복의 두번째 파라미터를 설정해서 반복 상태 확인 가능
  - 두번째 파라미터는 생략 가능한데, 생략하면 지정한 변수명( user ) + Stat
  - 여기서는 user + Stat = userStat 이므로 생략 가능

- 반복 상태 유지 기능
  - index : 0부터 시작하는 값
  - count : 1부터 시작하는 값
  - size : 전체 사이즈
  - even, odd : 홀수, 짝수 여부( boolean ) 
  - first, last : 처음, 마지막 여부( boolean )
  - current : 현재 객체

### 조건부 평가

타임리프의 조건식

if, unless(if의 반대)

#### 세팅

```java
public String condition(Model model) {
    addUsers(model);
    return "basic/condition";
}
```

#### 사용

```html
<h1>if, unless</h1>
<table border="1">
    <tr>
        <th>count</th>
        <th>username</th>
        <th>age</th>
    </tr>
    <tr th:each="user, userStat : ${users}">
        <td th:text="${userStat.count}">1</td>
        <td th:text="${user.username}">username</td>
        <td>
            <span th:text="${user.age}">0</span>
            <span th:text="'미성년자'" th:if="${user.age lt 20}"></span>
            <span th:text="'미성년자'" th:unless="${user.age ge 20}"></span>
        </td>
    </tr>
</table>
<h1>switch</h1>
<table border="1">
    <tr>
        <th>count</th>
        <th>username</th>
        <th>age</th>
    </tr>
    <tr th:each="user, userStat : ${users}">
        <td th:text="${userStat.count}">1</td>
        <td th:text="${user.username}">username</td>
        <td th:switch="${user.age}">
            <span th:case="10">10살</span>
            <span th:case="20">20살</span>
            <span th:case="*">기타</span>
        </td>
    </tr>
</table>
```

- if, unless
  - 타임리프는 해당 조건이 맞지 않으면 태그 자체를 렌더링하지 않음
  - 만약 다음 조건이 false 인 경우 `<span>...<span>` 부분 자체가 렌더링 되지 않고 삭제 
  - ```html
    <span th:text="'미성년자'" th:if="${user.age lt 20}"></span>
    ```
- switch
  - `*` 은 만족하는 조건이 없을 때 사용하는 디폴트

### 주석



#### 세팅

```java
public String comments(Model model) {
    model.addAttribute("data", "Spring!");
    return "basic/comments";
}
```

#### 사용

```html
<h1>예시</h1>
<span th:text="${data}">html data</span>

<h1>1. 표준 HTML 주석</h1>
<!--
<span th:text="${data}">html data</span> 
-->

<h1>2. 타임리프 파서 주석</h1> 
<!--/* [[${data}]] */-->

<!--/*-->
  <span th:text="${data}">html data</span>
<!--*/-->

<h1>3. 타임리프 프로토타입 주석</h1>
<!--/*/
<span th:text="${data}">html data</span>
/*/-->
```

#### 결과

```html
<h1>예시</h1>
<span>Spring!</span>

<h1>1. 표준 HTML 주석</h1>
<!--
<span th:text="${data}">html data</span>
-->

<h1>2. 타임리프 파서 주석</h1>

<h1>3. 타임리프 프로토타입 주석</h1>
<span>Spring!</span>
```

1. 표준 HTML 주석
- 자바스크립트의 표준 HTML 주석은 타임리프가 렌더링 하지 않고, 그대로 남겨둠
2. 타임리프 파서 주석
- 타임리프 파서 주석은 타임리프의 진짜 주석(렌더링에서 주석 부분을 제거)
3. 타임리프 프로토타입 주석
- HTML 파일을 웹 브라우저에서 그대로 열어보면 HTML 주석이기 때문에 이 부분이 웹 브라우저가 렌더링하지 않음
- 타임리프 렌더링을 거치면 이 부분이 정상 렌더링
- 쉽게 이야기해서 HTML 파일을 그대로 열어보면 주석처리, 타임리프를 렌더링 한 경우에만 보임

### 블록

`<th:block>` 은 HTML 태그가 아닌 타임리프의 유일한 자체 태그

#### 세팅

```java
public String block(Model model) {
    addUsers(model);
    return "basic/block";
}
```

#### 사용

```html
<th:block th:each="user : ${users}">
    <div>
        사용자 이름1 <span th:text="${user.username}"></span>
        사용자 나이1 <span th:text="${user.age}"></span> </div>
    <div>
        요약 <span th:text="${user.username} + ' / ' + ${user.age}"></span>
    </div>
</th:block>
```

#### 결과

```html
<div>
사용자 이름1 <span>userA</span> 
사용자 나이1 <span>10</span>
</div>
<div>
요약 <span>userA / 10</span>
</div>

<div>
사용자 이름1 <span>userB</span>
사용자 나이1 <span>20</span>
</div>
<div>
요약 <span>userB / 20</span>
</div>

<div>
사용자 이름1 <span>userC</span>
사용자 나이1 <span>30</span>
</div>
<div>
요약 <span>userC / 30</span>
</div>
```

타임리프의 특성상 HTML 태그안에 속성으로 기능을 정의해서 사용

위 예처럼 이렇게 사용하기 애매한 경우에 사용

`<th:block>`은 렌더링시 제거

### 자바스크립트 인라인

타임리프는 자바스크립트에서 타임리프를 편리하게 사용할 수 있는 자바스크립트 인라인 기능을 제공

```html
<script th:inline="javascript">
```

#### 세팅

```java
public String javascript(Model model) {
    model.addAttribute("user", new User("userA", 10));
    addUsers(model);
    return "basic/javascript";
}
```

#### 사용

```html
<!-- 자바스크립트 인라인 사용 전 --> 
<script>
    var username = [[${user.username}]];
    var age = [[${user.age}]];

    //자바스크립트 내추럴 템플릿
    var username2 = /*[[${user.username}]]*/ "test username";

    //객체
    var user = [[${user}]];
</script>

<!-- 자바스크립트 인라인 사용 후 --> 
<script th:inline="javascript">
    var username = [[${user.username}]];
    var age = [[${user.age}]];

    //자바스크립트 내추럴 템플릿
    var username2 = /*[[${user.username}]]*/ "test username";

    //객체
    var user = [[${user}]];
</script>
```

#### 자바스크립트 인라인 사용 전

```html
<script>
    var username = userA;
    var age = 10;

    //자바스크립트 내추럴 템플릿
    var username2 = /*userA*/ "test username";

    //객체
    var user = BasicController.User(username=userA, age=10);
</script>
```

#### 자바스크립트 인라인 사용 후

```html
<script>
    var username = "userA";
    var age = 10;

    //자바스크립트 내추럴 템플릿
    var username2 = "userA";

    //객체
    var user = {"username":"userA","age":10};
</script>
```

#### 자바스크립트 인라인을 사용하지 않은 경우 문제점들

##### 텍스트 렌더링

- `var username = [[${user.username}]];`
  - 인라인 사용 전 → `var username = userA;`
  - 인라인 사용 후 → `var username = "userA";`

인라인 사용 전 렌더링 결과를 보면 userA 라는 변수 이름이 그대로 남아있다. 

타임리프 입장에서는 정확하게 렌더링 한 것이지만 아마 개발자가 기대한 것은 다음과 같은 "userA"라는 문자일 것이다. 

결과적으로 userA가 변수명으로 사용되어서 자바스크립트 오류가 발생한다. 

다음으로 나오는 숫자 age의 경우에는 `"`가 필요 없기 때문에 정상 렌더링 된다.

인라인 사용 후 렌더링 결과를 보면 문자 타입인 경우 "를 포함해준다. 

추가로 자바스크립트에서 문제가 될 수 있는 문자가 포함되어 있으면 이스케이프 처리도 해준다. 예) `"` → `\"`

##### 자바스크립트 내추럴 템플릿

타임리프는 HTML 파일을 직접 열어도 동작하는 내추럴 템플릿 기능을 제공한다. 

자바스크립트 인라인 기능을 사용하면 주석을 활용해서 이 기능을 사용할 수 있다.

- `var username2 = /*[[${user.username}]]*/ "test username";`
  - 인라인 사용 전 → `var username2 = /*userA*/ "test username";`
  - 인라인 사용 후 → `var username2 = "userA";`

인라인 사용 전 결과를 보면 정말 순수하게 그대로 해석을 해버렸다. 

따라서 내추럴 템플릿 기능이 동작하지 않고, 심지어 렌더링 내용이 주석처리 되어 버린다.

인라인 사용 후 결과를 보면 주석 부분이 제거되고, 기대한 "userA"가 정확하게 적용된다.

##### 객체

타임리프의 자바스크립트 인라인 기능을 사용하면 객체를 JSON으로 자동으로 변환해준다.

- var user = [[${user}]];
  - 인라인 사용 전 → `var user = BasicController.User(username=userA, age=10);`
  - 인라인 사용 후 → `var user = {"username":"userA","age":10};`

인라인 사용 전은 객체의 toString()이 호출된 값이다. 

인라인 사용 후는 객체를 JSON으로 변환해준다.

##### 자바스크립트 인라인 each

자바스크립트 인라인은 each를 지원하는데, 다음과 같이 사용

```html
<!-- 자바스크립트 인라인 each -->
<script th:inline="javascript">
    [# th:each="user, stat : ${users}"]
    var user[[${stat.count}]] = [[${user}]];
    [/]
</script>
```

##### 자바스크립트 인라인 each 결과

```html
<script>
    var user1 = {"username":"userA","age":10};
    var user2 = {"username":"userB","age":20};
    var user3 = {"username":"userC","age":30};
</script>
```

### 템플릿 조각

웹 페이지를 개발할 때는 상단 영역이나 하단 영역, 좌측 카테고리 등등 여러 페이지에서 함께 사용하는 영역 등의 공통 영역들이 많음

이런 부분을 코드를 복사해서 사용한다면 변경시 여러 페이지를 다 수정해야 하므로 상당히 비효율적

타임리프는 이런 문제를 해결하기 위해 템플릿 조각과 레이아웃 기능을 지원

#### 세팅

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
@Controller
@RequestMapping("/template")
public class TemplateController {
    @GetMapping("/fragment")
    public String template() {
        return "template/fragment/fragmentMain";
    }
}
```

#### 사용

footer.html
```html
<footer th:fragment="copy"> 
    footer 자리입니다.
</footer>

<footer th:fragment="copyParam (param1, param2)">
    <p>parameter 자리입니다.</p>
    <p th:text="${param1}"></p>
    <p th:text="${param2}"></p>
</footer>
```

th:fragment가 있는 태그는 다른곳에 포함되는 코드 조각

fragmentMain.html
```html
<h1>부분 포함</h1>
<h2>부분 포함 insert</h2>
<div th:insert="~{template/fragment/footer :: copy}"></div>
<h2>부분 포함 replace</h2>
<div th:replace="~{template/fragment/footer :: copy}"></div>
<h2>부분 포함 단순 표현식</h2>
<div th:replace="template/fragment/footer :: copy"></div>
<h1>파라미터 사용</h1>
<div th:replace="~{template/fragment/footer :: copyParam ('데이터1', '데이터 2')}"></div>
```

template/fragment/footer :: copy : template/fragment/footer.html 템플릿에 있는 th:fragment="copy"라는 부분을 템플릿 조각으로 가져와서 사용한다는 의미

- th:insert를 사용하면 현재 태그( div ) 내부에 추가
- th:replace를 사용하면 현재 태그( div )를 대체
- ~{...}를 사용하는 것이 원칙이지만 템플릿 조각을 사용하는 코드가 단순하면 생략 가능
- 파라미터를 전달해서 조각을 동적 렌더링 가능 

### 템플릿 레이아웃

이전에는 일부 코드 조각을 가지고와서 사용했다면, 이번에는 개념을 더 확장해서 코드 조각을 레이아웃에 넘겨서 사용

예를 들어 `<head>`에 공통으로 사용하는 css, javascript같은 정보들이 있는데, 이러한 공통 정보들을 한 곳에 모아두고, 공통으로 사용하지만, 각 페이지마다 필요한 정보를 더 추가해서 사용하고 싶다면 다음과 같이 사용

#### 세팅

```java
public String layout() {
    return "template/layout/layoutMain";
}
```

#### 사용

base.html
```html
<head th:fragment="common_header(title,links)">
    <title th:replace="${title}">레이아웃 타이틀</title>

    <!-- 공통 -->
    <link rel="stylesheet" type="text/css" media="all" th:href="@{/css/awesomeapp.css}">
    <link rel="shortcut icon" th:href="@{/images/favicon.ico}">
    <script type="text/javascript" th:src="@{/sh/scripts/codebase.js}"></script>

    <!-- 추가 -->
    <th:block th:replace="${links}" />
</head>
```

layoutMain.html
```html
<head th:replace="template/layout/base :: common_header(~{::title},~{::link})">
    <title>메인 타이틀</title>
    <link rel="stylesheet" th:href="@{/css/bootstrap.min.css}">
    <link rel="stylesheet" th:href="@{/themes/smoothness/jquery-ui.css}">
</head>
```

#### 결과

```html
<head>
<title>메인 타이틀</title>

<!-- 공통 -->
<link rel="stylesheet" type="text/css" media="all" href="/css/awesomeapp.css">
<link rel="shortcut icon" href="/images/favicon.ico">
<script type="text/javascript" src="/sh/scripts/codebase.js"></script>

<!-- 추가 -->
<link rel="stylesheet" href="/css/bootstrap.min.css">
<link rel="stylesheet" href="/themes/smoothness/jquery-ui.css">
</head>
```

common_header(~{::title},~{::link})이 부분이 핵심
::title은 현재 페이지의 title 태그들을 전달
::link는 현재 페이지의 link 태그들을 전달

결과를 보면 메인 타이틀이 전달한 부분으로 교체

공통 부분은 그대로 유지되고, 추가 부분에 전달한 <link>들이 포함된 것을 확인할 수 있음
이 방식은 사실 앞서 배운 코드 조각을 조금 더 적극적으로 사용하는 방식
레이아웃을 두고, 그 레이아웃에 필요한 코드 조각을 전달해서 완성

<br>

## 정리

Thymeleaf는 HTML과 XML의 문법을 따르기 때문에 난이도가 낮고 자연스럽게 뷰와 로직을 분리할 수 있어 코드의 유지 보수와 수정이 용이

라이브러리로도 제공되기 때문에 작성한 템플릿을 JAR 파일로 포함하여 배포할 수도 있고, 렌더링 전에도 미리 HTML로 확인할 수 있기 때문에 프론트엔드 개발자와 협업이 수월

[🔗 공식 사이트](https://www.thymeleaf.org/){:target="_blank"}

[🔗 기본 기능](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html){:target="_blank"}

[🔗 스프링 통합](https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html){:target="_blank"}

<br>
