---
layout: single
title: "객체 지향 프로그래밍(OOP)의 특징(characteristics)"
date: 2022-10-22 10:55:35 +0900
categories: knowledge
tags: [OOP, 객체지향프로그래밍, 캡슐화, 상속, 다형성]
typora-root-url: ../
---


## OOP(Object Oriented Programming)의 특징
> - OOP의 정의(Definition of OOP)
> - 캡슐화(Encapsulation)
> - 상속(Inheritance)
> - 다형성(Polymorphism)

<br>

## OOP의 정의

OOP(Object Oriented Programming, 객체 지향 프로그래밍)란 

큰 기능을 작게 쪼개는 것이 아니라, 

먼저 `작은 기능들을 독립적으로 담당하는 객체들`을 만든 뒤, 

이 `객체들을 조립해서 큰 기능을 구현하는 상향식(Bottom-up)` 프로그래밍 방식

![oop](/images/2022-10-24-characteristics-of-object-oriented-programming/oop.jpeg)

객체를 일단 한번 독립성/신뢰성이 높게 만들어 놓기만 하면, 

이후엔 그 객체를 수정 없이 재사용 가능 → 개발 기간과 비용이 대폭 감소

<br>

## 캡슐화(Encapsulation)

변수와 함수를 하나의 단위로 묶는 것을 의미(데이터의 bundling)

- 대개 bundling은 클래스를 통해 구현
- 해당 클래스의 인스턴스 생성 → 멤버 변수와 메소드에 접근 가능
- 클래스는 객체 지향 프로그래밍을 지원하는 거의 대부분의 언어가 제공하는 제1요소

### 정보 은닉(Information Hiding)

캡슐화로부터 파생된 개념

프로그램의 세부 구현을 외부로 드러나지 않는 것을 의미

![encapsulation](/images/2022-10-24-characteristics-of-object-oriented-programming/encapsulation.png){: width="560"}

객체 지향 언어에서 클래스 외부에서는 바깥으로 노출된 특정 메소드에만 접근이 가능

클래스 내부에서 어떤 식으로 처리가 이루어지는지는 알 수 없음

내부의 구현은 감추고(응집도↑), 외부로의 노출을 최소화(결합도↓) 

→ 유연함과 유지보수성 ↑

<br>

## 상속(Inheritance)

상속은 자식 클래스가 부모 클래스의 특성과 기능을 그대로 물려받는 것

기능의 일부를 변경해야 할 경우, 

### 오버라이드(Overriding)

자식 클래스가 부모 클래스에게서 상속받은 기능을 재정의하는 것

→ 캡슐화를 유지하면서도 클래스의 재사용 용이

<br>

## 다형성(Polymorphism)

하나의 변수, 또는 함수가 서로 다른 클래스의 객체에서 

각자의 방식과 상황에 따라 각자 다르게 해석되고 동작하는 성질

![polymorphism](/images/2022-10-24-characteristics-of-object-oriented-programming/polymorphism.png){: width="560"}

### 다형성을 사용하지 않을 경우
```java
  public class Dog {
      public void bark(){ System.out.println("멍멍"); }
  }
  public class Cat {
      public void meow(){ System.out.println("야옹"); }
  }
  public class Duck {
      public void quack(){ System.out.println("꽥꽥"); }
  }
  
  public class Main {
      public static void main(String[] args) {
        Dog dog = new Dog();
        Cat cat = new Cat();
        Duck duck = new Duck();
        // 애완동물 세 마리의 울음소리 호출
        dog.bark(); 
        cat.meow(); 
        duck.quack();
      }
  }
```
### 다형성을 사용할 경우
```java
// 부모 클래스
public abstract class Animal {
      public abstract void sound();
}
// 자식 클래스
public class Dog extends Animal {
      public void sound(){ System.out.println("멍멍"); }
}
public class Cat extends Animal {
      public void sound(){ System.out.println("야옹"); }
}
public class Duck extends Animal {
      public void sound(){ System.out.println("꽥꽥"); }
}

public class Main {
      public static void main(String[] args) {
        Animal[] animals = { new Dog(), new Cat(), new Duck() };
        // 애완동물 세 마리의 울음소리 호출
        for (int i = 0; i < 3; i++){
          // 실제 참조하는 객체에 따라 talk 메서드가 실행된다.
          animals.sound();
        }
      }
}
```

다형성을 이용해 오버라이딩과 오버로딩 등을 적용하면 

불필요한 코드의 중복을 막고 기작성 코드의 사용성 증대 및 코딩효율 증가

<br>
