# Concourse 구성
Concourse는 Control Plane의 CI/CD 도구이며 PCF 파이프라인을 통해 PCF를 배포 합니다.
## Concourse 배포시 UAA, CredHub, backup restore sdk을 포함하여 배포하도록 설정
* add-credhub-uaa-to-web.yml: Concourse Web(ATC)에 CredHub을 배포하기 위한 yaml
* versions.yml: 배포할 릴리즈들의 버전을 명시하는 yaml
```
cd ~/workspace/concourse-bosh-deployment/cluster/operations/
wget https://raw.githubusercontent.com/pivotalservices/concourse-credhub/master/operations/add-credhub-uaa-to-web.yml

cat << EOF >> ~/workspace/concourse-bosh-deployment/versions.yml
uaa_release_version: '60'
uaa_sha: 'a7c14357ae484e89e547f4f207fb8de36d2b0966'
credhub_release_version: '1.9.3'
credhub_sha: '648658efdef2ff18a69914d958bcd7ebfa88027a'
backup_restore_sdk_release_version: '1.9.0'
backup_restore_sdk_sha: '2f8f805d5e58f72028394af8e750b2a51a432466'
EOF
```
## vars.yml 설정
vars.yml를 통해 Concourse 배포 설정 값을 지정해줍니다.
```
cd ~/workspace/concourse-bosh-deployment/cluster
cat << EOF > ~/workspace/concourse-bosh-deployment/cluster/vars.yml
external_host: "${EXTERNAL_HOST}"
external_url: "https://${EXTERNAL_HOST}"
local_user:
    username: "${CC_ADMIN_USERNAME}"
    password: "${CC_ADMIN_PW}"
network_name: 'private'
web_network_name: 'private'
web_vm_type: 't2.micro'
web_network_vm_extension: 'lb'
web_instances: 1
db_vm_type: 't2.micro'
db_persistent_disk_type: '1GB'
worker_vm_type: 't2.micro'
deployment_name: 'concourse'
worker_instances: 3
worker_ephemeral_disk: '100GB_ephemeral_disk'
EOF
```
## aws-tls-vars.yml 설정
aws-tls-vars.yml을 통해 Concourse 배포시 EXTERNAL_HOST 환경 변수로 지정한 도메인에 대해 Concourse Web(ATC) 인증서를 생성 합니다.
```
cat << EOF >  ~/workspace/concourse-bosh-deployment/cluster/aws-tls-vars.yml
- type: replace
  path: /variables/-
  value:
    name: atc_ca
    type: certificate
    options:
      is_ca: true
      common_name: atcCA

- type: replace
  path: /variables/-
  value:
    name: atc_tls
    type: certificate
    options:
      ca: atc_ca
      alternative_names: [((external_host))]
      organization: atcOrg
EOF
```

## Concourse 배포를 위해 필요한 Stemcell 업로드 스크립트 생성
```
cd ~/workspace/bbl
cat << 'EOF' > ~/workspace/bbl/upload-stemcell-for-concourse.sh
export IAAS="$(cat bbl-state.json | jq -r .iaas)"
if [ "${IAAS}" = "aws" ]; then
  export EXTERNAL_HOST="$(bbl outputs | grep concourse_lb_url | cut -d ' ' -f2)"
  export STEMCELL_URL="https://bosh.io/d/stemcells/bosh-aws-xen-hvm-ubuntu-xenial-go_agent"
elif [ "${IAAS}" = "gcp" ]; then
  export EXTERNAL_HOST="$(bbl outputs | grep concourse_lb_ip | cut -d ' ' -f2)"
  export STEMCELL_URL="https://bosh.io/d/stemcells/bosh-google-kvm-ubuntu-xenial-go_agent"
else # Azure
  export EXTERNAL_HOST="$(bbl outputs | grep concourse_lb_ip | cut -d ' ' -f2)"
  export STEMCELL_URL="https://bosh.io/d/stemcells/bosh-azure-hyperv-ubuntu-xenial-go_agent"
fi

bosh upload-stemcell "${STEMCELL_URL}"
EOF

chmod +x ~/workspace/bbl/upload-stemcell-for-concourse.sh
```
## Concourse 배포를 위해 필요한 Stemcell 업로드
```
cd ~/workspace/bbl
./upload-stemcell-for-concourse.sh
```
## Concourse 배포 스크립트 생성
```
cd ~/workspace/concourse-bosh-deployment/cluster

cat << EOF > deploy-concourse.sh
 bosh deploy -d concourse concourse.yml \
      -l ../versions.yml \
      -l vars.yml \
      -o operations/basic-auth.yml \
      -o operations/privileged-http.yml \
      -o operations/privileged-https.yml \
      -o operations/tls.yml \
      -o aws-tls-vars.yml \
      -o operations/web-network-extension.yml \
      -o operations/add-credhub-uaa-to-web.yml \
      -o operations/worker-ephemeral-disk.yml \
      -o operations/scale.yml
EOF

chmod +x deploy-concourse.sh
```
## Concourse 배포
```
cd ~/workspace/concourse-bosh-deployment/cluster
./deploy-concourse.sh
```

## Concourse 배포 확인
Concourse 배포 후 총 5개의 VM들이 생성 됩니다.
* bosh deployment -d concourse 명령어를 통해 Name이 concourse인 배포가 있는지 확인
* bosh vms -d concourse 명령어를 통해 배포된 concourse의 VM 상태들이 running인지 확인
```
ubuntu@ip-0-0-0-0:~$ bosh deployment -d concourse
Using environment 'https://10.0.0.6:25555' as client 'admin'

Name       Release(s)          Stemcell(s)                                     Config(s)        Team(s)
concourse  bosh-dns/1.10.0     bosh-aws-xen-hvm-ubuntu-xenial-go_agent/170.13  1 cloud/default  -
           concourse/4.2.1                                                     2 runtime/dns
           credhub/1.9.3
           garden-runc/1.16.3
           postgres/30
           uaa/60

1 deployments

Succeeded

ubuntu@ip-0-0-0-0:~$ bosh vms -d concourse
Using environment 'https://10.0.0.6:25555' as client 'admin'

Task 21. Done

Deployment 'concourse'

Instance                                     Process State  AZ  IPs        VM CID               VM Type   Active
db/d8972961-9f4e-45a6-8872-ed0f15e2d3a7      running        z1  10.0.16.5  i-08f2cba620a71d59c  t2.micro  true
web/9792de26-2b7f-400a-baf6-b4ef478b9fec     running        z1  10.0.16.4  i-066931fd2ca7bbdba  t2.micro  true
worker/310b25ec-1335-4d08-93b4-81745b74f031  running        z1  10.0.16.6  i-0093c80c9a544bc81  t2.micro  true
worker/7231ceac-3722-4954-a6b1-4657cacf0d57  running        z1  10.0.16.7  i-0b46ffbdf6fdb5578  t2.micro  true
worker/860352bb-8020-414c-ae06-4651ff57ff0e  running        z1  10.0.16.8  i-02852570172a48946  t2.micro  true

5 vms

Succeeded

```
