# :space_invader: 비밀번호 및 API 키 노출 방지를 위한 자동화 시스템
"GitHub 중요정보 노출을 Trivy로 자정마다 스캔하고 Slack으로 즉시 알림을 보내는 자동화 시스템"


## 👨‍👨‍👧 개발팀원

| <img src="https://avatars.githubusercontent.com/u/65991884?v=4" width="80"> | <img src="https://avatars.githubusercontent.com/u/90691610?v=4" width="80"> | <img src="https://avatars.githubusercontent.com/u/79312705?v=4" width="80"> |
|:---:|:---:|:---:|
| [류채현](https://github.com/RyuChaeHyun) | [이석철](https://github.com/SeokCheol-Lee) | [김상민](https://github.com/isshomin) |

<br>

## 📌 개요

이 프로젝트는 **소스 코드 관리 시스템인 GitHub**에서 비밀번호와 같은 비밀정보(예: API 키, 패스워드)가 노출될 경우, 이를 **신속하게 탐지하고 알림**을 통해 **즉각적인 대응을 가능하게 하는 자동화 시스템**을 구축하는 것입니다. <br>

**Trivy**를 활용하여 비밀정보를 스캔하고, 스캔 결과를 **Slack**으로 자동 전송하여 팀원들이 즉시 조치를 취할 수 있도록 구현하였습니다. 이 시스템은 개발팀의 보안 인식을 높이고, 비밀정보 유출로 인한 사이버 공격을 예방하는 데 기여할 것입니다.

<br>

<br>

## 🎖️ 주요 기능

1. **비밀정보 스캔**:
   - Trivy를 이용해 GitHub 저장소에서 비밀정보(예: API 키, 비밀번호)를 자동으로 스캔합니다.

2. **자동 알림 전송**:
   - 스캔 결과 발견 시 Slack으로 자동 알림을 전송하여 팀원이 즉시 대응할 수 있도록 합니다.

3. **주기적인 스캔 설정**:
   - crontab을 사용해 비밀정보 스캔을 매일 자정에 자동 실행하여 최신 상태를 유지합니다.
<br>


## 📊 프로젝트 과정
본 프로젝트는 **Trivy를 사용하여 GitHub 저장소를 스캔**하고, 발견된 비밀정보를 **Slack을 통해 자동으로 알림을 전송하는 구조**로 설계되었습니다. 

<br>

### 1️⃣ Trivy 설치

```
# Trivy를 설치하기 위해 필요한 도구와 패키지를 설치
sudo apt-get install wget apt-transport-https gnupg lsb-release

# Trivy 저장소의 패키지를 신뢰할 수 있도록 GPG 키 추가
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

# APT 소스 목록에 Trivy 저장소 추가
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list

# 패키지 목록 업데이트
sudo apt-get update

# Trivy 설치
sudo apt-get install trivy
```
<br>

### 2️⃣ URL 요청 수신
- Spring Boot 애플리케이션에서 HTTP GET 요청을 통해 비밀정보 스캔을 위한 GitHub 리포지토리 URL을 수신합니다.
- `http://localhost:8081/api/scan?repoUrl=https%3A%2F%2Fgithub.com%2FSeokCheol-Lee%2FWebToonProject.git"`


<br>

### 3️⃣ 스캔 스크립트 실행
- 수신한 URL을 인자로 사용하여 `scan.sh` 스크립트를 실행합니다.
- 스크립트 내에서는 Trivy를 사용하여 리포지토리에서 비밀정보를 스캔합니다.

```
#!/bin/bash

# Git 리포지토리 URL
REPO_URL=$1
TEMP_DIR=$(mktemp -d)

# Git 리포지토리 클론
git clone $REPO_URL $TEMP_DIR

# Trivy로 비밀 정보 스캔
docker run --rm -v $TEMP_DIR:/repo aquasec/trivy fs --scanners secret /repo

# 클론한 디렉토리 삭제
rm -rf $TEMP_DIR
```

<br>

### 4️⃣ 스캔 결과 저장
- Trivy 스캔의 출력 결과를 `result` 변수에 저장합니다.
![image (10)](https://github.com/user-attachments/assets/6a9523f7-3de1-4c93-bf32-36a67d034cd1)
<br>
<br>

- 비밀정보가 발견되었는지 `SecretDetect`로 어떤 정보가 노출되었는지를 확인합니다.
![image (10)](https://github.com/user-attachments/assets/29f77d4c-845d-4d59-9461-e0acb938d36a)


<br>

### 5️⃣ Slack 알림 전송
<p align="left"><img src="https://github.com/user-attachments/assets/df30ea37-3231-4079-a239-583de0cd51dd"></p>

- 스캔 결과를 기반으로 **Slack으로 알림을 전송** 합니다. <br>


<br>

![image](https://github.com/user-attachments/assets/c2080700-b31c-40a3-b45f-22f5477c5f94)


- 발견된 비밀정보에 대한 상세 내용을 포함하여 **팀원들에게 신속하게 알림**을 보냅니다.
  
<br>

### 6️⃣ crontab 편집 및 cron job 추가

매일 자정에 스캐닝 될 수 있도록 crontab으로 자동화를 설정한다.
```
crontab -e

26 16 * * * /usr/bin/curl -s "http://localhost:8081/api/scan?repoUrl=https%3A%2F%2Fgithub.com%2FSeokCheol-Lee%2FWebToonProject.git -H accept: */* -d" >> /path/to/your/scan.log 2>&1
```

<br>



## 🧐 결론

이번 프로젝트를 통해 **Trivy와 crontab을 활용하여 비밀정보 탐지 및 슬랙 알림 자동화 시스템을 구축**함으로써, 현대의 DevOps 환경에서 필수적인 보안 솔루션의 필요성을 체감하게 되었습니다.

비밀정보가 코드에 노출될 경우 발생할 수 있는 보안 사고를 예방하기 위해 지속적인 모니터링의 중요성을 인식하게 되었습니다.

**자동화된 프로세스를 통해 팀원들이 신속하게 대응할 수 있는 환경을 마련**하였고, 이는 팀의 생산성을 향상시키는 데 크게 기여했습니다.


<br>

## 🤔 아쉬웠던 점

첫째,**포트 스캐닝 기능**을 추가하고 싶었으나 구현하지 못한 점이 아쉽습니다. 이는 시스템의 보안 상태를 보다 면밀하게 점검할 수 있는 중요한 요소로, 향후 개선사항으로 남겨두고 싶습니다.

둘째, **알림 과부하**가 발생할 수 있다는 점입니다. 주기적으로 수행되는 스캔으로 인해 많은 알림이 팀원들에게 부담이 될 수 있으며, 중요하지 않은 경고가 자주 발생하면 실제로 중요한 알림이 묻히는 문제가 발생할 수 있습니다.

<br>
