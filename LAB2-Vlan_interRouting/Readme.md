
---

# LAB2 – ROAS 기반 Inter-VLAN Routing 및 DHCP 분산 구성

## 1. LAB 개요 및 목표

본 LAB은 HQ와 Branch 환경에서 VLAN을 분리하고,
Router-on-a-Stick(ROAS) 방식을 통해 Inter-VLAN Routing을 구현하는 것을 목표로 한다.

또한 HQ와 Branch 각각에서 DHCP 기능을 직접 수행하도록 구성하여,
VLAN 단위 IP 할당 및 기본 게이트웨이 동작을 검증한다.

구체적인 목표는 다음과 같다.

* HQ 및 Branch 환경에서 VLAN 분리 구성
* ROAS 기반 Inter-VLAN Routing 구현
* 각 VLAN에 대한 DHCP 자동 할당
* HQ–Branch 간 라우팅 구성
* 내부 PC 간 및 외부 네트워크(Internet) 접근 검증

---

## 2. 전체 토폴로지 설명

* 네트워크는 HQ, Branch, ISP, Internet 영역으로 구성된다.
* HQ와 Branch 각각에 Access Switch가 위치하며, VLAN 10/20/30을 사용한다.
* HQ와 Branch 라우터는 각각 ROAS 방식으로 VLAN 게이트웨이 역할을 수행한다.
* ISP 및 Internet 영역은 외부 네트워크 역할을 수행하며,
  Internet은 Loopback 주소(8.8.8.8)로 단순화하였다.

---

## 3. 설계 및 구성 시 고려 사항

### 3.1 전체 설계 방향

* VLAN 간 트래픽은 라우터에서 처리하도록 하여 L2/L3 역할을 명확히 분리한다.
* 각 사이트(HQ, Branch)는 독립적인 DHCP 서버 역할을 수행하도록 구성한다.
* 이후 LAB 확장을 고려하여 VLAN 구조를 동일하게 유지한다.

---

### 3.2 VLAN 및 ROAS 설계

* HQ 및 Branch 모두 다음 VLAN을 사용한다.

  * VLAN 10
  * VLAN 20
  * VLAN 30
* Access Switch와 라우터 간 링크는 Trunk로 구성한다.
* ROAS를 위해 라우터의 서브인터페이스를 다음과 같이 구성한다.

  * `G0/2.10` – VLAN 10
  * `G0/2.20` – VLAN 20
  * `G0/2.30` – VLAN 30
  * `G0/2.99` – Native VLAN
* 각 VLAN의 Default Gateway는 `.1` 주소로 설정한다.

---

### 3.3 DHCP 설계

* HQ 라우터는 HQ VLAN에 대한 DHCP 서버 역할을 수행한다.

  * VLAN 10: `192.168.1.0/24`
  * VLAN 20: `192.168.2.0/24`
  * VLAN 30: `192.168.3.0/24`
* Branch 라우터는 Branch VLAN에 대한 DHCP 서버 역할을 수행한다.

  * VLAN 10: `192.168.10.0/24`
  * VLAN 20: `192.168.20.0/24`
  * VLAN 30: `192.168.30.0/24`
* 각 VLAN에서 `.1` 주소는 게이트웨이로 사용하며,
  `.0–.10` 범위는 DHCP 제외 주소로 설정한다.

---

### 3.4 라우팅 설계

* HQ와 Branch 간에는 정적 라우팅 또는 기본 라우팅을 통해 상호 통신을 구성한다.
* 외부 Internet 접근을 위해 HQ에서 Default Route를 설정한다.
* Internet은 실제 환경 대신 Loopback 주소 `8.8.8.8`로 대체하여 검증한다.

---

## 4. 사용 기술

* VLAN
* Trunk / Native VLAN
* Router-on-a-Stick (ROAS)
* DHCP
* Static / Default Routing

---

## 5. 검증 항목

* 동일 VLAN 내 PC 간 통신 가능 여부
* 서로 다른 VLAN 간 Inter-VLAN Routing 정상 동작 여부
* HQ VLAN ↔ Branch VLAN 간 통신 가능 여부
* 각 VLAN PC가 DHCP로 IP를 정상 할당받는지 확인
* 내부 PC에서 Internet(8.8.8.8) 접근 가능 여부

---

## 6. 트러블슈팅 포인트

* Trunk 설정 오류 시 VLAN 트래픽 전달 실패
* Native VLAN 불일치로 인한 통신 문제
* 서브인터페이스에 `encapsulation dot1q` 설정 누락
* DHCP Pool 네트워크 주소 불일치
* Default Route 미설정 시 외부 통신 실패

---

## 7. 설계 한계 및 범위

* 본 LAB은 단일 라우터 기반 구조로 구성되어,
  게이트웨이 및 라우터 이중화는 고려하지 않았다.
* DHCP를 각 라우터에서 개별적으로 수행하므로,
  중앙 집중형 DHCP 구조는 이후 LAB에서 다룬다.

---

