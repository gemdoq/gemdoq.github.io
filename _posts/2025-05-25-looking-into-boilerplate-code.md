---
title: "보일러플레이트 코드 알아보기"
categories:
  - CS Basics
tags:
  - CS Basics
last_modified_at: 2025-06-05
---

#### 📌 용어 한눈에
- 보일러플레이트 코드: 반복적으로 작성하는 형식적인 코드, 핵심 로직과는 무관  
- 추상화: 복잡한 코드를 단순화, 핵심만 남김  
- 템플릿: 미리 정의된 코드 구조, 일관성 유지  
- 어노테이션: 자바에서 메타데이터로 코드 간소화  
- 프레임워크: 애플리케이션 뼈대, 보일러플레이트 줄이는 도구  

---
#### 📌 보일러플레이트 코드란
핵심 로직과 상관없이  
작동을 위해 반복해서 작성해야 하는 코드  

예: 자바에서 클래스, 생성자, getter/setter, toString 같은 것들  
매번 비슷한 패턴으로 작성해야 하는 반복적인 코드  
이걸 **보일러플레이트 코드**라고 부름  

---
#### 📌 보일러플레이트 코드 예시
스프링부트에서 엔티티 클래스 만들 때 자주 마주치는 코드  

```java
public class User {
    private String name;
    private int age;

    public User() {}

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }

    @Override
    public String toString() { return "User{name='" + name + "', age=" + age + "}"; }
}
```  

핵심 로직은 없지만  
컴파일과 동작을 위해 꼭 작성해야 했던 코드  

---
#### 📌 보일러플레이트 줄이는 도구

##### ✅ Lombok
자바에서 보일러플레이트를 확 줄여주는 라이브러리  
어노테이션 몇 개로 위 코드 대체 가능  

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private String name;
    private int age;
}
```  

가독성 높이고, 실수 줄이고, 유지보수 편해짐  
IntelliJ에서 Lombok 플러그인 설치하면 코드 자동완성도 지원  

##### ✅ Hibernate
JPA 구현체로, SQL 직접 작성 없이 DB 작업 가능  
`@Entity`, `@Id`, `@GeneratedValue`로 반복적인 CRUD 코드 제거  
예: 엔티티 정의로 DB 테이블 매핑 간소화  

```java
@Entity
@Data
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private int age;
}
```

##### ✅ Spring Boot
복잡한 설정(서버, 의존성 주입, 빈 등록 등)을 자동화  
`application.yml`로 설정 간소화, 보일러플레이트 대폭 축소  
예: `@SpringBootApplication` 하나로 애플리케이션 초기화  

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

---
#### 📌 보일러플레이트가 생기는 이유
- 자바 같은 언어의 명시적 선언 요구  
- 코드의 안전성과 명확성 추구  
- 프레임워크나 시스템의 복잡한 구조  

반복이 많다고 꼭 나쁜 건 아님  
명확한 구조와 안정성을 위해 필요한 경우도 있음  

---
#### 📌 보일러플레이트 줄이는 의미
- 반복 줄여서 핵심 로직에 집중  
- 추상화로 코드 가독성과 유지보수성 향상  
- 개발 생산성 끌어올림  

Lombok, Hibernate, Spring Boot는 단순 편의 도구가 아니라  
효율적인 설계를 위한 **시작점**  
스프링부트 개발자라면 `application.yml` 잘 활용해서 설정 간소화 추천  

---
#### ✍ 느낀 점
보일러플레이트 코드는 처음엔 부담스러웠지만  
반복하다 보니 패턴이 익숙해지고 구조를 이해하는 계기 됨  

Lombok은 단순히 코드를 줄이는 게 아니라 시간 절약의 철학  
Hibernate는 DB를 객체로 바라보는 새로운 시각  
Spring Boot는 전체 흐름을 간소화하는 설계  

도구를 쓰는 건 단순히 편의를 넘어서  
**왜 이 도구가 필요한지**, **어떻게 설계를 단순화하는지** 고민하는 과정  
그 고민이 결국 좋은 코드를 만드는 첫걸음