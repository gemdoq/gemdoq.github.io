---
layout: single
title: "Spring에서 PageRequest와 Pageable에 대해"
date: 2023-04-04 11:43:27 +0900
categories: java
tags: [PageRequest, Pageable, Offset]
typora-root-url: ../
---

## Spring에서 PageRequest와 Pageable에 대해
> - 정의
> - PageRequest
> - Pageable
> - 결론

<br>

## 정의

스프링에서 데이터를 페이지별로 나누어 요청하고자 할 때, PageRequest와 Pageable이라는 기능을 사용

<br>

## PageRequest

PageRequest는 Pageable 인터페이스를 구현한 객체

페이징 처리를 편리하게 하기 위해 제공

### 예시

포스트 데이터를 페이지별로 나누어 요청하고자 할 때

PostService.java
```java
public Page<Post> getPosts(int page, int size) {
    return postRepository.findAll(PageRequest.of(page, size, Sort.by("id").descending()));
}
```

PostRepository.java
```java
public interface PostRepository extends JpaRepository<Post, Long> {
}
```

위 코드에서 PageRequest.of(page, size, Sort.by("id").descending())를 사용하여, 페이지 번호와 페이지 크기, 페이지 정렬 기준을 지정하는 것으로 페이징 처리 가능

PageRequest의 장점은 여러 파라미터를 한 번에 지정 가능

<br>

## Pageable

Pageable은 페이징 처리에 필요한 정보를 가지고 있는 인터페이스

### 예시

포스트 데이터를 페이지별로 나누어 요청하고자 할 때

PostService.java
```java
public Page<Post> getPosts(Pageable pageable) {
    return postRepository.findAll(pageable);
}
```

PostRestController.java
```java
@GetMapping("/posts")
public ResponseEntity<Page<Post>> getPosts(
    @RequestParam(value = "page", defaultValue = "0") int page, 
    @RequestParam(value = "size", defaultValue = "10") int size) {
        Page<Post> posts = postService.getPosts(PageRequest.of(page, size));
        return ResponseEntity.ok(posts);
}
```
위 코드에서는 먼저 컨트롤러에서 Pageable 객체를 생성하여 서비스에 전달하고, 서비스에서 이를 이용하여 페이징 처리

Pageable 인터페이스를 사용할 때의 장점은 사용자 정의 쿼리를 작성하여 자유롭고 유연하게 페이징 처리 수행 가능하고, 인터페이스이기 때문에 상속받는 클래스를 만들어 익명 클래스나 람다식 등으로 사용자가 원하는 대로 구현이 가능하고, JPA의 sort 기능과 같이 다양한 쿼리 처리 방식과 함께 사용할 수도 있음

예를 들어, Pageable 인터페이스를 구현한 OffsetBasedPageRequest를 사용하면, 페이징 처리 기능 뿐만 아니라 오프셋을 이용한 처리도 가능

#### 오프셋(offset)

데이터를 조회할 때 시작점(기준점)을 가리키는 용어

페이징 처리에서 오프셋은 결과 데이터의 시작지점을 나타내며, 주로 첫번째 페이지에 이어 다음 페이지부터 순서대로 불러올 때 사용

오프셋을 이용한 처리는 "처음부터 몇 번째 데이터까지 건너뛰겠다"는 개념으로 사용되어, 페이지 형식의 데이터를 조회할 경우 편리하게 결과를 가져와 처리 가능

예를 들어, 다음과 같은 데이터가 있을 때

```console
Data: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
```

페이지 크기를 3으로 설정하고, 오프셋을 0으로 설정하면, 아래와 같이 0(처음)부터 3개 데이터를 가져옴

```console
Page 1: 1, 2, 3 (offset: 0)
```

이 상태에서 오프셋을 3으로 변경하면, 다음과 같이 데이터를 가져옴

```console
Page 2: 4, 5, 6 (offset: 3)
```

이렇게 오프셋을 조절함으로써 원하는 페이지의 데이터를 조회 가능

<br>

## 결론

각 프로젝트의 상황과 개발자의 의도 및 요구 사항에 따라 PageRequest와 Pageable 인터페이스 중 적절한 방법으로 페이징 처리 필요

<br>
