---
layout: single
title: "Spring Boot 리팩토링하기"
date: 2022-12-23 11:21:37 +0900
categories: java
tags: [Springboot, Refactoring]
typora-root-url: ../
---


## Spring Boot 리팩토링하기
> - Raw 타입은 사용하지 말자
> - Restful API는 자원과 메소드로 표현하자
> - 데이터를 주고 받을 때에는 DTO를 이용하자
> - final과 함께 생성자 주입을 사용하라
> - Controller는 최대한 가볍게 만들어라
> - 불변 객체를 사용하라
> - 유틸성 또는 상수형 클래스는 내부 생성자를 통해 객체 생성을 제한하자

<br>

## Raw 타입은 사용하지 말자

다음과 같이 Controller에서 데이터를 반환할 때 ResponseEntity를 Raw 타입으로 반환하고 있다.

```java
@RestController
@RequestMapping(value = "/error")
@Log4j2
public class ErrorController {

    @GetMapping(value = "/unauthorized")
    public ResponseEntity unauthorized() {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }

}
```
```java
@RequiredArgsConstructor
@RestController
@RequestMapping(value = "/user")
@Log4j2
public class UserController {

    private final BCryptPasswordEncoder passwordEncoder;
    private final UserService userService;

    @PostMapping(value = "signUp")
    public ResponseEntity signUp(@RequestBody User user) {
        user.setRole(UserRole.ROLE_USER);
        user.setPw(passwordEncoder.encode(user.getPw()));
        return userService.findByEmail(user.getEmail()).isPresent()
                ? ResponseEntity.badRequest().build()
                : ResponseEntity.ok(TokenUtils.generateJwtToken(userService.signUp(user)));
    }

    @GetMapping(value = "/findAll")
    public ResponseEntity findAll() {
        return ResponseEntity.ok(userService.findAll());
    }

}
```
Raw 타입을 사용하면 컴파일 시점에 문제를 잡지 못하다가 런타임 시점에 ClassCastException 에러가 발생할 수 있다. 예를 들어 다음과 같은 Integer만을 갖는 List에 String이 추가되어도 오류 없이 컴파일되고 실행된다. 그러다가 해당 데이터를 꺼내거나 연산을 할 때 즉, 런타임 시점에 문제(ClassCastException 에러)가 발생하게 된다.
```java
List list = new ArrayList<>();
list.add("문자열");
list.add(123);

int sum = 0;
for (Object num : list) {
    // 런타임 시점에 ClassCastException 발생
    sum += (int)num;
}
```
하지만 만약 Raw 타입이 아니라 타입 파라미터를 List<Integer>로 명시해준다면, 컴파일러가 리스트에 Integer만 넣어야 함을 인지하여 컴파일 시점에 오류를 잡아낼 수 있다. 왜냐하면 컴파일러는 컬렉션에서 원소를 꺼내는 모든 곳에 보이지 않는 형변환을 추가하여 절대 실패하지 않음을 보장하기 때문이다.

그렇기 때문에 위와 같은 Raw 타입의 반환을 피하고 반환할 데이터 타입을 명시해는 것이 좋다.
```java
@RestController
@RequestMapping(value = "/error")
@Log4j2
public class ErrorController {

    @GetMapping(value = "/unauthorized")
    public ResponseEntity<Void> unauthorized() {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }

}
```
```java
@RequiredArgsConstructor
@RestController
@RequestMapping(value = "/user")
@Log4j2
public class UserController {

    private final BCryptPasswordEncoder passwordEncoder;
    private final UserService userService;

    @PostMapping(value = "/signUp")
    public ResponseEntity<String> signUp(@RequestBody User user) {
        user.setRole(UserRole.ROLE_USER);
        user.setPw(passwordEncoder.encode(user.getPw()));
        return userService.findByEmail(user.getEmail()).isPresent()
                ? ResponseEntity.badRequest().build()
                : ResponseEntity.ok(TokenUtils.generateJwtToken(userService.signUp(user)));
    }

    @GetMapping(value = "/findAll")
    public ResponseEntity<List<User>> findAll() {
        return ResponseEntity.ok(userService.findAll());
    }

}
```
<br>

## Restful API는 자원과 메소드로 표현하자

위의 예제에서는 Controller가 사용자 목록 조회 API(/user/findAll) 를 제공하고 있다.

```java
@RequiredArgsConstructor
@RestController
@RequestMapping(value = "/user")
@Log4j2
public class UserController {

    ... 생략

    @GetMapping(value = "/findAll")
    public ResponseEntity<List<User>> findAll() {
        return ResponseEntity.ok(userService.findAll());
    }

}
```
기능적인 문제는 없지만 Restful API는 자원(URI)과 메소드(GET, POST 등) 으로 해당 요청을 표현해야 한다. 하지만 사용자 목록 조회 API의 경우, URI에서도 조회라는 기능을 설명하고 있고(find...) GET이라는 메소드도 조회라는 기능을 설명하고 있다. 그렇기 때문에 해당 API의 URI를 다음과 같이 명사의 형태로 바꿔 자원과 메소드로 표현하도록 하자.
```java
@RequiredArgsConstructor
@RestController
@RequestMapping(value = "/users")
@Log4j2
public class UserController {

    private final BCryptPasswordEncoder passwordEncoder;
    private final UserService userService;

    ... 생략

    @GetMapping
    public ResponseEntity<List<User>> findAll() {
        return ResponseEntity.ok(userService.findAll());
    }

}
```

<br>

## 데이터를 주고 받을 때에는 DTO를 이용하자



<br>

## final과 함께 생성자 주입을 사용하라

<br>

## Controller는 최대한 가볍게 만들어라

<br>

## 불변 객체를 사용하라

<br>

## 유틸성 또는 상수형 클래스는 내부 생성자를 통해 객체 생성을 제한하자

<br>
