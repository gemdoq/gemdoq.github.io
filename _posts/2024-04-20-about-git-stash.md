---
layout: single
title: "git stash에 대하여"
date: 2024-04-20 11:26:23 +0900
categories: git
tags: [git, stash, version control]
typora-root-url: ../

---

## Git Stash 명령어 정리

> - Git Stash의 정의
> - 주요 Git Stash 명령어
> - Git Stash의 중요성

## Git Stash의 정의

> Git Stash is a feature that allows you to save your current changes temporarily and reapply them later.
> Git Stash는 현재 작업 중인 변경 사항을 임시로 저장하고, 나중에 다시 적용할 수 있도록 하는 기능

작업 중 변경 사항을 임시 저장 후, 다른 작업 수행 후 다시 복구(변경 사항을 "숨기는" 행위)

## 주요 Git Stash 명령어

> - 변경 사항 저장하기
> - 스택 목록 보기
> - 최신 스택 적용하기
> - 특정 스택 적용하기
> - 최신 스택 적용 후 삭제하기
> - 특정 스택 삭제하기
> - 모든 스택 삭제하기
> - 스택 이름 지정하기
> - 스택 내용 보기
> - 특정 스택 내용 보기
> - 스택 내용 비교하기
> - 특정 스택 내용 비교하기
> - 스택 브랜치 생성하기

1. **변경 사항 저장하기**:
```
git stash
```
또는
```
git stash save "메시지"
```
현재 작업 중인 변경 사항을 스택에 저장

메시지를 추가하면 나중에 스택을 확인할 때 어떤 변경 사항이 저장되었는지 쉽게 알 수 있음

2. **스택 목록 보기**:
```
git stash list
```
현재 저장된 모든 스택을 보여줌

3. **최신 스택 적용하기**:
```
git stash apply
```
가장 최근에 저장된 스택을 적용(스택은 여전히 스택 목록에 잔존)

4. **특정 스택 적용하기**:
```
git stash apply stash@{n}
```
여기서 n은 스택 목록에서 적용하고자 하는 스택의 인덱스

5. **최신 스택 적용 후 삭제하기**:
```
git stash pop
```
가장 최근에 저장된 스택을 적용하고, 그 스택을 스택 목록에서 삭제

6. **특정 스택 삭제하기**:
```
git stash drop stash@{n}
```
여기서 n은 삭제하고자 하는 스택의 인덱스

7. **모든 스택 삭제하기**:
```
git stash clear
```
모든 스택을 삭제

8. **스택 이름 지정하기**:
```
git stash save "메시지"
```
스택을 저장할 때 메시지를 추가하여 스택에 이름을 지정

9. **스택 내용 보기**:
```
git stash show
```
가장 최근에 저장된 스택의 내용

10. **특정 스택 내용 보기**:
```
git stash show stash@{n}
```
여기서 n은 스택 목록에서 보고자 하는 스택의 인덱스

11. **스택 내용 비교하기**:
```
git stash show -p
```
가장 최근에 저장된 스택의 변경 사항을 패치 형식

12. **특정 스택 내용 비교하기**:
```
git stash show -p stash@{n}
```
여기서 n은 스택 목록에서 비교하고자 하는 스택의 인덱스

13. **스택 브랜치 생성하기**:
```
git stash branch <branchname> [<stash>]
```
스택을 기반으로 새로운 브랜치를 생성

<stash> 인자를 생략하면 가장 최근의 스택을 사용

## Git Stash의 중요성

Git Stash는 개발 과정에서 매우 유용한 도구

작업 중인 변경 사항을 임시로 저장하고, 다른 작업을 수행한 후 다시 복구할 수 있는 기능을 제공

이를 통해 작업 흐름을 효율적으로 관리하고, 필요할 때 쉽게 복구

<br>