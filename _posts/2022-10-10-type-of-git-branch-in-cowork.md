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

![total-branch](/images/2022-10-10-type-of-git-branch-in-cowork/total-branch.png)

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

![develop-branch](/images/2022-10-10-type-of-git-branch-in-cowork/develop-branch.png)

<br>

