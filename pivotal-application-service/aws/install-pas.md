# Deploying Pivotal Application Service
## 사전 요구 사항
* Ops Manager와 PCF 버전 호환성 확인
https://docs.pivotal.io/resources/product-compatibility-matrix.pdf
* PAS 테라폼 상태 저장 버킷 생성
** S3 버킷 생성시 Versioning 기능을 활성화 후 생성
** PCF Pipeline의 params.yml 내용 중 S3_OUTPUT_BUCKET에 입력하게 됨

## Concourse 내 CredHub에 민감한 정보(AWS 키, 계정 정보 등) 추가
### Concourse URL 설정
```
export CONCOURSE_URL="https://${EXTERNAL_HOST}"
```
### Concourse Web(ATC) 내 있는 CredHub 접근
```
cd ~/workspace/bbl/
eval "$(bbl print-env)"
cd ~/workspace/concourse-credhub/
source ./target-concourse-credhub.sh
```
### Concourse 내 CredHub 접근 확인
```
ubuntu@ip-0-0-0-0:~$ credhub api
Connecting to Credhub on BOSH Director....
Setting the target url: https://10.0.0.6:8844
Login Successful
Reading environment details from Credhub on BOSH Director....
Connecting to Concourse Credhub...
Setting the target url: https://bbl-env-xx-xxxxxx-concourse-lb-xxxxxx.elb.ap-northeast-1.amazonaws.com:8844
Login Successful
```
## PCF Pipeline 배포 준비
PCF Pipeline을 Concourse를 통해 배포하여 PCF 배포를 자동화 할 수 있으며
PCF 배포 자동화를 위한 파이프라인에 사용 되는 파라미터(params.yml)을 설정해줘야 합니다.
* PCF Pipeline 경로
```
cd ~/workspace/pcf-pipelines/aws/install-pcf/
```

### Concourse 내 CredHub을 통한 파라미터 값 추가
Concourse에 파이프라인 배포시 괄호로 감싼 입력 값은 CredHub을 통해 로드 합니다.
CredHub에 값 추가시 이름을 형태로 추가해줘야 합니다.
CredHub - /concourse/<팀 이름>/<파이프라인 이름>/<값 이름>
본 실습에서는 아래와 같은 값을 지정하여 진행합니다.
* 팀명: main
* 파이프라인명: install-pcf
```
파라미터의 타입이 값인 경우
credhub set -t value -n /concourse/main/install-pcf/aws_access_key_id -v <AWS Access Key ID>

params.yml에는 아래와 같이 명시하여 로드합니다.
((aws_access_key_id))

파라미터의 타입이 유저인 경우
credhub set -t user -n /concourse/main/install-pcf/pcf-admin -z <사용자명> -w <password>

params.yml에는 아래와 같이 명시하여 로드합니다.
((pcf-admin.username))
((pcf-admin.password))

파라미터의 타입이 SSH 인증서인 경우
credhub generate -t ssh -n /concourse/main/install-pcf/pcf_git_private_key

params.yml에는 아래와 같이 명시하여 로드합니다.
((pcf_git_private_key.private_key))

```

### PCF Pipeline 내 params.yml 수정
params.yml 내용 중 CHANGEME에 해당하는 내용은 수정이 필요합니다.

#### NAT 인스턴스 AMI ID 지정
* AWS NAT 인스턴스 AMI ID를 지정해줍니다.
  * 도쿄 리전(ap-northeast-1) = ami-03cf3903
  * 서울 리전(ap-northeast-2) = ami-8e0fa6e0
