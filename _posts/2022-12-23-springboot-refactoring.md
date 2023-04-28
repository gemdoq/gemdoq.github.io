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

위의 예제에서는 데이터를 주고 받을때 엔티티를 사용하고 있다. 회원가입을 할 때는 User를 파라미터로 받고 있고, 사용자 목록을 조회할 때는 List<User>를 반환하고 있다.

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

    @GetMapping(value = "/list")
    public ResponseEntity<List<User>> findAll() {
        return ResponseEntity.ok(userService.findAll());
    }

}
```
지금은 간단한 프로젝트이기 때문에 User 객체에 @Valid와 @NotEmpty 같은 유효성 검사 코드나 패스워드 변경시 필요한 newPw와 같은 필드가 존재하지 않는다. 하지만 만약 프로젝트가 확장되어 User 객체와 같은 엔티티에 해당 코드들이 추가된다면 엔티티가 상당히 무겁고 복잡해지며 가독성이 떨어질 것이다. 그리고 만약 엔티티를 직접 반환한다면 필드명이 바뀌는 경우에 API의 스펙을 변경하게 되고, 해당 API를 사용중인 클라이언트에게 문제를 발생시킬 수 있다.

또한 지금은 사용자 목록을 반환하는 경우에는 List를 그대로 반환하고 있다. DTO를 사용하지 않고 있으므로, 만약 해당 API에 total count를 추가해달라는 요구사항이 생기는 경우에 상당히 유연성이 떨어진다. 또한 DTO의 이름을 SignUpDTO와 같이 작명한다면, 해당 요청을 통해 어떠한 파라미터를 받는지 직관적으로 짐작할 수 있다.

그렇기 때문에 우리는 다음과 같은 이유로 Entity와 DTO를 분리하여 사용해야 한다.
1. 불필요한 코드 및 로직을 엔티티로부터 분리할 수 있다.
2. 엔티티가 변경되어도 API 스펙이 변하지 않는다.
3. 요청으로 넘어오는 파라미터를 직관적으로 확인가능하며, API의 유연성을 확보할 수 있다.
그 외에 함수의 파라미터가 너무 긴 경우에도 가독성이 떨어지므로, DTO를 사용하는 것도 좋은 선택지이다.
위의 코드를 DTO로 변경하기 위해 우선 다음과 같은 회원가입 DTO와 사용자 목록 반환 DTO를 생성하도록 하자.
```java
@Getter
public class SignUpDTO {

    private String email;
    private String pw;

}
```
```java
@Getter
@Builder
public class UserListResponseDTO {

    private final List<User> userList;

}
```
또한 User 객체를 이제 직접 생성해주어야 하므로 lombok을 통해 builder 패턴을 추가해주도록 하자. 이에 맞게 User 클래스를 수정하면 다음과 같다.
```java
@Entity
@Table(name = "USER")
@Getter
@Builder
@NoArgsConstructor(force = true)
public class User extends Common implements Serializable {

    @Column(nullable = false, unique = true, length = 50)
    private final String email;

    @Setter
    @Column(nullable = false)
    private String pw;

    @Setter
    @Column(nullable = false, length = 50)
    @Enumerated(EnumType.STRING)
    private UserRole role;

}
```
그리고 UserController에 해당 DTO를 반영하여 다음과 같이 수정하도록 하자.
```java
@RequiredArgsConstructor
@RestController
@RequestMapping(value = "/user")
@Log4j2
public class UserController {

    private final BCryptPasswordEncoder passwordEncoder;
    private final UserService userService;

    @PostMapping(value = "/signUp")
    public ResponseEntity<String> signUp(@RequestBody SignUpDTO signUpDTO) {
        User user = User.builder()
                .email(signUpDTO.getEmail())
                .pw(passwordEncoder.encode(signUpDTO.getPw()))
                .role(UserRole.ROLE_USER)
                .build();

        return userService.findByEmail(user.getEmail()).isPresent()
                ? ResponseEntity.badRequest().build()
                : ResponseEntity.ok(TokenUtils.generateJwtToken(userService.signUp(user)));
    }

    @GetMapping(value = "/list")
    public ResponseEntity<UserListResponseDTO> findAll() {
        UserListResponseDTO userListResponseDTO = UserListResponseDTO.builder()
                .userList(userService.findAll()).build();

        return ResponseEntity.ok(userListResponseDTO);
    }

}
```

<br>

## final과 함께 생성자 주입을 사용하라

<br>

## Controller는 최대한 가볍게 만들어라

<br>

## 불변 객체를 사용하라

<br>

## 유틸성 또는 상수형 클래스는 내부 생성자를 통해 객체 생성을 제한하자

<br>
