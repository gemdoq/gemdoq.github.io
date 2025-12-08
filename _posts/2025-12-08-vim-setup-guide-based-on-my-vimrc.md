---
layout: single
title: "내 vimrc 기준으로 Vim 설정하기 (macOS 터미널 Vim 초보자 가이드)"
date: 2025-12-08 17:30:00 +0900
categories: Vim
tags: [Vim, Terminal, macOS, Editor]
typora-root-url: ../

---
# ✈️ 내 vimrc 기준으로 Vim 설정하기

## 🎯 개요

macOS 터미널에서 Vim을 사용하면 사람마다 화면이 전부 다르게 보임
누군가는 줄 번호가 없고, 누군가는 색상이 없고, 누군가는 검색이 불편함
이 문서는 내 맥북에서 실제로 사용 중인 `~/.vimrc` 기준으로 다른 사람도 그대로 따라 하면 같은 Vim 환경을 만들 수 있도록 정리한 글

## 📋 목록

- [ ] Vim 설정 파일 위치
- [ ] 줄 번호 설정
- [ ] 화면 이동 편의성
- [ ] 괄호 매칭 설정
- [ ] 검색 설정
- [ ] 상태 표시줄 설정
- [ ] 커서 위치 강조
- [ ] 공백 문자 표시
- [ ] 백업 파일 비활성화
- [ ] 색상 및 하이라이트 설정
- [ ] 전체 vimrc 예시

---

## ✏️ Vim 설정 파일 위치

Vim은 실행될 때 여러 설정 파일을 읽음
개인 설정 파일은 홈 디렉토리의 `.vimrc`에 위치함

```bash
~/.vimrc
```

파일이 없다면 직접 생성

```bash
touch ~/.vimrc
vim ~/.vimrc
```

---

## ✏️ 줄 번호 설정

```vim
set number
set relativenumber
set numberwidth=4
```

### set number
왼쪽에 절대 줄 번호 표시
파일 몇 번째 줄인지 바로 확인 가능

### set relativenumber
현재 커서를 기준으로 상대 줄 번호 표시
`5j`, `3k` 같은 이동이 쉬워짐

### set numberwidth=4
줄 번호 영역 최소 너비를 4칸으로 고정
파일 길이가 바뀌어도 화면이 흔들리지 않음

---

## ✏️ 화면 이동 편의성

```vim
set scrolloff=3
set sidescrolloff=5
set wrap
set linebreak
```

### scrolloff
커서 위아래로 최소 3줄 여유 유지
화면 끝에 커서가 붙지 않음

### sidescrolloff
좌우 이동 시 여유 공간 확보

### wrap
긴 줄 자동 줄바꿈
가로 스크롤 방지

### linebreak
단어 단위로 줄바꿈
글 읽기 편해짐

---

## ✏️ 괄호 매칭 설정

```vim
set showmatch
set matchtime=2
```

### showmatch
`()`, `{}` 괄호 짝 강조 표시

### matchtime=2
강조 유지 시간 0.2초

---

## ✏️ 검색 설정

```vim
set ignorecase
set smartcase
set incsearch
set hlsearch
```

### ignorecase
대소문자 무시 검색

### smartcase
대문자 포함 시 대소문자 구분

### incsearch
검색어 입력 중 실시간 검색

### hlsearch
검색 결과 전체 강조

---

## ✏️ 상태 표시줄 설정

```vim
set laststatus=2
set statusline=%f\ %h%m%r%=%-14.(%l,%c%V%)\ %P
```

### laststatus=2
항상 하단 상태 표시줄 표시

### statusline 구성 설명

- %f: 파일 이름
- %h: 도움말 여부
- %m: 수정 여부
- %r: 읽기 전용 여부
- %=: 좌우 정렬 기준
- %l: 현재 줄 번호
- %c: 현재 열 번호
- %V: 화면 기준 열 위치
- %P: 파일 진행률

왼쪽은 파일 상태, 오른쪽은 현재 위치 정보 표시

---

## ✏️ 커서 위치 강조

```vim
set cursorline
set cursorcolumn
```

### cursorline
현재 줄 배경 강조

### cursorcolumn
현재 열 강조
불필요하면 제거 가능

---

## ✏️ 공백 문자 표시

```vim
set list
set listchars=tab:>-,trail:-,extends:>,precedes:<
```

### set list
눈에 보이지 않는 문자 표시

### listchars 설명

- `tab:>-`: 탭 문자를 >--- 형태로 표시
- `trail:-`: 줄 끝 공백 표시
- `extends:>`: 화면 오른쪽 넘어가는 줄 표시
- `precedes:<`: 화면 왼쪽 넘어가는 줄 표시

공백 실수 바로 확인 가능
코드 스타일 관리에 매우 유용함

> 💡특이사항
>
> 공백 문자 표시는 코드 작성 시 탭과 스페이스 혼용 실수 방지에 필수
> 협업 환경에서 코드 스타일 통일에 큰 도움

---

## ✏️ 백업 파일 비활성화

```vim
set nobackup
set nowritebackup
set noswapfile
```

불필요한 파일 생성 방지
디렉토리 깔끔 유지

---

## ✏️ 색상 및 하이라이트 설정

```vim
set termguicolors

highlight Normal guifg=#fcf6e5 guibg=#0e2a35
highlight Search guifg=#0e2a35 guibg=#69e0e2
highlight Visual guibg=#618fce
highlight CursorLine guibg=#12303a
highlight LineNr guifg=#fcf6e5 guibg=#0e2a35
highlight CursorLineNr guifg=#69e0e2 guibg=#0e2a35
```

### set termguicolors
터미널에서 True Color 사용
이 설정 없으면 색상 깨짐

### highlight Normal
기본 텍스트 색상 지정

### highlight Search
검색 결과 강조 색상 지정

### highlight Visual
비주얼 모드 선택 영역 색상 지정

### highlight CursorLine
현재 줄 강조 색상 지정

### highlight LineNr
줄 번호 색상 지정

### highlight CursorLineNr
현재 줄 번호 강조 색상 지정

> 💡특이사항
>
> macOS 터미널에서 termguicolors 설정은 필수
> iTerm2 사용 시 색상 표현이 더 정확함

---

## ✏️ 전체 vimrc 예시

```vim
set number
set relativenumber
set numberwidth=4

set scrolloff=3
set sidescrolloff=5
set wrap
set linebreak

set showmatch
set matchtime=2

set ignorecase
set smartcase
set incsearch
set hlsearch

set laststatus=2
set statusline=%f\ %h%m%r%=%-14.(%l,%c%V%)\ %P

set cursorline
set cursorcolumn

set list
set listchars=tab:>-,trail:-,extends:>,precedes:<

set nobackup
set nowritebackup
set noswapfile

set termguicolors
highlight Normal guifg=#fcf6e5 guibg=#0e2a35
highlight Search guifg=#0e2a35 guibg=#69e0e2
highlight Visual guibg=#618fce
highlight CursorLine guibg=#12303a
highlight LineNr guifg=#fcf6e5 guibg=#0e2a35
highlight CursorLineNr guifg=#69e0e2 guibg=#0e2a35
```

---

## 💡 요약

이 설정은 플러그인 없이 순정 Vim 기준
macOS 터미널에서 바로 사용 가능
위 설정을 `~/.vimrc`에 복사하면 동일한 환경 구성 완료
