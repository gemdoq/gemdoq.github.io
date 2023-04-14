---
layout: single
title: "Web Server와 WAS"
date: 2022-11-10 09:53:33 +0900
categories: java
tags: [Web Server, WAS, Static Pages, Dynamic Pages]
typora-root-url: ../
---


## Web Server와 WAS
> - Static Pages와 Dynamic Pages
> - Web Server
> - WAS
> - 구분하는 이유

<br>

## Static Pages와 Dynamic Pages

![static-vs-dynamic](/images/2022-11-10-web-server-vs-was/static-vs-dynamic.png)

### Static Pages

  - Web Server는 파일 경로 이름을 받아 경로와 일치하는 file contents를 반환
  - 항상 동일한 페이지를 반환
  - image, html, css, javascript

### Dynamic Pages
  - 인자의 내용에 맞게 동적인 contents를 반환
    - 즉, 웹 서버에 의해서 실행되는 프로그램을 통해서 만들어진 결과물
  - Servlet: WAS 위에서 돌아가는 Java Program
    - 개발자는 Servlet에 doGet()을 구현

<br>

## Web Server

### 정의

웹 브라우저 클라이언트로부터 HTTP 요청을 받아 정적인 컨텐츠(.html .jpeg .css 등)를 제공하는 프로그램이 설치된 컴퓨터

### 기능

HTTP 프로토콜을 기반으로 하여 클라이언트(웹 브라우저 또는 웹 크롤러)의 요청을 서비스
- WAS를 거치지 않고 바로 정적 자원 제공
- 클라이언트(일반적으로 웹 브라우저)에서의 요청(request)을 WAS에 전달하고, WAS에서 처리된 결과를 클라이언트에게 응답(response)

### 예시

- Apache Server
- Nginx
- IIS(Windows 전용 Web 서버)

<br>

## WAS

### 정의

DB 조회나 다양한 로직 처리를 요구하는 동적인 컨텐츠를 제공하기 위해 만들어진 Application이 설치된 컴퓨터
  - HTTP를 통해 컴퓨터나 장치에 애플리케이션을 수행해주는 미들웨어(소프트웨어 엔진)
  - 웹 컨테이너(Web Container) 혹은 서블릿 컨테이너(Servlet Container)
  - Container란 JSP, Servlet 등을 구동할 수 있는 환경을 제공하는 소프트웨어

### 기능

- 프로그램 실행 환경과 DB 접속 기능 제공
- 여러 개의 트랜잭션(논리적인 작업 단위) 관리 기능
- 업무를 처리하는 비즈니스 로직 수행 

### 예시

- Tomcat
- JBoss
- Jeus
- Web Sphere

<br>

## 구분하는 이유

### Web Server가 필요한 이유
  - 클라이언트(웹 브라우저)에 이미지 파일(정적 컨텐츠)을 보내는 과정을 생각해보자.
    * 이미지 파일과 같은 정적인 파일들은 웹 문서(HTML 문서)가 클라이언트로 보내질 때 함께 가는 것이 아니다.
    * 클라이언트는 HTML 문서를 먼저 받고 그에 맞게 필요한 이미지 파일들을 다시 서버로 요청하면 그때서야 이미지 파일을 받아온다.
    * Web Server를 통해 정적인 파일들을 Application Server까지 가지 않고 앞단에서 빠르게 보내줄 수 있다.
  - 따라서 Web Server에서는 정적 컨텐츠만 처리하도록 기능을 분배하여 서버의 부담을 줄일 수 있다.

### WAS가 필요한 이유
  - 웹 페이지는 정적 컨텐츠와 동적 컨텐츠가 모두 존재한다.
    * 사용자의 요청에 맞게 적절한 동적 컨텐츠를 만들어서 제공해야 한다.
    * 이때, Web Server만을 이용한다면 사용자가 원하는 요청에 대한 결과값을 모두 미리 만들어 놓고 서비스를 해야 한다. 
    * 하지만 이렇게 수행하기에는 자원이 절대적으로 부족하다.
  - 따라서 WAS를 통해 요청에 맞는 데이터를 DB에서 가져와서 비즈니스 로직에 맞게 그때 그때 결과를 만들어서 제공함으로써 자원을 효율적으로 사용할 수 있다.

### Web Server와 WAS를 같이 사용하는 이유

![web-service-architecture](/images/2022-11-10-web-server-vs-was/web-service-architecture.png)

1. 기능을 분리하여 서버 부하 방지(분업)
- WAS는 DB 조회나 다양한 로직을 처리
- Web Server는 정적 컨텐츠 제공
2. 물리적으로 분리하여 보안 강화
- Web Server에서 SSL 암호화/복호화
- 접근 허용 IP 관리, 2대 이상의 서버에서의 세션 관리
3. 여러 대의 WAS를 연결 가능
- Load Balancing을 위해서 Web Server 사용
- fail over(장애 극복), fail back 처리에 유리
- 장애 극복에 쉽고 유연한 대응(=무중단 운영) 가능
4. 물리자원 활용의 유연성 증가
- 한 서버에서 PHP와 Java 동시 사용

즉, 자원 이용의 효율성 및 장애 극복, 배포 및 유지보수의 편의성을 위해 Web Server와 WAS를 분리하여 사용

<br>

