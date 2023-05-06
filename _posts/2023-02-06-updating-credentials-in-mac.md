---
layout: single
title: "GitHub credentials 업데이트(mac)"
date: 2023-02-10 11:27:14 +0900
categories: knowledge
tags: [Personal access token, Credentials, Osxkeychain]
typora-root-url: ../
---

## GitHub credentials 업데이트(mac)
> - keychain access로 자격 증명 업데이트
> - 명령줄을 통한 자격 증명 삭제
> - Git에서 GitHub 자격 증명 캐싱

<br>

Git 사용 시 Personal access token으로 자격 증명 가능

또는 Git Credential Manager와 같은 자격 증명 헬퍼 사용 가능

## keychain access로 자격 증명 업데이트

1. Spotlight에 Keychain Access를 입력
2. github.com을 검색
3. github.com에 대한 "Internet password" 기입란에 Personal access token 기입

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
