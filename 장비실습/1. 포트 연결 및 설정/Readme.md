# 선행 설정
- hostname: NonameL2
- no ip domain-lookup (실수로 인한 DNS 쿼리 방지)
- console 0
  - exec-time out 0 0 (자동 로그아웃 방지)
  - logging synchronous (명령어 입력 혼잡 방지)

# 포트 연결 및 설정

스위치 포트에 대한 기본 설정

## 상황
- Laptop 1대를 스위치 1번 포트에 연결함

## 수행 과정

- vlan 명령어로 vlan 생성
 vlan 10, 		vlan 10 생성
 name USER, 	vlan 10의 이름을 USER라고 명명

- interface 설정
 switchport mode access,	포트 (g0/1)을 액세스 포트로 설정
 switchport access vlan 10,	g0/1 포트를 vlan 10에 포함
 description USER-PC1,	주석 포함

- show vlan brief, show interface status, show interface g0/1 switchport 명령어로 검증
