# Concourse
## Concourse URL 설정
```
export CONCOURSE_URL="https://${EXTERNAL_HOST}"
```

## Concourse Web(ATC)에 있는 CredHub 연결
```
source ~/workspace/concourse-credhub/target-concourse-credhub.sh
```

##
```
ubuntu@ip-10-0-1-250:~/workspace/concourse-credhub$ source ./target-concourse-credhub.sh
Connecting to Credhub on BOSH Director....
Setting the target url: https://10.0.0.6:8844
Login Successful
Reading environment details from Credhub on BOSH Director....
Connecting to Concourse Credhub...
Setting the target url: https://bbl-env-sa-df0bee6-concourse-lb-e02a85cf5ceacc39.elb.ap-northeast-1.amazonaws.com:8844
Login Successful
```

