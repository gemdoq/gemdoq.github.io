---
layout: single
title: "JPA에서 Pageable에 대해"
date: 2023-02-14 10:19:22 +0900
categories: java
tags: [Pageable]
typora-root-url: ../
---

## JPA에서 Pageable에 대해
> - 페이징이란
> - Spring Data JPA에서의 페이징과 정렬
> - 쿼리 메서드에서 페이징 사용하기
> - 반환 타입에 따른 페이징 결과
> - Spring Web MVC에서 더 편하게 페이징하기

<br>

## 페이징이란

게시판이나 댓글, 블로그를 개발할 때 페이징은 중요한 역할

페이징은 수많은 게시글과 같은 정보들을 페이지로 나눠 효과적으로 정보를 제공

![google](/images/2023-02-14-about-jpa-pageable/google.png){: width="560"}

과거 페이징을 구현하기 위해서 page 관련 쿼리를 파라미터로 받아서 직접 처리하는 방법이 있었으나, Spring Data JPA에서 페이징을 효과적으로 처리할 수 있는 방법을 제공

<br>

## Spring Data JPA에서의 페이징과 정렬

다음은 JpaRepository 인터페이스의 상속 다이어그램

![jparepo](/images/2023-02-14-about-jpa-pageable/jparepo.png){: width="560"}

JpaRepository의 부모 인터페이스인 PagingAndSortingRepository에서 페이징과 소팅이라는 기능을 제공

![pagingandsort](/images/2023-02-14-about-jpa-pageable/pagingandsort.png){: width="560"}

findAll() 메서드가 있고, 그 메서드의 반환 타입과 파라미터

- org.springframework.data.domain.Pageable : 페이징을 제공
- org.springframework.data.domain.Page : 페이징의 findAll()의 여러 반환 타입 중 하나

JpaRepository<>를 사용할 때, findAll() 메서드를 Pageable 인터페이스로 파라미터를 넘기면 페이징 사용 가능

### 예시

```java
@RestController
public class UserController {

    private final UserRepository userRepository;

    public UserController(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @GetMapping("/users")
    public Page<User> getAllUsers() {
        PageRequest pageRequest = PageRequest.of(0, 5);
        return userRepository.findAll(pageRequest);
    }

    @PostConstruct
    public void initializing() {
        for (int i = 0; i < 100; i++) {
            User user = User.builder()
                    .username("User " + i)
                    .address("Korea")
                    .age(i)
                    .build();
            userRepository.save(user);
        }
    }
}
```

getAllUsers() 메서드에 보면 PageRequest 객체가 존재

PageRequest 객체는 Pageable 인터페이스를 상속

![pagerequest](/images/2023-02-14-about-jpa-pageable/pagerequest.png){: width="560"}

정렬, offset, page와 같은 Paging을 위한 정보를 넘길 수 있음

추가로 DB에 데이터가 있으면 좋지만 없기 때문에, 테스트를 위해 100명 분의 더미 데이터를 생성하는 initializing() 메서드를 이용하여 페이징을 구현

```
"content":[
{
    "content": [
        {"id": 1, "username": "User 0", "address": "Korea", "age": 0},
        // 중간 생략
        {"id": 5, "username": "User 4", "address": "Korea", "age": 4}
    ],
    "pageable": {
        "sort": {
            "sorted": false, // 정렬 상태
            "unsorted": true,
            "empty": true
        },
        "pageSize": 5, // 한 페이지에서 나타내는 원소의 수 (게시글 수)
        "pageNumber": 0, // 페이지 번호 (0번 부터 시작)
        "offset": 0, // 해당 페이지에 첫 번째 원소의 수
        "paged": true,
        "unpaged": false
    },
    "totalPages": 20, // 페이지로 제공되는 총 페이지 수
    "totalElements": 100, // 모든 페이지에 존재하는 총 원소 수
    "last": false,
    "number": 0,
    "sort": {
        "sorted": false,
        "unsorted": true,
        "empty": true
    },
    "size": 5,
    "numberOfElements": 5,
    "first": true,
    "empty": false
}
]
```

content 아래에 있는 데이터들이 paging 과 관련된 정보들

<br>

## 쿼리 메서드에서 페이징 사용하기

다음은 주소로 사용자를 조회하는 쿼리 메서드이고, 두번째 파라미터는 pageable 

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByAddress(String address, Pageable pageable);
}
```

다음은 쿼리 메서드로 모든 사용자 정보를 페이지로 받아오는 컨트롤러

```java
@RestController
public class UserController {

    private final UserRepository userRepository;

