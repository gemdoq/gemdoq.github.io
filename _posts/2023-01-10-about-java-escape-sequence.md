---
layout: single
title: "자바 특수문자, 괄호, 백슬래시, 따옴표 출력하기"
date: 2023-01-10 11:28:17 +0900
categories: java
tags: [Escape Sequence]
typora-root-url: ../
---

## 자바 특수문자, 괄호, 백슬래시, 따옴표 출력하기
> - Escape Sequence 정의
> - 종류
> - 사용 예시

<br>

## Escape Sequence 정의

이스케이프 시퀀스(Escape Sequence)는 자바 출력문에서 백슬래시가 등장하면 그 바로 다음 문자를 인식해 상황에 맞게 처리하는 것

자바에서 백슬래시와 따옴표는 문자 그 자체만으로는 출력되지 않는 특수문자

출력문에서는 첫 쌍따옴표부터 그 다음 쌍따옴표가 나올 때 까지 그 사이에 있는 것들만 출력하기 때문에 출력문 중간에 따옴표를 넣게 되면 거기서 출력 내용이 끝나는 것으로 인식

이를 출력하고 싶은 경우에는 \' 또는 \" 처럼 따옴표 바로 앞에 백슬래시 삽입

만약 백슬래시 자체를 출력하고 싶다면 \\로 백슬래시를 두 번 입력

## 종류

대표적 이스케이프 시퀀스(Escape Sequence)의 입력과 출력

![escape-sequence(1)](/images/2023-01-10-about-java-escape-sequence/escape-sequence(1).png){: width="350"}

`주의 : 괄호나 일반 슬래시는 \를 붙이지 않아도 됨`

![escape-sequence(2)](/images/2023-01-10-about-java-escape-sequence/escape-sequence(2).png){: width="350"}

<br>

## 사용 예시

![escape-sequence(3)](/images/2023-01-10-about-java-escape-sequence/escape-sequence(3).png){: width="440"}

\r 이전까지는 aaa bbb가 출력되다가, \r로 인해 커서가 줄의 맨 처음으로 돌아가고, 맨 앞의 a 위에 c가 덮어씌워져 caa bbb가 출력

<br>
