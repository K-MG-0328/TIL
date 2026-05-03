### EC2 인스턴스 초기 세팅
시스템 업데이트   
```
sudo dnf update -y
```
  
사용자 생성  
```
sudo adduser admin  
sudo passwd admin
```
  
권한 설정  
```
sudo usermod -aG wheel admin   
```
-a: 추가(append) 모드로, 기존 그룹에서 제거하지 않고 새 그룹을 추가.  
-G wheel: wheel 그룹에 추가.  
wheel 그룹은 Amazon Linux에서 sudo 명령을 사용할 수 있는 관리자 권한을 가진 그룹  
  
#### SSH 키 기반 접속 설정  

SSH 디렉토리 및 키 파일 생성  

```
sudo mkdir -p /home/admin/.ssh  
sudo touch /home/admin/.ssh/authorized_keys
```

공개 키 추가   
```
cat /home/ec2-user/.ssh/authorized_keys | sudo tee -a /home/admin/.ssh/authorized_keys  
```
권한 설정  
```
sudo chmod 700 /home/admin/.ssh  
sudo chmod 600 /home/admin/.ssh/authorized_keys  
sudo chown -R admin:admin /home/admin/.ssh  
```
chown: 파일 또는 디렉토리의 소유자(owner)와 그룹(group)을 변경하는 명령어  
-R: 재귀적(recursive) 옵션 지정된 디렉토리(/home/admin/.ssh)와 그 안의 모든 하위 디렉토리 및 파일에 대해 소유권 변경을 적용합니다.  
**admin:admin** : 소유자와 그룹을 지정

#### 유틸리티 설치 
```
sudo dnf install 유틸리티 -y
```
**시스템 관리 유틸리티**    
htop, curl, git  
**파일 유틸리티**  
tree 
**모니터링 및 로깅**   
amazon-cloudwatch-agent

#### 시간대 설정
```
sudo timedatectl set-timezone Asia/Seoul
```

### 프로젝트 환경설정
#### API 서버  
**java 17**    
```
sudo dnf install -y java-17-amazon-corretto-devel
```

**docker 25.0.8**  
```
sudo dnf install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker admin
```

**docker-compose 2.35.1**  
```
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
curl -L: URL에서 파일 다운로드  
$(uname -s)-$(uname -m): 시스템 아키텍처에 맞는 바이너리 선택 (예: Linux-x86_64)  

#### 빌드 배포 서버 
**java 17**    
```
sudo dnf install -y java-17-amazon-corretto-devel
```

**docker 25.0.8**  
```
sudo dnf install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker admin
```

**jenkins**
```
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

키 목록 확인  
rpm -qi gpg-pubkey

젠킨스 설치 
sudo yum install -y jenkins

sudo systemctl start jenkins
sudo systemctl enable jenkins
```

**젠킨스 해줘야할 것들**
```
젠킨스 계정 확인
/etc/passwd

젠킨스 계정을 도커 그룹에 추가  
sudo usermod -aG docker jenkins

도커 그룹 계정 확인
/etc/group
```

### 보안 강화 체크리스트
EC2를 외부에 노출시키므로 초기 세팅 직후 다음을 반드시 적용하는 것이 안전합니다.

**1. SSH 보안**
```bash
# /etc/ssh/sshd_config
PermitRootLogin no            # root 직접 로그인 차단
PasswordAuthentication no     # 비밀번호 로그인 차단 (키 인증만)
Port 22022                    # 기본 포트 변경 (선택)
AllowUsers admin              # 특정 사용자만 허용
```
설정 후 `sudo systemctl restart sshd`. **새 세션을 열어 로그인이 되는지 확인하기 전까지 기존 세션을 닫지 마세요.**

**2. 방화벽 (Security Group + OS firewall)**
- AWS Security Group: 필요한 포트만 inbound 허용. SSH는 가능하면 사무실/VPN IP만.
- OS 레벨: Amazon Linux는 기본적으로 firewalld가 비활성. 필요 시 `sudo systemctl enable --now firewalld`.

**3. 자동 보안 업데이트**
```bash
sudo dnf install -y dnf-automatic
sudo systemctl enable --now dnf-automatic.timer
```

**4. fail2ban으로 무차별 로그인 차단**
```bash
sudo dnf install -y epel-release
sudo dnf install -y fail2ban
sudo systemctl enable --now fail2ban
```

### 설정 검증 체크리스트
설정 후 다음으로 정상 동작 확인:
```bash
# 시간대 / 시스템
timedatectl
free -h
df -h

# Docker
docker --version
docker run --rm hello-world
groups admin                     # docker 그룹에 포함됐는지

# Java
java -version

# Jenkins (빌드 서버)
sudo systemctl status jenkins
curl -I http://localhost:8080    # 200 또는 403이면 살아있음

# SSH 로그인 (새 세션)
ssh -i <key> admin@<ec2-ip>
```

### 버전 선택 이유 정리
- **Java 17**: Amazon Corretto는 무료·LTS·AWS 지원. Java 17은 최신 LTS로 record/sealed class 등 모던 기능 사용 가능. 다음 LTS는 21이지만 라이브러리 호환성 점검 후 마이그레이션.
- **Docker 25.0.x**: BuildKit 기본 활성화, 최신 보안 패치 포함. Amazon Linux 2023 기본 저장소 사용.
- **Jenkins LTS**: 안정성 중심. 주간 릴리스보다 LTS를 권장. 플러그인 호환성도 LTS 기준.

### 다중 인스턴스 환경 확장 시 고려할 점
단일 EC2가 아닌 여러 대로 늘릴 때:
- **상태 관리**: 세션은 Redis/DB로, 파일 업로드는 S3로 → 무상태 인스턴스 만들기
- **로드 밸런서**: ALB(HTTPS termination + 헬스 체크) 앞에 두기
- **로그 중앙화**: CloudWatch Logs 또는 ELK로 인스턴스별 로그 통합
- **이미지화**: 매번 수동 세팅 대신 AMI로 베이크 또는 Terraform/Ansible로 IaC
- **시크릿 관리**: `.env`나 코드에 비밀 정보 하드코딩 금지. AWS Secrets Manager / Parameter Store 사용

### 일반적인 트러블슈팅
| 증상 | 의심 |
|---|---|
| `docker: permission denied` | 사용자가 `docker` 그룹에 미포함, `newgrp docker` 또는 재로그인 |
| Jenkins가 docker 명령 실행 실패 | jenkins 계정이 docker 그룹에 미포함, `sudo usermod -aG docker jenkins` 후 jenkins 재시작 |
| `Connection refused` (외부에서) | Security Group inbound, OS firewall, 애플리케이션 listen 주소(0.0.0.0 vs 127.0.0.1) 점검 |
| `No space left on device` | `df -h`로 확인, Docker 이미지 정리 `docker system prune -af` |
