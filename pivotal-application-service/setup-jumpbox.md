# Jumpbox 구성
핸즈온을 진행하기 위한 Jumpbox를 구성하여 Control Plane 및 Pivotal Application Service를 구성한다.

## Jumpbox 생성 조건
* Ubuntu Server 18.04 LTS (HVM), SSD Volume Type AMI(ami-07ad4b1c3af1ea214) 선택
* t2.micro 인스턴스 타입 선택
* 인스턴스 스토리지는 SSD gp2 타입 및 용량 50GB 설정
* 인스턴스의 VPC의 서브넷은 퍼블릭 서브넷 선택
* Auto-assign Public IP Enable 선택
* Jumpbox의 Security Groups에 SSH 포트(22번)에 대해 자신의 IP로 인바운드 설정
```
Type | Protocol | Port Range | Source               | Description
SSH  | TCP      | 22         | <Your IP Address>/32 | SSH Access from Your Network
```

## Jumpbox 내 환경 구성
### 필수 패키지 설치 및 디렉토리 생성
```
sudo apt-get update -y && sudo apt-get install -y unzip jq \
&& mkdir -p ~/workspace/bbl/terraform \ 
&& mkdir -p ~/workspace/downloads 
```
### 필수 Github 저장소 복제
* Concourse Bosh Deployment - PCF를 구성하기 위한 CI/CD 도구 Bosh 릴리즈 저장소
* Concourse CredHub - Concourse의 Web(ATC) VM에 있는 CredHub에 접근하기 위한 저장소
* PCF Pipelines - PCF를 구성하기 위한 Concourse 파이프라인 저장소

```
cd ~/workspace/ && \
git clone https://github.com/concourse/concourse-bosh-deployment.git && \
git clone https://github.com/pivotalservices/concourse-credhub.git && \
git clone https://github.com/pivotal-cf/pcf-pipelines.git

```


#### Bosh Bootloader(bbl) CLI
Bosh를 쉽게 구성 할 수 있도록 도움을 주는 커맨드 라인 도구 입니다.
```
cd ~/workspace/downloads
wget -O bbl https://github.com/cloudfoundry/bosh-bootloader/releases/download/v6.10.54/bbl-v6.10.54_linux_x86-64
chmod +x bbl
sudo mv bbl /usr/local/bin
```
#### Bosh Bootloader(bbl) CLI 설치 확인
```
ubuntu@ip-0-0-0-0:~$ bbl -v
bbl 6.10.54 (linux/amd64)
```
#### Bosh create-env 종속성 패키지 설치
Bosh create-env 명령어를 통해 Bosh 인스턴스를 구성하며 이 명령어를 위해 필요한 종속성 패키지들을 설치합니다.
```
sudo apt-get install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt-dev libxml2-dev libssl-dev libreadline-dev libxslt1-dev libyaml-dev libsqlite3-dev sqlite3
```
### Bosh CLI 설치
Bosh CLI는 Bosh를 관리 하기 위한 커맨드 라인 도구 입니다.
```
cd ~/workspace/downloads
wget -O bosh https://github.com/cloudfoundry/bosh-cli/releases/download/v5.4.0/bosh-cli-5.4.0-linux-amd64
chmod +x bosh
sudo mv bosh /usr/local/bin
```
#### Bosh CLI 설치 확인
```
ubuntu@ip-0-0-0-0:~$ bosh -v
version 5.4.0-891ff634-2018-11-14T00:22:02Z

Succeeded
```


