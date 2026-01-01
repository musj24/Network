
2. 설계 및 구성 시 고려 사항

LAB2의 토폴로지를 기반으로 구성하되, LAB1에서 사용한 기능을 단계적으로 추가하는 방식으로 설계하였다.

2.1 LAB 통합 전략

LAB2를 기본 골격으로 사용

VLAN 구조 및 ROAS 구성 유지

LAB1에서 사용했던 기술을 LAB2 구조에 추가

OSPF 기반 라우팅

별도의 DHCP 서버 사용

2.2 DHCP 구성 변경

LAB1, 별도의 DHCP 서버 사용

LAB2, 라우터를 DHCP 서버로 사용

LAB3, 라우터 DHCP 제거

LAB1과 동일하게 중앙 DHCP 서버 방식으로 통일

각 VLAN 게이트웨이 인터페이스에서 ip helper-address를 사용해 DHCP Relay 구성

2.3 라우팅 방식 변경

LAB2, Static route / Default route 중심

LAB3, LAB1과 동일하게 OSPF 사용

HQ–Branch 간 네트워크 확장성을 고려한 동적 라우팅 적용

2.4 NAT 설계

내부 네트워크에서 외부(인터넷 역할 네트워크)로 나가는 트래픽에 대해 NAT 적용

내부 사설 IP 대역 → 외부 인터페이스 IP로 변환

내부 PC에서 외부 서버(8.8.8.8 및 외부 Web Server) 접근 가능 여부 확인
