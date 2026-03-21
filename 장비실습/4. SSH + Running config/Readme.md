# 선행 설정
- hostname: NonameL2
- no ip domain-lookup (실수로 인한 DNS 쿼리 방지)
- console 0
  - exec-time out 0 0 (자동 로그아웃 방지)
  - logging synchronous (명령어 입력 혼잡 방지)
- spanning-tree portfast edge default (portfast 전역설정)
- spanning-tree portfast edge bpduguard default (bpduGuard 전역설정)
- spanning-tree mode rapid-pvst (RPVST 설정)
- logging on (logging 기능 설정)
- service timestamp log datetime msec (로그에 실제시간 밀리초 단위로 기록 남김)
- logging trap notification (severity level 5단계까지의 알림을 외부 서버로 전송)
- logging console notification (severity level 5단계까지의 알림을 콘솔 화면에 표시)


# SSH

원격 접속 사용 설정

## 상황
- 스위치에 원격 접속 구성을 설정한다

## 수행 과정

- ip domain-name local.intra,			RSA 키 생성을 위한 절차 (장비 식별)
- ip ssh version 2,				SSH 버전2 사용
- username Noname secret 1q2w3e4r,		장비에 접속 허가할 유저와 비밀번호 DB 생성
- crypto key generate rsa 1024,		RSA 키 길이를 1024bit으로 생성

- line vty 0 4,				가상 라인 0 - 4 번 구성
  - transport input ssh,			외부 접속 방법을 SSH만 허용
  - login local,				0 - 4 번까지의 가상 라인에 접속시 local 아이디를 이용하여 로그인 함


- show run 으로 확인
