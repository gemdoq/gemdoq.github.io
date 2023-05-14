---
layout: single
title: "자바스크립트에서의 객체 접근법 두 가지, 점표기법 vs 괄호표기법에 대해"
date: 2023-03-09 11:15:13 +0900
categories: javascript
tags: [리터럴, 생성자, Dot notation, Bracket notation]
typora-root-url: ../
---

## 자바스크립트에서의 객체 접근법 두 가지, 점표기법 vs 괄호표기법에 대해
> - 객체 접근법이란
> - 점 표기법
> - 괄호 표기법

<br>

## 객체 접근법이란

객체(Object)는 key와 value pair(키값쌍) 형태로 된 데이터

key는 문자열, value는 어느 데이터든 가능

참조형 데이터 타입이며, 배열도 객체, 함수도 객체

자바스크립트에서 객체 선언 방식은 다음과 같이 리터럴 방식과 생성자 방식 존재

### 객체 리터럴 방식

가장 일반적인 방법은 중괄호{ } 를 사용하여 객체를 생성하는 방법

```javascript
let person = {
    name: "gemdoq",
    email: "abc@example.com",
    birth: "0608"
}
```

위와 같이 정의한 person은 name, email, birth 프로퍼티를 갖는 객체

리터럴이란 고정된 값을 명시하는 문법 의미

리터럴 방식으로 객체를 정의할 때는 중괄호{ } 안에 프로퍼티 : 값의 쌍을 입력하고 쉼표(,)로 프로퍼티를 구분

즉, 프로퍼티와 값은 콜론(:)으로 정의하고, 각 데이터는 쉼표(,)로 구분

### 객체 생성자 방식

new 키워드를 이용하여 Object 생성자 함수를 호출하여 객체를 생성하는 방법

```javascript
let person = new Object();

person.name = "gemdoq";
person.email = "abc@example.com";
person.birth = "0608";
```

new Object()를 호출하면 비어있는 객체를 생성

따라서 생성 직후 person 객체에는 name, email, birth 프로퍼티가 없는 상태이므로, new Object()로 비어있는 객체를 생성했으면 프로퍼티를 추가해야 함

<br>

## 점 표기법(Dot Notation)

```javascript
let obj = {
	cat: 'meow',
	dog: 'woof'
};

let sound = obj.cat; // objectName.propertyName;
console.log(sound); // meow
```

객체명.프로퍼티로 해당 값을 호출하는 방식

프로퍼티는 알파벳으로 시작되어야 하며, 숫자로 시작할 수 없음

(_ & $ 포함 가능)

<br>

## 괄호 표기법(Bracket Notation)

```javascript
let obj = {
	cat: 'meow',
	dog: 'woof'
};

let sound = obj.['cat']; // objectName["propertyName"]
console.log(sound); // meow
```

객체명.['프로퍼티']로 해당 값을 호출하는 방식

프로퍼티는 문자열(String)이므로, 공백 포함 가능 및 숫자로 시작 가능

해당 프로퍼티가 만약 '문자열(String)'을 참조하고 있는 변수라면, 프로퍼티로 변수도 사용 가능

<br>
