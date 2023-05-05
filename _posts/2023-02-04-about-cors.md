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

#### Simple request

다음과 같은 조건을 만족하면, 브라우저는 CORS 요청을 Simple request로 처리

- GET, POST, HEAD
- CORS safe-listed request header을 사용하였을 때
- Content-Type의 헤더 값으로 다음과 같은 값을 사용하였을 때
  - application/x-www-form-urlencoded
  - multipart/form-data
  - text/plain
- XMLHttpRequestUpload 객체에 이벤트 리스너가 등록되지 않은 경우
- 요청에서 ReadableStream 객체를 사용하지 않은 경우

이 방식은 일반 요청이랑 동일하게 동작하며, 응답의 Access-Control-Allow-Origin에 따라 브라우저가 응답을 정상적으로 처리할지 결정

예를 들어 http://localhost:8000에서 서버에 요청을 보낸다면 다음과 같이 Origin 헤더만 추가

```
GET /api/menus HTTP/1.1

Host: localhost:9000
Referer: http://localhost:8080/
Origin: http://localhost:8080
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
```

Access-Control-Allow-Origin 헤더가 존재하지 않기 때문에 브라우저는 다음과 같은 응답을 자바스크립트 코드에 노출시키지 않고 오류

```
HTTP/1.1 200 OK

Content-Type: application/json; charset=utf-8
Content-Length: 155
```
```console
교차 출처 요청 차단: 동일 출처 정책으로 인해 http://localhost:9000/api/menus에 있는 원격 자원을 차단하였습니다. (원인: ‘Access-Control-Allow-Origin’ CORS 헤더가 없음)
```

서버에 Access-Control-Allow-Origin 헤더를 추가해서 응답하면 다음과 같이 정상적으로 자바스크립트 코드에서 응답 처리

```
HTTP/1.1 200 OK

Access-Control-Allow-Origin: http://localhost:8080
Content-Type: application/json; charset=utf-8
Content-Length: 155
```

#### Preflight request

Simple request가 아니면 브라우저는 실제 요청을 보내기 전에 preflight request를 실제 요청 메시지보다 먼저 서버에 전송

서버는 이에 대한 응답을 주고, 브라우저는 이를 통해 실제 요청을 보낼지 결정

preflight request는 OPTIONS 메서드를 사용하며, 다음과 같은 요청 헤더를 사용

- Access-Control-Request-Method : 실제 요청에서 사용하는 메서드를 서버가 알 수 있도록 설정하는 헤더
- Access-Control-Request-Headers : 실제 요청에 포함될 헤더를 서버가 알 수 있도록 설정하는 헤더
- Origin : 요청을 보낸 출처로, URL 중 scheme과 host, port만 명시

preflight request는 브라우저가 자동으로 생성

응답이 오면, 브라우저는 방금 언급한 3가지 헤더값과 응답의 Access-Control-*헤더값을 비교하여 실제 요청을 서버에게 보낼지 결정

예를 들어, 다음과 같은 요청을 보낸다면 Simple request의 조건을 만족하지 못하므로, 실제 요청을 보내기 전에 preflight request를 보냄

- prefilght request

```
OPTIONS /api/menus HTTP/1.1

Host: localhost:9000
Access-Control-Request-Method: GET
Access-Control-Request-Headers: content-type
Referer: http://localhost:8080/
Origin: http://localhost:8080
```

서버가 다음과 같이 응답을 했다면, 브라우저는 실제 요청을 보내도 문제가 없다고 판단하여 실제 요청을 서버에 전송

- preflight request에 대한 응답 : 성공

```
HTTP/1.1 200 OK

Access-Control-Allow-Origin: http://localhost:8080
Access-Control-Allow-Headers: Content-Type
Access-Control-Max-Age: -1
Connection: keep-alive
Content-Length: 0
```

- 실제 요청을 서버로 전송

```
GET /api/menus HTTP/1.1

Host: localhost:9000
Referer: http://localhost:8080/
Content-Type: application/json
Origin: http://localhost:8080
```

하지만 서버가 다음과 같은 응답을 했다면, 브라우저는 실제 요청을 보내지 않고 오류

- preflight request에 대한 응답 : 실패

```
HTTP/1.1 200 OK

Access-Control-Allow-Origin: http://localhost:8080
Access-Control-Max-Age: -1
Connection: keep-alive
Content-Length: 0
```
```console
교차 출처 요청 차단: 동일 출처 정책으로 인해 http://localhost:9000/api/menus에 있는 원격 자원을 차단하였습니다. (원인: CORS 사전 점검 응답의 헤더 ‘Access-Control-Allow-Headers’에 따라 헤더 ‘content-type’가 허용되지 않음)
```

