---
layout: single
title: "gRPC에 대해"
date: 2023-02-20 11:37:40 +0900
categories: knowledge
tags: [gRPC, REST API]
typora-root-url: ../
---

## gRPC에 대해
> - RPC와 스텁(stub)
> - gRPC
> - gRPC와 RESTAPI 비교
> - 장단점

<br>

## RPC와 스텁(stub)

### RPC(Remote Procedure Call)

프로세스 간 통신(IPC)의 한 형태

떨어져 있거나 분산되어 있는 컴퓨터 간 통신 기술

![rpc](/images/2023-02-20-about-grpc/rpc.jpg){: width="560"}

그림처럼 클라이언트가 스텁을 거쳐 RPC의 런타임을 거처 다시 런타임, 스텁, 서버로 전달

### 스텁(stub)

RPC의 핵심 개념

매개변수(parameter) 객체를 메세지(Message)로 변환(Marshalling)/역변환(Unmarshalling)하는 레이어를 의미

- 클라이언트의 스텁은 함수 호출에 사용된 파라미터의 변환 및 함수 실행 후 서버에서 전달된 결과의 변환 담당
- 서버의 스텁은 클라이언트가 전달한 매개 변수의 역변환 및 함수 실행 결과 변환을 담당

![stub](/images/2023-02-20-about-grpc/stub.jpg){: width="560"}

<br>

## gRPC

### 정의

gRPC는 구글에서 개발한 RPC(RPC, Remote Procedure Call) 시스템이며 모든 환경에서 실행할 수 있는 오픈소스 고성능 RPC 프레임워크

HTTP/2기반, Java, C ++, Python, Ruby, JavaScript, Objective-C 및 C#에서 사용이 가능

### 프로토콜 버퍼(protocol buffer)

gRPC는 IDL(Interface Definition Language)로 프로토콜 버퍼(protocol buffer)를 사용

프로토콜 버퍼는 구조화된 데이터를 직렬화하기 위한 구글의 언어 중립적, 플랫폼 중립적, 확장 가능한 메커니즘

XML이나 JSON보다 더 작고 빠름

### 구조

애플리케이션, 프레임워크, 전송계층으로 나눠져 구성

![grpcarchitect](/images/2023-02-20-about-grpc/grpcarchitect.png){: width="560"}

### 채널

gRPC는 여러 서브채널을 열어서 통신하고 재사용하여 통신비용을 절약

![grpcchannel](/images/2023-02-20-about-grpc/grpcchannel.png){: width="560"}

<br>

## gRPC와 RESTAPI 비교

gRPC는 다음과 같이 프로토콜버퍼를 중심으로 통신하지만 REST API 의 경우 HTTP로 통신

gRPC-WEB은 gRPC로만 통신하는 웹서비스

![grpcrestapidiffer](/images/2023-02-20-about-grpc/grpcrestapidiffer.png){: width="560"}

<br>

## 장단점

### 장점

대부분의 아키텍쳐에 사용할 수 있지만 MSA에 가장 적합

많은 서비스 간의 API 호출로 인한 성능 저하를 개선

보안, API 게이트웨이, 로깅 등 개발이 수월 

![grpcadv](/images/2023-02-20-about-grpc/grpcadv.png){: width="560"}

MSA 환경에서 gRPC는 지연시간 감소

- 프로토콜 버퍼의 IDL만 정의하면 서비스와 메세지에 대한 소스코드가 자동으로 생성되고 데이터 송수신 가능(높은 생산성)
- HTTP/2 기반으로 통신하며 이 때문에 양방향 스트리밍이 가능
- HTTP/2 레이어 위에서 프로토콜 버퍼를 사용해 "직렬화된 바이트 스트림"으로 통신하여 JSON 기반의 통신보다 더 가볍고 통신 속도가 빠름
- 런타임환경과 개발환경 구축 난이도 쉬움 
- 다양한 언어를 기반

### 단점

- 브라우저에서 gRPC 서비스 직접 호출 불가능
- 하지만 프록시서버를 통해서 해결 가능

<br>
