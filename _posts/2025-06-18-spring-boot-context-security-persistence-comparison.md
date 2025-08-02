---
layout: single
title: "스프링부트 컨텍스트와 시큐리티 퍼시스턴스 비교"
date: 2025-06-18 14:12:23 +0900
categories: Spring Boot
tags: [ApplicationContext, SecurityContextHolder, SecurityContext, PersistenceContext]
typora-root-url: ../

---
#### 📌 용어 한눈에
- ApplicationContext: 스프링 IoC 컨테이너, 빈과 의존성 관리  
- SecurityContextHolder: 스레드별 보안 정보 저장소  
- SecurityContext: 사용자 인증/인가 정보, SecurityContextHolder가 관리  
- PersistenceContext: JPA로 엔티티 관리, DB 동기화  
- 싱글톤 빈: 단일 인스턴스 객체, ApplicationContext 관리  
- 상태 없는 빈: 내부 데이터 없음, 스레드 안전  
- 상태 있는 빈: 내부 데이터 있음, 동시성 주의  

---
#### 📌 ApplicationContext란
스프링의 핵심 컨테이너  
빈 생성, 의존성 주입, 생명주기 관리  
설정 로드, 이벤트 처리  

##### 실생활 비유
카페 매장  
직원(빈) 배치하고 운영 환경 제공  
매장 없으면 직원들 역할 못 함  

---
#### 📌 SecurityContext란
스프링 시큐리티에서 인증/인가 정보 관리  
SecurityContextHolder가 스레드별로 저장  
Authentication 객체로 사용자 정보 제공  

##### 실생활 비유
카페 직원의 배지  
누가 바리스타인지 매니저인지 확인  
요청마다 배지 체크로 권한 판단  

---
#### 📌 PersistenceContext란
JPA로 엔티티 관리  
1차 캐시, 변경 감지, DB 동기화  
트랜잭션 단위로 엔티티 생명주기 처리  

##### 실생활 비유
카페 주문 노트  
주문(엔티티) 기록하고 주방(DB)에 반영  
중복 주문 방지, 상태 관리  

---
#### 📌 코드로 살펴보기

##### ✅ ApplicationContext
```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class ContextExample {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        MyService myService = context.getBean(MyService.class);
        myService.doSomething();
    }
}
```

> 💡 팁  
> IntelliJ에서 `@SpringBootApplication`으로 자동 컨텍스트 설정  
> application.yml로 빈 설정 커스터마이징  

##### ✅ SecurityContext
```java
import org.springframework.security.core.context.SecurityContextHolder;

public class SecurityExample {
    public void printCurrentUser() {
        String username = SecurityContextHolder.getContext().getAuthentication().getName();
        System.out.println("Current user: " + username);
    }
}
```

> 💡 팁  
> application.yml에 `spring.security`로 JWT 설정 추가 가능  
> IntelliJ 디버깅으로 Authentication 객체 확인  

##### ✅ PersistenceContext
```java
import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import org.springframework.stereotype.Service;

@Service
public class UserService {
    @PersistenceContext
    private EntityManager entityManager;

    public void saveUser(String username) {
        User user = new User();
        user.setUsername(username);
        entityManager.persist(user);
    }
}
```

> 💡 팁  
> application.yml에 `spring.jpa`로 Hibernate 설정  
> IntelliJ JPA 콘솔로 엔티티 상태 추적  

---
#### 📌 비교 정리

##### ✅ 비유
- ApplicationContext: 카페 매장, 전체 운영 조율  
- SecurityContext: 직원 배지, 권한 확인  
- PersistenceContext: 주문 노트, 주문 관리  

##### ✅ 코드 관점
- ApplicationContext: 빈과 DI, 애플리케이션 전반  
- SecurityContext: 스레드별 인증, 요청 단위  
- PersistenceContext: 엔티티와 DB, 트랜잭션 단위  

##### ✅ 표로 정리
| 컨텍스트 | 관리 대상 | 생명주기 | 목적 |
|----------|-----------|----------|------|
| ApplicationContext | 빈 | 애플리케이션 | 환경 제공 |
| SecurityContext | 인증 정보 | 요청/스레드 | 보안 처리 |
| PersistenceContext | 엔티티 | 트랜잭션 | DB 동기화 |

---
#### 📌 싱글톤 빈과 스프링
ApplicationContext는 기본 싱글톤 빈 관리  
단일 인스턴스로 메모리 절약, 공유 용이  
상태 없는 빈으로 설계하면 스레드 안전  

##### ✅ 싱글톤 이유
- 메모리 효율, 일관성 유지  
- 의존성 주입 간소화  

##### ✅ 상태 없는 빈
```java
import org.springframework.stereotype.Service;

@Service
public class CalculatorService {
    public int add(int a, int b) {
        return a + b;
    }
}
```

- 내부 데이터 없음  
- 여러 스레드 동시 호출 안전  

##### ✅ 상태 있는 빈
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
- 동시성 문제 위험, `synchronized`나 다른 제어 필요  

##### ✅ 설계 팁
- 상태 없는 빈 선호, 유지보수 쉬움  
- 상태 있는 빈은 `@Scope("prototype")` 또는 동시성 처리  
- application.yml로 스코프 설정 가능  

---
#### 📌 Configuration vs Component
- Configuration: `@Bean`으로 빈 정의, 설계도 역할  
- Component: `@Component`로 자동 스캔, 실제 빈  

##### ✅ 코드 예시
```java
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyService();
    }
}

@Component
public class MyComponent {
    public void execute() {
        System.out.println("Executing...");
    }
}
```

> 💡 팁  
> IntelliJ로 `@ComponentScan` 범위 확인  
> application.yml에 `spring.main.allow-bean-definition-overriding: true`로 충돌 관리  

---
#### ✍ 결론
컨텍스트 처음엔 단순 저장소로 보였음  
각각 환경, 보안, DB 다루는 역할 깨달음  

싱글톤 빈으로 메모리 절약, DI 편리  
상태 없는 빈과 상태 있는 빈 차이로 스레드 안전성 이해  
Configuration과 Component로 빈 생성 흐름 파악  

컨텍스트와 빈 스코프 선택이 설계의 핵심  
스프링의 체계적인 구조에 감탄