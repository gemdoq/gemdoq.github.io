---
layout: single
title: "자바 String, StringBuffer, StringBuilder 비교"
date: 2023-01-04 09:48:37 +0900
categories: java
tags: [String, StringBuffer, StringBuilder]
typora-root-url: ../
---

## 자바 String, StringBuffer, StringBuilder 비교
> - String vs StringBuffer/StringBuilder
> - StringBuffer vs StringBuilder

<br>

## String vs StringBuffer/StringBuilders

String과 StringBuffer/StringBuilder 클래스의 가장 큰 차이점은 String은 불변(immutable) 속성

```java
String str = "hello";   // String str = new String("hello");
str = str + " world";  // [ hello world ]
```

기존 "hello" 값의 String 클래스 참조변수 str이 "hello world"라는 값을 가지는 새로운 메모리 주소를 가리키게 되고 처음 선언했던 "hello" 값이 할당되어 있던 메모리 영역은 Garbage로 남아있다가 GC(garbage collection)에 의해 사라짐

String 클래스는 불변 → 문자열을 수정하는 시점에 새로운 String 인스턴스가 생성

![string](/images/2023-01-04-string-vs-stringbuffer-vs-stringbuilder/string.png){: width="560"}

위와 같이 String은 `불변성(immutable)`을 가지기 때문에 변하지 않는 문자열을 자주 읽어들이는 경우 String을 사용시 고성능

그러나 문자열 추가, 수정, 삭제 등의 연산이 빈번하게 발생하는 알고리즘에 String 클래스를 사용하면 힙 메모리(Heap)에 많은 임시 가비지(Garbage)가 생성 → 어플리케이션 성능 저하

이를 해결하기 위해 Java에서 `가변성(mutable)`을 가지는 `StringBuffer/StringBuilder` 클래스 도입

String과는 반대로 StringBuffer/StringBuilder는 가변성을 가지기 때문에 .append() .delete() 등의 API를 이용하여 동일 객체내에서 문자열을 변경하는 것이 가능

따라서 문자열의 추가, 수정, 삭제가 빈번하게 발생할 경우, String 클래스가 아닌 StringBuffer/StringBuilder를 사용

```java
StringBuffer sb= new StringBuffer("hello");
sb.append(" world");
```

![string-buffer](/images/2023-01-04-string-vs-stringbuffer-vs-stringbuilder/string-buffer.png){: width="560"}

<br>

## StringBuffer vs StringBuilder

StringBuffer는 동기화 키워드를 지원하여 멀티쓰레드 환경에서 안전(thread-safe)

참고로, String도 불변성을 가지기 때문에 멀티쓰레드 환경에서 안정(thread-safe)

반대로 StringBuilder는 동기화를 지원하지 않기 때문에 멀티쓰레드 환경에서 사용하는 것은 비적합하며, 동기화를 고려하지 않는 단일쓰레드 성능은 StringBuffer보다 유리

![compare](/images/2023-01-04-string-vs-stringbuffer-vs-stringbuilder/compare.png){: width="560"}

<br>
