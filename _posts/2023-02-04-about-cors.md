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

### CORS response

CORS 요청이 온다면, 서버는 Access-Control-* 헤더를 메시지에 포함

- Access-Control-Allow-Origin : 요청을 허용할 출처를 명시할 때 사용하며, *를 사용하면 모든 출처의 리소스 요청을 허용
- Access-Control-Allow-Methods : 어떤 메서드를 허용할 것인지
- Access-Control-Allow-Headers : 어떤 헤더들을 허용할 것인지
- Access-Control-Max-Age : preflight 요청에 대한 응답을 브라우저에서 얼마만큼 캐싱하고 있을지 설정
- Access-Control-Expose-Headers : 브라우저가 스크립트에 노출시킬 헤더의 목록을 명시할 때 사용
- 기본적으로 다음 7가지 헤더(일명 CORS safe-listed header)는 따로 설정하지 않아도 노출
  - Cache-Control
  - Content-Language
  - Content-Length
  - Content-Type
  - Expires
  - Last-Modified
  - Pragma
- Access-Control-Allow-Credentials : 이하 서술

서버는 위와 같은 헤더를 통해 리소스에 대한 CORS 요청을 어느 수준까지 허용할 것인지 클라이언트(브라우저)에게 알려주고, 이를 바탕으로 브라우저는 추후동작을 결정

### CORS request

CORS 요청을 보낼 때, 브라우저는 해당 요청을 두 가지 방식으로 처리

Preflight request을 사용한 방식과 그렇지 않은 방식(Simple request)




<br>
