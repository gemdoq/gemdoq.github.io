---
layout: single
title: "Spring Boot로 파일 업로드와 다운로드 구현"
date: 2025-07-30 17:33:00 +0900
categories: Spring Boot
tags: [Upload, Download]
typora-root-url: ../

---
# ✈️ Spring Boot로 파일 업로드와 다운로드 구현

## 🎯 개요

Spring Boot로 파일 업로드와 다운로드 기능 구현
JDK21과 Gradle 환경에서 MultipartFile과 Resource로 파일 처리
IntelliJ Ultimate로 로컬 테스트와 디버깅 편리

## 📋 목록

- [ ]  파일 업로드와 다운로드란?
- [ ]  준비 항목
- [ ]  구현 단계
- [ ]  로컬 실행과 디버깅
- [ ]  실무 팁과 주의사항

---

## ✏️ 파일 업로드와 다운로드란?

### 파일 업로드 정의

웹에서 파일을 서버로 전송하는 기능
MultipartFile로 파일 이름, 크기, 형식 관리

### 파일 다운로드 정의

서버에서 파일을 사용자에게 제공하는 기능
Resource로 파일 준비와 다운로드 지원

> 💡왜 필요하나?
>
> Spring Boot 프로젝트에서 이미지나 문서 처리로 기능 확장
> Gradle 빌드와 application.yml로 설정 간소화

---

## ✏️ 준비 항목

- JDK21: java.net에서 다운로드
- IntelliJ Ultimate: 개발 환경 구성
- Spring Initializr: start.spring.io로 프로젝트 생성
- Gradle: 빌드 툴로 의존성 관리
- Thymeleaf: HTML 템플릿 엔진

> 💡특이사항
>
> Gradle 8.x 이상으로 JDK21 호환성 확보
> application.yml로 파일 크기 제한 설정

---

## ✏️ 구현 단계

### 1단계: 프로젝트 설정

start.spring.io로 Spring Web, Thymeleaf 선택 후 다운로드
build.gradle에 의존성 확인
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
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
}
```

application.yml에 업로드 설정 추가
```yaml
spring:
  application:
    name: file-app
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 10MB
server:
  port: 8080
```

### 2단계: 파일 업로드 구현

컨트롤러에 업로드 메서드 추가
```java
@Controller
public class FileController {
    private static String UPLOAD_DIR = "uploads/";

    @GetMapping("/")
    public String index() {
        return "index";
    }

    @PostMapping("/upload")
    public String uploadFile(@RequestParam("file") MultipartFile file, RedirectAttributes attributes) {
        if (file.isEmpty()) {
            attributes.addFlashAttribute("message", "파일을 선택하세요");
            return "redirect:/";
        }
        try {
            byte[] bytes = file.getBytes();
            Path path = Paths.get(UPLOAD_DIR + file.getOriginalFilename());
            Files.createDirectories(path.getParent());
            Files.write(path, bytes);
            attributes.addFlashAttribute("message", "파일 업로드 성공: " + file.getOriginalFilename());
        } catch (IOException e) {
            attributes.addFlashAttribute("message", "파일 업로드 실패");
        }
        return "redirect:/";
    }
}
```

Thymeleaf 템플릿으로 업로드 폼
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>파일 업로드/다운로드</title>
</head>
<body>
    <h1>파일 업로드</h1>
    <form method="post" action="/upload" enctype="multipart/form-data">
        <input type="file" name="file">
        <button type="submit">업로드</button>
    </form>
    <p th:if="${message}" th:text="${message}"></p>
</body>
</html>
```

### 3단계: 파일 다운로드 구현

컨트롤러에 다운로드 메서드 추가
```java
@GetMapping("/download/{filename:.+}")
public ResponseEntity<Resource> downloadFile(@PathVariable String filename) {
    try {
        Path file = Paths.get(UPLOAD_DIR + filename);
        Resource resource = new UrlResource(file.toUri());
        if (resource.exists() && resource.isReadable()) {
            return ResponseEntity.ok()
                    .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + resource.getFilename() + "\"")
                    .body(resource);
        }
        return ResponseEntity.notFound().build();
    } catch (Exception e) {
        return ResponseEntity.badRequest().build();
    }
}
```

템플릿에 다운로드 링크 추가
```html
<a th:href="@{/download/{filename}(filename='example.txt')}" th:text="'다운로드'"></a>
```

> 💡특이사항
>
> IntelliJ Ultimate로 MultipartFile 디버깅 편리
> uploads 폴더 생성으로 파일 저장 위치 확인

---

## ✏️ 로컬 실행과 디버깅

1. Gradle로 빌드 및 실행:
```bash
./gradlew bootRun
```
2. http://localhost:8080 접속
3. 파일 선택 후 업로드
4. 다운로드 링크 클릭으로 파일 확인
5. IntelliJ 콘솔 로그로 에러 디버깅
6. 파일 형식 제한 추가:
```java
if (!file.getContentType().startsWith("image/")) {
    attributes.addFlashAttribute("message", "이미지만 허용");
    return "redirect:/";
}
```

> 💡특이사항
>
> 브라우저 개발자 도구로 네트워크 탭 점검
> application.yml로 파일 크기 조정

---

## ✏️ 실무 팁과 주의사항

### 파일 처리 팁

- 간단 시작: 단일 파일 업로드 테스트
- 보안 주의: 파일 형식과 크기 제한 필수
- 에러 처리: 사용자 메시지 출력
- 확장 가능: 다중 파일 업로드나 S3 연동
- Thymeleaf 활용: 동적 파일 목록 표시

### 주의사항

- 파일 저장 위치: uploads 폴더 권한 확인
- 의존성 호환: Gradle 빌드 시 spring-boot-starter-web 버전 점검
- 로그 확인: IntelliJ 콘솔로 업로드 실패 디버깅
- 테스트: 로컬 환경에서 업로드와 다운로드 동작 확인