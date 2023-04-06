---
layout: single
title: "[Git] Github에 잘못 올라간 파일 삭제하기"
date: 2022-04-10 11:40:22 +0900
categories: git
tags: [git 명령어]
---


## GitHub에 잘못 올린 파일 삭제하기
> - GitHub pages를 만들 때 올릴 필요가 없는 파일을 remote origin에 push한 경우
> - remote origin에 잘못 올라간 파일을 삭제하고 .gitignore에 등록해 무시하고 다시 push한다

<br>

## Github에 잘못 올라간 파일 삭제 과정
### 1. 원격 저장소에서 파일 삭제하기

이미 github remote에 push를 했기 때문에 로컬의 저장소에서 파일을 삭제해도 원격 저장소에서는 삭제되지 않는다.
** git rm VS git rm --cached **
~~~javascript
// 원격 저장소와 로컬 저장소에 있는 파일을 삭제한다.
$ git rm [File Name]
// 원격 저장소에 있는 파일을 삭제한다. 로컬 저장소에 있는 파일은 삭제하지 않는다.
$ git rm --cached [File Name]
~~~

따라서 아래와 같이 `git rm --cached [File Name]` 명령어를 이용하여 원격 저장소에서 잘못 올라간 파일을 삭제해야 한다.

~~~javascript
// .idea/modules.xml 파일 삭제
$ git rm --cached .idea/modules.xml
// .idea 폴더 하위의 모든 파일 삭제 
$ git rm --cached -r .idea/
~~~

<br>

### 2. .gitignore 설정하기
만약 .gitignore가 제대로 설정되어 있지 않다면 .gitigore 설정하여 다음에는 개인이 관리해야되는 파일들이 원격 저장소에 올라가지 않도록 해야한다. 

`TIP)` 또한 .gitignore는 git add 명령어 전에 설정되어 있어야 적용이 가능하다는 것을 알아두자.

### 3. 원격 저장소에 적용하기
버전 관리에서 완전히 제외하기 위해서는 반드시 commit 명령어와 push를 수행해야 한다.

~~~javascript
// 버전 관리에서 완전히 제외하기 위해서는 반드시 commit 명령어를 수행해야 한다.
$ git commit -m "Fixed untracked files"
// 원격 저장소(origin)에 push
$ git push origin master
~~~

<br>