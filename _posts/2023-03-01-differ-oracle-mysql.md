---
layout: single
title: "ORACLE과 MySQL의 차이"
date: 2023-03-01 10:17:42 +0900
categories: database
tags: [ORACLE, MySQL]
typora-root-url: ../
---

## ORACLE과 MySQL의 차이
> - DB란
> - ORACLE
> - MySQL
> - 차이

<br>

## DB란

DB(DataBase, 자료 저장소)란 여러 사람에 의해 공유되어 사용될 목적으로 통합하여 관리되는 데이터의 저장 장소

### DB 종류

1세대 파일시스템(ISAM, VSAM), 2세대 계층형 HDBMS(IMS, System2000), 3세대 네트워크형 NDBMS(IDS, TOTAL, IDMS) 등이 있지만, 아래의 3가지 DB 종류가 중요

#### 관계형 RDBMS

행(Column)과 열(Row)을 가지는 표 형식 데이터를 저장하는 형태의 데이터베이스

![RDBMS](/images/2023-03-01-differ-oracle-mysql/RDBMS.png){: width="560"}

SQL을 이용하여 관리 및 접근

ORACLE, MySQL 등

#### 비관계형 NOSQL

키(Key)-값(Value)형태로 저장되는 데이터베이스

정해진 스키마가 없어 보다 자유롭게 데이터 저장

![notonlysql](/images/2023-03-01-differ-oracle-mysql/notonlysql.png){: width="560"}

키(Key)를 사용해 데이터 관리 및 접근

읽기를 자주 하지만 데이터를 자주 변경하지 않는 경우 사용

NoSQL은 비정형 데이터(메신저 텍스트, 음성, 이미지 등)를 다룰 수 있음

수평 확장이 가능, 분산형 컴퓨팅(클라우드)에 적합

MongoDB, Cassandra, Redis, Neo4j 등

![rdbvsnosql](/images/2023-03-01-differ-oracle-mysql/rdbvsnosql.png){: width="560"}

<br>

## ORACLE

세계 점유율 1위 데이터베이스 관리 시스템이며 현재 유닉스 체제에서 가장 많이 사용되는 DBMS

### 장점

1. Multiple databases 튜닝 가능하고, 다수의 사용자가 동시에 접근 가능
2. 변경 plan을 작성하고 실제 구현하기 전에 변경 사항의 효과를 볼 수 있고, 생산 시스템을 방해하지 않음
3. 오류가 발생하면 설정되어 있는 계정 및 이메일로 연락이 가고, 가동 정지 시간 동안 차단할 수 있음
4. DBMS 실행 컴퓨터 / 서버 역할 컴퓨터 / DB 응용 프로그램 실행 컴퓨터 다르게 분산처리
5. Cost 비용을 최소화 하기 위해 테이블과 인덱스를 분석하고, 다른 데이터베이스보다 고성능 트랜잭션

### 단점

1. 비용부담
2. 복잡한 기능으로 진입장벽 높음
3. 높은 하드웨어 사양 요구

<br>

## MySQL

가장 널리 사용되고 있는 오픈 소스 DBMS

### 장점

1. 1MB RAM만 사용할만큼 낮은 사양으로 매우 적은 오버 헤드를 사용하여, 빠른 처리 속도와 대용량 데이터 처리에 용이
2. 간단하여 사용하기가 매우 쉬움
3. 거의 모든 운영체제 사용을 지원하고 다양한 프로그래밍 언어와 통합
4. MySQL 데이터베이스는 오픈소스라서 무료, 상업용은 유료

### 단점

1. 복잡한 쿼리 처리는 성능 저하
2. 트랜잭션 지원이 완벽하지 않음
3. 사용자 정의 함수의 사용이 쉽지 않음

<br>

## 차이

### 구조적 차이

오라클 : DB 서버가 통합된 하나의 스토리지를 공유하는 방식
MYSQL : DB 서버마다 독립적인 스토리지를 할당하는 방식

### 조인 방식의 차이

오라클 : 중첩 루프 조인, 해시 조인, 소트 머지 조인 방식 제공
MYSQL : 중첩 루프 조인 방식을 제공

### 확장성의 차이

오라클 : 별도의 Managing System 사용 불가
MYSQL : 별도의 Managing System 사용 가능

### 메모리 사용율의 차이

오라클 : 메모리 사용율이 커서 최소 수백MB 이상이 되어야 설치 가능
MYSQL : 메모리 사용율이 낮아서 1MB 환경에서도 설치 가능

### 파티셔닝

오라클 : Local Partitioned Index, Global Partitioned Index를 지원
MYSQL : Local Partitioned index만 지원

### 힌트 방식

오라클 : 문법적 오류가 있으면 힌트를 무시하고 쿼리를 수행
MYSQL : 문법적 오류가 있으면 오류 발생

### SQL 구문의 차이

#### Null 대체

오라클
```sql
NVL(열명, '대체값')
```

MYSQL
```sql
IFNULL(열명, '대체값')
```

#### SELECT 결과 갯수 제한(페이징처리)

오라클
```sql
ROWNUM <= 숫자
```

MYSQL
```sql
LIMIT 시작위치, 가져올 데이터 건수
```

#### 가상테이블 DUAL

오라클
```sql
SELECT 1 FROM DUAL;
```

MYSQL
```sql
SELECT 1;
```

#### 현재 날짜

오라클
```sql
SELECT SYSDATE FROM DUAL;
```

MYSQL
```sql
SELECT NOW();
```

#### 날짜 형식

오라클
```sql
SELECT TO_CHAR(SYSDATE, 'YYYY-MM-DD') FROM DUAL;
```

MYSQL
```sql
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d');
```

#### 조건식(IF)

오라클
```sql
//칼럼이 값과 일치하면 TRUE, 일치하지 않으면 FALSE
SELECT DECODE(칼럼, 값, TRUE일때 출력할 값, FALSE일때 출력할 값) FROM TABLE;
```

MYSQL
```sql
SELECT IFNULL(조건식, TRUE일때 값, FALSE일때 값) FROM TABLE;
```

#### 시퀀스

오라클
```sql
CREATE SEQUENCE [시퀀스명]
INCREMENT BY [증감숫자]
START WITH [시작숫자]
NOMINVALUE / MINVALUE [최소값]
NOMINVALUE / MINVALUE [최소값]
CYCLE / NOCYCLE
CACHE / NOCACHE
```
```sql
INSERT TABLE
(SEQ_NBR)
VALUES
(시퀀스명.NEXTVAL)
;
```

MYSQL
```sql
CREATE TABLE
( SEQ_NBR INT NOT NULL AUTO_INCREMENT PRIMARY KEY);
```

#### 문자열 합치기

오라클
```sql
SELECT "A" || "B" FROM DUAL;
SELECT CONCAT("A", "B") FROM DUAL;
```

MYSQL
```sql
SELECT CONCAT("A", "B") FROM DUAL;
```

#### 문자열 자르기

오라클
```sql
SELECT SUBSTR( 문자열/칼럼, 시작위치, 잘라낼 문자열의 길이) FROM DUAL;
```

MYSQL
```sql
SELECT SUBSTRING(문자열/칼럼, 시작위치, 잘라낼 문자열의 길이);
```

<br>
