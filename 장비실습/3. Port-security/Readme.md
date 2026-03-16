
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


# Port-security

기본적인 포트보안 설정

## 상황
- G0/1 , G0/2 에 기본적인 포트보안을 설정한다.
- G0/15 포트를 통해 DHCP를 수신받는다는 가정

## 수행 과정

- port-security,			포트 보안 기능 on
  - port-security maximum 1,		포트에서 허용하는 MAC 주소수 설정 (현재 1)
  - port-security violation shutdown,	포트 보안설정 위반시 수행할 보안활동 (현재 포트 다운)
  - port-security mac-address sticky,	포트에서 MAC 주소를 학습하는 방법 (호스트 연결시 동적으로 학습후 유지)

- ip dhcp snooping,				dhcp snooping 기능 on
  - ip dhcp snooping vlan 10,20,99,		영향 받을 vlan 범위 설정
  - interface g0/15
     - ip dhcp snooping trust,			g0/15 포트를 신뢰하는 포트로 설정
  - interface range g0/1 - 2
     - ip dhcp snooping limit rate 10,		초당 dhcp 요청 한도를 10으로 설정

- ip arp inspection vlan 10,20,99,		DAI 검사할 vlan 범위 설정
  - interface g0/15
     - ip arp inspection trust,			g0/15 포트를 신뢰하는 포트로 설정
  - interface range g0/1 - 2
     - ip arp inspection limit rate 10,		초당 arp 요청 한도를 10으로 설정


- show ip dhcp snooping, show ip arp inspection 등의 명령어로 확인
