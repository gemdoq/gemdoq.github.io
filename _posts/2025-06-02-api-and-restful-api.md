---
layout: single
title: "API와 RESTful API 파헤치기"
date: 2025-06-02 14:12:23 +0900
categories: concepts
tags: [RESTful API]
typora-root-url: ../

---

#### 📌 용어 한눈에
- API: 프로그램 간 소통을 위한 인터페이스, 기능과 데이터 주고받기  
- REST: 웹 시스템 설계를 위한 아키텍처 스타일  
- RESTful API: REST 원칙을 따라 설계된 API  
- HTTP 메서드: 클라이언트-서버 간 요청 방식, GET, POST, PUT, DELETE 등  
- 엔드포인트: API의 특정 자원에 접근하는 URL 경로  

---
#### 📌 API란
프로그램끼리 데이터를 교환하거나 기능을 호출하는 연결 고리  
내부 구현은 몰라도 정해진 규칙만 따르면 사용 가능  

예: 카카오맵 API 호출로 지도 띄우기  
지도 로직을 직접 짤 필요 없이 API로 간단히 해결  

---
#### 📌 RESTful API란
REST 원칙을 따르는 API 설계 방식  
자원을 URI로 표현하고 HTTP 메서드로 작업 정의  

RESTful API의 장점  
- 직관적이고 예측 가능한 URL 구조  
- HTTP 표준 활용으로 호환성 높음  
- 클라이언트-서버 역할 명확히 분리  

스프링부트 개발자라면 `@RestController`로 RESTful API 쉽게 구현 가능  

---
#### 📌 RESTful API 예시
블로그 서비스의 게시글 자원 다루기  

```plaintext
GET /api/posts → 전체 게시글 조회
GET /api/posts/1 → 특정 게시글 조회
POST /api/posts → 새 게시글 생성
PUT /api/posts/1 → 특정 게시글 수정
DELETE /api/posts/1 → 특정 게시글 삭제
```

URI는 자원을, HTTP 메서드는 작업을 나타냄  
스프링부트로 구현 시 `application.yml`로 기본 경로 설정 추천  

```java
@RestController
@RequestMapping("/api/posts")
public class PostController {
    private final PostService postService;

    public PostController(PostService postService) {
        this.postService = postService;
    }

    @GetMapping
    public List<PostResponse> getPosts() {
        return postService.findAll();
    }

    @GetMapping("/{id}")
    public PostResponse getPost(@PathVariable Long id) {
        return postService.findById(id);
    }
}
```

IntelliJ Ultimate에서 `@RequestMapping` 자동완성, 경로 오류 감지로 생산성 높아짐  

---
#### 📌 RESTful API의 특징

##### ✅ Stateless
각 요청은 독립적, 서버는 이전 요청 기억 안 함  
클라이언트가 상태 관리 (예: JWT 토큰)  

##### ✅ Uniform Interface
일관된 인터페이스 제공  
URI로 자원, HTTP 메서드로 행동 표현  

##### ✅ Cacheable
응답 캐싱 가능, 네트워크 부하 줄임  
예: GET 요청에 ETag 활용  

##### ✅ Layered System
프록시나 게이트웨이 같은 중간 레이어로 확장 가능  

---
#### 📌 RESTful하지 않은 API 예시
```plaintext
/api/getPosts
/api/savePost
```

동사를 URI에 넣거나  
모든 작업을 POST로 처리하면 RESTful 아님  
자원 중심 설계가 핵심  

---
#### 📌 API 설계 팁
- URI는 명사로, 자원을 명확히  
- HTTP 메서드 적절히 활용 (GET은 조회, POST는 생성)  
- 상태 코드 활용 (200 OK, 404 Not Found, 500 Internal Server Error)  
- 버저닝 고려 (예: `/api/v1/posts`)  
- 스프링부트라면 `@RestControllerAdvice`로 예외 처리 통합 추천  

---
#### ✍ 결론
API는 단순한 기능 호출이 아님  
RESTful API는 설계 철학이 담긴 소통 방식  
좋은 RESTful API는 유지보수와 확장을 쉽게 만듦  