
---

# LAB1 – OSPF / NAT / DHCP 기반 기본 네트워크 구성

## 1. LAB 개요 및 목표

본 LAB은 HQ와 Branch로 분리된 환경에서,
중앙 DHCP 서버를 통해 내부 호스트에 IP를 할당하고
외부 인터넷과의 통신에는 NAT(PAT)를 적용하는 기본 네트워크 구성을 목표로 한다.

또한 Branch 내부 서버는 Static NAT를 통해 외부에서 접근 가능하도록 구성한다.

구체적인 목표는 다음과 같다.

* HQ–Branch 간 OSPF 기반 동적 라우팅 구성
* 중앙 DHCP 서버를 통한 HQ / Branch 내부 PC IP 할당
* 공인 주소 절약을 위한 PAT 적용
* Branch 내부 서버에 대한 Static NAT 구성
* 내부 PC에서 Internet(Loopback 8.8.8.8) 접근 가능 여부 검증

---

## 2. 전체 토폴로지 설명

* 네트워크는 HQ, Branch, ISP, Internet 영역으로 구성된다.
* HQ와 Branch는 각각 내부 스위치 및 PC를 수용한다.
* DHCP 서버는 ISP 영역에 위치하며, HQ 및 Branch와 다른 네트워크에 존재한다.
* HQ 라우터는 내부 네트워크와 외부(Internet) 간의 경계 장비 역할을 수행한다.
* Branch 내부에는 외부 공개 대상 서버가 위치한다.

---

## 3. 설계 및 구성 시 고려 사항

### 3.1 전체 설계 방향

* 내부 네트워크(HQ, Branch)는 OSPF를 통해 라우팅 정보를 교환한다.
* 외부 ISP 및 Internet 영역으로 내부 라우팅 정보가 전달되지 않도록 제어한다.
* NAT 정책은 HQ 라우터에서 일괄 처리하여 구조를 단순화한다.

---

### 3.2 DHCP 설계

* 중앙 DHCP 서버를 사용하여 다음과 같이 주소를 할당한다.

  * HQ 내부 PC: `192.168.1.0/24`
  * Branch 내부 PC: `192.168.2.0/24`
* 각 네트워크의 Default Gateway는 `.1` 주소로 설정한다.
* DHCP 제외 주소는 각 네트워크에서 `0–10` 범위로 설정한다.
* DHCP 서버는 ISP 영역에 위치하므로,

  * HQ 및 Branch 라우터의 내부 인터페이스에 `ip helper-address`를 설정하여 DHCP Relay를 구성한다.

---

### 3.3 라우팅 설계 (OSPF)

* HQ–Branch 간에는 OSPF를 사용하여 내부 네트워크를 연결한다.
* ISP 및 Internet 영역으로 내부 라우팅 정보가 전달되지 않도록,

  * ISP와 연결된 인터페이스
  * DHCP 서버와 연결된 인터페이스에 `passive-interface`를 적용한다.
* 내부 네트워크가 외부로 통신할 수 있도록,

  * HQ 라우터에 Default Route를 설정하고 이를 내부로 전파한다.
* Internet은 실제 환경을 단순화하기 위해 Loopback 주소 `8.8.8.8`로 대체한다.

---

### 3.4 NAT 설계

* HQ–ISP 간 연결 인터페이스(G0/2)에 PAT를 적용하여,

  * HQ 및 Branch 내부 PC가 HQ의 공인 IP를 사용해 외부와 통신하도록 구성한다.
* Branch 내부 서버는 외부에서 접근 가능해야 하므로,

  * Static NAT를 사용해 `203.1.113.1` 주소로 변환한다.
* PAT와 Static NAT 주소가 충돌하지 않도록 주소 범위를 분리한다.

---

## 4. 사용 기술

* OSPF
* NAT (PAT / Static NAT)
* DHCP

---

## 5. 검증 항목

* HQ 내부 PC ↔ Branch 내부 PC 간 Ping 성공 여부
* HQ 및 Branch 내부 PC에서 Internet(8.8.8.8) Ping 성공 여부
* ISP의 G0/0, G0/1 인터페이스에 passive-interface 적용 여부
* 내부 트래픽이 HQ G0/2 인터페이스 주소로 PAT 변환되는지 확인
* Branch 내부 서버가 `203.1.113.1` 주소로 Static NAT 변환되는지 확인

---

## 6. 트러블슈팅 포인트

* Default Route를 전파하지 않을 경우 내부에서 외부 접근 불가
* passive-interface를 잘못된 인터페이스에 적용할 경우 OSPF 이웃 형성 실패
* `ip helper-address` 누락 시 DHCP 할당 실패
* Static NAT 주소와 PAT 주소가 충돌할 경우 NAT 변환 오류 발생

---

## 7. 설계 한계 및 범위

* Internet 환경은 실제 ISP 대신 Loopback 주소로 단순화하여 구성하였다.

---
