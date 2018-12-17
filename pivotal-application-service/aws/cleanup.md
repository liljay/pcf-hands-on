# PAS 핸즈온 환경 제거
Control Plane 및 Pivotal Application Service가 정상 배포된 경우 아래의 단계에 따라 환경을 제거 할 수 있습니다.
* Concourse Web에 로그인 하여 install-pcf 파이프라인 클릭 > wipe-env 클릭 > 우측 상단의 (+) 버튼 클릭
* PAS 환경 제거 후 Control Plane Bosh를 통해 concourse 배포 삭제
  * bosh delete-deployment -d concourse
* Bosh Bootloader를 통해 생성된 Control Plane Bosh, jumpbox, Concourse LB 제거
  * bbl down
