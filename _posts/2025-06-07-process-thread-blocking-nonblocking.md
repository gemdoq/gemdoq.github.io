---
title: "프로세스 쓰레드 블로킹 논블로킹 파악하기"
categories:
  - CS Basics
tags:
  - CS Basics
last_modified_at: 2025-06-17
---

#### 📌 용어 한눈에
- 프로세스: 실행 중인 프로그램, OS가 메모리와 자원 할당  
- 쓰레드: 프로세스 내 작업 단위, 자원 공유하며 병렬 처리  
- 동기: 요청 후 결과 기다림, 순서대로 진행  
- 비동기: 요청 후 기다리지 않음, 다른 작업 가능  
- 블로킹: 쓰레드가 대기 상태로 멈춤  
- 논블로킹: 쓰레드가 계속 실행 가능  
- 멀티플렉서: IO 상태 감시하고 이벤트 알림  
- I/O: 외부와 데이터 주고받는 작업  

---
#### 📌 프로세스와 쓰레드

##### ✅ 프로세스
- 프로그램 실행 시 OS가 프로세스 생성  
- 독립 메모리 공간과 자원 할당  
- 다른 프로세스와 메모리 공유 안 함  

##### ✅ 쓰레드
- 프로세스 내 최소 하나 이상 쓰레드 (메인 쓰레드)  
- 쓰레드끼리 메모리 공유 (Heap 영역)  
- 병렬 작업으로 효율 높임  

##### ✅ 실생활 비유
- 프로세스 = 카페 매장  
- 쓰레드 = 매장 안 직원들  
→ 매장은 독립적  
→ 직원들은 같은 재료 공유하며 협업  

---
#### 📌 블로킹과 논블로킹 원리

##### ✅ 블로킹
- 쓰레드가 IO 요청 (파일, DB, 네트워크)  
- 결과 올 때까지 쓰레드 대기  
- 요청 많으면 쓰레드 많아져 자원 낭비  

##### ✅ 논블로킹
- IO 요청 시 쓰레드 멈추지 않음  
- OS 멀티플렉서가 IO 감시  
- IO 끝나면 이벤트 발생, 쓰레드 처리  

##### ✅ 실생활 비유
카페 주문 상황  

- 블로킹  
→ 손님 카운터 앞에서 커피 나올 때까지 기다림  
→ 다음 손님 대기  

- 논블로킹  
→ 손님 번호표 받고 자리로  
→ 카운터 계속 다른 주문 처리  
→ 커피 완성 알림 후 손님 픽업  

---
#### 📌 멀티플렉서 역할

멀티플렉서 = IO 감시 도구  

> 💡 IO가 멀티플렉서 필요한 이유  
> IO는 CPU 계산과 달리 외부 기다림 (디스크, 네트워크)  
> CPU 연산 = 빠른 머릿속 계산  
> IO = 전화 걸어 답 기다림 (느림)  
> 웹 서버 성능 저하는 보통 IO 때문  

##### ✅ 작동 방식
1️⃣ 쓰레드 IO를 멀티플렉서에 등록  
2️⃣ 쓰레드 다른 작업 계속  
3️⃣ 멀티플렉서 여러 IO 동시 감시  
4️⃣ IO 완료 이벤트 발생  
5️⃣ 쓰레드 콜백으로 결과 처리  

##### ✅ 비유
- 멀티플렉서 = 주문 상태 확인 카운터  
- "이 주문 끝남", "저건 아직" 리스트 관리  
- 이벤트 루프에서 반복 확인  

##### ✅ 기술 예
- Java NIO Selector  
- Linux epoll  
- macOS kqueue  
- Node.js libuv (epoll/kqueue 기반)  
- 스프링부트 WebFlux에서 Reactor Netty로 논블로킹 지원  

---
#### 📌 Selector 내부 흐름

##### ✅ IO 처리 예시

