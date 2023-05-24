---
layout: single
title: "QueryDSL에 대해"
date: 2023-04-12 11:16:35 +0900
categories: java
tags: [QueryDSL]
typora-root-url: ../
---

## QueryDSL에 대해
> - 정의
> - 설정
> - QueryDSL 간단 예제
> - Repository에서 QueryDSL 사용하기
> - 커스텀 사용

<br>

## 정의

Spring Data JPA는 기본적으로 CRUD 메서드 및 쿼리 메서드 기능 제공

원하는 조건의 데이터를 수집하기 위해서는 JPQL을 작성할 필요

but 복잡한 로직의 경우, 개행이 포함된 쿼리 문자열이 상당히 길어지면서 휴먼 에러(JPQL 문자열에 오타 혹은 문법 오류) 발생 가능성 높음

정적 쿼리라면 컴파일 시 발견할 수 있으나, 그렇지 않은 경우 런타임 시점에서 에러가 발생

QueryDSL은 정적 타입을 이용, SQL 쿼리를 생성해 이러한 문제를 예방하는 프레임워크

<br>

## 설정

의존성 설정(build.gradle)

```gradle
dependencies {
    implementation 'com.querydsl:querydsl-jpa:4.4.0'
    annotationProcessor 'com.querydsl:querydsl-apt:4.4.0:jpa'
}
```

Querydsl 설정(build.gradle)

```gradle
// Querydsl 설정
def generatedDir = "$buildDir/generated/querydsl"

tasks.withType(JavaCompile) {
    options.annotationProcessorGeneratedSourcesDirectory = file(generatedDir)
}
sourceSets.main.java.srcDirs += "$buildDir/generated/querydsl"

clean.doLast {
    file(generatedDir).delete()
}
```

위와 같이 build.gradle 파일에 설정을 추가하면, 프로젝트 빌드할 때마다 자동으로 Querydsl 관련 클래스와 빌더가 생성

<br>

## QueryDSL 간단 예제



<br>

## Repository에서 QueryDSL 사용하기



<br>

## 커스텀 사용



<br>
