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
vim, htop, curl, wget, git  
**네트워크 유틸리티**    
net-tools, traceroute, tcpdump, bind-utils  
**보안 유틸리티**  
openssh-clients, firewalld  
**파일 유틸리티**  
tree, unzip, zip, rsync, lsof  
**모니터링 및 로깅**   
amazon-cloudwatch-agent, logrotate   

#### 시간대 설정
```
sudo timedatectl set-timezone Asia/Seoul
```

스토리지 확인 및 마운트
프로젝트 환경설정
