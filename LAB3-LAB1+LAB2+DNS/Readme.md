
---

# LAB3 – LAB1 + LAB2 통합 (OSPF / NAT / DHCP / DNS)

## 1. LAB 개요 및 목표

본 LAB은 기존 LAB1과 LAB2에서 각각 구성한 네트워크를 하나의 통합 토폴로지로 결합하고,
중앙 DHCP/DNS 서버를 기반으로 내부 및 외부 자원에 대해 **도메인 기반 접근**을 구현하는 것을 목표로 한다.

구체적인 목표는 다음과 같다.

* LAB1과 LAB2에서 사용한 네트워크 기술을 단일 토폴로지로 통합
* 중앙 DHCP/DNS 서버를 통해 내부 PC에 IP 주소 및 DNS 정보 제공
* 내부 PC에서 내부 서버 및 외부 서버(Internet 역할: 8.8.8.8, 외부 Web Server)에 대해
  IP 주소가 아닌 Domain Name 기반 접근 가능 여부 검증
* 내부 → 외부 트래픽에 대한 NAT 동작 확인

---

## 2. 전체 토폴로지 설명

* 네트워크는 HQ, Branch, ISP, Internet 영역으로 구성된다.
* HQ 및 Branch 환경은 VLAN 10 / 20 / 30으로 분리되어 있다.
* HQ 라우터는 ROAS 방식으로 VLAN 간 라우팅을 수행한다.
* 중앙 DHCP/DNS 서버와 내부 Web 서버는 Branch 영역에 위치한다.
* HQ–Branch 간에는 OSPF를 사용하여 라우팅 정보를 교환한다.
* Internet은 실제 외부망 대신 Loopback 주소(8.8.8.8)로 단순화하여 구성한다.

---

## 3. 설계 및 구성 시 고려 사항

### 3.1 LAB 통합 전략

* LAB2의 토폴로지를 기본 골격으로 사용하였다.
* VLAN 구조 및 ROAS 구성은 LAB2와 동일하게 유지하였다.
* LAB1에서 사용한 기능을 단계적으로 추가하여 통합하였다.

  * OSPF 기반 동적 라우팅
  * 중앙 DHCP 서버 사용
  * NAT 구성

---

### 3.2 DHCP 설계 변경

* LAB1: 중앙 DHCP 서버 사용
* LAB2: 라우터 자체 DHCP 서버 사용
* LAB3: 라우터 DHCP 기능 제거

LAB3에서는 LAB1과 동일하게 **중앙 DHCP 서버 방식으로 통일**하였다.

* 모든 VLAN의 게이트웨이 인터페이스에서 `ip helper-address`를 사용하여 DHCP Relay를 구성한다.
* DHCP 서버를 통해 IP 주소와 DNS 서버 주소를 함께 전달한다.

---

### 3.3 라우팅 설계 (OSPF)

* LAB2에서는 Static Route / Default Route 중심의 라우팅을 사용하였다.
* LAB3에서는 LAB1과 동일하게 OSPF를 사용한다.
* HQ–Branch 간 네트워크 확장성을 고려하여 동적 라우팅을 적용하였다.
* 외부 ISP 및 Internet 영역으로 내부 라우팅 정보가 전달되지 않도록,

  * ISP와 연결된 인터페이스에는 passive-interface를 적용한다.

---

### 3.4 NAT 설계

* 내부 네트워크에서 외부(Internet 역할 네트워크)로 나가는 트래픽에 대해 NAT를 적용한다.
* 내부 사설 IP 대역은 외부 인터페이스 IP로 변환된다.
* 내부 PC에서 외부 서버(8.8.8.8 및 외부 Web Server)에 대한 접근 가능 여부를 확인한다.

---

## 4. 사용 기술

* OSPF
* NAT
* DHCP
* DNS
* ROAS
* DAI

---

## 5. 검증 항목

* 내부 PC가 DHCP를 통해 IP 및 DNS 서버 정보를 정상적으로 할당받는지 확인
* 내부 PC에서 내부 서버에 대해 Domain Name 기반 접근 가능 여부 확인
* 내부 PC에서 외부 서버(8.8.8.8, 외부 Web Server)에 대해 Domain Name 기반 접근 가능 여부 확인
* HQ–Branch 간 OSPF 이웃 관계 형성 및 라우팅 정보 교환 여부 확인
* 내부 → 외부 트래픽에 대한 NAT 변환 동작 확인

---

## 6. 트러블슈팅 포인트

### – DNS

* DHCP를 통해 DNS 서버 주소가 정상적으로 전달되었는지 확인
* DNS 서버에 A 레코드가 정상적으로 등록되어 있는지 확인

### – Firewall / NAT

* ICMP와 TCP 트래픽을 구분하여 동작 여부 점검
* NAT 변환 테이블 및 세션 생성 여부 확인

### – DAI

* DHCP Snooping 바인딩 테이블 생성 여부 확인
* Trust / Untrust 포트 설정 점검
* VLAN 단위 DAI 적용 여부 확인

---

## 7. 설계 변경 사항 및 한계

1. 초기 설계에서는 내부 DHCP/DNS 서버와 Web 서버를 HQ 라우터에 직접 연결하려 하였다.
그러나 Packet Tracer 환경에서 다음과 같은 제약이 존재하였다.

* HQ 라우터의 인터페이스 수 제한
* HQ 라우터의 G0/2 인터페이스가 ROAS 구성으로 사용 중

이로 인해 토폴로지 구성에 제약이 발생하였고,
서버들은 Branch 라우터 측에 연결한 뒤 HQ–Branch 간 OSPF를 통해 접근성을 확보하는 방식으로 설계를 수정하였다.

또한 Branch-SW는 L3 스위치이며,
DHCP를 수신하는 G1/0/24 인터페이스가 `no switchport` 상태이기 때문에
L2 기능인 DHCP Snooping을 적용할 수 없다.

이에 따라 Branch 영역의 DAI 관련 구성은 본 LAB에서는 보류하고,
다음 LAB에서 토폴로지 변경과 함께 재구성한다.

2. ASA에서 NAT 설정을 해놓았지만,
   “Packet Tracer 환경에서 ASA NAT 동작/가시성 제약이 확인되어, NAT 변환 동작 자체를 명확히 증빙하기 위해 HQ 라우터에서 PAT을 수행하였다. 변환 주소는 공인 IP가 아닌 HQ–FW 구간 주소로 설정했으며, 이는 실환경의 공인 NAT을 단순화한 검증 목적의 구성이다.”
  
---