```plaintext
[main thread]
      |
      |--> IO 요청 (소켓 연결)
      |
[Selector 등록] ----> [IO Channel 1] (DB 쿼리)
                   |
                   +-> [IO Channel 2] (파일 읽기)
                   |
                   +-> [IO Channel 3] (API 호출)

[main thread]
      |
      |--> 다른 작업
      |
[Event Loop]
      |
      |--> Selector.poll() → 이벤트 확인
      |
      |--> 콜백 실행 → 결과 처리
```

##### ✅ 핵심
- IO 등록 후 쓰레드 자유  
- 이벤트 루프 지속 감시  
- 완료 시 알림과 콜백  

---
#### 📌 코드로 비교

##### ✅ 블로킹 IO (RestTemplate)

```java
import org.springframework.web.client.RestTemplate;

public class BlockingExample {
    public static void main(String[] args) {
        RestTemplate restTemplate = new RestTemplate();

        // 블로킹: 결과 올 때까지 대기
        String response = restTemplate.getForObject("https://jsonplaceholder.typicode.com/posts/1", String.class);

        System.out.println("Response: " + response);
    }
}
```

→ 요청 후 다음 줄 지연  
→ IntelliJ 디버깅 시 쓰레드 대기 확인  

##### ✅ 논블로킹 IO (WebClient)

```java
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

public class NonBlockingExample {
    public static void main(String[] args) {
        WebClient webClient = WebClient.create("https://jsonplaceholder.typicode.com");

        // 논블로킹: 바로 다음 코드 실행
        Mono<String> responseMono = webClient.get()
                .uri("/posts/1")
                .retrieve()
                .bodyToMono(String.class);

        responseMono.subscribe(response -> System.out.println("Response: " + response));

        System.out.println("이 출력은 즉시 나타남");

        // 예시용 대기
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

→ 콜백으로 처리  
→ application.yml에 `spring.main.web-application-type: reactive` 설정 추천  

---
#### 📌 논블로킹 IO 장단점

##### ✅ 장점
- 적은 쓰레드로 고부하 처리  
- TPS 높음, 자원 효율  
- 응답 속도 개선  

##### ✅ 단점
- 코드 복잡 (리액티브 이해 필요)  
- 디버깅 힘듦 (콜백 지옥 피하기)  
- 동기 코드와 섞을 때 주의  
- 학습 비용  

##### ✅ 사용 시기

| 상황 | 추천 |
|------|------|
| 트래픽 적음, 단순 구현 | RestTemplate 블로킹 |
| 고트래픽, 동시 요청 많음 | WebClient 논블로킹 |
| 마이크로서비스, API 호출 잦음 | WebFlux 필수 |

---
#### 📌 관계 요약

|               | 블로킹 | 논블로킹 |
|---------------|--------|----------|
| 동기          | 일반적 사용 | 드물음 |
| 비동기        | 드물음 | WebClient 등 |

##### ✅ 쓰레드 측면
- 블로킹: 요청당 쓰레드 필요, 낭비  
- 논블로킹: 적은 쓰레드로 효율  

##### ✅ 이벤트 루프
- 논블로킹 핵심  
- IO 등록 후 상태 확인, 완료 시 콜백  

---
#### 📌 마무르기
- 프로세스 = 독립 실행 단위  
- 쓰레드 = 공유 작업 흐름  
- 블로킹 = 쓰레드 대기 많음  
- 논블로킹 = 효율적 처리  
- 멀티플렉서 = IO 감시와 알림  
- WebClient = 고성능 키  
- 비동기 논블로킹 조합이 트렌드  

#### ✍ 느끼며
동기 비동기 블로킹 논블로킹 처음엔 혼란  
프로세스 쓰레드 원리 파악 후 차이 보임  

왜 논블로킹이 쓰레드 절약하고 성능 좋은지 이해  
IntelliJ로 WebClient 디버깅 해보면 실감  

상황별 선택이 핵심  
스프링부트에서 WebFlux 도입으로 논블로킹 시작 추천