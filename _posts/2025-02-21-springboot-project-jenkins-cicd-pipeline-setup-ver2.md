---
layout: single
title: "[v2.0] 스프링부트 프로젝트를 Jenkins CI/CD Pipeline 구축"
date: 2025-02-21 14:12:23 +0900
categories: cicd
tags: [Jenkins, CICD, Pipeline]
typora-root-url: ../

---

# ✈️ [v2.0] 스프링부트 프로젝트를 Jenkins CI/CD Pipeline 구축

## 🎯 v1.0과 차이

v1.0에서는 서버에 Jenkins를 직접 설치했다면, v2.0에서는 서버에 우선 Docker만 설치 

이후 JDK21과 git을 포함하는 Jenkins 이미지를 생성할 수 있는 Dockerfile을 정의하고, 해당 Dockerfile을 통해 커스텀된 Jenkins 이미지를 생성

이후 생성된 커스텀된 Jenkins 이미지를 Container로 실행한 뒤, 해당 Jenkins에 접속하여 필요한 Plugin 설정 및 Pipeline 구축하여 CI/CD

## 📋 목록

- [ ]  CI/CD란?
- [ ]  Jenkins란?
- [ ]  패키지 매니저 설치 및 사용
- [ ]  Docker 설치 및 사용
- [ ]  Dockerfile 정의
- [ ]  커스텀 Jenkins 이미지 생성
- [ ]  Jenkins Container 실행
- [ ]  Jenkins Plugin 설정
- [ ]  CI/CD Pipeline 구축
- [ ]  Github webhook 설정

---

## ✏️ CI/CD란?

### CI/CD 정의

CI/CD(Continuous Integration and Continuous Deployment, 지속적 통합 및 지속적 배포)는 소프트웨어 개발 과정에서 발생하는 코드 변경사항을 자동으로 반영하여 통합하고, 배포까지 자동화하는 방법

- CI (Continuous Integration, 지속적 통합): 개발자가 코드를 자주 커밋하고, 이를 자동으로 빌드 및 테스트하여 코드 품질을 유지
- CD (Continuous Deployment, 지속적 배포): 통합된 코드를 자동으로 프로덕션 환경에 배포하여 사용자에게 빠르게 제공

### CI/CD 하는 이유

- 수동 작업 감소: 통합과 배포 과정에서 수동 작업이 줄어 비용이 적게 들고, 배포 과정의 일관성을 높이며 오류를 최소화
- 빠른 피드백 및 배포: 짧은 배포 주기로 빠른 테스트 및 배포가 가능하며, 사용자 피드백을 신속하게 반영
- 팀워크 효율성: 여러 팀원이 동시에 작업할 수 있어 프로젝트 진행이 지연되지 않으며, 협업 효율이 증가
- 품질 보장: 자동화된 테스트를 통해 코드 품질을 지속적으로 검증

---

## ✏️ Jenkins란?

### Jenkins 정의

Jenkins는 빌드, 테스트, 배포 작업을 자동화하기 위해 설계된 오픈소스 CI/CD 도구
다양한 플러그인과 유연한 파이프라인 설정을 통해 복잡한 CI/CD 워크플로우를 지원

### Jenkins 사용하는 이유

- 빠른 반영: 새로운 기능 추가나 변경사항을 빠르게 소프트웨어에 반영
- 다양한 기능 제공: 빌드, 테스트, 배포를 효율적으로 수행할 수 있는 다양한 기능을 제공
- 확장성: 다양한 플러그인을 통해 GitHub, Docker, AWS 등과의 통합 가능
- 저비용: 시스템 요구사양이 높지 않고 무료로 사용 가능

### Jenkins 동작 원리

Jenkins와 유사한 서비스로는 GitHub Actions, GitLab CI/CD가 있으며, 각 서비스는 작업을 수행하는 "Agent" 통해 동작

- GitHub Actions: GitHub Runner
- GitLab CI/CD: GitLab Runner
- Jenkins: Jenkins Worker (Node Agent)

Jenkins의 경우, Jenkinsfile이라는 스크립트를 통해 작업을 정의하며, 이 스크립트는 빌드, 테스트, 배포 단계를 포함
Jenkins Worker는 이러한 작업을 실행하며, 작업 결과를 Jenkins 서버로 전송

