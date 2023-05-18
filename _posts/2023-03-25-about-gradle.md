---
layout: single
title: "Gradle에 대하여"
date: 2023-03-25 11:48:52 +0900
categories: java
tags: [Gradle]
typora-root-url: ../
---

## Gradle에 대하여
> - 정의
> - 명령어
> - build.gradle

<br>

## 정의 

그루비(Groovy)기반의 빌드 자동화 도구

Maven과 더불어 많이 쓰이며, XML 기반의 Maven보다 동적인 빌드를 유연하게 표현할 수 있고, 가독성이 좋음

<br>

## 명령어

### Gradle 프로젝트 생성

프로젝트 디렉토리에서 프로젝트 초기화

```bash
gradle init [--type 타입명]
```

프로젝트에 필요한 초기 환경 구성

타입을 주지 않는 경우 아래의 절차대로 진행되고, 타입을 주는 경우 'build script DSL' 절차부터 진행

```bash
Select type of project to generate:
  1: basic
  2: application
  3: library
  4: Gradle plugin
Enter selection (default: basic) [1..4]

Select implementation language:
  1: C++
  2: Groovy
  3: Java
  4: Kotlin
  5: Scala
  6: Swift
Enter selection (default: Java) [1..6]

Split functionality across multiple subprojects?:
  1: no - only one application project
  2: yes - application and library projects
Enter selection (default: no - only one application project) [1..2] 1

Select build script DSL:
  1: Groovy
  2: Kotlin
Enter selection (default: Groovy) [1..2]

Select test framework:
  1: JUnit 4
  2: TestNG
  3: Spock
  4: JUnit Jupiter
Enter selection (default: JUnit 4) [1..4]
```

#### basic 타입으로 생성 시 프로젝트 구조

Gradle 프로젝트의 필수 환경만 제공, 로우 레벨에서 프로젝트를 구성

.gradle 디렉토리 : 작업(Task) 파일이 생성
gradle 디렉토리 : 'gradle-wrapper'관련 디렉토리
gradlew, gradlew.bat : 각각 리눅스(맥) 환경과, 윈도우의 실행명령
build.gradle : 프로젝트에 필요한 의존성과 빌드처리 내용을 작성하는 파일
settings.gradle : 프로젝트에 대한 설정정보를 작성하는 파일

#### java-application 타입으로 생성 시 프로젝트 구조

Gradle 프로젝트 환경 + 자바 어플리케이션 환경이 구성

mainClass는 App.java로 설정("hello world!" 출력)

### 컴파일 및 실행(java-application)

gradle에서 task는 프로젝트의 작업 단위

gradle이 제공하는 task의 경우 다음 명령어로 확인 가능

```bash
gradle tasks
```

#### 프로젝트 빌드

```bash
gradle build
```

build.gradle에 apply plugin: 'java'가 추가된 경우 .jar파일로 패키징

컴파일된 파일들은 'app > build' 폴더 안에 생성되며, .jar파일은 'build > libs'에 패키징

#### 프로젝트 실행

```bash
gradle run
```

컴파일 후 메인클래스를 실행

스프링부트의 경우 다음 명령어로 앱 구동

```bash
gradle bootRun
```

#### 프로젝트 패키징

```bash
gradle jar
```

프로그램을 .jar로 패키징하고 'build > libs'에 생성
apply plugin: 'java'가 추가된 경우 build로 한번에 패키징까지 가능

#### 프로젝트 클린

```bash
gradle clean
```

build 폴더를 제거하여, 빌드 이전 상태로 되돌림

#### gradle-wrapper

새로운 환경에서 gradle을 설치하지 않고도 빌드 가능

gradle 명령어는 gradle이 로컬에 설치가 되어있어야 하고, 빌드를 할 경우 로컬에 설치된 gradle 버젼으로 빌드되기 때문에, 개발 당시 버젼과 다를 경우 문제를 일으킬 수도 있음

./gradlew build를 사용하면 개발버전과 동일한 버전으로 빌드를 할 수 있으며, gradle이 설치되지 않았어도 빌드 가능

리눅스
```bash
./gradlew [task명]
```

윈도우
```bash
gradlew [task명]
```

<br>

## build.gradle

- buildscript
  - gradle로 task 실행 시 사용되는 설정이다.
  - 즉 어플리케이션 빌드와는 별개의 설정이다.
  (위 처럼 repositories, dependencies를 따로 구현해야함.)
- ext
  - 전역변수 블록
  - 전역변수는 $전역변수명으로 사용할 수 있다.
- classpath
  - 라이브러리를 클래스 경로에 추가
  - 빌드에서 실행까지 의존하는 라이브러리를 지정한다.
- plugin
  - 프로젝트에서 사용하는 Gradle 플러그인을 추가한다.
  (위에 설정된 플러그인들은 부트 환경구성에 필요한 플러그인)
  - eclipse : eclipse IDE 에서도 해당 Gradle project를 개발할 수 있도록 플러그인이 설치됨.
- group / version / sourceCompatibility
  - 프로젝트 생성 시 groupId, 어플리케이션 버젼, 자바버젼
- repositories
  - 필요한 라이브러리를 다운로드할 저장소를 지정
  - 공개저장소(jcenter)와, maven저장소를 사용할 수 있다.
  - 상호보완 되도록 둘 다 사용하는 것을 권장한다.
- dependencies (compile, api, implementation)
  - 라이브러리 추가
  - compile, api
    - 모듈 수정 시, 해당 모듈을 의존하고있는 모듈을 모두 빌드 -> 느리다.
    - compile의 경우 Gradle 3.0부터는 사용안하는 것을 권장(api로 대체)
    - A(api) <- B <- C 로 의존하는 구조라면, A 수정 시 B,C 모두 빌드
  - implementation
    - 모듈 수정 시, 해당 모듈을 직접 의존하는 모듈만 빌드 -> 비교적 빠르다.
    - A(implementation) <- B <- C 로 의존하는 구조라면, A 수정 시 B 만 빌드
  - testImplementation
    - 테스트에 사용하는 라이브러리 추가
  - annotationProcessor
    - 어노테이션 기반 라이브러리를 컴파일러가 인식하도록 함
    ex.) lombok, queryDSL 등
  - compileOnly
    - compile에만 필요하고, runtime에는 필요없는 라이브러리를 추가

<br>
