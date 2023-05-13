---
layout: single
title: "다대다 관계의 문제점과 해결에 대해"
date: 2023-03-06 09:19:42 +0900
categories: knowledge
tags: [다대다 관계, 연관 테이블]
typora-root-url: ../
---

## 다대다 관계의 문제점과 해결에 대해
> - 다대다 관계의 예
> - 문제점
> - 해결

<br>

## 다대다 관계의 예

DB 모델링 과정에서 N:M(다대다) 관계가 나올 수 있다.

쉽게 생각할 수 있는 관계로는 학생과 과목의 관계를 들 수 있다.

학생은 학번, 이름, 학과에 대한 정보를 갖는다.

학번이 각각의 학생을 구별하는 PK가 된다.

![student](/images/2023-03-06-about-the-problems-and-solutions-of-nn-relationships/student.png)

학과는 과목코드, 과목명, 담당교수를 갖는다.

여기서는 과목코드가 각 과목을 구별하는 PK가 된다.

![subject](/images/2023-03-06-about-the-problems-and-solutions-of-nn-relationships/subject.png)

이 두 테이블의 관계를 보면 학생은 여러 과목을 수강할 수 있고, 한 과목은 여러 학생이 수강할 수 있다.

따라서 두 테이블은 양방향의 관계를 가진다.

만약 홍길동이 S1, S2를 수강하고 장길산은 S2와 S3를 수강했다고 생각해보자.

학생 테이블에서 학생의 수강 정보를 확인하기 위해서는 학생이 수강한 과목에 대한 정보가 필요하다.

또한 과목의 입장에서 과목의 수강생 정보를 관리하기 위해서는 수강 학생의 학번 정보가 유지되어야 한다.

그럼 테이블은 아래와 같이 구성된다.

![stu-sub](/images/2023-03-06-about-the-problems-and-solutions-of-nn-relationships/stu-sub.png)

<br>

## 문제점

위와 같이 구성된 DB 모델에는 다음과 같은 문제가 있다.

- 학생 테이블에서는 PK가 없기 때문에 학번만으로 데이터를 구분할 수 없다. 과목 테이블도 마찬가지다.
- 데이터가 중복되다 보니 어떤 테이블을 조회해야 할 지 명확하지 않다.
- 만약 홍길동이 수학과로 학과 정보가 변경된다면, 지금은 수강 과목이 2개 뿐이지만, 수강 과목이 많아질 수록 많은 수의 데이터를 수정해야 한다.

<br>

## 대책

N:M(다대다) 관계를 각각 1:N, 1:M으로 풀어줄 수 있게끔 수강정보를 담고 있는 연관 테이블을 추가한다.

![lecture](/images/2023-03-06-about-the-problems-and-solutions-of-nn-relationships/lecture.png)

<br>
