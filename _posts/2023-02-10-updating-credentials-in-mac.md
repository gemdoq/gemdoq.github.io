---
layout: single
title: "GitHub credentials 업데이트(mac)"
date: 2023-02-10 11:27:14 +0900
categories: github
tags: [Personal access token, Credentials, Osxkeychain]
typora-root-url: ../
---

## GitHub credentials 업데이트(mac)
> - keychain access로 자격 증명 업데이트
> - 명령줄을 통한 자격 증명 삭제
> - Git에서 GitHub 자격 증명 캐싱

<br>

과거 GitHub 비밀번호 입력 방식에서 현재 Personal Access Token 방식으로 자격 증명이 변경

MacOS에서 git push 실행 시, 다음과 같은 에러 로그 발생

```console
remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead. ... The requested URL returned error: 403
```

MacOS의 키체인에 GitHub 자격 증명으로 ID와 PW가 저장되어 있기 때문

이제 Git 사용 시 변경된 방식인 Personal access token으로 자격 증명 가능

또는 Git Credential Manager와 같은 자격 증명 헬퍼 사용 가능

<br>

## keychain access로 자격 증명 업데이트

1. Spotlight에 Keychain Access를 입력
2. github.com을 검색
3. github.com에 대한 "Internet password" 기입란에 Personal access token 기입

<br>

## 명령줄을 통한 자격 증명 삭제

터미널 명령줄을 이용해 자격 증명 키체인을 삭제

```console
$ git credential-osxkeychain erase
host=github.com
protocol=https
> [Press Return]
```

제대로 작동하면 아무것도 출력되지 않음

자격 증명이 제대로 지워졌는지 시험해보기 위해서는 private repository를 clone할 때, 패스워드를 요구함

<br>

## Git에서 GitHub 자격 증명 캐싱

HTTPS를 사용하여 GitHub repository를 clone하는 경우 GitHub CLI나 GCM(Git Credential Manager) 사용하여 자격 증명 시 자동으로 저장

### GitHub CLI

Git 작업을 위한 기본 프로토콜로 HTTPS를 선택하여 GitHub 자격 증명 확인을 실행하면 Git 자격 증명을 자동 저장

GitHub CLI에서 명령줄에 `gh auth login`을 입력하고, 기본 프로토콜을 HTTPS로 선택하고, Git에 대한 인증을 GitHub 자격 증명을 사용할 건지 물어보면 Y 입력

### GCM(Git Credential Manager)

GCM(Git Credential Manager)은 자격 증명을 안전하게 저장하고 HTTPS를 통해 GitHub에 연결

GCM을 이용하면 Personal access token을 생성하고 저장할 필요 없이 알아서 인증

```console
$ brew tap microsoft/git
$ brew install --cask git-credential-manager-core
```

인증이 요구되는 HTTPS URL을 복제하면, OAuth 인증 요구하는 브라우저 통해 로그인

한번 인증을 성공하면 자격 증명이 keychain에 저장되어 이후에는 자격 증명 필요 없음

<br>
