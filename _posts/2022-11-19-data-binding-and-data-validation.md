---
layout: single
title: "Data Binding, Data Validation"
date: 2022-11-19 10:33:19 +0900
categories: knowledge
tags: [Request Parameter, Data Binding, Data Validation]
typora-root-url: ../
---


## Request Parameter, Data Binding, Data Validation
> - Request Parameter
> - Data Binding
> - Data Validation

<br>

## Request Parameter

![springwebform-requestparam](/images/2022-11-19-data-binding-and-data-validation/springwebform-requestparam.png){: width="560"}

서버의 특정 동작에 필요한 HTTP Request message에 포함된 정보

두 가지 전달되는 방식 : Query String, HTTP Body

### Query String Parameter

![springwebform-requestparam-get](/images/2022-11-19-data-binding-and-data-validation/springwebform-requestparam-get.png){: width="560"}

도메인의 절대 경로 끝에 `?와 쿼리 문자열`이 붙는 형태(Origin format)로 전달

주로 조회(GET)에 사용되는 방식

### HTTP Body Parameter

![springwebform-requestparam-post](/images/2022-11-19-data-binding-and-data-validation/springwebform-requestparam-post.png){: width="560"}

HTTP Request의 Body에 담긴 형태로 전달

주로 생성(POST)이나 수정(PUT)에 사용되는 방식

<br>

## Data Binding

![springwebform-databinding](/images/2022-11-19-data-binding-and-data-validation/springwebform-databinding.png){: width="560"}

Request Parameters가 Form Bean(=command object)과 결합

### Data Binding 과정

![springwebform-databinding-process](/images/2022-11-19-data-binding-and-data-validation/springwebform-databinding-process.png){: width="560"}

1. Form bean(Offer 객체)를 생성
2. Request Parameter으로부터 Form Bean에 내용이 자동으로 binding
   (Controller의 method 인자에 Form Bean을 선언되어 있고, Form Bean에 setter가 있어야 binding 가능)
3. Controller에서 Form Bean을 자동으로 Model에 attribute
4. Controller에서 Form Bean을 Model에 넣어 View에 전달
5. View는 Form Bean 내용에 접근할 수 있고 이 내용을 rendering

```html
<html> 
<head> 
	<title>Thanks</title> 
</head> 
<body> 
    Hi, ${offer.name}.<br/>
    You have successfully registered.
</body> 
</html>
```

### Naive Solution

- @RequestParam 어노테이션 이용, method 인자에 Request Parameter을 binding
- query string의 key 이름이 동일해야 값을 호출 가능

```java
@RequestMapping("/docreate")
public String doCreate(@RequestParam("name") String name,
                        @RequestParam("email") String email,
                        Model model) {

       // we manually populate the Offer object with 
       // the data coming from the user
       Offer offer = new Offer();

       offer.setName(name);
       offer.setEmail(email);
       ...
}
```

### Spring Data Binding

- Spring이 자동으로 Request Paraters를 객체에 binding
- method 인자에 form bean을 선언

```java
@RequestMapping(value="/docreate", method=RequestMethod.POST)
public String doCreate(Offer offer) {
    // offer object will be automatically populated 
    // with request parameters
}
```

<br>

## Data Validation

사용자가 실수로 잘못된 정보를 입력했을 때 잘못된 양식임을 알려주기 위해 사용

### Hibernate Validator

사용자의 오류를 감지하기 위해 form bean에 캡슐화된 form data의 유효성을 검사

Bean Validation API(JSR-303)는 JavaBean 유효성 검사를 위한 API를 정의하는 명세서

- 이 명세의 구현체가 Hibernate Validator(라이브러리)
- 검증 제약 조건을 bean properties에 어노테이션을 달아 설정
- @NotNull, [@Pattern](http://www.rubular.com/), @Size, @Email 등
- `hibernate-validator` 의존성 추가

```java
public class Offer {
    private int id;

    @Size(min=5, max=100)
    @Pattern(regexp="^[A-Z]{1}[a-z]+$") // 정규표현(Regular Expression)
    private String name;

    @NotEmpty
    @Email
    private String email;
}
```

### Message Interpolation

- Bean 유효성 검증 제약 조건을 만족하지 못하면, 오류 메시지 생성
- 메시지 속성을 통해 각 속성의 message descriptor를 정의

```java
public class Offer {

   private int id;

   @Size(min=3, max=100, message="Name must be between 3 and 100 characters")
   private String name;

   @Email(message="please provide a valid email address")
   @NotEmpty(message="the email address cannot be empty")
   private String email;

   @Size(min=5, max=100, message="Text must be between 5 and 100 characters")
   private String text;
 }
```

### Validating Object

- `@Valid` 어노테이션을 통해 Hibernate가 자동으로 유효성 검사
- Controller method의 인자 중 검증할 객체 인자에 해당 어노테이션 사용
- `@Valid` 어노테이션은 객체의 유효성을 먼저 확인한 다음 모델에 객체 추가
- BindingResult Object
  * 유효성 검사 과정의 결과를 나타냄
  * 필요한 경우에 controller method의 인자에 추가
  * form bean과 같이 model에 들어가서 View에 전달
  * 이를 통해 View는 error message를 출력
- `@Valid`에 의해 Spring이 자동으로 유효성을 검사 후, BindingResult 객체를 자동으로 Model에 attribute

```java
@RequestMapping(...)
public String doCreate(@Valid Offer offer, BindingResult result) { 
    if(result.hasErrors()) {
	List<ObjectError> errors = result.getAllErrors();

       for(ObjectError error:errors) {
    		System.out.println(error.getDefaultMessage());
       }
       return "createoffer";
    }
}
```

<br>