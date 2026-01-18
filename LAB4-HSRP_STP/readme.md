

---

# LAB4 – Distribution Layer 이중화 및 캠퍼스 네트워크 설계 (HSRP / OSPF / STP / EtherChannel / ASA 연동)

## 1. LAB 개요 및 목표

본 LAB은 기존 LAB3의 단일 경로 기반 구조를 확장하여,
실제 캠퍼스 네트워크에서 일반적으로 사용되는 **계층형 구조(Access–Distribution–Edge/HQ)** 를 반영하고,
**Distribution Layer 이중화 및 상단(HQ) 이중 경로** 기반의 가용성 확보를 목표로 한다.

구체적인 목표는 다음과 같다.

* Access–Distribution–HQ 계층 구조를 명확히 분리한 캠퍼스 토폴로지 설계
* Distribution Switch에서 SVI 기반 Inter-VLAN Routing 수행
* 사용자 VLAN(10/20/30)에 대해 HSRP 기반 Default Gateway 이중화 구현
* DSW–HQ 및 DSW–DSW 간 OSPF 기반 라우팅 구성
* DSW 간 L3 백업 링크를 통한 업링크 장애 시 자동 우회 경로 확보(OSPF 수렴 기반)
* STP Root 및 EtherChannel(LACP)을 활용한 L2 안정성 확보
* Firewall(ASA) 연동 및 외부 트래픽 NAT 동작 검증(PT 제약 포함)

---

## 2. 전체 토폴로지 설명

* 네트워크는 **Internet / ISP(이중) / ASA / HQ(이중) / Distribution(이중) / Access(이중) / 내부 VLAN(10/20/30) / 서버 VLAN(50)** 영역으로 구성된다.
* 사용자 단말은 VLAN 10 / 20 / 30으로 분리되어 Access Layer에 수용된다.
* Distribution Layer(DSW1/DSW2)는 SVI 기반으로 Inter-VLAN Routing을 수행한다.
* 사용자 VLAN의 Default Gateway는 HSRP Virtual IP를 사용한다.
* 상단 HQ 라우터(HQ1/HQ2)는 ASA(inside)와 연결되어 외부망으로의 출구 역할을 수행한다.
* DSW–HQ 및 DSW–DSW 링크는 L3 링크로 구성되며 OSPF로 라우팅 정보를 교환한다.
* Internet은 실제 외부망 대신 **Loopback 8.8.8.8 및 External Web Server**로 단순화하여 구성한다.

---

## 3. 설계 및 구성 시 고려 사항

### 3.1 LAB 확장 전략

* LAB3에서는 ROAS 기반 단일 경로(상단 중심) 구조로 기능 통합을 수행하였다.
* LAB4에서는 캠퍼스 구조에 맞춰 **Inter-VLAN Routing을 Distribution으로 이동**하고,
  **Distribution/HQ/ISP 구간을 이중화**하여 장애 대응 흐름을 검증한다.
* 설계 확장 요소는 다음과 같다.

  * Distribution 이중화(HSRP)
  * 상단 이중 경로(DSW–HQ1, DSW–HQ2 분리)
  * DSW–DSW L3 백업 링크(OSPF 수렴 기반 우회)
  * L2 안정화(STP Root 설계 + EtherChannel)

---

### 3.2 Inter-VLAN Routing 설계 변경

* LAB3: HQ 라우터에서 ROAS 방식으로 Inter-VLAN Routing 수행
* LAB4: **Distribution Switch에서 SVI 기반 Inter-VLAN Routing 수행**

이를 통해,

* 내부 VLAN 간 트래픽이 상단 라우터를 불필요하게 경유하지 않도록 최적화하고,
* 캠퍼스 설계 관점에서 Distribution의 역할(집선/라우팅/이중화)을 명확히 한다.

---

### 3.3 게이트웨이 이중화 설계 (HSRP)

* 사용자 VLAN(VLAN 10/20/30)에 대해 HSRP를 적용한다.
* 단말의 Default Gateway는 HSRP Virtual IP를 사용한다.
* DSW1/DSW2 간 priority 차이를 두어 Active/Standby 역할을 구분한다.
* Packet Tracer 환경에서는 Object Tracking 지원이 제한적이므로,
  본 LAB에서는 Priority/Preempt 기반으로 이중화 동작을 단순화한다.

> 서버 VLAN(50)은 이번 LAB에서 “서버/관리 영역 분리” 목적의 VLAN로 구성하며,
> 게이트웨이 이중화(HSRP)까지는 확장 범위에서 제외한다

---

### 3.4 라우팅 설계 (OSPF)

* DSW–HQ 간 링크는 L3 포인트투포인트 링크로 구성하고 OSPF Area 0에 참여한다.
* HQ 라우터는 외부로 나가는 Default Route를 보유한다.
* 필요 시 HQ에서 `default-information originate`를 통해 DSW로 default route를 전파한다.
* 내부 라우팅 정보가 불필요하게 외부(ISP/Internet)로 확산되지 않도록,
  외부 방향 인터페이스에는 passive-interface 적용 등 라우팅 범위를 제어한다.

---

### 3.5 DSW–DSW 백업 링크 설계

