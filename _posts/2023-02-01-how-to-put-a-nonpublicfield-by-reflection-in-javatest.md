---
layout: single
title: "자바 테스트 시, public이 아닌 필드를 Reflection으로 넣어주는 방법"
date: 2023-02-01 10:19:51 +0900
categories: java
tags: [Reflection]
typora-root-url: ../
---

## 자바 테스트 시, public이 아닌 필드를 Reflection으로 넣어주는 방법
> - Reflection 정의
> - 예시
> - 해결

<br>

## Reflection 정의

자바에서 구체적인 클래스 타입을 알지 못하더라도 그 클래스의 메서드, 타입, 변수들에 접근할 수 있도록 해주는 API

컴파일 시간이 아닌 실행 시간에 동적으로 특정 클래스의 정보를 추출하여 객체를 생성하거나 변수를 변경할 수 있고 메소드를 호출할 수도 있음

테스트 코드 작성을 위해 private 변수를 변경할 때 리플렉션을 사용할 수 있음

Reflection은 다음과 같은 정보를 가져와서 객체를 생성하거나 메소드를 호출하거나 변수의 값을 변경할 수 있음

- Class
- Constructor
- Method
- Field

<br>

## 예시

다음과 같은 Entity를 DTO로 변환해주는 Service Layer의 Unit test를 진행하고자 한다.

![service](/images/2023-02-01-how-to-put-a-nonpublicfield-by-reflection-in-javatest/service.png)

![dto](/images/2023-02-01-how-to-put-a-nonpublicfield-by-reflection-in-javatest/dto.png)

위와 같이 DTO 내부를 보면 Post가 만들어진 LocalDate 타입 createdAt을 클라이언트에게 반환해주어야 하는 상황

![entity](/images/2023-02-01-how-to-put-a-nonpublicfield-by-reflection-in-javatest/entity.png)

Post Entity는 JPA Auditing 적용되어 BaseEntity를 extends하고 createdAt, updatedAt을 자동 생성

![baseentity](/images/2023-02-01-how-to-put-a-nonpublicfield-by-reflection-in-javatest/baseentity.png)

createdTime의 접근 지정자를 보면 외부에서 접근할 수 있는 public이 아닌 protected

작성한 테스트 코드

![testcode](/images/2023-02-01-how-to-put-a-nonpublicfield-by-reflection-in-javatest/testcode.png)

Repository를 Mocking 하고(given) Service를 호출(when)

![runfailnpe](/images/2023-02-01-how-to-put-a-nonpublicfield-by-reflection-in-javatest/runfailnpe.png)

NPE가 발생

원인은 Entity -> DTO 변환할 때 createdAt 값이 필요한데 넣지 않았기 때문

![entitymethod](/images/2023-02-01-how-to-put-a-nonpublicfield-by-reflection-in-javatest/entitymethod.png)

Post Entity를 만드는 createPost() 메소드에서 createdAt을 넣어주고 싶지만, createdAt은 부모 클래스인 BaseEntity에 존재하기 때문에 값을 넣을 수 없음

![basemethod](/images/2023-02-01-how-to-put-a-nonpublicfield-by-reflection-in-javatest/basemethod.png)

그래서 위와 같이 값을 넣어주기 위해서 필드에 Setter 어노테이션을 만들었습니다.

![setter](/images/2023-02-01-how-to-put-a-nonpublicfield-by-reflection-in-javatest/setter.png)

그래서 위와 같이 BaseEntity에 setter를 사용

테스트 코드에서 꺼낸 Post Entity에 LocalDateTime 값을 설정 

가능은 하지만, 테스트로 인해 운영 코드에 Setter를 추가하는 것은 바람직하지 못함 

<br>

## 해결

이럴 때 사용할 수 있는 것이 ReflectionTestUtils 클래스

setField를 사용하여 값을 지정해 넣는 방법

- 자식 클래스 객체 (getClass 아님)
- 부모 클래스.class
- 부모 클래스에서 set Reflection 할 필드 이름
- 필드 타입의 값 (현재라면 LocalDateTime)
- 필드 타입.class

위 Reflection 매개변수를 통해 Post Entity의 public이 아닌 protected의 createdAt 필드 값을 지정

![reflection](/images/2023-02-01-how-to-put-a-nonpublicfield-by-reflection-in-javatest/reflection.png)

그리고 테스트를 실행하면 테스트 성공

<br>