---
## ✏️ 패키지 매니저 설치 및 사용

### 패키지 매니저란?

패키지 매니저는 소프트웨어 설치, 업데이트, 제거를 간소화하는 도구

Amazon Linux 2023 환경에서는 dnf(Dandified YUM)라는 패키지 매니저를 사용

### 설치 및 사용 방법

- 패키지 매니저 업데이트: 시스템의 패키지 목록을 최신 상태로 유지

```bash
sudo dnf update -y
```

- 필요한 기본 패키지 설치:

```bash
sudo dnf install -y curl wget
```
curl, wget: 파일 다운로드 및 API 호출에 사용

> 💡왜 필요하나?
>
> Docker 설치와 Jenkins 설정을 위해 필수 도구를 빠르게 설치
> 최신 패키지로 유지해 호환성과 보안성을 확보

## ✏️ Docker 설치 및 사용

### Docker란?

Docker는 컨테이너 기반 가상화 플랫폼으로, 애플리케이션과 의존성을 패키징해 일관된 실행 환경을 제공

Jenkins를 컨테이너로 실행하기 위해 설치

### 설치 방법 (Amazon Linux 2023 기준)

Docker 설치:
```bash
sudo dnf update -y
sudo dnf install -y docker
sudo systemctl start docker
sudo systemctl enable docker
```

Docker 권한 설정: 비root 사용자가 Docker를 실행할 수 있도록 설정
```bash
sudo usermod -aG docker $USER
newgrp docker
```

설치 확인:
```bash
docker --version
```

💡왜 Docker를 사용하나?

이미지를 컨테이너로 실행해 환경 독립성과 유지보수 용이성을 확보
스프링부트 애플리케이션 빌드와 배포 환경을 컨테이너로 통합 가능

## ✏️ Dockerfile 정의

### Dockerfile이란?

Dockerfile은 Docker 이미지를 생성하기 위한 스크립트 파일

JDK21, Docker, Git이 포함된 커스텀 Jenkins 이미지를 정의

### Dockerfile 예시

```dockerfile
FROM jenkins/jenkins:latest
USER root

# 필수 패키지 설치
RUN apt-get update && apt-get install -y wget ca-certificates git

# JDK 21 설치
RUN mkdir -p /usr/lib/jvm && \
    wget https://download.java.net/java/GA/jdk21.0.2/f2283984656d49d69e91c558476027ac/13/GPL/openjdk-21.0.2_linux-x64_bin.tar.gz -O /tmp/jdk-21.tar.gz && \
    tar -xzf /tmp/jdk-21.tar.gz -C /usr/lib/jvm/ && \
    mv /usr/lib/jvm/jdk-21.0.2 /usr/lib/jvm/jdk-21 && \
    rm /tmp/jdk-21.tar.gz

# Docker 설치
RUN apt-get install -y apt-transport-https curl gnupg && \
    curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian bookworm stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    apt-get update && apt-get install -y docker-ce docker-ce-cli containerd.io

# GID를 외부에서 넘겨받도록 ARG 사용
ARG DOCKER_GID=988

# docker 그룹 재생성 및 jenkins 사용자 추가
RUN groupdel docker || true && \
    groupadd -g ${DOCKER_GID} docker && \
    usermod -aG docker jenkins

# JAVA 환경변수 설정
ENV JAVA_HOME=/usr/lib/jvm/jdk-21
ENV PATH=$JAVA_HOME/bin:$PATH

USER jenkins
```

### 주요 구성 설명

- FROM: 최신 Jenkins 이미지를 기반으로 설정

- USER root: 패키지 설치 권한을 위해 루트로 전환

- RUN (JDK21): OpenJDK 21을 수동으로 다운로드 및 설치

- RUN (Docker): Docker 공식 저장소에서 docker-ce, docker-ce-cli, containerd.io 설치

- ARG DOCKER_GID: 호스트의 Docker 그룹 ID를 동적으로 받아 호환성 보장

- RUN (그룹 설정): Docker 그룹을 재생성하고 Jenkins 사용자를 추가

- ENV: Java 환경 변수 설정

- USER jenkins: 보안을 위해 Jenkins 사용자로 복귀

