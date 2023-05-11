---
layout: single
title: "curl에 대하여"
date: 2023-02-25 09:38:17 +0900
categories: knowledge
tags: [curl]
typora-root-url: ../
---

## curl에 대하여
> - curl이란
> - 설치 및 주요 옵션
> - HTTP/HTTPS Download(GET Method)
> - HTTP 인증
> - POST/File Upload(POST, PUT)
> - PATCH, DELETE 메서드
> - HTTP Header 설정
> - SSL/TLS 옵션
> - Cookie
> - REST API 와 연계

<br>

## curl이란

curl은 Linux/Unix계열 및 Windows 등 주요 OS에서 구동되는 command line용 data transfer tool

Download/Upload 모두 가능하며, HTTP/HTTPS/FTP/LDAP/SCP/TELNET/SMTP/POP3 등 주요한 프로토콜을 지원

libcurl이라는 PHP, ruby, PERL 및 여러 언어에 바인딩되어 있는 C기반 library가 제공되므로 C/C++ 프로그램 개발 시 위의 protocol과 연계가 필요하다면 libcurl을 사용하여 손쉽게 연계 가능


<br>
