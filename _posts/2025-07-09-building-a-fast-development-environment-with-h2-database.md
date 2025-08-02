---
layout: single
title: "H2 데이터베이스로 빠른 스프링부트 개발 환경 세팅"
date: 2025-07-09 14:12:23 +0900
categories: Spring Boot
tags: [H2]
typora-root-url: ../

---
#### 📌 용어 한눈에
- H2: 경량 인메모리 데이터베이스, 개발/테스트 특화  
- 인메모리: 메모리에 데이터 저장, 빠른 처리  
- JDBC: 자바와 DB 연결 표준  
- JPA: 객체와 DB 매핑 API  
- Hibernate: JPA 구현체, 매핑 처리  
- EntityManager: 엔티티 생명주기 관리  

---
#### 📌 H2 데이터베이스란
H2는 가볍고 빠른 DB  
개발 환경에서 빠른 테스트 가능  
인메모리 모드로 데이터 메모리 저장  
스프링부트와 통합 간편  

JPA와 Hibernate로 객체 기반 데이터 조작  
EntityManager가 H2와 엔티티 조율  

---
#### 📌 H2 쉽게 이해
노트북 메모장으로 비유  
메모장(H2)은 데이터 빠르게 기록  
노트북(애플리케이션)은 메모장 활용  
JPA는 메모장 사용 규칙  
Hibernate는 기록/수정 로직(ORM)  
EntityManager는 메모장 관리자, 데이터 정리  

이름 기록(데이터 저장) 요청  
사용자가 입력(엔티티)하면  
EntityManager가 정리 후 Hibernate 전달  
Hibernate가 메모장(H2)에 저장  

H2는 빠른 메모장  
JPA, Hibernate, EntityManager가 협력  

---
#### 📌 H2 설정

##### ✅ application.yml
```yaml
spring:
  datasource:
    url: ${H2_URL:jdbc:h2:mem:testdb}
    driver-class-name: org.h2.Driver
    username: sa
    password:
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
  h2:
    console:
      enabled: true
      path: /h2-console
```

인메모리 모드: `jdbc:h2:mem:testdb`  
H2 콘솔로 데이터 시각화  

> 💡 팁  
> 환경 변수 `H2_URL`로 DB 설정 동적 관리  
> IntelliJ H2 콘솔로 테이블 확인  

---
#### 📌 H2 개발 환경 구축

##### ✅ 의존성 추가
build.gradle에 추가  
```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
runtimeOnly 'com.h2database:h2'
```

> 💡 팁  
> Gradle 빌드 후 IntelliJ로 의존성 확인  

##### ✅ 엔티티 클래스
User 테이블 매핑  
```java
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Entity
@Table(name = "users")
@Getter
@Setter
@NoArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;

    private String email;

    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }
}
```

@Entity: 테이블 연결  
Lombok으로 코드 간소화  

메모장 비유로 User는 페이지  
username, email은 항목  
EntityManager가 기록 조정  

##### ✅ 리포지토리
CRUD 자동화  
```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
}
```

JpaRepository가 Hibernate 활용  
기본 메서드 제공  

##### ✅ 서비스
```java
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;

    public User saveUser(String username, String email) {
        User user = new User(username, email);
        return userRepository.save(user);
    }
}
```

saveUser는 메모장 기록  
EntityManager 조정, Hibernate 저장  

> 💡 팁  
> IntelliJ JPA 콘솔로 쿼리 확인  
> @Transactional로 트랜잭션 관리  

##### ✅ H2 콘솔
`http://localhost:8080/h2-console` 접속  
JDBC URL: `jdbc:h2:mem:testdb`  
테이블과 데이터 확인  

---
#### 📌 H2 장점
- 설치 없이 즉시 사용  
- 인메모리 모드로 빠른 테스트  
- H2 콘솔로 데이터 직관 확인  
- JPA, Hibernate 통합 간편  

---
#### 📌 주의점
- 인메모리 모드는 종료 시 데이터 삭제  
- 프로덕션에 부적합, 테스트용 추천  
- ddl-auto는 validate로 전환  
- H2 콘솔 프로덕션에서 비활성화  
- 영속성 컨텍스트 관리 주의  

---
#### 📌 H2로 개발의 가치
H2는 빠른 개발/테스트 지원  
JPA, Hibernate로 데이터 관리 간소화  
EntityManager가 연결 조율  
핵심 로직에 집중 가능  

---
#### ✍ 결론
H2 처음엔 단순 DB로 보였음  
메모장 비유로 역할 이해  
잘못된 설정은 테스트 왜곡  
H2로 개발 속도 높아짐  
설계와 로직에 더 몰두 가능