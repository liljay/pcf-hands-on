# 사전 준비 사항
* AWS 계정 생성 - https://aws.amazon.com/ko/premiumsupport/knowledge-center/create-and-activate-aws-account
* 공개 도메인 (소유하고 있는 공개 도메인이 없는 경우 구매 필요)
* AWS Limit Increase 요청 - https://docs.pivotal.io/pivotalcf/2-3/customizing/aws.html
* Route 53에 도메인 명으로 Hosted Zone 생성
* Certificate Manager를 통한 인증서 발급 (DNS 인증)
  * \<domain\> 예) example.com
  * \*.\<domain\> 예) \*.example.com
  * \*.apps.\<domain\> 예) \*.apps.example.com
  * \*.system.\<domain\> 예) \*.system.example.com
* Pivotal 계정 생성
  * https://network.pivotal.io/ > Register
* Pivotal 계정 생성 후 LEGACY API TOKEN 확인
  * https://network.pivotal.io/ > Sign In > 화면 우측 상단 이름 클릭 > Edit Profile 클릭 > LEGACY API TOKEN [DEPRECATED] 값 확인
* Control Plane 구성을 위한 IAM 계정 및 권한 생성 (Access Key ID 및 Secret Access Key가 필요하므로 Programmatic Access 체크 후 IAM 계정 생성)
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
* PCF 구성을 위한 IAM 계정 및 권한 생성 (Access Key ID 및 Secret Access Key가 필요하므로 Programmatic Access 체크 후 IAM 계정 생성)
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
                "s3:*",
                "rds:*"
            ],
            "Resource": "*"
        }
    ]
}
```
