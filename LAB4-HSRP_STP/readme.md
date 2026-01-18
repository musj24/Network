
---

# LAB4 – Distribution Layer 이중화 및 캠퍼스 네트워크 설계 (HSRP/OSPF/STP/EtherChannel + ASA 연동)

## 1. 목표

본 LAB은 LAB3의 단일 경로 기반 구조를 확장하여, **캠퍼스 표준 계층형 설계(Access–Distribution–Edge/HQ)** 를 기반으로 다음을 구현·검증하는 것을 목표로 한다.

* Access–Distribution–HQ 계층 구조 분리 및 역할 명확화
* Distribution Switch에서 SVI 기반 Inter-VLAN Routing 수행
* 사용자 VLAN(10/20/30)에 대해 **HSRP 기반 Default Gateway 이중화** 구현
* DSW–HQ 및 DSW–DSW 구간 **OSPF 기반 라우팅** 구성
* DSW 간 L3 백업 링크를 통한 업링크 장애 시 자동 우회 경로 확보(수렴 기반)
* Access–Distribution 구간 **EtherChannel(LACP)** 및 **STP Root 설계**로 L2 안정성 확보
* Firewall(ASA) 연동 및 외부 트래픽 NAT 동작 검증(PT 제약 포함)

---

## 2. 사용 기술

* Inter-VLAN Routing (SVI)
* HSRP
* OSPF (Area 0)
* STP (Rapid-PVST, Root Bridge 설계)
* EtherChannel (LACP)
* DHCP Relay (ip helper-address)
* DAI / DHCP Snooping (Access Layer)
* DNS
* NAT (HQ Router PAT + ASA NAT, PT 제약 회피 목적)

---

## 3. 토폴로지 및 주소 설계 요약

### 3.1 계층별 역할

* **Access Layer (ACC1/ACC2)**

  * 단말 수용, VLAN 분리
  * L2 보안(DAI, DHCP Snooping) 적용 지점

* **Distribution Layer (DSW1/DSW2)**

  * SVI 기반 Inter-VLAN Routing 수행
  * 사용자 VLAN에 대해 HSRP 게이트웨이 이중화
  * STP Root(Primary/Secondary) 역할 수행
  * 상단(HQ) 및 백업 링크(OSPF)로 라우팅 경로 제어

* **HQ Layer (HQ1/HQ2)**

  * 외부(ASA/Internet)로 나가는 Edge 역할
  * 기본 경로(Default Route) 보유
  * OSPF를 통해 Distribution에 기본 경로 제공(필요 시)
  * **PT ASA 제약을 회피하기 위한 1차 PAT 수행(전처리 NAT)**

* **Firewall (ASA)**

  * 내부/외부 경계(Inside/Outside)
  * 외부 방향 NAT 수행(PT 환경 기준)

### 3.2 VLAN/게이트웨이 정책(

* 사용자 VLAN: **VLAN10/20/30**

  * Default Gateway는 **HSRP Virtual IP (각 VLAN의 .1)** 를 사용
  * DSW1/DSW2 우선순위로 Active/Standby 역할 분리

* 서버 VLAN: **VLAN50 (Servers)**

  * 서버/관리 자원을 수용
  * **본 LAB 범위에서는 VLAN50의 게이트웨이 이중화(HSRP)까지는 확장하지 않고**, 서버 접근/라우팅 분리 관점에서만 사용한다.
  * (확장 가능 항목) VLAN50 HSRP/VIP 추가, 서버 이중화 설계



---

## 4. 설계 상세

### 4.1 Inter-VLAN Routing 위치 변경

* LAB3: ROAS 기반(라우터에서 Inter-VLAN Routing)
* LAB4: **Distribution(SVI) 기반 Inter-VLAN Routing**

목적:

* 내부 VLAN 간 트래픽이 불필요하게 상단 라우터를 경유하지 않도록 최적화
* 캠퍼스 구조에서 Distribution의 역할을 명확화

### 4.2 HSRP 설계(사용자 VLAN)

* VLAN10/20/30에 대해 HSRP 적용
* 단말의 Default Gateway는 HSRP Virtual IP 사용
* DSW1/DSW2 Priority 차이를 통해 Active/Standby 구분(필요 시 VLAN별 분산 가능)
* PT 제약으로 Object Tracking은 제외하고, 기본 Priority/Preempt 기반으로 구성

