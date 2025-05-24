---
title: "라이브러리, 프레임워크, IDE 차이점 정리"
categories:
  - CS Basics
tags:
  - CS Basics
last_modified_at: 2025-06-04
---
#### 📌 용어 한눈에
- 라이브러리: 특정 기능을 모아둔 코드 뭉치, 내가 호출해서 쓰는 도구  
- 프레임워크: 애플리케이션 전체 구조를 잡아주는 뼈대, 흐름을 주도  
- IDE: 코드 작성부터 디버깅, 배포까지 돕는 통합 개발 환경  

---
#### 📌 개념 잡기

- 라이브러리  
필요한 기능을 내가 골라서 호출하는 코드 모음  
흐름은 개발자가 주도  
예: Lombok, Hibernate, Jackson, JavaScript의 Axios  

- 프레임워크  
애플리케이션의 전체 흐름과 구조를 미리 설계  
개발자는 정해진 규칙에 따라 빈칸 채우기  
흐름은 프레임워크가 주도  
예: Spring Boot, Django, JavaScript의 Next.js  

- IDE  
코딩, 디버깅, 테스트, 배포까지 한 번에 지원하는 소프트웨어  
개발 생산성을 끌어올리는 도구  
예: IntelliJ IDEA Ultimate, Visual Studio Code, Eclipse  

---
#### 📌 코드로 비교

- 라이브러리 예시  
```java
// Lombok으로 반복 코드 줄이기
@Data
@NoArgsConstructor
public class User {
    private String name;
    private int age;
}
```
Lombok은 getter/setter 같은 보일러플레이트 코드를 줄여주는 유틸성 라이브러리  
개발자가 어노테이션 붙여서 원하는 기능만 호출  

```java
// Hibernate로 DB 객체 다루기
User user = session.get(User.class, 1L);
```
Hibernate는 ORM 기능을 제공  
개발자가 메서드 호출로 DB 조작  

- 프레임워크 예시  
```java
// Spring Boot로 REST API 만들기
@RestController
@RequestMapping("/api")
public class UserController {

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}
```
Spring Boot가 서버 실행, 라우팅, 의존성 주입까지 제어  
개발자는 비즈니스 로직만 작성  

- IDE 예시  
IntelliJ IDEA Ultimate는 스프링부트 개발에 최적화  
코드 자동완성, Gradle 의존성 관리, Spring Initializr 연동, 테스트 실행, 디버깅까지 지원  
예: `application.yml` 자동 포맷팅, `@Autowired` 경고 감지  

---
#### 📌 한눈에 정리

| 구분         | 라이브러리                          | 프레임워크                          | IDE                                  |
|--------------|-------------------------------------|-------------------------------------|--------------------------------------|
| 핵심         | 기능 단위 코드                      | 애플리케이션 구조                   | 개발 환경                            |
| 흐름 제어     | 개발자                              | 프레임워크                          | 해당 없음                           |
| 예시         | Lombok, Hibernate, Jackson          | Spring Boot, Django, Next.js        | IntelliJ, VS Code, Eclipse          |
| 목적         | 재사용 가능한 기능                  | 일관된 설계와 흐름                  | 생산성 향상                         |

---
#### ✍ 도구를 대하는 마음

라이브러리, 프레임워크, IDE는 코드를 더 잘 짜기 위한 도구  
동시에 문제를 깊게 파고드는 시작점  

Lombok은 단순히 코드를 줄이는 게 아니라 생산성을 높이는 선택  
Hibernate는 DB를 객체로 바라보는 새로운 관점  
Spring Boot는 복잡한 애플리케이션 구조를 체계적으로 설계하는 기반  

도구의 철학과 흐름을 이해하고  
주체적으로 활용하는 개발자  
그게 시스템을 설계하는 사람의 첫걸음  

기능을 넘어 구조를, 구조를 넘어 태도를  
코드를 쓰는 건 결국 도구를 다루는 시선에서 시작