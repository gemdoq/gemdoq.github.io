---
layout: single
title: "HTTP 클라이언트 알아보기: RestTemplate WebClient 중심"
date: 2025-06-08 14:12:23 +0900
categories: Spring Boot
tags: [RestTemplate, WebClient, HttpClient, FeignClient]
typora-root-url: ../

---

#### 📌 HTTP 클라이언트란
서버에 HTTP 요청을 보내고 응답을 받는 도구  
프로그램이나 라이브러리로 구현  

실생활 비유  
- 배달 앱에서 '주문' 버튼 누름  
- 앱이 서버에 주문 정보 전송  
- 서버가 '주문 접수' 응답  
- 이 과정에서 HTTP 클라이언트가 요청과 응답 처리  

스프링부트 개발자라면 RestTemplate이나 WebClient 자주 접함  

---
#### 📌 요청 흐름
1️⃣ 요청 생성  
주문 정보(메뉴, 주소)를 HTTP 본문에 담아 서버로 전송  
2️⃣ 서버 처리  
서버가 요청 처리 후 응답 준비  
3️⃣ 응답 수신  
'주문 완료' 같은 응답 받음  
4️⃣ 응답 활용  
앱이 응답 기반으로 화면 업데이트  

IntelliJ에서 디버깅하면 흐름 파악 쉬움  

---
#### 📌 동기 vs 비동기
- 동기  
요청 후 응답 올 때까지 기다림  
화면 멈춤 (로딩 표시)  
RestTemplate 기본 방식  

- 비동기  
요청 보내고 바로 다른 작업 가능  
응답 오면 알림으로 처리  
WebClient로 구현, UX 부드러움  

스프링부트에서 WebClient 쓰려면 `application.yml`에 `spring.main.web-application-type: reactive` 설정 추천  

---
#### 📌 주요 HTTP 클라이언트 도구

##### ✅ RestTemplate (동기)
스프링의 기본 동기 클라이언트  
간단한 REST API 호출에 최적  

```java
import org.springframework.web.client.RestTemplate;

public class RestTemplateExample {
    public static void main(String[] args) {
        RestTemplate restTemplate = new RestTemplate();

        // 동기 GET 요청
        String url = "https://jsonplaceholder.typicode.com/posts/1";
        String response = restTemplate.getForObject(url, String.class);

        System.out.println("Response: " + response);
    }
}
```

> 💡 팁  
> IntelliJ로 디버깅 시 쓰레드 대기 확인  
> application.yml에 `spring.http.rest-template`로 타임아웃 설정 가능  

##### ✅ WebClient (비동기, 논블로킹)
스프링 5+ 리액티브 클라이언트  
고트래픽 환경에 적합  

```java
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

public class WebClientExample {
    public static void main(String[] args) {
        WebClient client = WebClient.create("https://jsonplaceholder.typicode.com");

        // 비동기 GET 요청
        Mono<String> responseMono = client.get()
                .uri("/posts/1")
                .retrieve()
                .bodyToMono(String.class);

        responseMono.subscribe(response -> System.out.println("Response: " + response));

        System.out.println("요청 후 바로 실행");

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
> IntelliJ 코드 자동완성으로 WebClient 설정 간편  

##### ✅ Apache HttpClient (동기, 세밀 제어)
강력한 커넥션 관리 지원  
스프링 외부에서도 사용  

```java
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

public class ApacheHttpClientExample {
    public static void main(String[] args) throws Exception {
        try (CloseableHttpClient client = HttpClients.createDefault()) {
            HttpGet request = new HttpGet("https://jsonplaceholder.typicode.com/posts/1");
            try (CloseableHttpResponse response = client.execute(request)) {
                String responseBody = EntityUtils.toString(response.getEntity());
                System.out.println("Response: " + responseBody);
            }
        }
    }
}
```

> 💡 팁  
> Gradle에 `implementation 'org.apache.httpcomponents:httpclient:4.5.14'` 추가  
> 복잡한 커넥션 풀 설정 필요 시 유용  

##### ✅ OkHttp (동기/비동기)
안드로이드와 자바에서 인기  
HTTP/2, 웹소켓 지원  

```java
import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;

import java.io.IOException;

public class OkHttpExample {
    public static void main(String[] args) {
        OkHttpClient client = new OkHttpClient();

        Request request = new Request.Builder()
                .url("https://jsonplaceholder.typicode.com/posts/1")
                .build();

        // 비동기 요청
        client.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                System.out.println("Failed: " + e.getMessage());
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                System.out.println("Response: " + response.body().string());
            }
        });

        System.out.println("요청 후 바로 실행");

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
> Gradle에 `implementation 'com.squareup.okhttp3:okhttp:4.12.0'` 추가  
> 스프링부트와 함께 쓰려면 RestTemplate 대체 가능  

##### ✅ Feign (선언적 클라이언트)
스프링 클라우드에서 REST 호출 간소화  
인터페이스 정의로 코드 줄임  

```java
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(name = "jsonPlaceholder", url = "https://jsonplaceholder.typicode.com")
public interface JsonPlaceholderClient {
    @GetMapping("/posts/{id}")
    String getPostById(@PathVariable("id") Long id);
}

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class FeignExample implements CommandLineRunner {

    @Autowired
    private JsonPlaceholderClient client;

    @Override
    public void run(String... args) throws Exception {
        String response = client.getPostById(1L);
        System.out.println("Response: " + response);
    }
}
```

> 💡 팁  
> Gradle에 `implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'` 추가  
> application.yml에 Feign 설정 (타임아웃, 로깅) 추천  

---
#### ✍ 느끼며
HTTP 클라이언트는 서버와 소통하는 전화기 같은 존재  
RestTemplate은 간단하지만 고트래픽엔 한계  
WebClient는 비동기와 논블로킹으로 미래 지향적  

스프링부트 개발자라면 WebClient와 Feign 적극 활용 추천  
IntelliJ로 WebClient 디버깅, Postman으로 응답 테스트하면 흐름 빠르게 익힘  
application.yml로 타임아웃과 로깅 설정하면 유지보수 쉬워짐  

트래픽과 요구사항에 따라 도구 선택  