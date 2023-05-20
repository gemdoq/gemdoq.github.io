---
layout: single
title: "Spring에서 간단한 예외와 복잡한 예외의 비교"
date: 2023-04-01 10:13:25 +0900
categories: java
tags: [Simple exception, Complex exception, Throw new exception]
typora-root-url: ../
---

## Spring에서 간단한 예외와 복잡한 예외의 비교
> - 문제
> - throw new exception()
> - new exception()
> - 분석
> - 결론

<br>

## 문제

프로젝트 중 DB에 유저 정보를 생성하기 위해 DAO(UserRepository.java)와 서비스 레이어(UserService.java), api(UserRestController.java)를 다음과 같이 생성

UserRepository.java
```java
public interface UserRepository extends JpaRepository<UserEntity, UUID> {
    Optional<UserEntity> findByEmailAddress(String emailAddress);
}
```

UserService.java
```java
@Service
@AllArgsConstructor
public class UserService {
    private final UserRepository userRepository;

    public UUID createUser(UserCreateRequest req) {
        duplicateCheck(req.getEmailAddress());
        UserEntity reqUser = UserEntity.toEntity(req);
        UUID savedUserId = userRepository.save(reqUser).getId();
        return savedUserId;
    }

    private void duplicateCheck(String emailAddress) {
        userRepository.findByEmailAddress(emailAddress).ifPresent(userEntity -> { throw new AppException(ErrorCode.DUPLICATED_USER);});
    }
}
```

UserRestController.java
```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
public class UserRestController {
    private final UserService userService;

    @PostMapping("")
    public String userCreate(@RequestBody UserCreateRequest req) {
        UUID userId = userService.createUser(req);
        return "유저 생성 성공, UUID : " + userId;
    }
}
```

UserSerivce.java에서 duplicateCheck()를 통해 회원 중복을 체크하고 만약 중복일 경우, `throw new AppException(ErrorCode.DUPLICATED_USER)`으로 다음과 같은 예외 객체 AppException을 생성 

AppException.java
```java
@Getter
@AllArgsConstructor
public class AppException extends RuntimeException {
    private ErrorCode errorCode;
    private String appExceptionMessage;

    public AppException(ErrorCode errorCode) {
        this.errorCode = errorCode;
        this.appExceptionMessage = errorCode.getErrorCodeMessage();
    }
}
```

그리고 전역 예외 처리를 할 수 있게끔 ResponseEntityExceptionHandler를 상속받는 ExceptionManager 클래스를 만들고 @RestControllerAdvice 애노테이션을 붙여서 커스텀 예외 처리 내용을 정의함

문제는 유저 중복으로 인한 예외가 발생하게끔 테스트를 시도했을 때, ExceptionManager 클래스의 ResponseEntity의 body에 throw new exception()로 생성한 예외 객체가 들어갈 때의 ResponseBody 내용과 new exception()로 생성한 예외 객체가 들어갈 때의 ResponseBody 내용이 다름

UserService 클래스 내부의 duplicateCheck() 메서드 내부에서 throw new exception()로 생성한 예외 객체가 들어갈 때의 ResponseBody 내용은 간단하게 AppException.java에 새롭게 정의한 필드값만 json형식으로 반환

반면, ExceptionManager 클래스 내부의 appException() 메서드 내부에서 새롭게 AppException appExceptionErrorCode = new AppException()로 생성한 예외 객체가 들어간 ResponseBody 내용은 AppException.java에 새롭게 정의한 필드값과 함께 AppException 클래스가 상속받는 RuntimeException 클래스의 상위 클래스인 Throwable의 필드값(cause, stackTrace, message, suppressed, localizedMessage)도 함께 json형식으로 반환

분명 두 객체는 모두 AppException.class 객체인데, 왜 ResponseBody의 내용(Json형식)은 다른지 궁금해서 찾아봤으나, 납득가는 명쾌한 설명이 나오지 않음

<br>

## throw new exception()로 생성한 예외 객체

아래 appException()는 ResponseEntity의 body로 `UserService.java`에서 `throw new exception()`로 생성한 `AppException e`를 매개 변수로 받음

ExceptionManager.java
```java
@RestControllerAdvice
public class ExceptionManager extends ResponseEntityExceptionHandler {
    @ExceptionHandler(AppException.class)
    private ResponseEntity<?> appException(AppException e) {
        return ResponseEntity.status(e.getErrorCode().getHttpStatus()).body(Response.error(e.getErrorCode()));
    }
}
```

그리고 유저 중복으로 인한 예외가 발생하게끔 테스트를 시도한 결과, ResponseBody

ResponseBody
```json
{
    "resultCode": "ERROR",
    "result": "DUPLICATED_USER"
}
```

