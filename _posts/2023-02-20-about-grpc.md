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
> - gRPC란
> - gRPC와 RESTAPI 비교
> - 장단점

<br>

## RPC와 스텁(stub)

### RPC

RPC(Remote Procedure Call)는 프로세스 간 통신 (IPC)의 한 형태로 떨어져 있거나 분산되어 있는 컴퓨터간의 통신을 위한 기술

![rpc](/images/2023-02-20-about-grpc/rpc.jpg){: width="560"}

그림처럼 클라이언트가 스텁을 거처 RPC의 런타임을 거처 다시 런타임, 스텁, 서버로 전달되는 과정이 들어가있는 것이 RPC의 매커니즘

### 스텁(stub)

스텁(Stub)은 RPC의 핵심 개념으로 매개변수(parameter) 객체를 메세지(Message)로 변환(Marshalling)/역변환(Unmarshalling)하는 레이어이며 클라이언트의 스텁과 서버의 스텁으로 나눠짐

- 클라이언트의 스텁은 함수 호출에 사용된 파라미터의 변환 및 함수 실행 후 서버에서 전달된 결과의 변환 담당
- 서버의 스텁은 클라이언트가 전달한 매개 변수의 역변환 및 함수 실행 결과 변환을 담당

![stub](/images/2023-02-20-about-grpc/stub.jpg){: width="560"}

<br>

## 

<br>

## 

<br>

## 

<br>

## 

<br>
