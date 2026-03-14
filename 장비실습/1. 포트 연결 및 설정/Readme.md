# 선행 설정
- hostname: NonameL2
- no ip domain-lookup (실수로 인한 DNS 쿼리 방지)
- console 0
  - exec-time out 0 0 (자동 로그아웃 방지)
  - logging synchronous (명령어 입력 혼잡 방지)

# 포트 연결 및 설정

스위치 포트에 대한 기본 설정 및 VLAN 할당

## 상황
- Laptop 1대를 스위치 1번 포트에 연결함 (후에 2번 포트로 옮겨 연결함)

## 수행 과정

- vlan 명령어로 vlan 생성
  - vlan 10,20,99 		          vlan 들 생성
  - name USER,ADMIN,Management 	vlan 들 이름 명명

- interface 설정
   - switchport mode access,	        포트를 액세스 포트로 설정
   - switchport access vlan x,	      포트를 vlan x에 포함
   - description USER-PC1,ADMIN-PC1	   인터페이스에 주석 포함

- SVI 설정
  - interface vlan 99,                      vlan 99의 가상 인터페이스 설정
  - ip address 192.168.99.2,               스위치 관리용 주소 할당
  ## + ip default-gateway 192.168.99.1,    업링크 통신용 주소


- show vlan brief, show interface status, show interface g0/x switchport, show ip interface brief등 명령어로 검증
