---
layout: single
title: "어떤 JDK를 써야 할까"
date: 2023-02-22 10:57:12 +0900
categories: java
tags: [JDK, SDKMan]
typora-root-url: ../
---

## 어떤 JDK를 써야 할까
> - JDK란
> - JDK 종류
> - SDKMan을 통한 JDK 목록/설치/관리
> - release 주기

<br>

## JDK란

JDK(Java Development Kit)는 자바 프로그래밍 언어를 사용하여 소프트웨어를 개발할 때 필요한 도구들을 포함하는 개발 환경

JDK에는 자바 컴파일러, 디버거, 자바 API 라이브러리 등이 포함

개발자는 JDK를 사용하여 자바 언어로 작성된 코드를 컴파일하고 실행할 수 있으며, 자바 애플리케이션, 앱렛, 서블릿 등을 개발

<br>

## JDK 종류

### 1. Adoptium Eclipse Temurin(강력 추천)

이클립스 재단의 Adoptium은 고품질의 Java 제품을 만들기 위한 프로젝트로 Adoptium Working Group에는 MS, IBM ,RedHat 등 쟁쟁한 오픈소스 기업들이 참여 중

Adoptium에서 개발하는 JDK는 Eclipse Temurin이라 부르는데 local, 운영, CI 환경 모두에 권장하는 제품

Termurin은 고품질에 벤더 중립적이고 TCK 테스트도 통과

### 2. Azul Systems JDK(권장)

검증된 Java 기술 전문 회사인 Azul Systems이 만드는 JDK로 Zulu는 무료로 사용할 수 있는 제품

예산이 있거나 전문 기술 지원을 받고 싶다면 상용 제품인 Azul Zing 구매해서 사용

### 3. Oracle JDK(비추천)

오라클 JDK는 자주 라이센스가 바뀌는데 17 버전은 Oracle No-Fee Terms and Conditions(NFTC) 라이센스

이 라이센스는 Internal Business operations는 무료로 사용할 수 있는데 문제는 이게 무엇인지 명확하게 정의되지 않음(라이센스 위반 여지)

### 4. Alibaba JDK(?!)

중국 정부랑 일한다면 사용

<br>

## SDKMan을 통한 JDK 목록/설치/관리

Local이나 개발서버라면 SDKMan을 사용해서 JDK를 관리하는 게 좋음

### SDKMan이란

SDK(Software Development Kit) Manager CLI는 커맨드 라인에서 다양한 종류의 Open JDK와 ant, gradle, maven등 Java기반 개발 도구를 설치하고 관리할 수 있게 해주는 command line 유틸리티

yum이나 apt, brew같은 패키지 매니저에 등록된 Open JDK는 벤더가 다양하지 않고 업데이트가 자주 되지 않으며 하나의 버전밖에 사용하지 못하는 단점이 있음

SDKMan은 RVM(Ruby Version Manager)처럼 다양한 벤더와 버전의 Open JDK를 사용할 수 있게 해주며 Linux와 OSX, Solaris 등 여러 Unix 계열 운영체제를 지원

### SDKMan 설치

SDKMAN은 의존성 최소화를 위해 shell script로 작성

curl과  zip/unzip만 있으면 잘 동작

먼저 sdkman 을 curl 로 설치

```bash
$ curl -s https://get.sdkman.io | bash
```

쉘의 환경 설정 파일(Ex: .bash_profile)의 맨 밑에 다음 내용을 추가하고 명령행에서도 실행(또는 logoff 후 다시 login)

```bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
```

제대로 설치되었는지 다음 명령어로 확인

```bash
$ sdk version

SDKMAN 5.11.5+713
```

### JDK 목록 보기

list(축약: l) 명령으로 설치 가능한 개발 도구와 버전 정보를 확인

```bash
$ sdk l

-------------------------------------------------------------------------------
Apache ActiveMQ (Classic) (5.16.2)                  https://activemq.apache.org/
...
```

특정 제품의 목록을 보려면 list 뒤에 제품명을 입력하면 되며 다음 명령은 지원하는 gradle 버전을 출력

```bash
$ sdk l gradle

7.2 61. 6.2
...
```

다음 명령으로 지원하는 OpenJDK 목록을 확인

