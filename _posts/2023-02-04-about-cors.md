---
layout: single
title: "CORS에 대하여"
date: 2023-02-04 09:57:42 +0900
categories: knowledge
tags: [CORS]
typora-root-url: ../
---

## CORS에 대하여
> - 정의
> - 필요성
> - request와 response
> - Access-Control-Allow-Credentials
> - Access-Control-Allow-Origin

<br>

## 정의

CORS(Cross-origin resource sharing, 교차 출처 자원 공유)는 최초에 리소스를 제공한 출처(origin)와 다른 출처의 리소스를 요청하는 경우(cross-origin 요청), 특정 HTTP header를 사용하여 웹 애플리케이션의 cross-origin 요청을 브라우저가 제한적으로 허용하는 정책

> Origin(출처)란 URL의 scheme(protocol), host(domain), 그리고 port로 이루어진 값(객체)을 의미<br>
> Origin이 같다는 것은 URL의 scheme과 host, port가 모두 같음을 의미

<br>

## 필요성

기본적으로 보안상의 이유로 브라우저는 Same-origin policy(SOP, 동일 출처 정책)을 따르기 때문에 XMLHttpRequest와 fetch API는 동일 출처 정책에 따라 동일한 출처의 리소스만을 요청할 수 있음

하지만 웹 애플리케이션을 구현하다보면, 다른 출처에 있는 리소스를 요청하는 경우가 많음

- SPA(Single Page Application)로 구현된 웹 애플리케이션에서 다른 도메인을 가진 API 서버에 요청하는 경우
- Web Font를 요청하는 경우

cross-origin 요청을 필요한 경우가 많아지고 있기 때문에, 이를 안전하게 처리할 정책이 필요

CORS는 cross-origin 요청을 제한적으로 허용함으로써 좀 더 안전하게 cross-origin 요청을 처리

<br>

## request와 response




<br>
