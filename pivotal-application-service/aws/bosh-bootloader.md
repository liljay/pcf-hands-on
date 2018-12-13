# Bosh Bootloader 구성
Bosh Bootloader는 Control Plane의 Bosh를 쉽게 구성하기 위해 사용 합니다.
# Bosh Bootloader를 위한 스크립트 구성
## bblup.sh
Bosh Bootloader의 bbl up 명령어를 통해 Bosh를 구성하는 스크립트 입니다.
```
mkdir -p ~/workspace/bbl/terraform
cd ~/workspace/bbl/terraform
wget https://raw.githubusercontent.com/pivotalservices/concourse-credhub/master/bbl-terraform/aws/concourse-lb_override.tf
cd ~/workspace/bbl
cat << 'EOF' > bblup.sh 
bbl up --aws-access-key-id ${BBL_ACCESS_KEY_ID} \
         --aws-secret-access-key ${BBL_SECRET_ACCESS_KEY} \
         --aws-region ${REGION} \
         --iaas aws \
         --lb-type concourse
EOF
chmod +x bblup.sh
```
## bbldown.sh
Bosh Bootloader의 bbl down 명령어를 통해 구성했던 Bosh를 제거하는 스크립트 입니다.
```
cd ~/workspace/bbl
cat << 'EOF' > bbldown.sh 
bbl down --aws-access-key-id ${BBL_ACCESS_KEY_ID} \
         --aws-secret-access-key ${BBL_SECRET_ACCESS_KEY} \
         --aws-region ${REGION} \
         --iaas aws \
         --lb-type concourse
EOF
chmod +x bbldown.sh
```
# Bosh Bootloader를 통한 Bosh 구성 
bblup.sh 스크립트를 통해 Control Plane Bosh를 생성합니다.
```
cd ~/workspace/bbl
./bblup.sh
```
# Bosh Bootloader를 통해 생성한 Bosh 생성 확인
eval "$(bbl print-env)"를 통해 Control Plane의 Bosh에 접근 할 수 있으며
위의 명령어는 반드시 ~/workspace/bbl 디렉토리 경로에서 실행해줘야 합니다.
그 다음 bosh env 명령어를 통해 출력 결과를 확인합니다.
bosh 명령어는 어느 경로에서 실행해도 상관 없습니다.
```
ubuntu@ip-0-0-0-0:~/workspace/bbl$ eval "$(bbl print-env)"
ubuntu@ip-0-0-0-0:~/workspace/bbl$ bosh env
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
# AWS 콘솔에서 인스턴스 확인
Bosh Bootloader를 통해 Bosh를 구성하면 2개의 인스턴스 및 1개의 네트워크 로드 밸런서가 생성 됩니다.
* EC2 인스턴스
  * Bosh/0: Control Plane Bosh
  * Jumpbox/0: Control Plane Jumpbox
* ELB 
  * bbl-env-\<random-value\>-\<random-values\>-concourse-lb: Concourse Load Balancer
  * 리스너 및 타겟 그룹 확인
    * 80, 443: Concourse Web(ATC)에 접근 하기 위한 포트
    * 2222: Concourse의 Worker들을 Web(ATC)에 등록 및 Worker들이 접근 하기 위한 포트
    * 8443: Concourse의 UAA에 접근 하기 위한 포트
    * 8844: Concourse의 CredHub에 접근 하기 위한 포트
         
