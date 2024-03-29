---
layout: single
title: "SQL 단일행 함수에 대해"
date: 2024-03-07 18:05:47 +0900
categories: sql
tags: [SQL, DBMS, POSIX regular expression, Meta characters]
typora-root-url: ../
---

## SQL 단일행 함수에 대해

> - 문자함수
> - 숫자함수
> - 날짜함수
> - 일반함수
> - 정규식함수

## 문자함수

- INITCAP:  입력값의 첫 글자만 대문자로 변환
- LOWER: 입력값을 전부 소문자로 변환
- UPPER: 입력값을 전부 대문자로 변환
- LENGTH: 입력된 문자열의 길이 출력
- LENGTHB: 입력된 문자열의 길이의 바이트값 출력
- CONCAT: 두 문자열을 결합해서 출력. || 연산자와 동일하나 || 연산자는 다중 요소 결합가능
- SUBSTR: 주어진 문자에서 특정 문자만 추출
- SUBSTRB: 주어진 문자에서 특정 바이트만 추출
- INSTR: 주어진 문자에서 특정 문자의 위치 추출
- INSTRB: 주어진 문자에서 특정 문자의 바이트값 추출
- LPAD: 주어진 문자열에서 왼쪽으로 특정문자를 채움
- RPAD: 주어진 문자열에서 오른쪽으로 특정문자를 채움
- LTRIM: 주어진 문자열에서 왼쪽의 특정문자를 삭제
- RTRIM: 주어진 문자열에서 오른쪽의 특정문자를 삭제
- REPLACE: 주어진 문자열에서 A를 B로 치환
- TRANSLATE: 주어진 문자열에서 대상문자를 변환문자로 치환

<br>

## 숫자함수

- ROUND: 주어진 숫자를 반올림 후 출력함
- TRUNC: 주어진 숫자를 버림 후 출력함
- MOD: 주어진 숫자를 나눈 후 나머지값을 출력함
- CEIL: 주어진 숫자와 가장 근접한 큰 정수를 출력함
- FLOOR: 주어진 숫자와 가장 근접한 작은 정수를 출력함
- POWER: 주어진 숫자 A의 숫자 B승을 출력함

<br>

## 날짜함수

- SYSDATE: 시스템의 현재 날짜와 시간
- MONTHS_BETWEEN: 두 날짜 사이의 개월 수
- ADD_MONTHS: 주어진 날짜에 개월을 더함
- NEXT_DAY: 주어진 날짜를 기준으로 돌아오는 날짜 출력
- LAST_DAY: 주어진 날짜가 속한 달의 마지막 날짜 출력
- ROUND: 주어진 날짜를 반올림
- TRUNC: 주어진 날짜를 버림

<br>

## 일반함수

- NVL: null값을 만나면 다른 값으로 치환해서 출력
- NVL2: null이 아니면 출력할 값을 지정해서 출력
- DECODE: 일반 개발용 언어에서 사용중인 분기문인 IF문처럼 조건에 맞는 값을 지정해서 출력
- CASE WHEN THEN: DECODE와 같이 조건에 맞는 값을 지정해서 출력

<br>

## 정규식함수

- ^(carrot): 해당 문자로 시작
- $(dollar): 해당 문자로 끝
- .(dot): 하나의 문자
- *(asterisk): 바로 앞에 있는 항목이 0번 이상 반복되는 모든 문자를 의미(greedy)
- \+(plus sign): 바로 앞에 오는 패턴이 최소 1개 이상 반복되는 것을 의미(non-greedy)
- ?(question mark): 바로 앞에 있는 항목이 존재할 수도 있고 없을 수도 있는 모든 문자를 의미(non-greedy)
- []: 해당 문자에 해당하는 한 문자
- [^]: 해당 문자에 해당하지 않는 한 문자(1개 이상의 문자가 들어갈 수 있음)
- REGEXP_REPLACE: 주어진 문자열에서 특정패턴을 찾아 치환
- REGEXP_INSTR: 주어진 문자열에서 특정패턴의 시작 위치를 반환
- REGEXP_SUBSTR: 주어진 문자열에서 특정패턴을 찾아 반환
- REGEXP_LIKE: 주어진 문자열에서 특정패턴을 찾아 반환
- REGEXP_COUNT: 주어진 문자열에서 특정패턴의 횟수를 반환

정규식함수에서는 메타문자를 활용한 정규식을 잘 알고 응용하는 것이 중요하다.

대괄호[] 옆의 중괄호{} 안의 숫자는 대괄호의 정규식 조건에 만족하는 문자의 갯수를 의미한다.

중괄호{} 안 숫자가 comma(,) 문자와 함께 있을 경우 "from 첫번째 숫자갯수 to 두번째 숫자갯수"를 의미한다. 

carrot(^) 문자가 대괄호[] 밖에 있는 경우는 시작문자를 의미하고, 대괄호[] 안에 있는 경우는 해당 문자가 아님을 의미한다.

dot(.) 문자는 하나의 문자를 의미하고 예를 들어 ^....g 와 같은 정규식은 처음 시작으로부터 5번째 문자가 g인 행을 특정한다.