```
# AMI to use for nat box
# Update with the correct NAT AMI for your region. List of AMIs can be found
# here: https://github.com/pivotal-cf/terraforming-aws/blob/master/modules/infra/variables.tf
# If the above has moved, please go to https://github.com/pivotal-cf/terraforming-aws and search
# for "nat_ami_map"
amis_nat: <NAT 인스턴스 AMI ID>
```
#### PCF 배포에 사용할 IAM 계정 Credential 지정
* 이전 단계에서 생성했던 bbl 계정의 IAM Access Key Id 및 Secret Access Key를 지정해줍니다.
```
# This key must be a key with admin access
aws_access_key_id: <PCF 배포에 사용할 IAM 계정 Access Key ID>
aws_secret_access_key: <PCF 배포에 사용할 IAM 계정 Secret Access Key>
```
#### PAS가 배포될 가용 영역 지정
* PAS가 배포될 가용 영역을 지정해 줍니다.
```
aws_az1: <AWS 가용 영역 입력> 예) ap-northeast-1a
aws_az2: <AWS 가용 영역 입력> 예) ap-northeast-1c
aws_az3: <AWS 가용 영역 입력> 예) ap-northeast-1d
```
#### AWS Cert Manager를 통해 발급 받은 PAS가 사용할 인증서 ARN 지정
```
# ARN of the wildcard certificate to use; upload this in [AWS](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_server-certs.html#upload-server-certificate). This must be done prior to running the pipeline.
aws_cern_arn: AWS Cert Manager를 통해 발급 받은 인증서의 ARN
```
#### PAS 서비스 VM 생성시 사용할 키페어명 및 AWS 리전 지정
```
# Upload PEM to AWS as the below key pair name
aws_key_name: AWS에 등록된 키페어명
aws_region: <AWS 리전> 예) ap-northeast-1
```
#### PAS 서비스들의 계정 지정
PAS 서비스들에서 사용할 계정들을 지정 합니다.
```
# ERT Database Credentials - Require
PAS에서 사용할 각 서비스 컴포넌트들의 계정
db_accountdb_password: CHANGEME
db_accountdb_username: CHANGEME
db_app_usage_service_password: CHANGEME
db_app_usage_service_username: CHANGEME
db_autoscale_password: CHANGEME
db_autoscale_username: CHANGEME
db_ccdb_password: CHANGEME
db_ccdb_username: CHANGEME
db_diego_password: CHANGEME
db_diego_username: CHANGEME
db_locket_password: CHANGEME
db_locket_username: CHANGEME
db_networkpolicyserverdb_password: CHANGEME
db_networkpolicyserverdb_username: CHANGEME
db_nfsvolumedb_password: CHANGEME
db_nfsvolumedb_username: CHANGEME
db_notifications_password: CHANGEME
db_notifications_username: CHANGEME
db_routing_password: CHANGEME
db_routing_username: CHANGEME
db_silk_password: CHANGEME
db_silk_username: CHANGEME
db_uaa_password: CHANGEME
db_uaa_username: CHANGEME
```
#### PAS DB의 마스터 계정 지정
```
# RDS Master Credentials - Required
db_master_password: <DB 계정명>
db_master_username: <DB 비밀번호>
```

#### PAS에서 사용할 도메인명 지정
```
# Domain Names for ERT
pcf_ert_domain: <PCF 도메인명> # This is the domain you will access ERT with, for example: pcf.example.com.
system_domain: system.<PCF 도메인명> # e.g. system.pcf.example.com
apps_domain: apps.<PCF 도메인명>   # e.g. apps.pcf.example.com
```
#### PAS 버전 지정
```
# PCF Elastic Runtime minor version to track
ert_major_minor_version: ^2\.3\.[0-9]+$ # ERT minor version to track (e.g ^2\.1\.[0-9]+$ will track 2.0.x versions)
```

#### Git Private Key 및 haproxy 비활성화 지정
* Git Private Key
  * SSH를 통한 Git 접속이 필요하므로 CredHub에서 SSH 인증서를 생성한다.
  * 소유한 GitHub 계정 접속 > 우측 상단 메뉴 클릭 > Settings 클릭 > SSH and GPG Keys 클릭 > New SSH Key 클릭 > CredHub을 통해 생성한 SSH 인증서의 Public Key를 등록해준다.

* haproxy
  * 본 구성에서는 haproxy를 구성하지 않으므로 ha_proxy_foward_tls를 disable로 지정하고 haproxy_backend_ca를 비워둔다.
```
# Optional - if your git repo requires an SSH key.
git_private_key: <Git Private Key>

haproxy_forward_tls: disable # If enabled HAProxy will forward all requests to the router over TLS (enable|disable)
haproxy_backend_ca:         # Required if haproxy_forward_tls is enabled - HAProxy will use the CA provided to verify the certificates provided by the router.
```

#### MySQL 모니터링 이메일 입력
* 필수로 입력해줘야 하는 항목이며 적당한 이메일 주소를 지정한다.
```
# Email address for sending mysql monitor notifications
mysql_monitor_recipient_email: user@example.com
```





