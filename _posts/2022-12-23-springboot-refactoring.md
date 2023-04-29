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

Controller에서 데이터를 반환할 때 ResponseEntity를 Raw 타입으로 반환

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
Raw 타입을 사용하면 컴파일 시점에 문제를 잡지 못하다가 런타임 시점에 ClassCastException 에러 발생

예를 들어 다음과 같은 Integer만을 갖는 List에 String이 추가되어도 오류 없이 컴파일되고 실행

해당 데이터를 꺼내거나 연산을 할 때 즉, 런타임 시점에 ClassCastException 발생

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
하지만 Raw 타입이 아니라 타입 파라미터를 List<Integer>로 명시해준다면, 컴파일러가 리스트에 Integer만 넣어야 함을 인지하여 컴파일 시점에 오류 검출

Raw 타입의 반환을 지양하고, 반환할 데이터 타입을 명시
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

Restful API는 자원(URI)과 메소드(GET, POST 등) 으로 해당 요청을 표현

사용자 목록 조회 API의 경우, API의 URI를 명사의 형태로 바꿔 자원과 메소드로 표현

```java
@RequiredArgsConstructor
@RestController
@RequestMapping(value = "/users")
@Log4j2
public class UserController {

    private final BCryptPasswordEncoder passwordEncoder;
    private final UserService userService;

    @GetMapping
    public ResponseEntity<List<User>> findAll() {
        return ResponseEntity.ok(userService.findAll());
    }
}
```

<br>

## 데이터를 주고 받을 때에는 DTO를 이용하자

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
다음과 같은 이유로 Entity와 DTO를 분리하여 사용

1. 불필요한 코드 및 로직을 엔티티로부터 분리
2. 엔티티가 변경되어도 API 스펙 불변
3. 요청으로 넘어오는 파라미터를 직관적으로 확인가능, API의 유연성 확보

그 외에 함수의 파라미터가 너무 긴 경우에도 가독성이 떨어지므로, DTO를 사용

회원가입 DTO와 사용자 목록 반환 DTO를 생성

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
User 객체를 이제 직접 생성해주어야 하므로 lombok을 통해 builder 패턴을 추가
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
DTO를 반영하여 리팩토링
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

```java
@RequiredArgsConstructor
@Service
public class UserServiceImpl implements UserService {

    @NonNull
    private UserRepository userRepository;
}
```
객체가 변하지 않는 경우, 모두 final과 @RequiredArgsConstructor 이용

```java
@RequiredArgsConstructor
@Service
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;
}
```

<br>

## Controller는 최대한 가볍게 만들어라

Controller는 어떠한 데이터를 주고 받는지만을 명확하게 작성

```java
@RequiredArgsConstructor
@RestController
@RequestMapping(value = "/user")
@Log4j2
public class UserController {

    private final UserService userService;

    @PostMapping(value = "/signUp")
    public ResponseEntity<String> signUp(@RequestBody SignUpDTO signUpDTO) {
        return userService.findByEmail(signUpDTO.getEmail()).isPresent()
                ? ResponseEntity.badRequest().build()
                : ResponseEntity.ok(TokenUtils.generateJwtToken(userService.signUp(signUpDTO)));
    }
}
```
컨트롤러 단순화

비밀번호 암호화 같은 로직은 서비스 레이어로 이동

```java
@RequiredArgsConstructor
@Service
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;
    private final BCryptPasswordEncoder passwordEncoder;

    @Override
    public User signUp(SignUpDTO signUpDTO) {
        User user = User.builder()
                .email(signUpDTO.getEmail())
                .pw(passwordEncoder.encode(signUpDTO.getPw()))
                .role(UserRole.ROLE_USER)
                .build();

        return userRepository.save(user);
    }
}
```

<br>

## 불변 객체를 사용하라

변경가능성이 없는 객체라면 final로 선언하여 불변성을 확보

내 코드를 읽거나 수정하는 다른 누군가는 final로 선언된 객체를 안전하게 사용

```java
@RequiredArgsConstructor
@RestController
@RequestMapping(value = "/user")
@Log4j2
public class UserController {

    private final UserService userService;

    @PostMapping(value = "/signUp")
    public ResponseEntity<String> signUp(@RequestBody final SignUpDTO signUpDTO) {
        return userService.findByEmail(signUpDTO.getEmail()).isPresent()
                ? ResponseEntity.badRequest().build()
                : ResponseEntity.ok(TokenUtils.generateJwtToken(userService.signUp(signUpDTO)));
    }

    @GetMapping(value = "/list")
    public ResponseEntity<UserListResponseDTO> findAll() {
        final UserListResponseDTO userListResponseDTO = UserListResponseDTO.builder()
                .userList(userService.findAll()).build();

        return ResponseEntity.ok(userListResponseDTO);
    }
}
```
매개변수, 지역변수, 클래스 변수 등 변경 가능성이 없는 모든 변수들에는 final 사용

<br>

## 유틸성 또는 상수형 클래스는 내부 생성자를 통해 객체 생성을 제한하자

일반적인 유틸리티성 클래스라면 static 메소드를 제공하기 때문에 객체를 생성할 필요 없음

```java
@Log4j2
public final class TokenUtils {

    private static final String secretKey = "ThisIsA_SecretKeyForJwtExample";

    public static String generateJwtToken(User user) {
        JwtBuilder builder = Jwts.builder()
                .setSubject(user.getEmail())
                .setHeader(createHeader())
                .setClaims(createClaims(user))
                .setExpiration(createExpireDateForOneYear())
                .signWith(SignatureAlgorithm.HS256, createSigningKey());

        return builder.compact();
    }
}
```
유틸성 클래스를 객체를 생성하도록 만든 것은 불필요하게 코드를 열어두는 것이므로, 내부 생성자를 통해 이를 제한

롬복을 사용중이라면 NoArgsConstructor를 이용, 그렇지 않다면 직접 내부 생성자를 추가

누군가 객체를 생성하려고 할 때 컴파일 오류를 발생

```java
@Log4j2
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public final class TokenUtils {

    private static final String secretKey = "ThisIsA_SecretKeyForJwtExample";

    public static String generateJwtToken(User user) {
        JwtBuilder builder = Jwts.builder()
                .setSubject(user.getEmail())
                .setHeader(createHeader())
                .setClaims(createClaims(user))
                .setExpiration(createExpireDateForOneYear())
                .signWith(SignatureAlgorithm.HS256, createSigningKey());

        return builder.compact();
    }
}
```

<br>
