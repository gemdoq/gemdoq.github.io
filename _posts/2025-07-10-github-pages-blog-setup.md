---
layout: single
title: "GitHub Pages로 블로그 만들기"
date: 2025-07-10 14:12:23 +0900
categories: github
tags: [Blog, Jekyll]
typora-root-url: ../

---
# ✈️ GitHub Pages로 블로그 만들기

## 🎯 개요

GitHub Pages와 Jekyll로 개인 블로그를 만드는 방법
Spring Boot 프로젝트 기록이나 학습 내용을 마크다운으로 정리해 깃허브 잔디 심기에 활용
무료 호스팅과 간단한 설정으로 초보자도 쉽게 블로그 운영 가능

## 📋 목록

- [ ]  GitHub Pages와 Jekyll이란?
- [ ]  블로그 만드는 이유
- [ ]  준비 항목
- [ ]  블로그 생성 단계
- [ ]  로컬 테스트 방법
- [ ]  실무 팁과 주의사항

---

## ✏️ GitHub Pages와 Jekyll이란?

### GitHub Pages 정의

GitHub에서 제공하는 무료 호스팅 서비스
마크다운 파일을 웹사이트로 변환해 블로그나 프로젝트 페이지로 활용

### Jekyll 정의

마크다운 파일을 정적 웹사이트로 변환하는 도구
GitHub Pages와 통합해 블로그 포스트 자동 정리

> 💡왜 필요하나?
>
> Spring Boot 프로젝트의 CI/CD 기록이나 학습 내용을 깃허브에 올려 잔디 심기 가능
> 마크다운으로 간단히 작성하고 무료로 배포

---

## ✏️ 블로그 만드는 이유

- Git 실습: 커밋, 푸시로 버전 관리 연습
- 포트폴리오: Spring Boot 프로젝트 기록으로 경력 증명
- 학습 공유: 코드와 설정 공유로 팀 협업 지원
- 웹사이트 경험: 기본적인 웹 배포 이해
- 무료 호스팅: 비용 없이 블로그 운영

---

## ✏️ 준비 항목

- GitHub 계정: github.com에서 가입
- Git: git-scm.com에서 다운로드
- 텍스트 에디터: IntelliJ Ultimate 또는 VS Code 추천
- Ruby: Jekyll 실행을 위해 rubyinstaller.org(Windows) 또는 Homebrew(macOS/Linux)로 설치
- 명령줄: macOS/Linux는 터미널, Windows는 Git Bash

> 💡특이사항
>
> IntelliJ Ultimate에서 마크다운 플러그인 설치로 편집 편리
> Ruby 설치 시 최신 버전 사용으로 호환성 문제 방지

---

## ✏️ 블로그 생성 단계

### 1단계: GitHub 리포지토리 생성

1. GitHub 로그인 후 상단 + 아이콘 → New repository 선택
2. 이름: username.github.io (username은 GitHub 사용자 이름, 예: johndoe.github.io)
3. Public으로 설정
4. README, .gitignore 없이 Create repository 클릭

### 2단계: 로컬 Jekyll 설정

1. 터미널 또는 Git Bash 열기
2. Jekyll과 Bundler 설치:
```bash
gem install jekyll bundler
```
3. 새 Jekyll 사이트 생성:
```bash
jekyll new my-blog
```
4. 생성된 폴더로 이동:
```bash
cd my-blog
```

### 3단계: 로컬 블로그와 GitHub 연결

1. Git 리포지토리 초기화:
```bash
git init
```
2. 모든 파일 추가:
```bash
git add .
```
3. 커밋:
```bash
git commit -m "초기 블로그 설정"
```
4. GitHub 리포지토리 연결:
```bash
git remote add origin https://github.com/<username>/<username>.github.io.git
```
5. GitHub 푸시:
```bash
git push -u origin main
```

### 4단계: GitHub Pages 활성화

1. GitHub 리포지토리(username.github.io)로 이동
2. Settings → Pages 섹션
3. Source에서 main 브랜치 선택 후 Save
4. 몇 분 후 https://username.github.io 접속으로 블로그 확인

> 💡특이사항
>
> 리포지토리 이름이 username.github.io가 아니면 블로그 URL이 달라짐
> GitHub Actions로 Jekyll 빌드 에러 시 _config.yml 확인

---

## ✏️ 로컬 테스트 방법

로컬 환경에서 블로그 테스트로 푸시 전 확인 가능
1. my-blog 폴더로 이동:
```bash
cd 경로/내/my-blog
```
2. Jekyll 서버 실행:
```bash
bundle exec jekyll serve
```
3. 브라우저에서 http://localhost:4000 접속
4. IntelliJ Ultimate로 _posts 폴더의 마크다운 파일 편집
5. 저장 후 브라우저 새로고침으로 변경 확인
6. 서버 종료: Ctrl + C

---

## ✏️ 실무 팁과 주의사항

### 블로그 운영 팁

- 간단 시작: 완벽한 포스트보다 학습 기록 우선
- 마크다운 연습: GitHub README 작성에 활용
- 테마 변경: Jekyll 테마로 나중에 블로그 꾸미기
- Git 연습: 커밋과 푸시로 잔디 심기

### 주의사항

- Ruby 버전 확인: Jekyll 호환 버전 사용
- _config.yml 백업: 설정 변경 시 복사본 저장
- 푸시 전 테스트: 로컬 서버로 오류 확인
- Public 리포지토리: 비공개 설정 시 Pages 작동 안 함

> 💡특이사항
>
> IntelliJ Ultimate의 마크다운 미리보기 기능으로 포스트 확인
> GitHub 푸시 후 1-2분 대기 후 URL 접속
> 이 설정으로 깃허브 잔디 심기와 블로그 운영 간소화