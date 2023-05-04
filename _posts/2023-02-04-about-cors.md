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

Cross-origin resource sharing(CORS)는 최초에 리소스를 제공한 출처(origin)와 다른 출처의 리소스를 요청하는 경우(cross-origin 요청), 특정 HTTP header를 사용하여 웹 애플리케이션의 cross-origin 요청을 브라우저가 제한적으로 허용하는 정책입니다.

## 필요성

기본적으로 보안상의 이유로 브라우저는 Same-origin policy(SOP, 동일 출처 정책)을 따릅니다. 그렇기 때문에 XMLHttpRequest와 fetch API는 동일 출처 정책에 따라 동일한 출처의 리소스만을 요청할 수 있습니다.

하지만 웹 애플리케이션을 구현하다보면, 다른 출처에 있는 리소스를 요청하는 경우가 많습니다. 일례로, SPA(Single Page Application)로 구현된 웹 애플리케이션에서 다른 도메인을 가진 API 서버에 요청하는 경우는 매우 비일비재합니다. 또한, web font을 요청하는 경우도 마찬가지입니다.

cross-origin 요청을 필요한 경우가 많아지고 있기 때문에, 이를 안전하게 처리할 정책이 필요했습니다. 그래서 CORS가 나온 것입니다. CORS는 cross-origin 요청을 제한적으로 허용함으로써 좀 더 안전하게 cross-origin 요청을 처리할 수 있는 정책입니다.






<br>
