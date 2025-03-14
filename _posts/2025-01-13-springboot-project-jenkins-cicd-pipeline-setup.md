---
layout: single
title: "스프링부트 프로젝트를 Jenkins CI/CD Pipeline 구축"
date: 2025-01-13 10:15:23 +0900
categories: cicd
tags: [Jenkins, CICD, Pipeline]
typora-root-url: ../

---

# ✈️ 스프링부트 프로젝트를 Jenkins CI/CD Pipeline 구축

## 🎯 프로젝트 목적

스프링부트 프로젝트를 AWS EC2에 설치된 Jenkins를 활용하여 CI/CD 파이프라인을 구축하고, 
GitHub Webhook을 통해 자동화된 빌드, 테스트, 배포 프로세스를 구현
AWS EC2 t3.small 인스턴스의 메모리 부족 문제를 스왑 메모리 설정으로 해결하여 저사양 환경에서도 안정적인 파이프라인 운영 환경 구축

## 📋 목록

- [x]  CI/CD란?
- [x]  Jenkins란?
- [x]  AWS EC2 t3.small 환경 설정
- [x]  Jenkins 설치 및 초기 설정
- [x]  GitHub Webhook 설정
- [x]  메모리 부족 문제 및 스왑 메모리 설정
- [x]  Dockerfile 정의
- [x]  Jenkinsfile 정의
- [x]  환경변수 보안 조치
- [x]  파이프라인 테스트
- [x]  개선할 점

---

## ✏️ CI/CD란?

### CI/CD 정의

CI/CD(Continuous Integration and Continuous Deployment, 지속적 통합 및 지속적 배포)는 소프트웨어 개발 과정에서 발생하는 코드 변경사항을 자동으로 반영하여 통합하고, 배포까지 자동화하는 방법입니다.

- CI (Continuous Integration, 지속적 통합): 개발자가 코드를 자주 커밋하고, 이를 자동으로 빌드 및 테스트하여 코드 품질을 유지.
- CD (Continuous Deployment, 지속적 배포): 통합된 코드를 자동으로 프로덕션 환경에 배포하여 사용자에게 빠르게 제공.

### CI/CD 하는 이유

- 수동 작업 감소: 통합과 배포 과정에서 수동 작업이 줄어 비용이 적게 들고, 배포 과정의 일관성을 높이며 오류를 최소화.
- 빠른 피드백 및 배포: 짧은 배포 주기로 빠른 테스트 및 배포가 가능하며, 사용자 피드백을 신속하게 반영.
- 팀워크 효율성: 여러 팀원이 동시에 작업할 수 있어 프로젝트 진행이 지연되지 않으며, 협업 효율이 증가.
- 품질 보장: 자동화된 테스트를 통해 코드 품질을 지속적으로 검증.

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

## ✏️ AWS EC2 t3.small 환경 설정

AWS EC2 t3.small 인스턴스를 사용하여 Jenkins를 설치하고 CI/CD 파이프라인을 구축합니다.

- 인스턴스 사양: t3.small (2 vCPU, 2GB RAM)
- 운영체제: Amazon Linux 2023

### EC2 인스턴스 생성

1. AWS Management Console → EC2 대시보드
2. "인스턴스 시작" 클릭 → Amazon Linux 2023 AMI 선택
3. 인스턴스 유형 t3.small 선택
4. 보안 그룹 설정:
    - SSH (포트 22) 허용
    - Jenkins 웹 UI 접근을 위한 포트 8080 허용
5. 키 페어 생성 및 다운로드
6. 인스턴스 시작 후 퍼블릭 IP 확인

---

## ✏️ Jenkins 설치 및 초기 설정

### Jenkins 설치

1. EC2 인스턴스에 SSH로 접속:

```shell
ssh -i jenkins-key.pem ec2-user@<EC2_PUBLIC_IP>
```

2. Jenkins 설치 전 필요한 패키지 업데이트 (Amazon Linux 2023에서는 dnf 사용):

