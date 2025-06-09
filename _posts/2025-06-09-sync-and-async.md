---
title: "동기와 비동기 구분 짓기"
categories:
  - CS Basics
tags:
  - CS Basics
last_modified_at: 2025-06-21
---

#### 📌 용어 한눈에
- 동기: 요청 후 결과 기다림, 순서대로 진행  
- 비동기: 요청 후 기다리지 않음, 다른 작업 가능  
- 블로킹: 쓰레드 대기 상태로 멈춤  
- 논블로킹: 쓰레드 계속 실행 가능  
- 쓰레드: 프로그램 내 독립 실행 흐름  
- 콜백: 작업 완료 후 호출되는 함수  
- Future: 비동기 결과 담는 자바 객체  
- CompletableFuture: 자바 8+ 비동기 처리 도구  

---
#### 📌 동기란
작업 하나 끝날 때까지 다음 작업 시작 안 함  
순서 명확하고 직관적  

##### 실생활 비유
은행 창구에서 번호표 들고 대기  
앞사람 업무 끝날 때까지 기다림  
내 차례 올 때까지 다른 일 못 함  

---
#### 📌 비동기란
작업 요청 후 바로 다른 일 시작  
결과 준비되면 콜백이나 알림으로 처리  

##### 실생활 비유
패스트푸드점에서 주문 후 번호표 받고 자리로  
음식 기다리는 동안 책 읽거나 전화 가능  
음식 완성 알림 오면 픽업  

---
#### 📌 코드로 비교

##### ✅ 동기 (RestTemplate)
```java
import org.springframework.web.client.RestTemplate;

public class SyncExample {
    public static void main(String[] args) {
        RestTemplate restTemplate = new RestTemplate();

        // 동기 요청
        String response = restTemplate.getForObject("https://jsonplaceholder.typicode.com/posts/1", String.class);

        System.out.println("Response: " + response);
        System.out.println("응답 후 실행");
    }
}
```

> 💡 팁  
> IntelliJ 디버깅으로 쓰레드 대기 확인  
> application.yml에 `spring.http.rest-template` 타임아웃 설정 추천  

##### ✅ 비동기 (WebClient)
```java
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

public class AsyncExample {
    public static void main(String[] args) {
        WebClient webClient = WebClient.create("https://jsonplaceholder.typicode.com");

        // 비동기 요청
        Mono<String> responseMono = webClient.get()
                .uri("/posts/1")
                .retrieve()
                .bodyToMono(String.class);

        responseMono.subscribe(response -> System.out.println("Response: " + response));

        System.out.println("요청 직후 실행");

        // 예시용 대기
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

> 💡 팁  
> Gradle에 `implementation 'org.springframework.boot:spring-boot-starter-webflux'` 추가  
> application.yml에 `spring.main.web-application-type: reactive` 설정  

---
#### 📌 비동기와 논블로킹 차이
비동기와 논블로킹은 비슷해 보이지만 다름  
쉽게 혼동되는 개념  

##### ✅ 실생활 비유
카페 주문 상황  
- 블로킹: 카운터 앞에서 커피 나올 때까지 대기  
- 논블로킹: 번호표 받고 자리로, 다른 손님도 주문 가능  
- 동기: 커피 받아야 친구 만남  
- 비동기: 커피 기다리지 않고 친구 먼저 만남  

##### ✅ 코드 관점
- 블로킹: 쓰레드 결과 기다림  
- 논블로킹: 쓰레드 다른 작업 가능  
- 동기: 코드 흐름 멈춤  
- 비동기: 코드 흐름 계속, 결과 나중에 처리  

##### ✅ 관계 정리
|               | 블로킹 | 논블로킹 |
|---------------|--------|----------|
| 동기          | RestTemplate 기본 | 드물음 |
| 비동기        | 드물음 | WebClient, WebFlux |

##### ✅ 중요성
- 동기 + 블로킹: 구현 쉬움, 자원 소모 많음  
- 비동기 + 논블로킹: 복잡하지만 성능 효율적  

---
#### 📌 언제 쓰나
- 동기  
작업 순서 중요  
간단한 로직 선호  
예: 소규모 API 호출  

- 비동기  
긴 대기 작업 (네트워크, 파일 IO)  
UI 반응성 필요  
고트래픽 서버 (스프링 `@Async` 또는 WebFlux)  

---
#### 📌 비동기 도구
- 자바: Future, CompletableFuture  
- 스프링: `@Async`, WebClient  
- 스프링부트: WebFlux로 논블로킹 API 구현  
- IntelliJ: 비동기 코드 디버깅, 콜백 흐름 추적 지원  

> 💡 팁  
> application.yml로 `@Async` 스레드 풀 설정  
> `spring.task.execution.pool.max-size: 10` 예시  

---
#### ✍ 느끼며
동기 비동기 처음엔 속도 차이로만 봤음  
알고 보니 **흐름과 자원**의 문제  

RestTemplate으로 동기 시작, WebClient로 비동기 익히는 여정  
스프링부트에서 WebFlux 도입하면 비동기 감 잡힘  
IntelliJ 디버깅과 Postman 테스트로 흐름 이해  

비동기는 성능 향상의 열쇠  
복잡성 감수하더라도 고트래픽 환경엔 필수  
상황별 도구 선택이 개발자의 성장