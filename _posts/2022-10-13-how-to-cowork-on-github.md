---
layout: single
title: "GitHub로 협업하는 방법"
date: 2022-10-13 11:52:22 +0900
categories: knowledge
tags: [Git]
typora-root-url: ../
---


## GitHub로 협업할 때 알아둬야 할 것
> - 저장소에 대한 개념
> - GitHub로 협업하는 방법과 이유 

<br>

## 0. Forking Workflow
* 모든 프로젝트 참여자가 개인적인 로컬 저장소와 공개된 자신의 원격 저장소, 
  즉 **두 개씩의 Git 저장소** 를 가지는 방식
* 모든 코드 기여자가 하나의 중앙 저장소에 푸시하는 것이 아니라, 
  각자 자신의 원격 저장소에 푸시하고, 
  프로젝트 관리자만 다른 개발자들의 기여분을 중앙 원격 저장소에 병합
  (소규모의 팀에서는 팀원 모두가 중앙 원격 저장소를 관리할 수도 있음)
* 아주 큰 규모의 분산된 팀에서도 안전하게 협업하기에 좋은 협업 방법
* **오픈 소스 프로젝트에서 많이 사용하는 방식**

<br>

## 1. 중앙 원격 저장소, 자신의 원격 저장소, 로컬 저장소의 개념
* 중앙 원격(remote) 저장소  
  * 여러 명이 같은 프로젝트를 관리하는 데 사용하는 그룹 계정의 중립된 원격 저장소
* 자신의 원격(remote) 저장소
  * *remote repository*
  * 파일이 GitHub 전용 서버에서 관리되는 원격 저장소(Origin)
* 로컬(local) 저장소
  * *local repository*
  * 파일을 작업하고 저장되는 내 개인 로컬 저장소(PC)

<br>

## 2. 중앙 원격 저장소를 포크(fork)해서 자신만의 원격 저장소를 생성
중앙 원격 저장소를 복제한 저장소는 개인의 공개 저장소(remote repository)
다른 개발자는 자신의 원격 저장소에 푸시할 수 없음(내려받기는 가능)

<br>

## 3. 프로젝트 참여자는 git clone 명령으로 로컬 저장소를 생성
git clone 명령으로 자신의 원격 저장소(remote repository)를 복제하여 

개인 로컬 저장소(local repository)를 생성 

프로젝트 참여자는 이 로컬 저장소에서 작업을 수행

~~~javascript
$ git clone [내 remote repository URL]
~~~

<br>

## 4. 두 개의 원격 저장소를 연결
프로젝트 중앙 원격 저장소와 포크한 자신의 원격 저장소(remote repository) 연결

일반적으로 포크한 원격 저장소는 origin, 

프로젝트 중앙 원격 저장소는 upstream으로 호칭 

(upstream 별칭은 자동으로 생성되지 않으므로, 직접 지정)

로컬 저장소(local repository)를 프로젝트 중앙 원격 저장소와 같은 상태로 유지 가능

~~~javascript
$ git remote add origin [내 원격 URL]
$ git remote add upstream [중앙 원격 저장소 URL]
$ git remote -v
~~~

<br>

## 5. 설명을 위해 현재 로컬에서 작업 중인 branch 위치를 표시
중앙 원격 저장소와 자신의 원격 저장소에는 각각 master branch가 있고, 

자신의 로컬 저장소에는 master branch가 있다고 가정 

<br>

## 6. 새로운 기능 개발을 위해 격리된 branch를 생성
로컬 저장소에서 기능구현을 위한 branch를 생성해서 해당 branch로 변경,

코드를 수정 후 변경 내용을 커밋

~~~javascript
$ git checkout -b [branch name]

# 위의 명령어는 아래의 두 명령어를 합한 것
$ git branch [branch name]
$ git checkout [branch name]
~~~

<br>

## 7. 로컬 저장소의 커밋 이력을 자신의 원격 저장소(remote repository)에 푸시
기능을 구현한 후 커밋한 이력을 푸시할 때, 자신의 원격 복제본 저장소에 푸시  
~~~javascript
$ git commit -a -m "Write commit message"

# 위의 명령어는 아래의 두 명령어를 합한 것
$ git add . # 변경된 모든 파일을 스테이징 영역에 추가
$ git add [some-file] # 스테이징 영역에 some-file 추가
$ git commit -m "Write commit message" # local 작업폴더에 history 하나를 쌓는 것
~~~
자신의 원격 저장소에 변경 내용을 push하기 전까지는 변경 내용은 비공개

다른 개발자가 볼 수 있도록 자신의 원격 저장소에 push

~~~javascript
$ git push origin [branch name] # branch name에 해당하는 branch를 자신의 원격 저장소에 푸시
~~~

<br>

## 8. 프로젝트 관리자에게 자신의 기여를 반영해 달라는 요청
Pull requests 이용하여 프로젝트 중앙 원격 저장소의 master branch에 

자신의 원격 저장소에 있는 특정 branch의 기여를 merge를 통해 반영해달라고 요청

<br>

## 9. 프로젝트 관리자는 변경 내용을 검토 후 중앙 원격 코드 베이스에 merge
변경된 코드 내용을 검토 후 변경된 코드를 중앙 원격 코드 베이스에 merge
1. Pull requests - File changed 탭에서 변경 내용을 확인
2. Conversation 탭으로 이동하여 Confirm merge
3. 충돌이 일어난 경우, 합의하여 코드 수정한 후 merge

<br>

## 10. 중앙 원격 저장소와 자신의 로컬 저장소를 동기화하기 위해 로컬 저장소의 branch를 master branch로 변경

<br>

## 11. 중앙 원격 저장소에 새로운 커밋이 있다면 pull
프로젝트 참여하는 모든 개발자가 자신의 로컬 저장소를 변경사항이 반영된 최신의 메인 코드 베이스와 동기화
~~~javascript
$ git pull upstream master
~~~

<br>

## 12. 새로운 기능을 추가하기 위해서 그 작업에 대한 branch를 생성하여 작업
중앙 원격 저장소와 동기화된 로컬 저장소의 master branch에서 새로운 작업에 대한 branch를 생성하여 작업을 시작

<br>
