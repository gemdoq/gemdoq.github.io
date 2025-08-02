---
layout: single
title: "JPA 기초와 엔티티 매핑 정리"
date: 2025-06-25 14:12:23 +0900
categories: Spring Boot
tags: [JPA, Hibernate, EntityManager]
typora-root-url: ../

---
#### 📌 용어 한눈에
- JPA: 자바 객체와 DB 매핑 표준 API  
- 엔티티: DB 테이블과 매핑되는 자바 클래스  
- ORM: 객체와 관계형 DB 연결 기술  
- Hibernate: JPA 구현 ORM 프레임워크  
- EntityManager: 엔티티 생명주기 관리 객체  
- @Entity: 엔티티 클래스 표시  
- @Id: 기본 키 지정  

---
#### 📌 JPA란
JPA는 객체 중심 DB 관리 표준  
SQL 직접 작성 없이 데이터 처리  
스프링부트에서 Spring Data JPA로 간편 사용  

Hibernate는 JPA 구현체로 매핑 실행  
EntityManager는 Hibernate와 엔티티 조율  

엔티티 매핑은 객체-테이블 관계 정의  
데이터 작업 효율 높임  

---
#### 📌 JPA Hibernate EntityManager 이해
스마트 TV로 비유  
TV(DB)는 콘텐츠(데이터) 저장  
리모컨(엔티티)은 TV 조작 인터페이스  
JPA는 리모컨 표준 설계도  
Hibernate는 리모컨 회로(ORM), 신호 처리  
EntityManager는 버튼 관리자, 신호 조정  

채널 변경(데이터 수정) 요청  
리모컨 버튼(속성) 누르면  
EntityManager가 Hibernate에 전달  
Hibernate가 TV에 신호 보내 변경  

마리오네트 인형 비유  
인형(DB)은 줄(매핑)로 움직임  
인형사(엔티티)가 줄 조작  
JPA는 줄 조작 규칙  
Hibernate는 줄 연결(ORM)  
EntityManager는 인형사 손, 당기는 방식 결정  

JPA 표준 제공  
Hibernate 매핑 실행  
EntityManager 조율 역할  

---
#### 📌 JPA 설정
application.yml에서 시작  

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
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
```

H2 DB 예시  
ddl-auto로 테이블 자동 생성  
Hibernate 매핑 처리  

> 💡 팁  
> IntelliJ H2 콘솔로 DB 확인  
> application.yml에 환경 변수로 DB URL 동적 설정  

---
#### 📌 엔티티 매핑 구현

##### ✅ 의존성 추가
build.gradle에 JPA 의존성  
```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
runtimeOnly 'com.h2database:h2'
```

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

@Entity: 테이블 매핑  
@Id: 기본 키  
Lombok으로 getter/setter 간소화  

리모컨 비유로 User는 리모컨  
username, email은 버튼  
EntityManager가 신호 전달  

##### ✅ 리포지토리
CRUD 자동화  
```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
}
```

EntityManager와 Hibernate 활용  
기본 메서드 제공  

##### ✅ 서비스 사용
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

saveUser는 버튼 누르기  
EntityManager 조정  
Hibernate DB 저장  

> 💡 팁  
> IntelliJ로 JPA 쿼리 콘솔 확인  
> @Transactional로 트랜잭션 관리  

---
#### 📌 주요 어노테이션
- @Table: 테이블 이름  
- @Column: 컬럼 속성  
- @GeneratedValue: 키 생성 전략  
- @ManyToOne, @OneToMany: 관계 매핑  
- @Transient: 매핑 제외  

---
#### 📌 주의점
- @Id 필수  
- 기본 생성자 필요  
- ddl-auto 프로덕션에서 validate 추천  
- 관계 매핑 순환 주의  
- EntityManager 영속성 컨텍스트 이해  

---
#### 📌 JPA 매핑의 가치
JPA 표준으로 편의성 제공  
Hibernate 복잡 매핑 처리  
EntityManager 생명주기 관리  

매핑 잘하면 유지보수 향상  
스프링부트 설정 줄여 로직 집중  

---
#### ✍ 결론
JPA Hibernate EntityManager 처음엔 헷갈렸음  
객체-DB 패러다임 연결의 묘미  
코드 넘어 설계 집중 가능