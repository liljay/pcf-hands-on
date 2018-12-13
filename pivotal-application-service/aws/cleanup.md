# PAS 핸즈온 환경 제거
Control Plane 및 Pivotal Application Service가 정상 배포된 경우 아래의 단계에 따라 환경을 제거 할 수 있습니다.
* Concourse Web에 로그인 하여 install-pcf 파이프라인 클릭 > wipe-env 클릭 > 우측 상단의 (+) 버튼 클릭
* PAS 환경 제거 후 Control Plane Bosh를 통해 concourse 배포 삭제
  * bosh delete-deployment -d concourse
* Bosh Bootloader를 통해 생성된 Control Plane Bosh, jumpbox, Concourse LB 제거
  * bbl down

## PCF Pipeline 진행 중 create-infrastructure에 Terraform 관련 에러 발생시
install-pcf 파이프라인에서 create-infrastructure 단계에서 실패하는 경우 아래와 같이 진행해야 합니다.
### create-infrastructure 단계에서 Terraform이 생성한 리소스 삭제
\<terraform-prefix\>는 PCF Pipeline 배포시 params.yml의 terraform_prefix 변수에 지정된 값 입니다.

* Route 53
  * \*.apps.\<domain\> - CNAME 레코드
  * \*.system.\<domain\> - CNAME 레코드
  * opsman.\<domain\> - A 레코드
  * ssh.system.\<domain\> - CNAME 레코드
* EC2
  * \<terraform prefix\>-OpsMan az1 
  * \<terraform prefix\>-NAT Instance az1
  * \<terraform prefix\>-NAT Instance az2
  * \<terraform prefix\>-NAT Instance az3
* ELB
  * \<terraform prefix\>-Pcf-Http-Elb
  * \<terraform prefix\>-Pcf-Ssh-Elb
  * \<terraform prefix\>-Pcf-Tcp-Elb
* S3
  * \<terraform prefix\>-bosh
  * \<terraform prefix\>-droplets
  * \<terraform prefix\>-packages
  * \<terraform prefix\>-resources
  * \<terraform prefix\>-buildpacks
  * PCF pipeline에 포함된 params.yml 내용 중 S3_OUTPUT_BUCKET에 지정한 버킷 내 terraform.tfstate 파일 삭제
* VPC
  * \<terraform prefix\>-terraform-pcf-vpc

* RDS
  * 인스턴스 - \<terraform prefix\>-pcf
  * 서브넷 그룹 - \<terraform prefix\>-rds_subnet_group

* Security Groups
  * \<terraform prefix\>-RDS Security Group
  * \<terraform prefix\>-PcfHttpElb Security Group
  * \<terraform prefix\>-Ops Manager Director Security Group
  * \<terraform prefix\>-PcfSshElb Security Group
  * \<terraform prefix\>-NAT instance security group
  * \<terraform prefix\>-PCF VMs Security Group 

* IAM
  * User: \<terraform prefix\>_pcf_iam_user
  * Role: \<terraform prefix\>_pcf_admin_role
  * Policy: \<terraform prefix\>_PcfAdminPolicy
  * Instance Profile: \<terraform prefix\>_pcf_admin_role_instance_profile

* 인스턴스 프로파일 삭제 방법
인스턴스 프로파일은 AWS 콘솔에서 삭제 할 수 없으며 AWS CLI를 통해 조회 및 삭제해야 한다.
  * \<terraform prefix\>_pcf_admin_role_instance_profile 이름을 가진 인스턴스 프로파일 있는지 확인
    * aws iam list-instance-profiles
  * 인스턴스 프로파일 존재 여부 확인 후 인스턴스 프로파일 삭제 
    * aws iam delete-instance-profile --instance-profile-name \<terraform prefix\>_pcf_admin_role_instance_profile
    
### Concourse에 배포했던 PCF Pipeline 삭제 및 재배포
* fly -t <target> dp -p install-pcf
* fly -t <target> sp -p install-pcf -c pipeline.yml -l params.yml