> 💡특이사항
>
> wget으로 JDK를 직접 설치해 최신 버전 보장
> Docker GID를 동적으로 설정해 호스트와의 권한 충돌 방지
> 불필요한 파일(jdk-21.tar.gz) 삭제로 이미지 크기 최적화

## ✏️ 커스텀 Jenkins 이미지 생성

### 이미지 생성 방법

- Dockerfile 저장: 위의 Dockerfile을 Dockerfile 이름으로 프로젝트의 루트 디렉토리에 저장

- 이미지 빌드:

```bash
docker build \
  --build-arg DOCKER_GID=$(getent group docker | cut -d: -f3) \
  -t my-jenkins:jdk21 .
```

--build-arg: 호스트의 Docker 그룹 ID를 동적으로 전달

-t: 이미지 이름과 태그 지정 (my-jenkins:jdk21)

.: 특수문자 .(점)은 현재 디렉토리의 Dockerfile 사용하겠다는 현재의 디렉토리를 의미

위 이미지 빌드 명령어를 통해 명령어를 실행하는 디렉토리에 존재하는 Dockerfile에 정의해놓은 대로 커스텀된 Jenkins 이미지를 my-jenkins:jdk21라는 이름으로 이미지 빌드를 하겠다는 의미

- 빌드 확인:

```bash
docker images
```

my-jenkins:jdk21이 목록에 나타나는지 확인

> 💡특이사항
>
> 호스트의 Docker GID를 정확히 전달해야 권한 문제가 발생하지 않음
> 빌드 중 네트워크 오류 시, apt-get update 명령을 별도로 실행해 디버깅

> 💡왜 그냥 docker hub의 jenkins이미지가 아닌 커스텀 이미지(my-jenkins:jdk21)를 사용하나?
>
> 스프링부트 프로젝트 빌드에 필요한 JDK21과 Docker를 사전 설치해 환경 설정 간소화
> 호스트와의 Docker 소켓 통신을 지원해 파이프라인에서 Docker 명령 실행 가능

## ✏️ Jenkins Container 실행

### 컨테이너 실행 방법

- Jenkins 컨테이너 실행:

```bash
docker run -d --name jenkins \
  -p 8100:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  my-jenkins:jdk21
```

-d: 백그라운드 실행

-p 8100:8080: Jenkins 웹 UI를 8100 포트로 매핑

-p 50000:50000: 에이전트 통신 포트

-v jenkins_home: Jenkins 설정을 호스트에 저장

-v /var/run/docker.sock: 호스트의 Docker 데몬과 통신

my-jenkins:jdk21: 빌드해두었던 도커 커스텀 이미지(my-jenkins:jdk21)를 사용

- 컨테이너 상태 확인:

```bash
docker ps
```

- Jenkins 접속: 브라우저에서 http://<서버_IP>:8100에 접속

- 초기 비밀번호는 컨테이너 로그에서 확인:

```bash
docker logs jenkins
```

> 💡특이사항
>
> 방화벽에서 8100, 50000 포트 오픈
> /var/run/docker.sock 마운트로 Jenkins가 호스트의 Docker를 제어 가능
> 볼륨 미설정 시 컨테이너 삭제로 설정 초기화 위험 주의

## ✏️ Jenkins Plugin 설정

### 필수 플러그인 설치

- Jenkins 초기 설정 후 다음 플러그인을 설치:
    - Git Plugin: GitHub 리포지토리와 연동
    - Pipeline Plugin: 파이프라인 스크립트 작성 지원
    - Docker Pipeline Plugin: Docker 빌드 및 배포 지원
    - JDK Tool Plugin: JDK 환경 관리
    - Credentials Plugin: GitHub 및 Docker Hub 인증 정보 관리
    - Publish Over SSH: 배포 서버로 파일 전송

- 설치 방법
    - Jenkins UI에서 Manage Jenkins → Manage Plugins 이동
    - Available 탭에서 위 플러그인 검색 후 설치
    - 설치 후 Jenkins 재시작

> 💡특이사항
>
> 플러그인 버전 호환성 확인
> 스프링부트 프로젝트 특성상 Maven 또는 Gradle 플러그인 추가 고려

## ✏️ CI/CD Pipeline 구축

### 파이프라인 개요

