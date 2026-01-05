
# LAB4 – Distribution Layer 이중화 및 캠퍼스 네트워크 설계

## 목표

본 LAB은 기존 LAB3의 단일 경로 기반 구조를 확장하여,
**Distribution Layer(DSW) 이중화와 상단(HQ) 이중 경로를 고려한 캠퍼스 네트워크 구조를 설계·구현**하는 것을 목표로 한다.

구체적인 목표는 다음과 같다.

* Access–Distribution–HQ 계층 구조를 명확히 분리한 캠퍼스 토폴로지 설계
* Distribution Switch에서 Inter-VLAN Routing 수행
* 사용자 VLAN에 대해 HSRP 기반 Default Gateway 이중화 구현
* DSW–HQ 및 DSW–DSW 간 OSPF 기반 라우팅 구성
* DSW 간 L3 백업 링크를 통한 업링크 장애 시 자동 우회 경로 확보
* STP Root 및 EtherChannel을 활용한 L2 안정성 확보
* 서버 VLAN과 사용자 VLAN의 역할 분리 및 설계 의도 명확화

---

## 사용 기술

* Inter-VLAN Routing (SVI 기반)
* HSRP
* OSPF
* STP (Root Bridge 설계)
* EtherChannel (LACP)
* DHCP Relay (ip helper-address)
* DAI / DHCP Snooping (Access Layer)
* DNS

---

## 설계 및 구성 시 고려 사항

### 2.1 전체 설계 방향

LAB3에서는 단일 라우터 중심 구조로 기능 통합을 목표로 했다면,
LAB4에서는 **캠퍼스 네트워크에서 일반적으로 사용되는 계층형 구조**를 반영하여 다음과 같이 설계하였다.

* Access Layer

  * 단말 수용
  * VLAN 분리
  * L2 보안(DHCP Snooping, DAI) 적용

* Distribution Layer

  * Inter-VLAN Routing 수행
  * 사용자 VLAN에 대해 HSRP 적용
  * STP Root 역할 수행
  * 상단 및 백업 경로 제어

* HQ Layer

  * 외부(Internet / Firewall) 방향 Edge 역할
  * Default Route 보유
  * OSPF를 통해 Distribution Layer에 default route 전파

---

### 2.2 Inter-VLAN Routing 위치 변경

* LAB3

  * ROAS 기반, 라우터에서 Inter-VLAN Routing 수행

* LAB4

  * Distribution Switch에서 SVI 기반 Inter-VLAN Routing 수행

이를 통해,

* 내부 트래픽이 불필요하게 상단 라우터를 경유하지 않도록 하고
* 캠퍼스 설계 관점에서 Distribution Layer의 역할을 명확히 하였다.

---

### 2.3 HSRP 설계

* 사용자 VLAN(VLAN 10/20/30)에 대해 HSRP 적용
* Default Gateway는 HSRP Virtual IP 사용
* DSW1/DSW2 간 Priority 차이를 두어 Active/Standby 역할 구분

서버 VLAN(VLAN 50)은:
본 LAB에서는 서버의 물리적 이중화까지는 고려하지 않았으나,
게이트웨이 장애로 인한 추가적인 단절을 방지하기 위해 VLAN 50 SVI에도 HSRP를 적용하였다.
---

### 2.4 DSW–HQ 라우팅 설계

* DSW–HQ 간 링크는 L3 포인트투포인트 링크로 구성
* OSPF Area 0 사용
* HQ는 외부로 나가는 default route를 보유
* `default-information originate`를 통해 DSW에 default route 전파

이를 통해:

* DSW는 내부 VLAN 경로만 직접 보유
* 외부 트래픽은 모두 HQ 방향으로 단순하게 전달

---

### 2.5 DSW–DSW 백업 링크 설계

DSW 간 링크는 VLAN 확장이 아닌 **라우팅 우회 전용 링크**로 설계하였다.

* `no switchport` 기반 L3 링크
* /30 주소 사용
* OSPF Area 0 참여

이 구조를 통해:

* DSW 업링크(HQ) 장애 시
* DSW → DSW → 반대편 HQ 경로로 자동 우회 가능
* Static route나 Tracking 없이도 OSPF 수렴으로 장애 대응 가능

---

### 2.6 STP 및 EtherChannel 설계

* Access–Distribution 구간에 EtherChannel(LACP) 적용
* STP Root Bridge를 Distribution Switch로 명확히 지정
* HSRP Active 장비와 STP Root를 일치시켜 트래픽 비효율 최소화

이를 통해:

* L2 루프 방지
* 링크 장애 시 빠른 수렴
* 불필요한 STP 차단 경로 최소화

---

## 트러블슈팅 포인트

### – OSPF

* Area 불일치 여부
* 인터페이스에 OSPF 활성화 여부
* L2 상태(switchport) 잔존 여부

### – HSRP

* SVI 상태(up/up) 확인
* Priority 및 preempt 설정 점검

### – EtherChannel

* VLAN mask 불일치
* speed/duplex 불일치
* LACP mode 설정 확인

### – DHCP / DAI

* ip helper-address 설정 여부
* DHCP Snooping 바인딩 테이블 생성 여부
* Trust / Untrust 포트 구분

---

## 설계 판단 및 한계

* Packet Tracer 환경에서는 Object Tracking(track) 기능이 지원되지 않아
  HSRP Priority 기반 이중화로 설계를 단순화하였다.
* 해당 제약 하에서도,
  상단 장애는 OSPF 수렴으로 처리하도록 역할을 분리하였다.