AppException 클래스에 정의했던 필드값 resultCode와 result가 포함된 `단순한 예외` 반환

<br>

## new exception()로 생성한 예외 객체

아래 appException()는 ResponseEntity의 body로 `appException()`에서 `new exception()`로 생성한 `AppException appExceptionErrorCode`를 매개 변수로 받음

ExceptionManager.java
```java
@RestControllerAdvice
public class ExceptionManager extends ResponseEntityExceptionHandler {
    @ExceptionHandler(AppException.class)
    private ResponseEntity<?> appException(AppException e) {
        AppException appExceptionErrorCode = new AppException(e.getErrorCode());
        return ResponseEntity.status(e.getErrorCode().getHttpStatus()).body(Response.error(appExceptionErrorCode));
    }
}
```

그리고 유저 중복으로 인한 예외가 발생하게끔 테스트를 시도한 결과, ResponseBody

ResponseBody
```json
{
    "resultCode": "ERROR",
    "result": {
        "cause": null,
        "stackTrace": [
            ...
        ],
        "errorCode": "DUPLICATED_USER",
        "appExceptionMessage": "이미 등록된 유저입니다.",
        "message": null,
        "suppressed": [],
        "localizedMessage": null
    }
}
```

AppException 클래스에 정의했던 필드값 resultCode와 result를 포함하여, Throwable의 필드값인 cause, stackTrace, message, suppressed, localizedMessage 까지 포함된 `복잡한 예외` 객체 반환

<br>

## 분석

첫 번째 경우인 `throw new exception()`로 생성한 예외 객체는 `UserService.java`에서 `duplicateCheck()` 메서드 내에서 직접 생성

이 예외 객체는 생성될 때 `ErrorCode`와 `appExceptionMessage` 필드만 설정

그리고 `ExceptionManager`에서 이 예외를 처리할 때, 해당 필드값들을 이용하여 응답을 생성

이 경우 `ErrorCode`와 `appExceptionMessage` 필드만 응답에 포함되며, 이는 단순히 예외를 나타내기 위한 최소한의 정보만을 포함

두 번째 경우인 `new exception()`로 생성한 예외 객체는 `ExceptionManager` 내에서 생성

`AppException e`를 매개 변수로 받아 새로운 `AppException appExceptionErrorCode` 객체를 생성

이 경우 `ErrorCode`와 `appExceptionMessage` 필드뿐만 아니라, `AppException` 클래스가 `RuntimeException`을 상속받고 있기 때문에 `Throwable`의 필드값들(cause, stackTrace, message, suppressed, localizedMessage)도 상속

따라서 이 예외 객체를 응답으로 반환할 때는 `AppException` 객체의 모든 필드값이 함께 반환

첫 번째 경우에서는 `throw new exception()`로 생성한 예외 객체는 단순히 예외를 나타내기 위한 목적이므로 필요한 최소한의 정보만을 포함

반면에 두 번째 경우에서는 `ExceptionManager`에서 새로운 예외 객체를 생성하기 때문에 해당 객체는 예외 발생 시의 모든 정보를 포함

이러한 차이는 예외 객체를 생성하는 시점과 응답을 생성하는 시점에서의 차이에서 기인

첫 번째 경우에서는 예외가 발생하는 시점에 필요한 정보만을 담은 예외 객체가 생성되고, 두 번째 경우에서는 예외를 처리하고 응답을 생성하는 시점에 해당 정보를 포함한 예외 객체가 생성

<br>

## 결론

### throw new exception()

첫 번째 경우인 `throw new exception()`로 생성한 예외 객체를 사용하는 방식은 필요한 최소한의 정보만을 포함하므로, 응답은 간결하고 이해하기 쉬움

오류 코드와 메시지만을 포함하고 있기 때문에 클라이언트는 이를 기반으로 오류 처리 가능

또한, 응답의 크기가 작아지기 때문에 네트워크 트래픽과 전송 시간 절약 가능

### new exception()

반면에 두 번째 경우인 `new exception()`로 생성한 예외 객체를 사용하는 방식은 오류 발생 시의 모든 정보를 포함

따라서 클라이언트는 오류에 대한 자세한 정보를 얻을 수 있지만, 불필요한 정보도 함께 전달되므로 응답의 크기가 크고 분석하기 어려움

또한, 네트워크 트래픽과 전송 시간이 증가

### 정리

예외 발생 시 클라이언트는 오류에 대한 정보를 이해하고 처리하기 위해 응답을 분석해야 함

따라서 클라이언트의 입장에서는 필요한 최소한의 정보만을 포함한 단순하고 명확한 응답을 선호

<br>