    public UserController(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @GetMapping("/users")
    public Page<User> getAllUserWithPageByQueryMethod(@RequestParam("page") Integer page, @RequestParam("size") Integer size) {
        PageRequest pageRequest = PageRequest.of(page, size);
        return userRepository.findByAddress("Korea", pageRequest);
    }


    @PostConstruct
    public void initializing() {
        for (int i = 0; i < 100; i++) {
            User user = User.builder()
                    .username("User " + i)
                    .address("Korea")
                    .age(i)
                    .build();
            userRepository.save(user);
        }
    }
}
```

이렇게 만들고 http://localhost:8080/users/?page=3&size=4 로 요청을 보내면 다음과 같은 결과가 출력

```
{
  "content": [
    { "id": 13, "username": "User 12", "address": "Korea", "age": 12 },
    { "id": 14, "username": "User 13", "address": "Korea", "age": 13 },
    { "id": 15, "username": "User 14", "address": "Korea", "age": 14 },
    { "id": 16, "username": "User 15", "address": "Korea", "age": 15 }
  ],
  "pageable": {
    "sort": { "sorted": false, "unsorted": true, "empty": true },
    "pageNumber": 3,
    "pageSize": 4,
    "offset": 12,
    "paged": true,
    "unpaged": false
  },
  "totalPages": 25,
  "totalElements": 100,
  "last": false,
  "numberOfElements": 4,
  "number": 3,
  "sort": { "sorted": false, "unsorted": true, "empty": true },
  "size": 4,
  "first": false,
  "empty": false
}
```

<br>

## 반환 타입에 따른 페이징 결과

Spring Data JPA 에는 반환 타입에 따라서 각기 다른 결과를 제공

1. Page<T>
2. Slice<T>
3. List<T>

### 1. Page<T>

Page<T> 타입을 반환 타입으로 받게 된다면 offset과 totalPage 이용

Page<T> 는 일반적인 게시판 형태의 페이징에서 사용

중요한 정보는 총 페이지 수이기 때문에 총 페이지 수를 포함하여 반환

카운트 쿼리 자동 생성

### 2. Slice<T>

Slice<T> 타입을 반환 타입으로 받게 된다면 더보기 형태의 페이징에서 사용

```
{
  "content": [
    { "id": 13, "username": "User 12", "address": "Korea", "age": 12 },
    { "id": 14, "username": "User 13", "address": "Korea", "age": 13 },
    { "id": 15, "username": "User 14", "address": "Korea", "age": 14 },
    { "id": 16, "username": "User 15", "address": "Korea", "age": 15 }
  ],
  "pageable": {
    "sort": { "sorted": false, "unsorted": true, "empty": true },
    "pageNumber": 3,
    "pageSize": 4,
    "offset": 12,
    "paged": true,
    "unpaged": false
  },
  "number": 3,
  "numberOfElements": 4,
  "first": false,
  "last": false,
  "size": 4,
  "sort": { "sorted": false, "unsorted": true, "empty": true },
  "empty": false
}
```

Page<T> 타입에는 없던 number과 numberOfElements들이 생기고, Page<T> 에 존재하던 totalPages, totalElements가 없어짐

Slice<T> 타입은 추가 count 쿼리 없이 다음 페이지 확인 가능

내부적으로 limit+1 조회를 해서 totalCount 쿼리가 나가지 않음

### 3. List<T>

```java
@GetMapping("/users")
public List<User> getAllUsers(Pageable pageable) {
  return userRepository.findAll(pageable);
}
```

List 반환 타입은 가장 기본적인 방법으로 count 쿼리 없이 결과만 반환

<br>

## Spring Web MVC에서 더 편하게 페이징하기

사용자가 정의한 파라미터에 따라서도 페이징이 가능

컨트롤러에서 @GetMapping에 파라미터로 Pageable 추가

```java
@GetMapping("/users")
public Page<User> getAllUsers(Pageable pageable) {
    return userRepository.findAll(pageable);
}
```

Springboot에서 url파라미터가 컨트롤러에 바인딩될 때, Pageable이 존재하면 PageRequest 객체 생성

해당 객체에서 정렬도 제공하는데, url을 다음과 같이 치면 정렬과 페이징이 동시에 수행되게 할 수 있음

- http://localhost:8080/members?page=0 : 0번 페이지 부터 20개 조회(default가 20개로 default 수정 가능)
- http://localhost:8080/members?page=0&size=5 : 0번 페이지부터 5개 조회
- http://localhost:8080/members?page=0&size=5&sort=id.desc : 0번 페이지부터 5개 id역순으로 조회

<br>