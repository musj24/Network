
# 선행 설정
- hostname: NonameL2
- no ip domain-lookup (실수로 인한 DNS 쿼리 방지)
- console 0
  - exec-time out 0 0 (자동 로그아웃 방지)
  - logging synchronous (명령어 입력 혼잡 방지)
- spanning-tree portfast edge default (portfast 전역설정)
- spanning-tree portfast edge bpduguard default (bpduGuard 전역설정)
- spanning-tree mode rapid-pvst (RPVST 설정)

# Trunk

G0/16 포트를 Trunk 포트로 구성

## 상황
- G0/16 포트가 다른 스위치와 연결됐다고 가정

## 수행 과정

- vlan 999
  - name native                               native vlan 생성

- interface g0/16
  - switch mode trunk			                    포트를 트렁크모드로 설정
  - switch trunk allowed vlan 10,20,99,999		포트 통신에 허용될 VLAN 목록들 설정
  - switch trunk native vlan 999		        	native vlan 명시


- show interface g0/16 switchport 명령어로 검증

