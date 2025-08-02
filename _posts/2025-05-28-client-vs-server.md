---
layout: single
title: "클라이언트와 서버 구분 짓기"
date: 2025-05-28 11:09:23 +0900
categories: concepts
tags: [Client, Server]
typora-root-url: ../

---

#### 📌 용어 한눈에
- 클라이언트: 서비스를 요청하는 쪽, 사용자나 사용자 프로그램  
- 서버: 요청을 받아 응답을 제공하는 쪽, 시스템이나 프로그램  
- 요청: 클라이언트가 서버에 보내는 데이터나 작업 지시  
- 응답: 서버가 클라이언트에 돌려주는 결과  
- 프로세스: 실행 중인 프로그램 단위, 한 기기에서 클라이언트와 서버 역할 동시 가능  

---
#### 📌 클라이언트와 서버의 차이점
클라이언트와 서버는 기기가 아니라 **역할**로 구분  
- 클라이언트: 요청을 보내는 쪽  
- 서버: 요청을 처리하고 응답하는 쪽  

노트북, 스마트폰, PC 같은 기기는  
상황에 따라 클라이언트도 될 수 있고 서버도 될 수 있음  
중요한 건 **어떤 프로세스가 어떤 역할을 하느냐**  

---
#### 📌 예시로 풀어보기

##### 🖥️ 브라우저로 웹사이트 접속
- Chrome에서 `https://example.com` 입력  
- 브라우저(클라이언트)가 서버에 HTTP 요청 보냄  
- 서버가 HTML, CSS, JS로 응답  
- 이 경우 노트북은 **클라이언트 역할**  

##### 🖥️ IntelliJ로 스프링부트 서버 띄우기
- IntelliJ에서 `@SpringBootApplication` 실행  
- `localhost:8080` 같은 포트로 서버 프로세스 구동  
- 노트북이 **서버 역할**  

##### 🖥️ 같은 노트북에서 `localhost:8080` 접속
- 브라우저가 `localhost:8080`으로 요청 → **클라이언트 역할**  
- 로컬 스프링부트 서버가 응답 → **서버 역할**  
- 한 노트북에서 클라이언트와 서버 프로세스 동시 실행  

---
#### 📌 코드로 살펴보기

##### ✅ 서버 (Spring Boot)
```java
@RestController
@RequestMapping("/api")
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello from Server!";
    }
}
```
`@RestController`로 요청 처리  
`application.yml`에서 포트 설정 가능 (예: `server.port=8080`)  
노트북이 서버 역할로 동작  

##### ✅ 클라이언트 (Java HTTP 클라이언트)
```java
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.net.URI;

public class ClientExample {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("http://localhost:8080/api/hello"))
                .build();
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        System.out.println(response.body());
    }
}
```
이 코드는 요청을 보내는 클라이언트 역할  
IntelliJ에서 실행하면 디버깅도 편리  

---
#### 📌 노트북은 클라이언트일까 서버일까
"노트북이 클라이언트인가 서버인가"는 잘못된 질문  
기기가 아니라 **프로세스의 역할**로 판단  
- 브라우저로 URL 접속 → 클라이언트  
- 스프링부트 서버 실행 → 서버  
- 둘 다 동시에 가능  

스프링부트 개발자라면 IntelliJ로 서버 띄우고  
브라우저나 Postman으로 테스트하며 역할 구분 익히는 게 추천  

---
#### 📌 혼동되는 이유
기기 중심으로 생각하려는 습관 때문  
"URL 치면 클라이언트", "서버 코드 돌리면 서버" 같은 맥락을 헷갈림  
**요청과 응답**의 흐름으로 역할 구분하면 명확해짐  

---
#### ✍ 결론
클라이언트와 서버는 역할의 문제  
노트북 하나로도 두 역할을 다 경험 가능  
핵심은 **역할의 구분**과 **흐름의 이해**  

스프링부트로 API 만들 때  
`@RestController`로 서버 역할 구현하고  
Postman이나 HTTP 클라이언트로 요청 날려보면서 이해  