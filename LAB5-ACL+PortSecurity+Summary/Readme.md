
---

# LAB5 – 트래픽 제어 및 Access Layer 보안 설계

## 목표

본 LAB은 **LAB4에서 구성한 Distribution Layer 이중화 기반 캠퍼스 토폴로지**를 그대로 사용하여,
단순한 연결성 확보를 넘어 **트래픽 제어 및 Access Layer 보안 요소를 추가로 설계·구현**하는 것을 목표로 한다.

구체적인 목표는 다음과 같다.

* 서버 VLAN과 사용자 VLAN 간 접근 정책 명확화
* VLAN별 Internet 접근 권한 분리
* Distribution Layer 기준 트래픽 제어(ACL) 적용
* Access Layer에서 단말 보안 강화
* STP 보호 기능을 통한 계층 구조 안정성 보완

---

## 사용 기술

* Extended ACL (SVI 기반 적용)
* Port-Security
* STP Protection

  * PortFast
  * BPDU Guard
  * Root Guard
* EtherChannel (LAB4 구성 유지)

---

## 설계 및 구성 시 고려 사항

### 2.1 설계 전제

* 본 LAB은 **LAB4의 토폴로지 및 이중화 구조를 변경하지 않는다.**
* 라우팅(OSPF), HSRP, EtherChannel 구성은 그대로 유지한다.
* 본 LAB에서는 **“누가, 어디로, 접근할 수 있는가”**에 대한 정책 설계에 집중한다.

---

### 2.2 서버 접근 제어 설계 (ACL)

#### 설계 목표

* Internal Server VLAN(VLAN 50)은 **특정 사용자 VLAN만 접근 가능**
* 서버 보호 정책을 단일 지점에서 일관되게 관리

#### 설계 방식

* **서버 VLAN(SVI 50) 기준으로 Extended ACL 적용**
* VLAN 10 사용자만 서버 접근 허용
* VLAN 20 / VLAN 30의 서버 접근은 차단

#### 설계 의도

* 서버 접근 제어를 **목적지 기준(Server VLAN 경계)**으로 통합
* 사용자 VLAN별 ACL 중복 구성 방지
* 서버 VLAN 확장 시에도 정책 유지 용이

---

### 2.3 Internet 접근 제어 설계 (ACL)

#### 설계 목표

* VLAN 10은 내부 통신만 허용
* VLAN 20 / VLAN 30만 Internet 접근 허용

#### 설계 방식

* **출발지 기준 정책**
* VLAN 10 SVI에 ACL을 **in 방향**으로 적용
* RFC1918 대역은 허용, 그 외 목적지는 차단

#### 설계 의도

* “누가 외부로 나갈 수 있는가”를 출발지 기준으로 명확히 표현
* 서버 접근 정책과 Internet 접근 정책을 분리하여 상호 영향 최소화
* Distribution Layer에서 1차 트래픽 통제 수행

---

### 2.4 Access Layer 보안 설계 (Port-Security)

Access Switch의 단말 연결 포트에 다음 보안 정책을 적용하였다.

* Port-Security 활성화
* 단일 MAC 주소만 허용 (`maximum 1`)
* Sticky MAC 주소 사용
* Violation 발생 시 포트 Shutdown

이를 통해,

* 허브/스위치 무단 연결 방지
* 비인가 단말 연결 시 즉각적인 포트 차단
* 실제 캠퍼스 환경의 기본적인 단말 보안 정책 반영

---

### 2.5 STP 보호 설계

* Access 단말 포트

  * PortFast 적용
  * BPDU Guard 적용

* Distribution–Access 경계 트렁크(EtherChannel)

  * **Root Guard 적용**
  * Access 계층에서 Root Bridge가 선출되는 상황 방지

본 LAB에서는 Packet Tracer IOS 제약으로 Loop Guard가 지원되지 않아,
Root Guard를 통해 STP 보호 의도를 반영하였다.

---

## 트러블슈팅 포인트

### – ACL

* ACL 적용 인터페이스(SVI) 및 방향(in / out) 확인
* 서버 접근 제어 ACL이 서버 VLAN SVI 기준으로 적용되었는지 확인
* Internet 접근 제어 ACL이 출발지 VLAN SVI 기준(in 방향)으로 적용되었는지 확인
* ACL 항목 순서로 인한 의도치 않은 permit/deny 여부 점검
* `permit ip any any`로 정책이 무력화되지 않았는지 확인
* ACL hit count 증가 여부 확인

---

### – 서버 접근 제어

* 서버 VLAN 목적지 대역이 ACL에 정확히 명시되었는지 확인
* 특정 VLAN만 허용하도록 설계한 정책이 다른 VLAN에 영향 주지 않는지 점검
* 서버 VLAN 외 트래픽이 ACL 적용 대상에 포함되지 않았는지 확인

---

### – Internet 접근 제어

* RFC1918 대역 정의 누락으로 내부 통신까지 차단되지 않았는지 확인
* VLAN 10의 외부 통신 차단 여부 확인
* VLAN 20 / VLAN 30의 Internet 접근 정상 동작 여부 확인
* ACL 적용 이후 NAT 동작에 영향이 없는지 점검

---

### – Port-Security

* `switchport port-security` 활성화 여부 확인
* Sticky MAC 주소 정상 학습 여부 확인
* Violation 발생 시 포트 상태(shutdown / err-disable) 확인
* 단말 교체 시 Sticky MAC 재학습 필요 여부 확인

---

### – STP 보호

* Root Guard 적용 포트가 Distribution–Access 경계 트렁크인지 확인
* Root Guard 동작 시 `root-inconsistent` 상태 여부 확인
* STP 보호 설정이 EtherChannel 구성에 영향을 주지 않는지 점검

---

## 검증 요소

* VLAN 10 → Server VLAN 접근 성공 여부
* VLAN 20 / VLAN 30 → Server VLAN 접근 차단 여부
* VLAN 10 → Internet 접근 차단 여부
* VLAN 20 / VLAN 30 → Internet 접근 허용 여부
* Port-Security 위반 발생 시 포트 Shutdown 동작 확인
* Root Guard 트리거 시 STP 상태 변화 확인

---

## 설계 판단 및 한계

* ACL은 Distribution Layer에서 1차 통제로 구성하였으며,
  실제 운영 환경에서는 Firewall 정책과 병행 운용이 필요하다.
* Packet Tracer IOS 제약으로 Loop Guard는 적용하지 못하였으며,
  Root Guard를 통해 STP 보호 의도를 반영하였다.
* 본 LAB은 기능 구현 자체보다 **정책 분리 및 설계 의도 명확화**에 중점을 두었다.

---

※ 본 LAB은 LAB4 기반 확장 LAB이며,
기존 이중화 및 라우팅 구조는 변경하지 않는다.

---

