---
layout: single
title: "자바 String Constant Pool에 대해"
date: 2023-01-12 10:17:18 +0900
categories: java
tags: [String Constant Pool, String literal, intern메서드]
typora-root-url: ../
---

## 자바 String Constant Pool에 대해
> - String 객체를 생성하는 방법
> - String Constant Pool
> - String interning

<br>

## String 객체를 생성하는 방법

Java에서 String 객체를 생성하는 방법은 2가지가 있음

1. String literal, 즉 큰 따옴표("")를 사용하는 것
2. new 연산자를 사용하는 것

![test1](/images/2023-01-12-about-java-string-constant-pool/test1.png){: width="440"}

간단한 테스트 결과, String literal로 생성한 객체는 내용이 같다면 같은 객체, 즉 동일한 메모리 주소를 가리키고 있음

하지만 new 연산자로 생성한 String 객체는 내용이 같더라도 개별적인 객체임을 알 수 있음

<br>

## String Constant Pool

흔히 new 연산자로 String 객체를 생성하지 않는 것이 좋다는 말이 있음

String literal로 생성하면 해당 String 값은 Heap 영역 내 "String Constant Pool"에 저장되어 재사용되지만, new 연산자로 생성하면 같은 내용이라도 여러 개의 객체가 각각 Heap 영역을 차지하기 때문

![string-pool](/images/2023-01-12-about-java-string-constant-pool/string-pool.png){: width="560"}

생성된 String 객체는 Stack 영역에 저장

String literal로 생성한 객체는 "String Pool"에 위치

String literal로 생성한 객체의 값(ex. "Cat")이 이미 String Pool에 존재한다면, 해당 객체는 String Pool의 reference를 참조

new 연산자로 생성한 String 객체는 같은 값이 String Pool에 이미 존재하더라도, Heap 영역 내 별도의 객체를 생성해서 참조

<br>

## String interning

String 클래스에는 intern()이라는 메서드가 존재

intern()은 해당 String과 동등한(equal; 값이 같음) String 객체가 이미 String Pool에 존재하면 그 객체를 그대로 반환

그렇지 않다면, 호출된 String 객체를 String Pool에 추가하고 객체의 reference 반환

![test2](/images/2023-01-12-about-java-string-constant-pool/test2.png){: width="560"}

간단한 테스트 결과, new 연산자를 사용해 생성한 String 객체는 String Pool 바깥에 있었지만, intern() 메서드를 호출한 후엔 String Pool로 이동해 String Pool에 원래 있던 객체와 동일한 reference값 확인

String 객체를 new 연산자로 생성하면, 같은 값이라 할지라도 Heap 영역에 매번 새로운 객체가 생성

따라서, 메모리를 효율적으로 사용하기 위해서는 String literal(큰 따옴표)로 String 생성

<br>