* DSW 간 링크는 VLAN 확장 목적이 아니라 **라우팅 우회 전용 L3 링크**로 설계한다.

  * `no switchport` 기반 L3 링크
  * /30 주소 사용
  * OSPF Area 0 참여

이를 통해,

* 특정 DSW–HQ 업링크 장애 시
  **DSW → DSW → 반대편 HQ** 방향으로 OSPF 수렴을 통한 자동 우회가 가능하다.

---

### 3.6 STP 및 EtherChannel 설계

* Access–Distribution 구간에 EtherChannel(LACP)을 적용하여 링크 집성과 장애 내성을 확보한다.
* STP Root Bridge를 Distribution Switch로 명확히 지정하여,
  Access Layer에서 Root가 형성되지 않도록 한다.
* 가능하면 HSRP Active 장비와 STP Root를 동일 VLAN 기준으로 일치시켜,
  불필요한 L2 우회/차단을 최소화한다.

---

### 3.7 Firewall 및 NAT 설계 (Packet Tracer 환경 고려)

* ASA는 내부(inside)–외부(outside) 경계 장비로 배치하여 외부 트래픽에 대한 NAT를 수행한다.
* 다만 Packet Tracer ASA는 내부 라우팅된 서브넷(DSW SVI 뒤 VLAN) 트래픽에 대해
  **NAT 매칭/표시/검증이 불안정한 케이스가 존재**한다.

이에 따라 본 LAB에서는,

* ASA 단일 NAT 설계를 의도한 것이 아니라,
* **PT 제약을 회피하기 위한 전처리 NAT(PAT)을 HQ 라우터에서 수행**하여
  내부 트래픽의 소스 주소를 ASA inside 대역으로 변환한 뒤,
  ASA NAT 동작을 검증할 수 있도록 구성하였다.

또한 HQ1/HQ2 모두 동일한 PAT 구성을 적용하여,
상단 라우터 장애 시에도 외부 트래픽 경로가 유지되도록 한다.

---

## 4. 사용 기술

* Inter-VLAN Routing (SVI)
* HSRP
* OSPF
* STP (Rapid-PVST)
* EtherChannel (LACP)
* DHCP Relay (ip helper-address)
* DAI / DHCP Snooping (Access Layer)
* DNS
* NAT (HQ Router PAT + ASA NAT, PT 제약 회피 목적)

---

## 5. 검증 항목

* 내부 PC가 DHCP를 통해 IP 및 DNS 서버 정보를 정상적으로 할당받는지 확인
* VLAN/Trunk 확장 및 EtherChannel 번들링 상태 정상 여부 확인
* STP Root가 Distribution으로 고정되어 L2 경로가 설계 의도대로 동작하는지 확인
* 사용자 VLAN의 Default Gateway가 HSRP Virtual IP로 제공되고 Active/Standby가 정상인지 확인
* DSW–HQ 및 DSW–DSW 구간 OSPF 이웃 관계 형성 및 라우팅 정보 교환 여부 확인
* 업링크 장애 시 OSPF 수렴 및 HSRP 전환을 통한 자동 우회 경로 유지 여부 확인
* 내부 → 외부 트래픽에 대한 NAT 변환 동작 확인(PT 제약 반영: HQ PAT + ASA NAT 증빙)

---

## 6. 트러블슈팅 포인트

### – OSPF

* Area 불일치 여부
* 인터페이스 OSPF 활성화 여부
* L3 링크 구간에서 `no switchport` 미적용 여부
* passive-interface 처리 누락 여부(불필요한 인접 형성 방지)

### – HSRP

* SVI 상태(up/up) 확인
* 그룹/VIP 불일치 여부
* Priority 및 preempt 설정 점검

### – STP / EtherChannel

* trunk allowed/native VLAN 불일치
* EtherChannel 멤버 포트 설정 불일치(speed/duplex/trunk mode)
* STP Root 설정(Primary/Secondary) 누락 또는 VLAN별 우선순위 오설정

### – DHCP / DAI

* ip helper-address 설정 여부/오타
* DHCP Snooping 바인딩 테이블 생성 여부
* Trust / Untrust 포트 설정 점검(업링크/서버 포트 구분)

### – Firewall / NAT

* HQ 라우터 NAT(PAT) 동작 여부(inside/outside 지정, ACL 매칭)
* ASA inside/outside 라우팅 및 NAT 규칙 매칭 여부


---

## 7. 설계 변경 사항 및 한계

1. Packet Tracer 환경에서는 Object Tracking(track) 기능 지원이 제한적이므로,
   HSRP는 Priority/Preempt 기반 이중화로 설계를 단순화하였다.

2. Packet Tracer ASA는 내부 라우팅된 서브넷 트래픽에 대해 NAT 매칭/표시가 불안정한 케이스가 있어,
   NAT 변환 동작 자체를 명확히 증빙하기 위해 **HQ 라우터에서 전처리 PAT을 수행**하였다.
   변환 주소는 공인 IP가 아닌 HQ–ASA inside 구간 대역으로 설정했으며, 이는 실환경 설계의 권장안이 아니라
   **PT 시뮬레이터 제약을 회피하기 위한 검증 목적 구성**이다.

---
