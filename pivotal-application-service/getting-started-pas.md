# Pivotal Application Service on AWS
AWS 위에 Pivotal Application Service를 구축하는 핸즈온 입니다.

# 사전 준비 사항
* AWS 계정 생성 - https://aws.amazon.com/ko/premiumsupport/knowledge-center/create-and-activate-aws-account
* 공개 도메인 (소유하고 있는 공개 도메인이 없는 경우 구매 필요)
* AWS Limit Increase 요청 - https://docs.pivotal.io/pivotalcf/2-3/customizing/aws.html

# Jumpbox 생성 및 설정
* Ubuntu Server 18.04 LTS (HVM), SSD Volume Type AMI(ami-07ad4b1c3af1ea214) 선택
* t2.micro 타입 선택
* 스토리지는 SSD gp2 타입 및 용량 50GB 설정
* VPC의 서브넷 중 퍼블릭 서브넷 선택
* Auto-assign Public IP Enable 설정
* Jumpbox의 Security Groups에 SSH 포트(22번)에 대해 자신의 IP로 인바운드 설정
```
Type | Protocol | Port Range | Source               | Description
SSH  | TCP      | 22         | <Your IP Address>/32 | SSH Access from Your Network
```
# Control Plane 구성
## Apt 업데이트 및 workspace 폴더 생성
```
sudo apt-get update -y
mkdir ~/workspace
cd ~/workspace
```

## Terraform 설치 (버전 0.11.0 이상)
```
sudo apt-get install -y unzip
mkdir ~/workspace/downloads
cd ~/workspace/downloads
wget https://releases.hashicorp.com/terraform/0.11.10/terraform_0.11.10_linux_amd64.zip
unzip terraform*.zip
chmod +x terraform
sudo mv terraform /usr/local/bin
rm terraform*.zip
```
### Terraform 설치 완료 확인
```
ubuntu@ip-0-0-0-0:~$ terraform -v
Terraform v0.11.10
```
## Bosh CLI 설치
```
cd ~/workspace/downloads
wget -O bosh https://github.com/cloudfoundry/bosh-cli/releases/download/v5.4.0/bosh-cli-5.4.0-linux-amd64
chmod +x bosh
sudo mv bosh /usr/local/bin
```
### Bosh CLI 설치 완료 확인
```
ubuntu@ip-0-0-0-0:~$ bosh -v
version 5.4.0-891ff634-2018-11-14T00:22:02Z

Succeeded
```
## Bosh create-env 종속성 패키지 설치
```
sudo apt-get install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt-dev libxml2-dev libssl-dev libreadline libxslt1-dev libyaml-dev libsqlite3-dev sqlite3
```
## Bosh Bootloader(bbl) 설치
```
cd ~/workspace/downloads
wget -O bbl https://github.com/cloudfoundry/bosh-bootloader/releases/download/v6.10.54/bbl-v6.10.54_linux_x86-64
chmod +x bbl
sudo mv bbl /usr/local/bin
```
### Bosh Bootloader(bbl) 설치 확인
```
ubuntu@ip-0-0-0-0:~$ bbl -v
bbl 6.10.54 (linux/amd64)
```
## UAA CLI(uaac) 설치
```
sudo gem install cf-uaac
```
### UAA CLI(uaac) 설치 확인
```
ubuntu@ip-0-0-0-0:~$ uaac -v
UAA client 4.1.0
```
## Control Plane 구성을 위한 IAM 계정 생성
### IAM 권한 생성 (bbl-policy)
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "logs:*",
                "elasticloadbalancing:*",
                "cloudformation:*",
                "iam:*",
                "kms:*",
                "route53:*",
                "ec2:*",
                "s3:*"
            ],
            "Resource": "*"
        }
    ]
}
```
### IAM 계정 생성
* Services -> IAM -> Users -> Add User -> 계정명 bbl 입력 및 Programmatic access 체크
* 위에서 생성한 bbl-policy 정책을 설정
### Jumpbox 내 환경 변수 설정
* BBL_ACCESS_KEY_ID: bbl IAM 계정의 Access Key ID 입력
* BBL_SECRET_ACCESS_KEY: bbl IAM 계정의 Secret Access Key 입력
* REGION: PCF를 구축할 리전명 (ap-northeast1, ap-northeast-2) 입력
```
BBL_ACCESS_KEY_ID=<Your BBL Access Key ID>
BBL_SECRET_ACCESS_KEY=<Your BBL Secret Access Key>
REGION=<Your Region>
```

### Bosh Bootloader 구성 (bbl up)
bbl 명령어는 반드시 bbl 폴더 경로에서 실행해줘야함
```
mkdir -p ~/workspace/bbl/terraform
wget https://raw.githubusercontent.com/pivotalservices/concourse-credhub/master/bbl-terraform/aws/concourse-lb_override.tf
cd ~/workspace/bbl
cat << EOF > bblup.sh 
bbl up --aws-access-key-id ${BBL_ACCESS_KEY_ID} \
         --aws-secret-access-key ${BBL_SECRET_ACCESS_KEY} \
         --aws-region ${REGION} \
         --iaas aws \
         --lb-type concourse
EOF
chmod +x bblup.sh
./bblup.sh
cd ~/workspace/bbl/
eval "$(bbl print-env)"
```

### Bosh Bootloader 작업 완료 확인
#### Bosh 및 Jumpbox 인스턴스 생성 확인
* Bosh: AWS 콘솔에서 bosh/0으로 생성된 인스턴스 확인
* Jumpbox: AWS 콘솔에서 jumpbox/0으로 생성된 인스턴스 확인
#### Concourse Load Balancer 생성 확인
* Concourse Load Balancer: bbl-env-xx-xxxxxx-concourse-lb 생성(Network Load Balancer) 확인
#### Bosh 연결 확인
```
ubuntu@ip-0-0-0-0:~$ bosh env
Using environment 'https://10.0.0.6:25555' as client 'admin'

Name               bosh-bbl-env-titicaca-2018-12-10t06-11z
UUID               2ab57208-1a1b-4bd4-b9b9-170273c87c46
Version            268.3.0 (00000000)
Director Stemcell  ubuntu-xenial/170.9
CPI                aws_cpi
Features           compiled_package_cache: disabled
                   config_server: enabled
                   local_dns: enabled
                   power_dns: disabled
                   snapshots: disabled
User               admin

Succeeded
```
#### External Host 환경 변수 설정
1. External Host 값 확인
```
cd ~/workspace/bbl
ubuntu@ip-0-0-0-0:~/workspace/bbl$ bbl lbs
Concourse LB: bbl-xxx-xx-xxxxx-concourse-lb [bbl-env-xx-xxxx-concourse-lb-xxxxxxxxxxxx.elb.ap-northeast-1.amazonaws.com]
```

2. External Host 값 설정
1번 단계에서 나온 출력 결과의 []안에 있는 값을 EXTERNAL_HOST 변수에 설정
```
export EXTERNAL_HOST=bbl-env-xx-xxxx-concourse-lb-xxxxxxxxxxxx.elb.ap-northeast-1.amazonaws.com
```


# 참고
* Bosh CLI Download - https://github.com/cloudfoundry/bosh-cli/releases
* Terraform Download - https://www.terraform.io/downloads.html
* Bosh Bootloader - https://github.com/cloudfoundry/bosh-bootloader
* Concourse Bosh Deployment - https://github.com/concourse/concourse-bosh-deployment
* Concourse with CredHub - https://github.com/pivotalservices/concourse-credhub
* PCF Pipeline - https://github.com/pivotal-cf/pcf-pipelines