```bash
$ sdk list java

================================================================================
Available Java Versions
================================================================================
 Vendor        | Use | Version      | Dist    | Status     | Identifier
--------------------------------------------------------------------------------
 AdoptOpenJDK  |     | 16.0.1.j9    | adpt    |            | 16.0.1.j9-adpt      
               |     | 16.0.1.hs    | adpt    |            | 16.0.1.hs-adpt      
               |     | 11.0.11.j9   | adpt    |            | 11.0.11.j9-adpt     
               |     | 11.0.11.hs   | adpt    |            | 11.0.11.hs-adpt     
               |     | 8.0.292.j9   | adpt    |            | 8.0.292.j9-adpt     
               |     | 8.0.292.hs   | adpt    |            | 8.0.292.hs-adpt     
 Alibaba       |     | 11.0.9.4     | albba   |            | 11.0.9.4-albba      
               |     | 8.5.5        | albba   |            | 8.5.5-albba         
 Amazon        |     | 16.0.1.9.1   | amzn    |            | 16.0.1.9.1-amzn     
               |     | 11.0.11.9.1  | amzn    |            | 11.0.11.9.1-amzn    
               |     | 8.292.10.1   | amzn    |            | 8.292.10.1-amzn   
...
```

눈여겨볼 부분은 Version 과 Dist 를 결합한 Identifier 이며 설치시에 꼭 필요

![sdkman](/images/2023-02-22-which-jdk-version-is-best/sdkman.png){: width="560"}

### JDK 설치

설치하려면 install java  명령어뒤에 Identifier 를 기술

#### Adoptium Eclipse Temurin

whichjdk.com에서 추천하는 JDK 구현물인 Temurin을 설치하려면 다음 명령을 실행

먼저 버전 목록을 확인

```bash
$ sdk l java |grep -i tem


 Temurin       |     | 17.0.0       | tem     |            | 17.0.0-tem          
               |     | 16.0.2       | tem     |            | 16.0.2-tem          
               |     | 11.0.12      | tem     |            | 11.0.12-tem         
               | >>> | 8.0.302      | tem     | installed  | 8.0.302-tem 
```

LTS 버전인 8, 11, 17을 설치

```bash
$ sdk i java 8.0.302-tem
$ sdk i java 11.0.12-tem  
$ sdk i java 17.0.0-tem
```

#### AdoptOpenJDK

AdoptOpenJDK는 다음 명령어로 설치

```bash
$ sdk i java 8.0.292.j9-adpt
```

11버전은 다음 명령어로 설치 가능

```bash
$ sdk i java 11.0.11.j9-adpt
```

### JDK 버전 관리

#### 기본 버전 설정

default 명령으로 사용할 기본 버전을 설정

```bash
$ sdk default java 8.0.292.hs-adpt

Default java version set to 8.0.292.hs-adpt
```

#### 현재 버전 보기

current 명령으로 현재 기본 버전을 확인

```bash
$ sdk current

Using:

gradle: 7.0.2
java: 11.0.11.9.1-amzn
```

#### 사용 버전 지정

use 명령으로 사용할 버전을 지정

다음은 기본 Java를 MS의 OpenJDK 11로 설정

```bash
$ sdk use java 11.0.11.9.1-ms
```

### JDK 개발도구 설치

#### gradle

list gradle 명령으로 설치 가능한 버전을 확인

```bash
$ sdk l gradle

================================================================================
Available Gradle Versions
================================================================================
 > * 7.3                 6.2.1               4.6                 2.7            
     7.3-rc-5            6.2                 4.5.1               2.6            
     7.3-rc-3            6.1.1               4.5                 2.5            
     7.3-rc-2            6.1                 4.4.1               2.4            
     7.3-rc-1            6.0.1               4.4                 2.3            
     7.2                 6.0                 4.3.1               2.2.1          
     7.2-rc-3            5.6.4               4.3                 2.2  
```

install gradle 명령어로 설치할 버전을 지정하여 설치

```bash
$ sdk i gradle 7.3
```

#### maven

list maven 명령으로 설치 가능한 버전을 확인

```bash
$ sdk l maven

================================================================================
Available Maven Versions
================================================================================
     3.8.4                                                                      
     3.8.3                                                                      
     3.8.2
```

install maven 뒤에 설치할 버전을 지정하여 설치

```bash
$ sdk i maven 3.8.4
```

#### tomcat

install tomcat 뒤에 설치할 버전을 지정하여 설치

```bash
$ sdk i tomcat 9.0.40
```

<br>

## Release 주기

새로운 기능을 추가한 major 버전은 1년에 2번, 3월과 9월에 발표, 추가로 분기별로 버그 픽스 업데이트

3년마다 9월에 발표되는 버전은 LTS(Long-Term Support)릴리스로 최소 3년 간 지원

그래서 JDK는 LTS인 8, 11, 17 중 고르는 게 좋음

<br>
