---
layout: single
title: "Spring Boot와 JPA로 게시판 만들기"
date: 2025-08-06 17:31:00 +0900
categories: spring-boot
tags: [Spring Boot, JPA, Board]
typora-root-url: ../

---

# ✈️ Spring Boot와 JPA로 게시판 만들기

## 🎯 개요

Spring Boot와 JPA로 간단한 게시판 구현
JDK21과 Gradle 환경에서 데이터베이스 연동으로 CRUD 기능 제공
IntelliJ Ultimate로 빠른 개발과 테스트 가능

## 📋 목록

- [ ]  게시판이란?
- [ ]  준비 항목
- [ ]  게시판 구현 단계
- [ ]  실행과 테스트
- [ ]  실무 팁과 주의사항

---

## ✏️ 게시판이란?

### 게시판 정의

사용자가 글 작성, 조회, 수정, 삭제 가능한 웹 기능
JPA로 데이터베이스 테이블과 자바 객체 매핑

### 주요 구성 요소

- 엔티티: 데이터베이스 테이블과 매핑된 객체
- 리포지토리: 데이터베이스 상호작용 인터페이스
- 서비스: 비즈니스 로직 처리 계층
- 컨트롤러: HTTP 요청 처리 계층

> 💡왜 필요하나?
>
> Spring Boot 프로젝트에서 CRUD 기능 구현으로 데이터 관리 실습
> Gradle 빌드와 application.yml로 설정 간소화

---

## ✏️ 준비 항목

- JDK21: java.net에서 다운로드
- Spring Initializr: start.spring.io로 프로젝트 생성
- IntelliJ Ultimate: 개발 환경 구성
- Gradle: 빌드 툴로 의존성 관리
- H2 데이터베이스: 테스트용 메모리 DB
- Postman: API 테스트 도구

> 💡특이사항
>
> Gradle 8.x 이상으로 JDK21 호환성 확보
> application.yml로 데이터소스 설정 관리

---

## ✏️ 게시판 구현 단계

### 1단계: 프로젝트 설정

build.gradle에 의존성 추가
```groovy
plugins {
    id 'org.springframework.boot' version '3.4.5'
    id 'io.spring.dependency-management' version '1.1.7'
    id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '21'
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
}
```

application.yml에 H2 설정 추가
```yaml
spring:
  application:
    name: board-app
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password:
  h2:
    console:
      enabled: true
server:
  port: 8080
```

### 2단계: 엔티티 클래스 생성

게시판 글 정보 저장 엔티티
```java
@Entity
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private String content;
    private String author;

    public Post() {}
    public Post(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }
}
```

### 3단계: 리포지토리 인터페이스 생성

데이터베이스 연동 인터페이스
```java
public interface PostRepository extends JpaRepository<Post, Long> {
}
```

### 4단계: 서비스 레이어 구현

비즈니스 로직 처리
```java
@Service
public class PostService {
    @Autowired
    private PostRepository postRepository;

    public List<Post> getAllPosts() {
        return postRepository.findAll();
    }

    public Post getPostById(Long id) {
        return postRepository.findById(id).orElse(null);
    }

    public Post createPost(Post post) {
        return postRepository.save(post);
    }

    public Post updatePost(Long id, Post postDetails) {
        Post post = postRepository.findById(id).orElse(null);
        if (post != null) {
            post.setTitle(postDetails.getTitle());
            post.setContent(postDetails.getContent());
            post.setAuthor(postDetails.getAuthor());
            return postRepository.save(post);
        }
        return null;
    }

    public void deletePost(Long id) {
        postRepository.deleteById(id);
    }
}
```

### 5단계: 컨트롤러 레이어 구현

HTTP 요청 처리
```java
@RestController
@RequestMapping("/api/posts")
public class PostController {
    @Autowired
    private PostService postService;

    @GetMapping
    public List<Post> getAllPosts() {
        return postService.getAllPosts();
    }

    @GetMapping("/{id}")
    public Post getPostById(@PathVariable Long id) {
        return postService.getPostById(id);
    }

    @PostMapping
    public Post createPost(@RequestBody Post post) {
        return postService.createPost(post);
    }

    @PutMapping("/{id}")
    public Post updatePost(@PathVariable Long id, @RequestBody Post postDetails) {
        return postService.updatePost(id, postDetails);
    }

    @DeleteMapping("/{id}")
    public void deletePost(@PathVariable Long id) {
        postService.deletePost(id);
    }
}
```

> 💡특이사항
>
> IntelliJ Ultimate로 JPA 엔티티 디버깅 편리
> H2 콘솔로 데이터베이스 상태 확인 가능

---

## ✏️ 실행과 테스트

1. Gradle로 빌드 및 실행:
```bash
./gradlew bootRun
```
2. Postman으로 API 테스트:
   - GET /api/posts: 모든 글 조회
   - GET /api/posts/{id}: 특정 글 조회
   - POST /api/posts: 글 생성
   - PUT /api/posts/{id}: 글 수정
   - DELETE /api/posts/{id}: 글 삭제
3. http://localhost:8080/h2-console로 DB 확인

---

## ✏️ 실무 팁과 주의사항

### 게시판 구현 팁

- 간단 시작: H2 메모리 DB로 테스트
- 디버깅: IntelliJ Ultimate로 JPA 쿼리 확인
- 공식 문서: Spring Boot 3.x와 JPA 가이드 참고
- 확장: 페이징이나 검색 기능 추가
- 테스트: Postman으로 CRUD 동작 점검

### 주의사항

- 의존성 호환: Gradle 빌드 시 spring-boot-starter-data-jpa 버전 확인
- 데이터베이스 설정: application.yml에서 H2 URL 정확히 입력
- 로그 확인: IntelliJ 콘솔로 에러 디버깅
- 테스트: 로컬 환경에서 CRUD 기능 모두 확인

> 💡특이사항
>
> JDK21 환경에서 Gradle 8.x 이상 추천
> application.yml로 데이터소스 설정 중앙화
> 이 설정으로 Spring Boot 프로젝트의 CRUD 기능 안정화