그리고 정규식 안에서 pipe(|) 문자를 통해서 연결할 수도 있다.

<br>

### POSIX 정규표현식의 종류

대괄호[] 안에 양 옆을 콜론(:)으로 감싼 형태로 표현되는 다음 정규표현식들은 각각의 자료형을 가진 패턴의 문자열과 매칭된다.

- **[:alnum:]**: 알파벳과 숫자 문자
- **[:alpha:]:** 알파벳 문자
- **[:upper:]:** 대문자 알파벳
- **[:lower:]:** 소문자 알파벳
- **[:digit:]:** 10진수 숫자 (0-9)
- **[:xdigit:]:** 16진수 숫자 (0-9, a-f, A-F)
- **[:blank:]:** 공백 문자 (스페이스와 탭 문자)
- **[:space:]:** 공백 문자 (스페이스, 탭, 줄 바꿈 등)
- **[:graph:]:** 출력 가능한 문자 (공백 문자를 제외한 모든 문자)
- **[:print:]:** 출력 가능한 문자와 공백 문자
- **[:punct:]:** 구두점 문자(쉼표 `,`, 마침표 `.`, 물음표 `?`, 느낌표 `!` 등)
- **[:cntrl:]:** 제어 문자(줄바꿈(`\n`), 탭(`\t`), 백스페이스(`\b`)  등)

<br>

### 백슬래쉬(\\)의 역할과 의미

백슬래쉬(\\)는 여러 역할을 가진다.

백슬래쉬(\\) 뒤에 나오는 문자에 따라 역할과 의미가 다르다.

<br>

#### 백슬래쉬(\\) 뒤 일반문자

- '\\d' : 숫자문자와 일치하여 패턴매칭에 사용된다. '[0-9]'와 동일한 의미를 가진다.
- '\\w' : 단어문자와 일치하여 패턴매칭에 사용된다. 알파벳(대문자 및 소문자), 숫자, 밑줄('_')과 일치한다.
- '\\s'  : 공백문자와 일치하여 패턴매칭에 사용된다. 스페이스, 탭, 개행문자와 일치한다.
- '\\b' : 단어 경계와 일치하여 패턴매칭에 사용된다. 단어문자와 비단어 문자 사이의 경계를 의미한다.

<br>

#### 백슬래쉬(\\) 뒤 숫자문자

정규표현식에서 '( )' 안에 하나의 패턴을 정의한다면 그것은 하나의 그룹이다. 

이전에 '정의된 그룹'을 통해 패턴을 참조하고자 할 때  '\\숫자문자'를 사용하면 해당 숫자문자의 순서에 해당하는 '정의된 그룹'을 사용할 수 있다.

<br>

#### 백슬래쉬(\\) 뒤 기호문자

기호문자의 일부는 메타문자로 쓰인다.

메타문자를 원래 기호문자처럼 사용하고 싶을 때, 탈출문자인 백슬래쉬(\\)를 사용해야 한다.

<br>

### Greedy 매칭과 Non-Greedy 매칭

#### Greedy 매칭

Greedy한 매칭은 정규 표현식에서 사용되는 기본적인 매칭 방식이다. 

예를 들어, `.*` 패턴은 가능한 한 많은 문자열을 매칭하여 문자열의 시작부터 끝까지 가능한 많은 문자열을 포함하려고 시도한다. 

"abcdef" 문자열에서 `a.*b` 패턴은 "ab"로 시작하고 "b"로 끝나는 가능한 한 많은 문자열을 매칭하므로 "abcdef" 전체 문자열에 매칭된다. 

#### Non-Greedy 매칭

Non-greedy한 매칭은 우리가 원하는 가장 짧은 부분을 찾고자 할 때 유용한 방식이다. 

Non-greedy 연산자는 다음과 같이 사용될 수 있다.

1. `*?`: 가능한 한 적은 수의 반복을 매칭한다. `*`는 해당 항목이 0번 이상 반복됨을 나타내고, `?`는 그것이 최소한으로 매칭되도록 한다.
2. `+?`: 가능한 한 적은 수의 반복을 매칭한다. `+`는 해당 항목이 1번 이상 반복됨을 나타내고, `?`는 그것이 최소한으로 매칭되도록 한다.
3. `??`: 가능한 한 적은 수의 반복을 매칭한다. `?`는 해당 항목이 0번 또는 1번만 나타날 수 있음을 나타내고, `?`는 그것이 최소한으로 매칭되도록 한다.
4. `{n,m}?`: 가능한 한 적은 수의 반복을 매칭한다. `{n,m}`는 해당 항목이 최소 n번부터 최대 m번까지 반복됨을 나타내고, `?`는 그것이 최소한으로 매칭되도록 한다.

예를 들어, `.*?` 패턴은 가능한 한 많은 문자열을 매칭하되, 최소한의 반복만 수행하려고 시도한다.

"ababcdefcdaf" 문자열에서 `a.*?e` 패턴은 "a"로 시작하고 "e"로 끝나는 가능한 긴 문자열을 가장 적은 횟수로 매칭하므로 "ababcde"에만 매칭된다. 
