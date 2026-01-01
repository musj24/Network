LAB3-LAB1+LAB2+DNS

1. 목표
본 LAB은 기존 LAB1과 LAB2를 하나의 통합 토폴로지로 구성하고,
중앙 DHCP/DNS 서버를 기반으로 내부 및 외부 자원에 대한 도메인 기반 접근을 구현하는 것을 목표로 한다.

구체적인 목표는 다음과 같다.

LAB1과 LAB2에서 각각 사용한 네트워크 기술을 단일 토폴로지로 통합

중앙 DHCP/DNS 서버를 통해 내부 PC에 IP 및 DNS 정보 제공

내부 PC에서, 내부 서버 및 외부 서버(인터넷 역할: 8.8.8.8, 외부 Web Server) 에 대해 IP 주소가 아닌 Domain name 기반으로 접근 가능하도록 구성

내부 → 외부 트래픽에 대해 NAT 동작 확인


사용할 기술 OSPF, NAT, DHCP, DNS, ROAS, DAI
-------

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


3. 트러블슈팅

– DNS
- 도메인 접속 실패 시, DHCP를 통해 DNS 서버 주소가 전달되었는지 확인
- DNS 서버에 A 레코드가 등록되어 있는지 확인
- 라우터에서 DNS 질의가 필요한 경우 ip name-server 설정 여부 점검



– Firewall
- ICMP와 TCP 동작을 구분하여 점검
- NAT 변환 테이블 및 세션 생성 여부 확인
- 내부/외부 경계에서 트래픽이 차단되는지 확인



– DAI
- DHCP Snooping 바인딩 테이블 생성 여부 확인
- Trust / Untrust 포트 설정 점검
- VLAN 단위 DAI 적용 여부 확인




4. 설계 변경 사항

초기 설계에서는 내부 DHCP/DNS 서버와 Web 서버를 HQ 라우터에 직접 연결하려 했으나,

Packet Tracer 환경에서 라우터 인터페이스 수 제한이 있고, HQ 라우터의 g0/2 인터페이스가 ROAS 구성으로 사용 중이라는 제약으로 인해 토폴로지 변경에 제한이 생겼다.

이에 따라,

서버들은 Branch 라우터 측에 연결, HQ–Branch 간 OSPF를 통해 서버 접근성 확보라는 방식으로 설계를 수정하였다.