<br>

## Access-Control-Allow-Credentials

CORS는 기본적으로 보안상의 이유로 쿠키를 요청으로 보낼 수 없음

하지만 다른 도메인을 가진 API 서버에 자신을 인증해야 정상적인 응답을 받을 수 있는 경우, 쿠키를 통한 인증 필요

이를 위해서 두 가지 작업이 필요

- 요청을 credentials 모드로 설정

```javascript
// fetch case
fetch(url, {
  credentials: 'include'
}) 

// XHR case
const xhr = new XMLHttpRequest();
xhr.withCredentials = true;
```

- 서버에서 응답 헤더로 Access-Control-Allow-Crendentials: true 설정

CORS는 Access-Control-Allow-Credentials 헤더를 지원

서버가 Access-Control-Allow-Credentials: true로 응답하면, credentials 이용 요청 처리

prefilght request의 경우, 실제 요청과 함께 credentials를 보낼지 판단하는데 사용

Access-Control-Allow-Credentials: false이거나 생략한 경우, 실제 요청을 서버에 보낼 떄 credentials 정보를 함께 보내지 않음

간단한 GET 요청의 경우는 prelight 과정이 없기에, credentials 정보와 함께 CORS 요청

서버에서 Access-Control-Allow-Credentials: false이거나 생략한 경우, 브라우저가 응답을 자바스크립트 코드에 노출시키지 않고 전달하지 않음

<br>

## Access-Control-Allow-Origin

Access-Control-Allow-Origin은 하나의 origin이나 *, null만 가질 수 있음

그렇기에 서브 도메인의 접근 허용을 위해 *.example.com로 헤더값을 설정하는 것은 스펙상 맞지 않음

그래서 여러 출처의 접근 허용을 위해 * 값을 쓰는 경우가 많음

하지만 *는 모든 출처의 접근을 허용하는 것이기 때문에 보안상 좋지 않음

### Whitelist 관리를 통한 여러 출처 허용

여러 출처를 허용하고 싶다면, Origin 헤더를 이용한 Whitelist 관리를 통한 방법을 사용

요청이 오면 CORS 요청 헤더의 Origin 값을 확인하여, 별도로 관리하고 있는 Whitelist에 포함되고 있는지 확인

만약 존재한다면, Access-Control-Allow-Origin 헤더값으로 해당 Origin의 값을 넣어 응답할 것이고, 브라우저는 실제 요청을 서버에 보낼 것

악의적인 사이트에서 요청을 보냈더라도, Whitelist에 포함되어 있지 않기 때문에 안전하게 CORS 요청 처리

CORS의 Whitelist를 직접 처리할 수도 있지만, cors 미들웨어가 이미 지원하고 있기 때문에 다음과 같이 작성하면 Whitelist를 편하게 관리할 수 있음

```javascript
// express cors
// https://www.npmjs.com/package/cors#configuration-options
app.use(
  cors({
    origin: ["http://localhost:8080", "http://localhost:7070"],
  })
);
```

### scheme과 host, port 명시

Accecc-Control-Allow-Origin의 헤더 값은 origin 스펙을 만족하는 값이기 때문에 [scheme]://[host]:[port] 형태로 명시

Origin 스펙을 준수하지 않으면, 브라우저에서 Origin 헤더값과 잘 비교를 못할 수 있음

### Access-Control-Allow-Origin: *와 Access-Control-Allow-Credentials: true는 함께 사용 불가

CORS는 응답이 Access-Control-Allow-Credentials: true 을 가질 경우, Access-Controll-Allow-Origin의 값으로 *를 사용 불가

Access-Control-Allow-Credentials: true를 사용하는 경우는 사용자 인증이 필요한 리소스 접근이 필요한 경우인데, 만약 Access-Control-Allow-Origin: *를 허용한다면, CSRF 공격에 매우 취약해져 악의적인 사용자가 인증이 필요한 리소스를 마음대로 접근할 수 있음

그렇기 때문에 CORS 정책에서 아예 동작하지 않도록 막아버림

Access-Contorl-Allow-Credentials: true인 경우에는 반드시 Access-Control-Allow-Origin의 값이 하나의 origin 값으로 명시되어 있어야 정상적으로 동작

<br>
