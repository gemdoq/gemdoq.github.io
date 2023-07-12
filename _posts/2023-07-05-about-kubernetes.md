---
layout: single
title: "쿠버네티스에 대하여"
date: 2023-07-05 18:40:41 +0900
categories: kubernetes
tags: [kubernetes]
typora-root-url: ../
---

## 쿠버네티스에 대하여
> - 쿠버네티스?
> - minikube vs kubernetes
> - ReplicationController vs ReplicaSet vs Deployment
> - kubectl apply(declarative) vs kubectl create(imperative)

<br>

## 쿠버네티스?

### 쿠버네티스Kubernetes

쿠버네티스 어원은 항해할 때 키잡이(helmsman)나 파일럿을 뜻하는데서 옴

K8s라는 표기는 K와 s와 그 사이 8글자를 나타내는 약식 표기

역할은 Docker Container Orchestration, 즉 도커 컨테이너를 지휘하듯 컨테이너를 띄우고 설정한 의도에 맞게 애플리케이션에 대한 상태를 유지 관리 역할 담당

![kubearch](/images/2023-07-05-about-kubernetes/kubearch.jpg){: width="560"}사용하는 이유는 모놀리틱 애플리케이션이 MSA로 전환되며 확장성, 가용성, 관리 편의성, 보안성 등을 추구하기 위함

좀 더 상세히는 애플리케이션 배포의 단순화, 하드웨어 활용도 높이기, 항상성 유지, 오토 스케일링, 관심사 분리로 개발 집중도 향상을 기대할 수 있음

<br>

## minikube vs kubernetes

일단 쿠버네티스kubernetes란 "개발을 가속하고, 유지관리를 간단히 하기 위해, 리눅스 컨테이너들의 클러스터를 단일 시스템으로 관리해주는 서비스"라 정의할 수 있다.

미니쿠베minikube는 "로컬 쿠버네티스 엔진"이라고 설명할 수 있다.

로컬 쿠버네티스 클러스터를 각종 OS에서 구현할 수 있다.

### kubernetes 핵심 특징

- 가벼움, 간단함, 높은 접근성
- 다양한 클라우드 환경에 대한 설계와 적용이 가능하고 높은 보안성
- 컴포넌트를 쉽게 교체할 수 있게끔 고도로 모듈화된 디자인

### minikube 핵심 특징

- 로컬 쿠버네티스
- 로드밸런서
- 다중클러스터

<br>

## ReplicationController vs ReplicaSet vs Deployment

ReplicationController와 ReplicaSet, 그리고 Deployment는 모두 쿠버네티스에서 Pod를 관리하는데 사용되는 컨트롤러

그러나 세 컨트롤러는 서로 다른 목적과 기능을 가짐

### ReplicationController

Kubernetes의 기본 컨트롤러

ReplicationController는 특정 수의 Pod를 유지하는 데 사용

Pod가 실패하면, ReplicationController는 새로운 Pod를 생성

### ReplicaSet

Kubernetes의 권장 컨트롤러

ReplicaSet은 Pod의 집합을 관리하는 데 사용

Pod의 개수를 유지하고, 상태를 모니터링하며, Pod가 실패하면 새로운 Pod를 생성

### Deployment

Kubernetes의 가장 강력한 컨트롤러

애플리케이션의 버전을 관리하는 데 사용

ReplicaSet을 사용하여 애플리케이션의 버전을 배포하고, 롤백함

<br>

## kubectl apply(declarative) vs kubectl create(imperative)

kubectl apply는 버전관리에 이점이 있는 방식인 반면, kubectl create는 테스트하거나 문제해결에 대한 좋은 방식

### kubectl apply(declarative)

선언적인 명령어이기 때문에 우리가 정의한 요구사항에 대한 목표만 정의할 뿐 그 목표에 도달하는 과정에서의 개별적 단계는 정의하지 않음

리소스에 대한 설정을 파일이름이나 표준입력에 의해 적용하는 방식이며, 리소스 명이 정의되어야 함

만약 리소스가 존재하지 않을 경우엔 새로 만들어질 것이고, 만약 자원이 존재한다면 이 명령은 에러를 발생시키지 않는다.

kubectl create(imperative)와 차이는 우리가 원하는 걸 디스크럽터(yaml, JSON)에 정의하면, Controller가 상태를 일치시키는 과정을 수행한다. 

### kubectl create(imperative)

명령적인 명령이기 때문에 목표에 도달하는 각 과정이나 단계에 대한 정의가 필요하기 때문에 CLI에 의해 객체가 생성되고 관리됨

리소스의 생성을 파일이나 표준입력에 의해 적용되는 방식이며, 만약 리소스가 존재하는 경우 명령어는 에러를 발생시킨다.

kubectl apply(declarative)와 차이는 무엇을 생성하고, 대체하고, 삭제할지에 대해 일일히 CLI를 통해 명령하기 때문에 절차적인 과정의 무시로 인해 필수적인 부분이 누락되어 에러가 발생할 수 있음

<br>
