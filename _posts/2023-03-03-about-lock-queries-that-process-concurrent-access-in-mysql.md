---
layout: single
title: "MySQL에서 잠금 쿼리로 동시 접근을 처리하는 방법에 대해"
date: 2023-03-03 11:41:13 +0900
categories: Database
tags: [ORACLE, MySQL, Shared Lock, Exclusive Lock]
typora-root-url: ../
---

## MySQL에서 잠금 쿼리로 동시 접근을 처리하는 방법에 대해
> - 공유 락
> - 배타 락
> - MySQL에서 공유 락과 배타 락
> - 주의점

<br>

DBMS에서 특정 데이터에 대한 동시 접근이 발생한 경우, 일관성과 무결성을 지키기 위해 해당 데이터에 잠금을 걸 수 있다. 

이때, 이 잠금을 구현하는 방식에 따라 크게 공유락과 배타락으로 나뉜다.

대표적인 DBMS인 MySQL InnoDB에서는 기본 'Isolation Level'을 변경할 수도 있겠지만, 간단한 잠금 쿼리를 이용해서 처리할 수 있다.

락의 종류와 전략은 DBMS 벤더사마다 상이할 수 있다.

## 공유 락

공유 락(Shared Lock)은 읽기 락(Read Lock)이라고도 불린다. 

공유 락을 사용하면 조회한 데이터가 트랜잭션 내내 변경되지 않음을 보장한다.

즉, 공유 락이 걸린 데이터에 대해서는 쓰기 연산(UPDATE, DELETE)은 실행이 불가능하며, 읽기 연산(SELECT)은 얼마든지 여러 세션이 동시에 실행 가능하다. 

<br>

## 배타 락

배타 락은 쓰기 락(Write Lock)이라고도 불린다. 

데이터에 대해 배타 락을 획득한 트랜잭션은, 읽기 연산과 쓰기 연산을 모두 실행할 수 있고, 다른 트랜잭션은 배타 락이 걸린 데이터에 대해 읽기 작업과 쓰기 작업 모두 수행할 수 없다. 

즉, 배타 락이 걸려있다면 다른 트랜잭션은 공유 락, 배타 락 둘 다 획득 할 수 없다. 

배타 락을 획득한 트랜잭션은 해당 데이터에 대한 독점권을 갖는 것이다.

<br>

## MySQL에서 공유 락과 배타 락

공유 락, 배타 락의 잠금 옵션은 Auto Commit이 비활성화 되거나, BEGIN 혹은 START TRANSACTION 명령을 통해 트랜잭션이 시작된 상태에서만 잠금이 유지된다.

### 공유 락(SELECT FOR SHARE)

아래와 같이 SELECT FOR SHARE를 사용하여 특정 데이터로부터 공유 락을 획득할 수 있다.

```shell
SELECT * 
FROM table_name 
WHERE id = 1 
FOR SHARE;
```

### 배타 락(SELECT FOR UPDATE)

아래와 같이 SELECT FOR UPDATE를 사용하여 특정 데이터로부터 배타 락을 획득할 수 있다.

```shell
SELECT * 
FROM table_name 
WHERE id = 1 
FOR UPDATE;
```

다만, SELECT한 Row의 락을 획득하기 위해 무한정 기다리는 장애를 야기할 수 있음

그래서 해결책으로 오라클의 경우 9i 버전 이후부터는 WAIT라는 키워드도 같이 제공

이는 WAIT 뒤에 나오는 숫자만큼 기다려서 해당 Row에 대한 Lock을 획득하지 못할 경우, 더 이상 락을 기다리지 않고 에러가 발생

WAIT 키워드 외에 NOWAIT도 있는데, 해당 키워드는 락을 획득하기 위한 대기시간 없이 바로 에러 발생

```shell
SELECT * 
FROM table_name 
WHERE id = 1 
FOR UPDATE WAIT 3;
```

<br>

## 주의점

### 잠금 없는 읽기

FOR UPDATE 혹은 FOR SHARE를 가지지 않는 SELECT 쿼리는 잠금 없는 읽기 가능

따라서 특정 데이터가 FOR UPDATE로 락이 걸린 상태라도 FOR UPDATE, FOR SHARE가 없는 단순 SELECT쿼리는 아무런 대기 없이 데이터 조회 가능

### 데드락

![deadlock](/images/2023-03-03-about-lock-queries-that-process-concurrent-access-in-mysql/deadlock.png){: width="560"}

데드락은 서로가 점유하고 있는 자원에 대해 무한정 대기하고 있는 상황을 의미

특정 데이터를 점유하는 락의 특성 상 데드락이 발생할 수 있음

락과 같이 특정 리소스를 점유하는 매커니즘을 사용할 때에는 위와 같은 데드락을 주의해야 함

<br>
