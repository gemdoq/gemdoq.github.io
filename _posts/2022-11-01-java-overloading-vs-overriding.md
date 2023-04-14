---
layout: single
title: "오버로딩과 오버라이딩"
date: 2022-11-01 09:31:41 +0900
categories: java
tags: [Java, 오버로딩, overloading, 오버라이딩, overriding]
typora-root-url: ../
---


## 오버로딩과 오버라이딩
> - 오버로딩(Overloading)
> - 오버라이딩(Overriding)

<br>

## 오버로딩(Overloading)

### 오버로딩의 정의

오버로딩(Overloading)이라는 뜻은 사전적으로 `과적하다`라는 의미

### 오버로딩 예시
```java
public double computeArea(Circle c) { ... }
public double computeArea(Circle c1, Circle c2) { ... }
public double computeArea(Square c) { ... }
```

### 오버로딩의 조건

한 클래스 내에 이름이 같으면서 매개변수의 개수나 타입이 다른 메소드가 있을 경우, 

`오버로딩` 되었다고 한다.

즉, 자바의 한 클래스 내에 이미 사용하려는 이름과 같은 이름을 가진 메소드가 있더라도 

매개변수의 개수 또는 타입이 다르면, 같은 이름을 사용해서 메소드를 정의 가능

### 오버로딩을 사용하는 이유

1. 같은 기능을 하는 메소드를 하나의 이름으로 사용 가능(예시 : println)
2. 메소드의 이름을 절약하고 용도 이외에 발생할 수 있는 간섭 방지

<br>

## 오버라이딩(Overriding)

### 오버라이딩의 정의

부모 클래스로부터 상속받은 메소드를 자식 클래스에서 필요에 따라 `내용부만 새로 정의`

### 오버라이딩 예시
```java
class Point {
	int x;
	int y;

	String getLocation() {
		return "x :" + x + ", y :" + y;
	}
}

class Point3D extends Point {
	int z;
	String getLocation() { // 오버라이딩
		return "x :" + x + ", y :" + y + ", z:" + z;
	}
}
```

### 오버라이딩의 조건

선언부의 메소드의 이름, 매개변수, 리턴값이 모두 일치해야 함

### 어노테이션 @Override 

어노테이션(Annotation)이라는 것으로 직역하면 주석

일반적인 주석과 다르게 컴파일 시 검증하는 기능

### 오버라이딩 규칙

1. 자식 클래스에서 오버라이딩하는 메소드의 접근 제어자는 부모 클래스보다 더 좁게 설정불가
2. 예외(Exception)는 부모 클래스의 메소드보다 많이 선언불가
3. static 메소드를 인스턴스의 메소드로 또는 그 반대로 교체불가

<br>

## 오버로딩 vs 오버라이딩

이 둘은 이름만 비슷하고 명백히 다른 것

> 오버로딩 - 기존에 없는 새로운 메소드를 추가하는 것
>
> 오버라이딩 - 상속받은 메소드를 재정의 하는 것

<br>