```shell
sudo dnf update -y
```

3. Jenkins 공식 저장소 추가 및 설치:

```shell
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo 
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key 
sudo dnf install jenkins -y
```

4. JDK 설치 (Jenkins는 Java에 의존, Amazon Corretto 21 JDK 설치):
    - Amazon Corretto 21은 JRE와 JDK로 나뉘며, Jenkins 빌드 작업을 위해 JDK 필요
    - java-21-amazon-corretto는 JRE, java-21-amazon-corretto-devel은 JDK

```shell
sudo dnf install java-21-amazon-corretto-devel -y
```

5. Jenkins 서비스 시작 및 부팅 시 자동 시작 설정:

```shell
sudo systemctl start jenkins 
sudo systemctl enable jenkins
```

### Jenkins 초기 설정

1. 웹 브라우저에서 `http://<EC2_PUBLIC_IP>:8080` 으로 접속

2. 초기 비밀번호를 입력:

```shell
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

3. 비밀번호 입력 후, "Install suggested plugins" 선택하여 Jenkins 설치 마법사 진행:
    - 계정 생성 및 Jenkins URL 설정
    - Jenkins 플러그인 설치
	    - Manage Jenkins > Manage Plugins > Available 탭에서 Git, GitHub Integration, Docker, Docker Pipeline, Pipeline 등 설치
	    - 해당 설치화면 하단에 "설치 후 Jenkins 재시작" 체크하여 설치 과정에서 자동으로 재시작

4. Jenkins URL 설정: `http://<EC2_PUBLIC_IP>:8080`

---

## ✏️ GitHub Webhook 설정

GitHub Webhook을 통해 코드 변경사항이 있을 때 Jenkins 파이프라인을 자동으로 트리거

### GitHub 설정

1. GitHub 리포지토리 "Settings" 페이지로 이동
2. "Webhooks" 메뉴에서 "Add webhook" 클릭
3. Payload URL: Jenkins URL에 Jenkins의 깃허브 웹훅(/github-webhook/)를 추가합니다. (예: `http://<EC2_PUBLIC_IP>:8080/github-webhook/`)
4. Content type: application/json
5. Which events would you like to trigger this webhook? 에서 이벤트 선택: "Just the push event" 선택
6. Active 체크박스를 선택 후 "Add webhook" 클릭

## ✏️ 젠킨스에 파이프라인 구축(깃허브 원격 레포지토리로부터 온 요청에 의해 트리거되게끔 설정)

### Jenkins Credentials 설정

1. Manage Jenkins > Manage Credentials > (global) > Add Credentials 클릭
2. Credentials에 github-pat 추가
- Kind: Secret text
- Scope: Global
- Secret: ghp_xxxxxxxxxxxxxxxx와 같은 형식의 GitHub Personal Access Token 값
- ID: github-pat
- Description: "GitHub Personal Access Token"
3. Credentials에 환경변수를 아래와 같이 추가:
- Kind: Secret text
- ID: `<변수Key>`
- Secret: `<변수Value>`
(예시 ID: `TOMCAT_PORT`, Secret: `8080`)

### Jenkins 파이프라인 생성

1. Jenkins 대시보드에서 **New Item**을 클릭
2. 이름: `<PipelineName>`, 타입: Pipeline 선택
3. General:
    - GitHub project에 GitHub 리포지토리 URL 입력
4. Build Triggers:
    - GitHub hook trigger for GITScm polling 체크
5. Pipeline:
    - Definition: Pipeline script from SCM
    - SCM: Git
    - Repository URL: `https://github.com/<깃허브username>/<원격리포지토리명>.git`
    - Credentials: github-pat
    - Branch: main
    - Script Path: Jenkinsfile
6. "Save" 클릭

---

## 🚨 디스크 공간 부족 문제 및 해결

### 문제 현상

