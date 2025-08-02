---
layout: single
title: "스프링부트 싱글톤 빈으로 리소스 절약하기"
date: 2025-06-13 14:12:23 +0900
categories: Spring Boot
tags: [Singleton Bean, Resource]
typora-root-url: ../

---

#### 📌 용어 한눈에
- 싱글톤 빈: ApplicationContext가 관리하는 단일 인스턴스  
- 리소스: 메모리, CPU 등 시스템 자원  
- 상태 없는 빈: 내부 데이터 없음, 스레드 안전  
- 상태 있는 빈: 내부 데이터 있음, 동시성 주의  
- 프로토타입 스코프: 요청마다 새 인스턴스 생성  

---
#### 📌 싱글톤 빈이란
스프링에서 기본적으로 빈을 싱글톤으로 관리  
ApplicationContext에 단일 인스턴스 저장  
여러 컴포넌트가 같은 인스턴스 공유  

##### 실생활 비유
카페의 커피 머신  
손님마다 새 머신 설치 대신 하나로 모든 주문 처리  
공간, 시간 절약  

---
#### 📌 싱글톤 빈의 필요성
단일 인스턴스로 메모리와 CPU 절약  
의존성 주입 간소화, 일관성 유지  

##### 실생활 비유
카페에 머신 하나로 모든 손님 주문 처리  
머신 여러 개면 공간, 비용 낭비  

##### 그림으로 보기
비-싱글톤 (매번 새 객체)  
```
[메모리]
컨트롤러 A -> [CalculatorService 객체1] (100KB)
컨트롤러 B -> [CalculatorService 객체2] (100KB)
컨트롤러 C -> [CalculatorService 객체3] (100KB)
=> 메모리: 300KB, 생성 3번
```

싱글톤 (단일 인스턴스)  
```
[메모리]
ApplicationContext
   |
   v
[CalculatorService 객체1] (100KB)
   ^         ^         ^
컨트롤러 A  컨트롤러 B  컨트롤러 C
=> 메모리: 100KB, 생성 1번
```

> 💡 팁  
> IntelliJ로 빈 스코프 확인, 메모리 사용량 디버깅 가능  

---
#### 📌 코드로 살펴보기

##### ✅ 상태 없는 싱글톤 빈
```java
import org.springframework.stereotype.Service;

@Service
public class CalculatorService {
    public int add(int a, int b) {
        return a + b;
    }
}
```

- 내부 상태 없음  
- 여러 스레드 동시 호출 안전  
- 싱글톤으로 리소스 효율  

##### ✅ 상태 있는 빈 (주의)
```java
import org.springframework.stereotype.Service;

@Service
public class CounterService {
    private int count = 0;

    public void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}
```

- `count`로 상태 유지  
- 싱글톤 시 동시성 문제 위험  
- `@Synchronized`나 프로토타입 스코프 고려  

##### ✅ 프로토타입 스코프
```java
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Service;

@Service
@Scope("prototype")
public class CalculatorService {
    public int add(int a, int b) {
        return a + b;
    }
}
```

- 요청마다 새 객체 생성  
- 메모리 소모 늘어남  
- application.yml로 스코프 설정 가능  

> 💡 팁  
> application.yml에 `spring.main.allow-bean-definition-overriding: true`로 빈 충돌 관리  

---
#### 📌 리소스 절약의 가치
소규모 앱은 싱글톤 효과 덜함  
대규모 앱은 사용자 많고 빈 많아 필수  
예: 사용자 10,000명, 빈 100개  
- 비-싱글톤: 객체 1,000,000개, 메모리 폭발  
- 싱글톤: 객체 100개, 서버 안정  

##### 실생활 비유
대형 카페, 손님 1,000명  
머신 하나로 효율 처리 vs 머신 1,000개 설치  

---
#### 📌 설계 팁
- 상태 없는 빈으로 설계, 스레드 안전  
- 상태 있는 빈은 `@Scope("prototype")` 또는 동시성 제어  
- 싱글톤은 설정, 공유 자원에 적합  
- IntelliJ로 빈 의존성 그래프 확인 추천  
- application.yml로 빈 스코프와 설정 관리  

---
#### ✍ 결론
싱글톤 처음엔 단순 객체 공유로 봤음  
커피 머신 비유로 리소스 절약 실감  
상태 없는 빈의 스레드 안전성 중요성 이해  
빈 설계와 스코프 선택이 서버 안정성의 핵심  