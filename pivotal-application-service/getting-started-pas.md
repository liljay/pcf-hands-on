# Pivotal Application Service on AWS
AWS 상에 Pivotal Application Service를 구축하는 핸즈온 입니다.

# 사전 준비 사항
* AWS 계정 생성 - https://aws.amazon.com/ko/premiumsupport/knowledge-center/create-and-activate-aws-account
* 공개 도메인 구매 (소유하고 있는 공개 도메인이 없는 경우)
* Limit Increase 요청

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
Services -> IAM -> Users -> Add User -> 계정명 bbl 입력 -> Programmatic access 체크
정책을 bbl-policy 설정

### Bosh Bootloader (배포 설정) 
```
BBL_ACCESS_KEY_ID=<Your BBL Access Key ID>
BBL_SECRET_ACCESS_KEY=<Your BBL Secret Access Key>
REGION=ap-northeast-1

mkdir ~/workspace/bbl
cd ~/workspace/bbl
cat << EOF > bblup.sh 
bbl up --aws-access-key-id $(BBL_ACCESS_KEY_ID) \
         --aws-secret-access-key $(BBL_SECRET_ACCESS_KEY) \
         --aws-region ap-northeast-1 \
         --iaas aws \
         --lb-type concourse
EOF
chmod +x bblup.sh
./bblup.sh
```

# 참고
* Bosh CLI Download - https://github.com/cloudfoundry/bosh-cli/releases
* Terraform Download - https://www.terraform.io/downloads.html
* Bosh Bootloader - https://github.com/cloudfoundry/bosh-bootloader
* Concourse Bosh Deployment - https://github.com/concourse/concourse-bosh-deployment
* Concourse with CredHub - https://github.com/pivotalservices/concourse-credhub
* PCF Pipeline - https://github.com/pivotal-cf/pcf-pipelines