- AWS EC2 t3.small 인스턴스에서 Jenkins를 실행하면, 디스크 공간 부족으로 인해 Jenkins Built-In Node가 오프라인으로 전환됨
- Jenkins 대시보드에 다음 메시지 출력:
<img width="1400" alt="스크린샷" src="https://github.com/user-attachments/assets/bbeac900-d0f5-4eb6-9e4a-f18e0a3400d3">

```plaintext
Disk space is below threshold of 1.00 GiB. Only 946.26 MiB out of 950.71 MiB left on /tmp
```

- GitHub Webhook 요청이 Jenkins로 전달되지만, 디스크 공간 부족으로 인해 파이프라인이 트리거되지 않음

### 문제 원인

- Jenkins는 빌드 및 테스트 작업 중 임시 파일을 /tmp 디렉토리에 저장하며, 이 디렉토리의 디스크 공간이 부족하면 작업이 실패하거나 노드가 오프라인으로 전환됨
- EC2 t3.small 인스턴스의 기본 EBS 볼륨 크기가 작아 디스크 공간이 빠르게 소진됨
- Jenkins 작업 로그, 캐시, 빌드 아티팩트 등이 디스크 공간을 차지

### 로그 확인

1. 디스크 사용량 확인:

```shell
df -h
```

    출력 예시:

```text
Filesystem Size Used Avail Use% Mounted on /dev/nvme0n1p1 8.0G 7.1G 950M 89% / tmpfs 990M 0 990M 0% /tmp
```

- /tmp 디렉토리의 사용 가능한 공간이 950MB로 부족

2. /tmp 디렉토리 내 파일 확인:

```shell
sudo du -sh /tmp/*
```

    출력 예시:

```text
500M /tmp/jenkins-12345.tmp 200M /tmp/build-cache
```

### 해결 방안: 디스크 공간 부족 해결

디스크 공간 부족 문제를 해결하기 위해 임시 파일 정리와 디스크 공간 확보 방법을 적용하며, 장기적인 해결책으로 EBS 볼륨 확장을 고려

#### 1. 임시 파일 정리

- /tmp 디렉토리의 불필요한 파일 제거:

```shell
sudo rm -rf /tmp/jenkins-*.tmp sudo rm -rf /tmp/build-cache
```

- Jenkins 작업 로그 및 아티팩트 정리:
    - Jenkins 대시보드 → "Manage Jenkins" → "System Configuration" → "Workspace Cleanup" 플러그인 설치
    - 오래된 빌드 아티팩트 삭제 설정:

```groovy
stage('Clean Workspace') {
    steps {
        cleanWs()
    }
}
```

#### 2. 디스크 공간 모니터링 및 임계값 조정

- Jenkins 디스크 공간 임계값 조정:
    - Jenkins 대시보드 → "Manage Jenkins" → "Configure System" → "Disk Usage" 섹션
    - 디스크 공간 임계값을 500MB로 낮춤 (기본값 1GB)
- 디스크 사용량 모니터링 스크립트 추가:

```shell
df -h | grep "/tmp" | awk '{print $4}' | grep -i "m" | sed 's/M//'
```

- 사용 가능한 공간이 500MB 미만일 경우 알림 전송

#### 3. 장기적인 해결책: EBS 볼륨 확장

- AWS EC2 인스턴스의 EBS 볼륨 크기 확장:
1. AWS Management Console에서 EC2 대시보드로 이동
2. 해당 인스턴스의 EBS 볼륨 선택 → "Modify Volume" → 용량을 16GB로 확장
3. 인스턴스에서 파일 시스템 확장:

```shell
sudo growpart /dev/nvme0n1 1 sudo xfs_growfs /
```

1. 확장된 디스크 확인:

```shell
df -h
```

출력 예시:

```text
Filesystem Size Used Avail Use% Mounted on /dev/nvme0n1p1 16G 7.1G 8.9G 45% / tmpfs 990M 0 990M 0% /tmp
```

