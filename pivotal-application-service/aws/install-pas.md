# Deploying Pivotal Application Service
## 사전 요구 사항
* Ops Manager와 PCF 버전 호환성 확인
https://docs.pivotal.io/resources/product-compatibility-matrix.pdf


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
### Concourse 내 CredHub 접근 완료 여부
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


