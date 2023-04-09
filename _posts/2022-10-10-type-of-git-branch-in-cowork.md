---
layout: single
title: "협업할 때 사용하는 Git Branch"
date: 2022-10-10 10:42:22 +0900
categories: knowledge
tags: [Git]
typora-root-url: ../
---


## 협업할 때 사용하는 Git Branch의 종류
> - Gitflow Workflow에서 사용하는 Git Branch 종류를 이해한다.
> - Gitflow Workflow에서 사용하는 Git Branch 사용법을 이해한다.

<br>

# Git Branch 종류 (5가지)
Gitflow Workflow에서는 항상 유지되는 메인 브랜치들(master, develop)과 

일정 기간 동안만 유지되는 보조 브랜치들(feature, release, hotfix)을 포함하여 

총 5가지의 브랜치를 사용한다.

![total-branch](/images/2022-10-10-type-of-git-branch-in-cowork/total-branch.png){: width="560"}

<br>

## 1. Master Branch
<span style="color:#4d0000">**제품으로 출시될 수 있는 브랜치**</span>

배포(Release) 이력을 관리하기 위해 사용

즉, 배포 가능한 상태만을 관리

<br>

## 2. Develop Branch
<span style="color:#4d0000">**다음 출시 버전을 개발하는 브랜치**</span>

기능 개발을 위한 브랜치들을 병합하기 위해 사용

모든 기능이 추가되고 버그가 수정되어 배포 가능한 안정적인 상태라면 

develop 브랜치를 'master' 브랜치에 병합(merge)

<br>

평소에는 이 브랜치를 기반으로 개발을 진행

![develop-branch](/images/2022-10-10-type-of-git-branch-in-cowork/develop-branch.png){: width="560"}

<br>

## 3. Feature branch
<span style="color:#4d0000">**기능을 개발하는 브랜치**</span>

feature 브랜치는 새로운 기능 개발 및 버그 수정이 필요할 때마다 

'develop' 브랜치로부터 분기

feature 브랜치에서의 작업은 기본적으로 공유할 필요가 없기 때문에, 

자신의 로컬 저장소에서 관리

개발이 완료되면 'develop' 브랜치로 병합(merge)하여 다른 사람들과 공유

1. 'develop' 브랜치에서 새로운 기능에 대한 feature 브랜치를 분기
2. 새로운 기능에 대한 작업 수행
3. 작업이 끝나면 'develop' 브랜치로 merge
4. 더 이상 필요하지 않은 feature 브랜치는 삭제
5. 새로운 기능에 대한 'feature' 브랜치를 중앙 원격 저장소에 push

* feature 브랜치 이름 정하기
  * master, develop, release-(RB_), or hotfix- 제외
  * [feature/기능요약] 형식을 추천  EX) feature/login

* feature 브랜치 생성 및 종료 과정

~~~javascript
// feature 브랜치(feature/login)를 'develop' 브랜치('master' 브랜치에서 따는 것이 아니다!)에서 분기
$ git checkout -b feature/login develop

/* ~ 새로운 기능에 대한 작업 수행 ~ */

/* feature 브랜치에서 모든 작업이 끝나면 */
// 'develop' 브랜치로 이동한다.
$ git checkout develop
// 'develop' 브랜치에 feature/login 브랜치 내용을 병합(merge)한다.
# --no-ff 옵션: 아래에 추가 설명
$ git merge --no-ff feature/login
// -d 옵션: feature/login에 해당하는 브랜치를 삭제한다.
$ git branch -d feature/login
// 'develop' 브랜치를 원격 중앙 저장소에 올린다.
$ git push origin develop
~~~

* <mark>--no-ff 옵션</mark>
  * 새로운 커밋 객체를 만들어 'develop' 브랜치에 merge
  * 이것은 'feature' 브랜치에 존재하는 커밋 이력을 **모두 합쳐서** 하나의 새로운 커밋 객체를 만들어 'develop' 브랜치로 merge
  * ![feature-branch-merge](/images/2022-10-10-type-of-git-branch-in-cowork/feature-branch-merge.png){: width="560"}

<br>