### 4.3 OSPF 설계(상단 이중 경로 + 백업 링크)

* DSW–HQ 링크는 L3 P2P 링크로 구성, OSPF Area 0 참여
* DSW–DSW 간 별도 **L3 백업 링크(/30)** 를 OSPF에 참여시켜 우회 경로 확보
* HQ는 외부로 나가는 Default Route를 보유하며, 필요 시 `default-information originate`로 내부에 전파

**우회 경로 예시:**

* DSW1–HQ1 업링크 장애 시
  → DSW1 → (DSW1–DSW2 L3 링크) → DSW2 → HQ2 방향으로 OSPF 수렴을 통해 자동 우회

### 4.4 STP 및 EtherChannel 설계

* Access–Distribution 구간은 EtherChannel(LACP)로 구성하여:

  * 링크 집성(대역폭 증가)
  * 단일 링크 장애 시에도 포트채널 유지(가용성)
* STP는 Rapid-PVST 기반으로:

  * Root Primary/Secondary를 Distribution에 고정
  * HSRP Active 장비와 STP Root를 가능하면 일치시켜(동일 VLAN 기준) 불필요한 우회/차단 최소화

---

## 5. Firewall 및 NAT 설계 (Packet Tracer 제약 포함)

### 5.1 기본 의도(실환경 기준)

* 일반적인 실환경에서는 **ASA 단일 NAT**로 내부(사설) → 외부(공인) 변환을 수행하는 것이 자연스럽다.

### 5.2 PT 환경에서의 제약과 우회 설계

* Packet Tracer ASA는 내부 라우팅된 서브넷(DSW SVI 뒤 VLAN) 트래픽에 대해 **NAT 매칭/표시/검증이 불안정**한 케이스가 있다.
* 본 LAB에서는 “이중 NAT”를 의도한 것이 아니라, **PT 제약 회피를 위한 전처리 NAT(PAT)** 를 적용한다.

우회 흐름:

1. 사용자 VLAN(192.168.x.x) 트래픽이 HQ로 올라옴
2. **HQ에서 1차 PAT** 수행 → 소스가 ASA inside 대역으로 변환
3. ASA에서 외부 방향 NAT/정책 처리 수행

또한 HQ1/HQ2 모두 동일한 PAT 구성을 적용하여, 상단 라우터 장애 시에도 외부 트래픽 경로가 유지되도록 한다.

---

## 6. 검증(Verification)

다음 검증은 “CLI 출력 + 트래픽 테스트”를 기본 근거로 한다.

* **VLAN/Trunk 정상**: VLAN 존재, trunk allowed/native 일치
* **EtherChannel 정상**: Port-Channel 번들링(SU/RU, 멤버 P 상태)
* **STP Root 정상**: VLAN별 Root가 Distribution에 고정
* **HSRP 정상**: VLAN별 Active/Standby 및 VIP 응답 확인
* **OSPF 정상**: Neighbor 형성, 라우트 교환, 우회 링크를 통한 수렴 확인
* **외부 통신/NAT 검증(PT 기준)**:

  * HQ: `show ip nat translations / statistics` 기반으로 PAT 동작 증빙
  * ASA: `show nat` hit 카운터(필수) + (가능하면) `show conn` + Simulation 캡처 보조

---

## 트러블슈팅 포인트 
### – OSPF
* Area 불일치 여부
* 인터페이스 OSPF 활성화 여부
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

### – Firewall / NAT
* 내부 트래픽 소스 주소 변환 여부
* HQ 라우터 NAT 동작 확인
* ASA inside / outside 인터페이스 역할 확인


---

## 8. 설계 판단 및 한계

* PT 환경에서는 Object Tracking 지원이 제한적이므로, HSRP는 Priority/Preempt 기반으로 단순화하였다.
* PT ASA의 NAT 동작/표시 제약으로 인해 HQ에서 전처리 PAT을 적용하였다(실환경 권장 아님).
* 그럼에도 **OSPF 수렴 + HSRP + 상단 이중 경로 + L2 안정화(EC/STP)** 를 통해 캠퍼스 네트워크 관점의 가용성과 장애 우회 흐름을 검증하였다.

---

