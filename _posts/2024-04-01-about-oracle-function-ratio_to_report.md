---
layout: single
title: "Oracle 비율 함수 RATIO_TO_REPORT()에 대해"
date: 2024-04-01 18:10:21 +0900
categories: sql
tags: [SQL, DBMS, 오라클 비율 함수, 오라클 퍼센트 함수]
typora-root-url: ../
---

## Oracle 비율 함수 RATIO_TO_REPORT()에 대해

> - RATIO_TO_REPORT() 함수란
> - 사용방법
> - 사용예제
> - 주의사항

## RATIO_TO_REPORT() 함수란

분석 함수(Analytic Function)의 한 종류로, 결과 집합의 각 행에 대해 특정 값의 비율을 계산하는 데 사용된다. 

이 함수는 전체 그룹 내에서 해당 Row가 차지하는 비율을 반환한다.

<br>

## 사용방법

기본 문법은 아래와 같다.

```sql
RATIO_TO_REPORT(값value) OVER (구분기준partitionn_clause)
```

- 값value: 비율을 계산할 대상 컬럼 또는 값
- 구분기준partition_clause: 결과를 구분 짓는 기준을 정의하는 절(PARTITION BY절을 사용하여 서브셋으로 나누고, 각각에 대해 비율을 계산)

<br>

## 사용예제

각 부서별로 직원들의 **급여**가 전체 **부서** 급여에서 **차지하는 비율**을 계산할 때 다음과 같이 사용한다.

```sql
SELECT department_id, 
       employee_id, 
       salary,
       RATIO_TO_REPORT(salary) OVER (PARTITION BY department_id) AS salary_ratio
FROM employees;
```

직원employees 테이블에서 각 부서department_id별로 직원들의 급여salary가 전체 부서 급여에서 차지하는 비율salary_ratio을 계산한다. 

여기서 `RATIO_TO_REPORT(salary) OVER (PARTITION BY department_id)`는 각 부서 내에서 해당 급여의 전체 대비 비율을 계산한다.

<br>

## 주의사항

RATIO_TO_REPORT() 함수는 그룹 내 각 Row 값이 전체 합에 대한 비율을 계산하기 때문에, 전체 합이 0인 경우 결과가 정의되지 않을 수 있다.

<br>
