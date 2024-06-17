---
layout: single
title: "Git Branch 전략에 대하여"
date: 2024-04-10 06:10:21 +0900
categories: git
tags: [commit, branch, merge, rebase]
typora-root-url: ../
---

## Git Branch 전략에 대하여

> - 왜 git을 사용해야 하는가?
> - commit이란?
> - branch란?
> - merge란?
> - rebase란?

## 왜 git을 사용해야 하는가?

좋은 개발자는 기억하는 개발자가 아니다. 

시대가 어느 때인데 정보를 기억하는 것을 미덕으로 여기는가?

정보는 저장해두고 필요할 때 적합한 정보를 검색할 능력을 가지고 있으면 된다.

**좋은 개발자**는 `정보를 지속적으로 저장하고 저장된 정보를 상황과 필요에 맞게 활용`하는 개발자이다. 

개발자가 git을 사용하는 이유도 **history를 저장하기 위해서**이다. 

git은 단순히 저장하고 불러오는 작업만 하는 것이 아니라 다른 사용자와도 history를 공유관리하여 협업하는 데 강력한 도움을 준다.

개발자가 혼자이더라도, 과거와 미래, 그리고 현재의 개발자는 모두가 상황과 필요가 다르기 때문에 다른 개발자라고 볼 수 있다. 

즉 개발자는 git을 통해 과거와 미래, 그리고 현재의 개발자들과 소통하여 상황과 필요에 따라 필요한 정보를 활용할 수 있게 된다.

<br> 

## Commit이란?

**commit**이란 프로젝트의 현재 상태를 **snapShot**으로 나타내는 **checkPoint**를 말한다.

### 좋은 commit의 기준
> https://github.com/joelparkerhenderson/git-commit-message
> 위 내용을 요약하면 **prefix를 잘 활용하고, 짧고 굵게 기록하라**
> ![prefix](/images/2024-06-10-about-git-branch-strategy/prefix.png)

<br>

## Branch란?

**branch**란 **commit** 사이를 가볍게 이동할 수 있는 어떤 **pointer** 같은 것을 말한다.

많은 회사에서 사용 중인 Git Flow를 따르고 있다.

### branch 종류 및 설명
- 주 branch: main, dev
	- `main`: 기준이 되는 branch로 제품을 배포하는 branch
	- `dev`: 개발 branch로 다음 release를 위해 준비
- `feature branches`: 단위별 프로그램 기능을 개발하는 branch로 기능 개발이 완료되면 dev branch에 Merge
- `release branch`: 배포를 위해 main branch로 보내기 전에 먼저 QA를 하기 위한 branch(QA를 진행하면서 commit이 쌓이면 dev branch에 지속적으로 merge된다)
- `hotfix branch`: main branch로 배포를 했는데 bug가 생겼을 때 긴급히 수정하는 branch이며, bug 수정 후 재배포가 완료된 이후에는 dev branch에도 main과 동일하게 배포해야 함

![gitflowbranch](/images/2024-06-10-about-git-branch-strategy/gitflowbranch.png)

<br>

## Merge란?

**forked**된 **branch**를 합치는 것

### Merge의 유형

![mergetype](/images/2024-06-10-about-git-branch-strategy/mergetype.png)

1. Create a merge commit

    <img src="/images/2024-06-10-about-git-branch-strategy/merge1.png" alt="merge1" style="zoom:50%;" />

    parent branch에서 fork하여 작업을 한 branch를 merge하려고 했는데, 

    내가 merge하기 전에 누군가 parent branch에 다른 작업을 하여 commit하고 push한 경우이다.

    <img src="/images/2024-06-10-about-git-branch-strategy/merge2.png" alt="merge2" style="zoom:50%;" />

    하나의 branch와 다른 branch의 변경 이력 전체를 합치는 방법이다.

    <img src="/images/2024-06-10-about-git-branch-strategy/merge3.png" alt="merge3" style="zoom:50%;" />

2. Squash and merge

    <img src="/images/2024-06-10-about-git-branch-strategy/squashmerge1.png" alt="squashmerge1" style="zoom:50%;" />

    commit a + b + c를 합쳐서 새로운 commit, abc를 만들어 parent branch에 추가한다.

    보통 feature branch의 commit history를 합쳐서 깔끔하게 만들기 위해 사용한다. 

3. Rebase and merge
    <img src="/images/2024-06-10-about-git-branch-strategy/rebasemerge1.png" alt="rebasemerge1" style="zoom:50%;" />

    모든 commit들이 합쳐지지 않고 각각 parent branch에 추가된다. 
    Merge는 Merge commit 기록이 추가로 남게 되지만, Rebase의 경우에는 Merge commit의 기록이 남지 않는다. 

    따라서 하나의 브랜치에서 작업한 것처럼 보여진다.

<br>

## Rebase란?

> The primary reason for rebasing is to maintain a linear project history 
> - atlassian

- 선형적으로 히스토리를 유지하여 **작업 내용이 한 눈에 파악될 수 있도록** 하기 위한 방법
- command line으로 git branching에 대해 학습하고 익힐 수 있는 시각화된 웹서비스
> 🔗 [LEARN GIT BRANCHING](https://learngitbranching.js.org/?locale=ko){: target="_blank"}

<br>