스프링부트 프로젝트의 빌드, 테스트, 배포를 자동화하는 Jenkins 파이프라인을 설정
파이프라인은 Jenkinsfile에 정의되고, 코드 체크아웃, 빌드, 테스트, Docker 이미지 생성 및 배포와 같은 단계(stage)를 포함

### Jenkinsfile에서 환경변수 사용
Jenkinsfile에서 민감한 정보(예: GitHub PersonalсераAccess Token, Docker Hub 인증 정보, 배포 서버 SSH 키)를 안전하게 관리하기 위해 환경변수를 사용
이러한 환경변수는 Jenkins의 Credentials에 등록해 보안성을 증대

### Credentials 등록 방법

- Jenkins UI에서 Credentials 추가:
    Manage Jenkins → Manage Plugins에서 Credentials Plugin이 설치되어 있는지 확인
    Manage Jenkins → Manage Credentials → System → Global credentials 선택
    Add Credentials 클릭
- 환경변수 종류별 등록(예시):
    GitHub Personal Access Token (PAT):
    Kind: Username with password
    Username: <github-username> 또는 임의 값(예: git)
    Password: <your-github-pat> (예: ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx)
    ID: GITHUB_PAT
    Description: GitHub PAT for private repository
    참고: GitHub PAT는 비밀번호 필드에 입력하며, Jenkins의 Git 플러그인이 이 형식을 지원해 Private 리포지토리 클론에 적합
- Docker Hub 인증:
    Kind: Username with password
    Username: <docker-hub-username>
    Password: <docker-hub-password>
    ID: DOCKER_CREDENTIALS
    Description: Docker Hub Credentials
- SSH 키 (배포 서버 접근용):
    Kind: SSH Username with private key
    Username: <ssh-username>
    Private Key: <paste-private-key>
    ID: SSH_KEY
    Description: SSH Key for Deployment Server

- Jenkinsfile에서 Credentials 사용: 
    Jenkinsfile에서 아래와 같이 Git SCM 플러그인에 자격 증명을 직접 지정하거나 credentials 블록을 사용해 환경변수로 바인딩

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'GITHUB_PAT',
                    url: 'https://github.com/<owner>/<private-repo>.git',
                    branch: 'main'
            }
        }
    }
}
```
credentialsId: 'GITHUB_PAT': Username with password로 등록된 PAT를 사용해 Private 리포지토리 클론
Git SCM 플러그인이 Username with password 형식을 지원하므로 간단하게 설정 가능

- 파이프라인 설정
    Jenkins UI에서 New Item → Pipeline 선택
    Pipeline script from SCM 선택 후 GitHub 리포지토리와 Jenkinsfile 경로 지정
    Credentials 추가로 GitHub 접근 설정 (예: GITHUB_PAT 선택)
    Build Triggers에서 GitHub hook trigger for GITScm polling 체크하여 활성화

> 💡특이사항
>
> Credentials ID는 고유해야 하며, 팀원과 공유 시 일관성 유지
> 민감한 정보는 절대 Jenkinsfile에 하드코딩하지 않음
> GitHub PAT는 Fine-grained 또는 Classic 토큰을 사용하며, 리포지토리 읽기/쓰기 권한을 포함해야 함
> Credentials 사용 후 파이프라인 로그에서 민감한 정보가 마스킹되는지 확인

## ✏️ Github Webhook 설정

### Webhook이란?

GitHub Webhook은 리포지토리 이벤트(예: 푸시)를 Jenkins에 알리는 기능
코드 푸시 시 파이프라인이 자동 실행되도록 설정

### Webhook 설정 방법

- Jenkins URL 확인: Jenkins UI에서 Manage Jenkins → Configure System → Jenkins Location 확인
    예: http://<서버_IP>:8100/_
- GitHub Webhook 설정:
    GitHub 리포지토리 → Settings → Webhooks → Add webhook
    Payload URL: http://<서버_IP>:8100/github-webhook/
    Content type: application/json
    Events: Just the push event 선택
- Jenkins 설정:
    파이프라인 설정에서 GitHub hook trigger for GITScm polling 확인
    테스트: 리포지토리에 푸시 후 Jenkins 대시보드에서 파이프라인이 실행되는지 확인
    (에러가 난다면, 해당 파이프라인의 console 확인)

> 💡특이사항
>
> 서버 방화벽에서 8100와 프로젝트 배포할 포트를 방화벽 오픈 규칙에 추가