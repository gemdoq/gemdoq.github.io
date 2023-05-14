---
layout: single
title: "GitHub labels 좀 쉽게 설정하기"
date: 2023-03-12 10:42:44 +0900
categories: github
tags: [issue, labels]
typora-root-url: ../
---

## GitHub labels 좀 쉽게 설정하기
> - 라벨이란
> - 준비물
> - 진행과정
> - 결과화면

<br>

## 라벨이란

깃허브(GitHub)에서 개인 프로젝트를 할 때나 팀 프로젝트를 할 때 이슈 구분 · 역할 구분 · 내용 구분 등을 정확하게 위해서 라벨이라는 것을 사용

깃허브 label을 지정해 둔 형식으로 한 번에 업데이트 하는 방법

<br>

## 준비물

1. labels.json(아무 디렉토리나 생성하고 위치 기억)
```json
[
  {
    "name": "Assignee1",
    "color": "d5ecc2",
    "description": "담당자1"
  },
  {
    "name": "Assignee2",
    "color": "ffd3b4",
    "description": "담당자2"
  },
  {
    "name": "Assignee3",
    "color": "dbe6fd", 
    "description": "담당자3"
  },
  {
    "name": "bug",
    "color": "ee0701",
    "description": "버그입니다."
  },
  {
    "name": "chore",
    "color": "8c001a",
    "description": "세팅 관련입니다."
  },
  {
    "name": "cleanup",
    "color": "fef2c0",
    "description": "코드를 더 깔끔하게 만들기만 하고, 코드 작동 방식이나 출력에 대한 부분을 변경하지 않습니다."
  },
  {
    "name": "docs",
    "color": "d4c5f9",
    "description": "문서만 변경됩니다."
  },
  {
    "name": "feat",
    "color": "84b6eb",
    "description": "구현·개선 사항에 관련된 내용입니다."
  },
  {
    "name": "fix",
    "color": "de5b7b",
    "description": "버그를 수정합니다."
  },
  {
    "name": "help wanted",
    "color": "0e8a16",
    "description": "누구나 처리할 수 있는 이슈를 나타냅니다."
  },
  {
    "name": "question",
    "color": "cc317c",
    "description": "질문만 있는 이슈, 질문이 해결되면 이슈를 종료합니다."
  },
  {
    "name": "refactoring",
    "color": "fbca04",
    "description": "코드가 내부적으로 작동하는 방식을 변경합니다. cleanup과는 다릅니다."
  },
  {
    "name": "test",
    "color": "bfe5bf",
    "description": "테스트 코드만 변경됩니다."
  }
]
```

2. personal access token
  다음 🔗[링크](https://github.com/settings/tokens)에 들어가서 Select scope에서 repo 카테고리가 체크된 Personal access token을 발급받아 따로 잘 보관(만약 한번이라도 분실 시, 토큰 찾기 불가능)

  ![token](/images/2023-03-12-about-github-issue-labels/token.png)

<br>

## 진행과정

### CLI 켜기

맥북은 기본으로 깔려있는 터미널을 사용하고, 윈도우는 Git Bash를 설치하고 실행

### labels.json을 저장해뒀던 디렉토리로 이동

디렉토리 이동 명령어는 `cd 경로`이므로 만약 json파일이 downloads에 있을 경우 다음과 같이 입력

```bash
cd downloads
```

### GitHub label 동기화 명령어 실행

json파일이 있는 디렉토리로 이동했으면, 해당 디렉토리에서 다음 형식에 맞춰 명령어를 입력

```bash
npx github-label-sync --access-token {personal access token} --labels ./labels.json {repo}
```

token은 ghp_abc123이고, repo명은 gemdoq/testrepo일 경우, 위 명령어의 예시

```bash
npx github-label-sync --access-token ghp_abc123 --labels ./labels.json gemdoq/testrepo
```

<br>

## 결과화면

![labels](/images/2023-03-12-about-github-issue-labels/labels.png)<br>
