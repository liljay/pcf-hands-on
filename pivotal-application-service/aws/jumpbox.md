# Jumpbox 구성
Jumpbox를 구성하여 Control Plane 및 Pivotal Application Service를 구축한다.

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
&& cd ~/workspace/bbl/terraform \
&& wget https://raw.githubusercontent.com/pivotalservices/concourse-credhub/master/bbl-terraform/aws/concourse-lb_override.tf \
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
### 필수 도구 설치
#### Terraform 설치 (버전 0.11.0 이상)
Control Plane을 AWS에 구성하기 위해 사용 되는 커맨드 라인 도구 입니다.
```
mkdir -p ~/workspace/downloads
cd ~/workspace/downloads
wget https://releases.hashicorp.com/terraform/0.11.10/terraform_0.11.10_linux_amd64.zip
unzip terraform*.zip
chmod +x terraform
sudo mv terraform /usr/local/bin
rm terraform*.zip
```
#### Terraform 설치 확인
```
ubuntu@ip-0-0-0-0:~$ terraform -v
Terraform v0.11.10
```
#### Bosh create-env 종속성 패키지 설치
Bosh create-env 명령어를 통해 Bosh 인스턴스를 구성하며 이 명령어를 위해 필요한 종속성 패키지들을 설치 합니다.
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
#### Bosh Bootloader(bbl) CLI 설치
Bosh를 쉽게 구성 할 수 있도록 도움을 주는 커맨드 라인 도구 입니다.
```
cd ~/workspace/downloads && \
wget -O bbl https://github.com/cloudfoundry/bosh-bootloader/releases/download/v6.10.54/bbl-v6.10.54_linux_x86-64 && \
chmod +x bbl && \
sudo mv bbl /usr/local/bin
```
#### Bosh Bootloader(bbl) CLI 설치 확인
```
ubuntu@ip-0-0-0-0:~$ bbl -v
bbl 6.10.54 (linux/amd64)
```
#### CredHub CLI 설치
CredHub 관리를 위한 커맨드 라인 도구 입니다.
```
cd ~/workspace/downloads
wget https://github.com/cloudfoundry-incubator/credhub-cli/releases/download/2.2.0/credhub-linux-2.2.0.tgz
tar xvf credhub*.tgz
chmod +x credhub
sudo mv credhub /usr/local/bin
```
#### Credhub CLI 설치 확인
```
ubuntu@ip-0-0-0-0:~$ credhub --version
CLI Version: 2.2.0
```
#### Fly CLI 설치
Concourse 파이프라인 관리를 위한 커맨드 라인 도구 입니다.
```
cd ~/workspace/downloads
wget -O fly https://github.com/concourse/concourse/releases/download/v4.2.1/fly_linux_amd64
chmod +x fly
sudo mv fly /usr/local/bin
```
#### Fly CLI 설치 확인
```
ubuntu@ip-0-0-0-0:~$ fly -v
4.2.1
```
#### UAA Cient(uaac) 설치
UAA Client는 UAA에 접근하기 위한 커맨드 라인 도구 입니다.
```
sudo gem install cf-uaac
```
#### UAA Client(uaac) 설치 확인
```
ubuntu@ip-0-0-0-0:~$ uaac -v
UAA client 4.1.0
```
#### Jumpbox 내 핸즈온을 위한 환경 변수 설정
##### Control Plane 구성을 위한 IAM 계정 및 리전 설정
* Jumpbox에서 아래의 환경 변수 사전 설정 필요
* BBL_ACCESS_KEY_ID: bbl IAM 계정의 Access Key ID 입력
* BBL_SECRET_ACCESS_KEY: bbl IAM 계정의 Secret Access Key 입력
* AWS_REGION: PCF를 구축할 리전명 (ap-northeast1, ap-northeast-2) 입력
```
export BBL_ACCESS_KEY_ID=<Your BBL Access Key ID>
export BBL_SECRET_ACCESS_KEY=<Your BBL Secret Access Key>
export AWS_REGION=<Your Region>
```
##### AWS PAS 용도 인증서, PAS 용도 S3 버킷, 도메인, 도메인에 대한 Route 53 Hosted Zone ID, PAS 용도 리소스 접두사 설정
* Jumpbox에서 아래의 환경 변수 사전 설정 필요
```
export AWS_CERT_ARN = <PAS 핸즈온에 사용할 도메인들의 인증서 ARN>
export PAS_DOMAIN = <PAS 핸즈온에 사용할 도메인>
export ROUTE_53_ZONE_ID = <도메인을 등록한 Route 53 Hosted Zone ID>
export S3_OUTPUT_BUCKET = <Terraform이 PAS 구성시 Terraform 상태 저장을 위한 S3 버킷>
export TERRAFORM_PREFIX = <Terraform이 PAS 구성시 생성하는 리소스들의 접두사>
```
##### Concourse 관리자 계정 환경 변수 설정
* Jumpbox에서 아래의 환경 변수 사전 설정 필요
* CC_ADMIN_USERNAME = <Concourse 관리자 계정명>
* CC_ADMIN_PW = <Concourse 관리자 비밀번호>
```
export CC_ADMIN_USERNAME=<Your Concourse Admin Username>
export CC_ADMIN_PW=<Your Concourse Admin Password>
```