### 디스크 공간 정리 및 확장 후 결과

- /tmp 디렉토리의 사용 가능한 공간이 증가하여 Jenkins Built-In Node가 다시 온라인으로 전환
- GitHub Webhook 요청이 정상적으로 Jenkins 파이프라인을 트리거
- 빌드 및 배포 작업이 안정적으로 수행됨

---

## Jenkins CI/CD Pipeline 구축

### Dockerfile
> Dockerfile은 docker에서 image를 만들 때 사용되는 필수 요소
> image를 어떻게 만들 것인가에 대하여 정의하는 내용
> 이렇게 만들어진 docker image는 나중에 docker에 의해 Container로 실행됨으로써 **일관된 환경에서의 애플리케이션 실행** 을 보장

Jenkins를 활용한 CI/CD pipeline에서 Dockerfile은 Docker image build 단계에서 사용됨

```dockerfile
# 기반 이미지로 OpenJDK 21 사용
FROM openjdk:21-jdk-slim

# 작업 디렉토리 설정
WORKDIR /app

# 빌드된 JAR 파일 복사
COPY build/libs/myapp.jar app.jar

# 컨테이너 실행 시 실행할 명령어 (환경변수 주입 가능)
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Dockerfile 설명

- FROM openjdk:21-jdk-slim: 경량화된 OpenJDK 21 이미지를 기반으로 사용
- WORKDIR /app: 작업 디렉토리를 /app으로 설정
- COPY build/libs/myapp.jar app.jar: 빌드된 JAR 파일을 컨테이너 내로 복사
- ENTRYPOINT: 컨테이너 실행 시 JAR 파일을 실행하며, 환경변수 주입 가능

---

### Jenkinsfile

Jenkinsfile은 빌드, 테스트, 배포 단계와 같은 Jenkins 파이프라인을 정의하는 스크립트
환경변수를 안전하게 관리하기 위해 Jenkins 크리덴셜 사용

```Jenkinsfile
pipeline {
    agent any

    environment {
        // Jenkins에서 관리하는 환경변수 참조
        DOCKER_IMAGE_NAME = credentials('DOCKER_IMAGE_NAME')
        DOCKER_CONTAINER_NAME = credentials('DOCKER_CONTAINER_NAME')
        TOMCAT_PORT = credentials('TOMCAT_PORT')
    }

    stages {
        stage('Clone Code') {
            steps {
                echo 'Cloning the code from GitHub...'
                git url: 'https://github.com/your-username/your-repo.git', branch: 'main', credentialsId: 'github-token'
            }
        }

        stage('Build JAR') {
            steps {
                echo 'Building the Spring Boot JAR with Gradle...'
                sh './gradlew clean build -x test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t ${DOCKER_IMAGE_NAME}:latest .'
            }
        }

        stage('Deploy Docker Container') {
            steps {
                echo 'Deploying Docker container...'
                // 기존 컨테이너 중지 및 제거
                sh 'docker stop ${DOCKER_CONTAINER_NAME} || true'
                sh 'docker rm ${DOCKER_CONTAINER_NAME} || true'
                // 새 컨테이너 실행 (환경변수 주입)
                sh """
                    docker run -d --name ${DOCKER_CONTAINER_NAME} \
                    -e SERVER_PORT=${TOMCAT_PORT} \
                    -p ${TOMCAT_PORT}:${TOMCAT_PORT} ${DOCKER_IMAGE_NAME}:latest
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
```

### Jenkinsfile 설명

- agent any: 모든 Jenkins Worker에서 작업 실행 가능
- environment: Jenkins 크리덴셜에서 환경변수를 참조
- stages: 빌드, 테스트, 배포 단계를 정의
    - Clone Code: GitHub에서 소스코드 복제
    - Build JAR: Gradle을 사용해 JAR 파일 빌드
    - Build Docker Image: Dockerfile을 사용해 Docker 이미지 빌드
    - Deploy Docker Container: 기존 컨테이너 제거 후 새 컨테이너 배포
- post: 파이프라인 성공/실패 시 알림

---

## 🚨 환경변수 보안 조치

스프링부트 프로젝트는 외부 라이브러리 및 API 통합 과정에서 다양한 환경변수를 필요로 하며, 이러한 환경변수는 민감한 정보로 노출되지 않도록 보호 조치

### ⚠️ 노출 포인트

1. 도커 및 컨테이너에서 환경변수를 출력하는 printenv 명령어를 통한 노출
2. 배포된 서버의 운영체제나 도커 컨테이너 내부에 존재하는 .env 파일을 통한 노출

### 해결 방안 및 대책

환경변수가 노출되지 않도록 하기 위해 **비밀 관리 도구와 애플리케이션 직접 통합** 방법을 사용

### 비밀 관리 도구란?

환경변수와 같은 민감한 정보를 안전하게 보관하고 애플리케이션에 제공하는 도구

- 보안: 암호화된 저장소에서 민감한 정보 관리
- 편리성: 애플리케이션에서 동적으로 환경변수 주입 가능
- 효율성: 중앙화된 관리로 유지보수 용이

대표적인 비밀 관리 도구:

- HashiCorp Vault
- AWS Secrets Manager

### 비밀 관리 도구 1: HashiCorp Vault

1. Vault 서버 설치 및 설정:

```shell
sudo dnf install vault -y sudo systemctl start vault
```

1. Vault에 환경변수 저장:

```shell
vault kv put secret/myapp DB_URL="jdbc:postgresql://..." DB_USERNAME="user" DB_PASSWORD="pass"
```

2. Jenkinsfile에서 Vault 통합:

```groovy
stage('Retrieve Secrets from Vault') {
    steps {
        withVault(
            vaultSecrets: [
                [path: 'secret/myapp', secretValues: [
                    [envVar: 'DB_URL', vaultKey: 'DB_URL'],
                    [envVar: 'DB_USERNAME', vaultKey: 'DB_USERNAME'],
                    [envVar: 'DB_PASSWORD', vaultKey: 'DB_PASSWORD']
                ]]
            ]
        ) {
            sh 'echo DB_URL=${DB_URL} > application.properties'
        }
    }
}
```

### 비밀 관리 도구 2: AWS Secrets Manager

1. AWS Secrets Manager에 환경변수 저장:
    - AWS Management Console에서 Secrets Manager로 이동
    - 새로운 비밀 생성: DB_URL, DB_USERNAME, DB_PASSWORD 등 저장
2. Jenkinsfile에서 AWS Secrets Manager 통합:

```groovy
stage('Retrieve Secrets from AWS Secrets Manager') {
    steps {
        withAWS(credentials: 'aws-credentials-id') {
            script {
                def secrets = sh(script: 'aws secretsmanager get-secret-value --secret-id myapp-secrets --query SecretString --output text', returnStdout: true).trim()
                def json = readJSON text: secrets
                env.DB_URL = json.DB_URL
                env.DB_USERNAME = json.DB_USERNAME
                env.DB_PASSWORD = json.DB_PASSWORD
            }
            sh 'echo DB_URL=${DB_URL} > application.properties'
        }
    }
}
```

---

## 🔍 파이프라인 테스트

### 테스트 환경

- Jenkins 서버: AWS EC2 t3.small (스왑 메모리 2GB 설정)
- GitHub 리포지토리: Webhook 설정 완료
- Docker 및 Gradle 설치 완료

### 테스트 시나리오

1. GitHub 리포지토리에 새로운 커밋 푸시
2. Jenkins 파이프라인이 Webhook을 통해 자동 트리거
3. 빌드, 테스트, 배포 단계 확인

### 테스트 결과

- 성공 케이스:
    - GitHub Webhook 요청이 Jenkins로 정상 전달
    - Jenkins 파이프라인이 자동 실행
    - Docker 컨테이너가 성공적으로 배포됨
    - 로그 출력 예시:
```text
[Pipeline] stage
[Pipeline] { (Clone Code)
[Pipeline] echo
Cloning the code from GitHub...
[Pipeline] git
[Pipeline] stage
[Pipeline] { (Build JAR)
[Pipeline] echo
Building the Spring Boot JAR with Gradle...
[Pipeline] sh
[Pipeline] stage
[Pipeline] { (Build Docker Image)
[Pipeline] echo
Building Docker image...
[Pipeline] sh
[Pipeline] stage
[Pipeline] { (Deploy Docker Container)
[Pipeline] echo
Deploying Docker container...
[Pipeline] sh
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // pipeline
```

- 실패 케이스 (메모리 부족):
    - 스왑 메모리 설정 전, 빌드 중 메모리 부족으로 파이프라인 실패
    - 스왑 메모리 설정 후 재시도 시 성공

---

## 🚀 개선할 점

1. 로그 레벨 조정:
    - 현재 Jenkins 로그가 상세하게 출력되어 디버깅에 유용하지만, 프로덕션 환경에서는 로그 레벨을 INFO로 조정 필요
    - Jenkins 로그 설정 파일 수정: /var/lib/jenkins/config.xml
2. 스왑 메모리 최적화:
    - 스왑 메모리 사용은 임시 해결책이며, t3.small 환경에서의 메모리 부족 문제를 완화하기 위해 사용됨
    - 장기적으로 메모리 부족 문제가 반복될 경우 t3.medium 이상으로 업그레이드 고려 가능
    - 스왑 메모리 사용량 모니터링 추가:
```shell
swapon --show
```

3. 파이프라인 병렬화:
    - 현재 파이프라인은 순차적으로 실행되지만, 빌드와 테스트 단계를 병렬화하여 실행 시간 단축 가능
    - Jenkinsfile 수정 예시:
```groovy
stage('Build and Test') {
    parallel {
        stage('Build JAR') {
            steps {
                sh './gradlew clean build -x test'
            }
        }
        stage('Run Tests') {
            steps {
                sh './gradlew test'
            }
        }
    }
}
```

4. 비밀 관리 도구 통합 강화:
    - Vault 또는 AWS Secrets Manager 통합 시, 캐싱 전략 추가하여 빈번한 API 호출 감소
    - Vault 플러그인 또는 AWS SDK 최적화

---

## 📝 결론

AWS EC2 t3.small 인스턴스 (Amazon Linux 2023)에서 Jenkins를 활용하여 스프링부트 프로젝트의 CI/CD 파이프라인 구축
GitHub Webhook을 통해 코드 변경사항을 자동으로 트리거하고, 빌드, 테스트, 배포 단계를 자동화
t3.small 환경에서의 메모리 부족 문제를 스왑 메모리 설정으로 해결하여 저사양 환경에서도 안정적인 파이프라인 운영 가능
환경변수 보안을 위해 HashiCorp Vault와 AWS Secrets Manager를 통합하여 민감한 정보의 노출 위험 최소화
장기적으로 메모리 부족 문제가 반복될 경우 t3.medium 이상으로의 업그레이드 고려가 필요하지만, 현재 t3.small 환경에서의 해결 방안을 통해 비용 효율적 운영 가능

---

## 📜 참조

- [🔗 Jenkins 공식 문서 🔗](https://www.jenkins.io/doc/)
- [🔗 AWS EC2 사용자 가이드 🔗](https://docs.aws.amazon.com/ec2/)
- [🔗 HashiCorp Vault 공식 문서 🔗](https://www.vaultproject.io/docs)
- [🔗 AWS Secrets Manager 공식 문서 🔗](https://docs.aws.amazon.com/secretsmanager/)
- [🔗 GitHub Webhook 설정 가이드 🔗](https://docs.github.com/en/developers/webhooks-and-events/webhooks/creating-webhooks)