---
layout: single
title: "AWS EC2에서 스프링부트 실행 실패 문제"
date: 2024-12-27 16:47:23 +0900
categories: springboot
tags: [JDK, JRE, AWS EC2]
typora-root-url: ../

---

# AWS EC2에서 스프링부트 실행 실패 문제: "Cannot find a Java installation"
## 🏗️ 초기 설정
- 서버: Amazon Linux 2023 on AWS EC2
- 목표: 스프링부트 프로젝트 배포 (Java 21 필요)
- 진행:  
    1. dnf search java-21 실행
    2. java-21-amazon-corretto.x86_64 설치
    3. Git에서 프로젝트 클론
    4. dev.env 파일 생성
    5. env $(grep -v '^#' dev.env | xargs) ./gradlew bootRun 실행

---

## 🚨 문제 발생
Spring Boot 애플리케이션을 실행하려고 다음 명령어를 입력
```shell
env $(grep -v '^#' dev.env | xargs) ./gradlew bootRun
```

### 오류 메시지
아래와 같은 현상 발생과 함께 스프링부트 실행 실패
<img width="1400" alt="스크린샷" src="https://github.com/user-attachments/assets/ef46b9be-8fef-4866-82e5-aa6971e774d0">

```
FAILURE: Build failed with an exception.

* What went wrong:
Could not determine the dependencies of task ':bootRun'.
> Could not resolve all dependencies for configuration ':runtimeClasspath'.
   > Failed to calculate the value of task ':compileJava' property 'javaCompiler'.
      > Cannot find a Java installation on your machine: {languageVersion=21, vendor=any vendor, implementation=vendor-specific} for LINUX on x86_64.
         > No locally installed toolchains match and toolchain download repositories have not been configured.
```
- **현상**: Gradle이 Java 21을 요구했지만, 시스템에서 이를 찾지 못했다고 보고함
- **이상한 점**: JDK 21은 이미 설치되어 있고, java -version과 ./gradlew --version에서도 정상적으로 인식됨

---

## 💻 문제 원인 확인

### java-21-amazon-corretto와 java-21-amazon-corretto-devel의 차이

dnf(패키지매니저)에는 search라는 명령어 존재
`dnf search java-21`로 `java-21` 검색 결과: 
<img width="800" alt="스크린샷" src="https://github.com/user-attachments/assets/fab48c91-b1c4-4bd6-b35c-d01b449646df">
- java-21-amazon-corretto.x86_64: Amazon Corretto development environment  
- java-21-amazon-corretto-devel.x86_64: Amazon Corretto 21 development tools  
    확인 완료: java-21-amazon-corretto는 JRE, -devel이 JDK 제공

### Gradle과 스프링부트 프로젝트에서 JDK가 필요한 이유

처음 설치한 `java-21-amazon-corretto.x86_64`는 `JDK`가 아니라 `JRE`만 포함하고 있었기 때문에, Gradle이 `컴파일러(javac)`를 찾지 못해 에러가 발생
반면, `java-21-amazon-corretto-devel`은 `JDK`를 포함하므로 `컴파일러(javac)`가 설치되어 Gradle이 **정상적으로 동작**

## 🏆 문제 해결

1. 새 EC2 인스턴스 생성
2. `dnf search java-21` 실행
3.  `sudo dnf install java-21-amazon-corretto-devel.x86_64 -y` 실행(java-21-amazon-corretto-devel.x86_64 설치)
4. Git 클론, 클론된 디렉토리에다가 dev.env 생성
5. env $(grep -v '^#' dev.env | xargs) ./gradlew bootRun 실행

빌드 및 실행 성공
<img width="1400" alt="스크린샷" src="https://github.com/user-attachments/assets/6ed1efef-1ca7-406f-91fb-d32d7f1c846c">
확정: java-21-amazon-corretto-devel이 JDK 제공, Gradle 빌드에 필수

## 🔍 원인 분석

- java-21-amazon-corretto: "development environment"는 only JRE만 포함 의미(Java 실행만 가능)
- java-21-amazon-corretto-devel: 실제 개발 도구(JDK) 포함(컴파일러(javac) 제공)
    주의: 이름만 보고서는 혼동되어 JRE/JDK 구분 어려움
- Gradle bootRun 과정: 의존성 확인 -> 소스 컴파일(이 단계 JDK 필요) -> 실행

## 📝 결론

- 스프링부트 Gradle 빌드 시 `JDK 필수`
- AWS EC2에서 Java 21 설치 시 `java-21-amazon-corretto-devel` 선택

## 💡 팁

- JDK 확인방법: 자바컴파일러는 JDK에 포함되어 있으므로, `javac --version` 실행해보고, 동작하면 JDK 설치 완료 의미
<img width="600" alt="스크린샷" src="https://github.com/user-attachments/assets/b7e36841-2f6f-435b-b7dc-fc2b083e4dcf">