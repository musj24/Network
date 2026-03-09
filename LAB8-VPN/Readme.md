

---

# LAB8 – Site-to-Site VPN (IPsec/IKEv1) 기반 HQ–Branch 사설망 연동 (Packet Tracer 제약 반영)

## 1. LAB 개요 및 목표

본 LAB은 기존 캠퍼스 토폴로지(HQ 중심 구조)에 **지사(Branch)** 를 추가하고,
공용망(Internet/ISP)을 경유하여 **HQ–Branch 간 Site-to-Site IPsec VPN(IKEv1, PSK)** 을 구성하는 것을 목표로 한다.

특히 Packet Tracer 환경에서 ASA 기능 제약(NAT/옵션/정책 처리)이 존재하므로,
“VPN 터널 성립”뿐 아니라 **트래픽이 실제로 내부망까지 전달되는지(End-to-End)** 를 증거 기반으로 검증한다.

구체적인 목표는 다음과 같다.

* HQ 보호망 `192.168.50.0/29` ↔ Branch 사용자망 `172.16.10.0/24` 간 **사설망 통신(Ping)**
* IPsec Phase1/Phase2 정상 성립(IKE SA / IPsec SA)
* `show crypto ipsec sa` 기반 **encaps/decaps 카운터 증가 증빙**
* 왕복 라우팅(리턴 패스) 확보 및 정책(ACL) 영향 분리


---

## 2. 전체 토폴로지 설명

* HQ 영역

  * 내부 VLAN(기존 캠퍼스 구성 유지)
  * **VPN 보호 대상:** `192.168.50.0/29` (서버 VLAN)
  * HQ ASA(Outside 공용망, Inside HQ 내부망)

* Branch 영역

  * Branch Router 뒤 사용자망 `172.16.10.0/24`
  * Branch ASA(Outside 공용망, Inside Branch Router와 트랜짓)

* 공용망 영역

  * ISP/Internet 역할 라우터
  * VPN은 공용망에서 **ASA Outside IP 간 캡슐화 트래픽(ESP/UDP4500)** 으로 전달됨

---

## 3. 설계 및 구성 시 고려 사항

### 3.1 전체 설계 방향

* VPN은 “암호화 터널”이며 라우팅을 대신하지 않으므로,

  * **왕복 라우팅** 를 먼저 확보해야 한다.
* VPN 보호 대역은 트랜짓(ASA–Router 연결망)이 아니라,

  * **실제 사용자/서버가 위치한 내부 대역**을 대상으로 정의한다.
* Packet Tracer ASA는 기능 옵션 제약으로 인해,

  * 실장비에서 “자동으로 통과”되는 정책이 PT에서는 차단될 수 있으므로
    **인터페이스 ACL을 명시적으로 적용**해 트래픽을 통과시킨다.

---

### 3.2 주소 체계(요약)

* HQ VPN 보호망: `192.168.50.0/29` 
* Branch 사용자망: `172.16.10.0/24` (Branch PC)
* Branch ASA Inside(트랜짓): `172.16.20.1/30`
* Branch Router 트랜짓: `172.16.20.2/30`
* HQ ASA Inside: `192.168.60.6/29`
* HQ ASA Outside: `203.0.60.6/29`
* Branch ASA Outside: `203.20.10.2/30`

---

### 3.3 라우팅 설계

#### Branch 측

* Branch PC(172.16.10.0/24) 기본 게이트웨이: Branch Router(172.16.10.1)
* Branch Router는 HQ 보호망(192.168.50.0/29)을 **Branch ASA(172.16.20.1)** 로 전달해야 함


#### HQ 측

* HQ 내부(서버 VLAN)에서 Branch 대역(172.16.10.0/24)으로 가는 리턴 경로가 필요
* HQ ASA는 내부(192.168.x)로 라우팅을 위해 inside 정적 라우트를 사용하며,
  외부로는 default route를 유지

---

### 3.4 VPN 설계 (IKEv1 / IPsec)

* IKEv1 기반 Site-to-Site (L2L) IPsec
* 인증: Pre-Shared Key(PSK)
* Phase1: AES / SHA / DH Group2
* Phase2: ESP-AES / ESP-SHA-HMAC (Tunnel mode)
* Crypto ACL(interesting traffic):

  * Branch: `172.16.10.0/24 → 192.168.50.0/29`
  * HQ: `192.168.50.0/29 → 172.16.10.0/24`
* Crypto map은 반드시 **outside 인터페이스에 적용**

---

### 3.5 Packet Tracer 제약 및 대응

* PT ASA는 일부 환경에서 VPN 트래픽이

  * `decaps`는 증가하지만 내부로 전달되지 않는 현상이 발생할 수 있음
* 실장비에서는 특정 옵션(sysopt 등)으로 완화되는 동작이,

  * PT에서는 미지원/제한되어 **outside 인바운드 ACL로 VPN 트래픽을 명시 허용**해야 정상 통신이 가능했음

---

## 4. 사용 기술

* Site-to-Site IPsec VPN (IKEv1, PSK)
* Crypto ACL(interesting traffic), Transform-set
* Crypto map

---

## 5. 검증 항목

### V1. IKE Phase1 성립

* 양쪽 ASA에서 `show crypto ikev1 sa` 확인
* SA 상태가 활성/유효 상태로 형성되는지 확인

### V2. IPsec Phase2 성립 및 카운터 증빙

* `show crypto ipsec sa`에서 SA 생성 확인
* ping 트리거 후 `#pkts encaps/decaps` 증가 확인

  * 정상: 양쪽 모두 encaps/decaps 증가(왕복 트래픽 성립)

### V3. End-to-End 통신

* Branch PC(172.16.10.x) → HQ Server(192.168.50.2) ping 성공


---

## 6. 트러블슈팅 포인트(이번 LAB에서 실제로 겪은 이슈 중심)

### TS1. Phase1만 뜨고 Phase2/IPsec SA가 안 생김

* 원인 예시

  * Crypto map 이름 불일치(맵 구성 분산)
  * Branch ASA 기본 경로(route) 오타/누락
  * Crypto ACL 보호 대역 오설정(트랜짓 대역을 넣는 실수)

* 조치

  * Crypto map 구성(ACL/peer/transform/interface)을 **단일 map name으로 일관성 유지**
  * Branch ASA default route(`route outside 0.0.0.0/0 …`) 확인

### TS2. IPsec SA는 뜨는데 ping 실패 (encaps/decaps 비대칭)

* 진단 방법(핵심)

  * `show crypto ipsec sa`에서 **encaps/decaps 비대칭**으로 문제 방향을 좁힘
  * 예: HQ에서 decaps만 증가하고 encaps 0이면 “요청은 도착, 응답이 안 나감(리턴 경로/정책/내부 전달 문제)”
* 조치

  * HQ 내부 리턴 라우팅 점검
  * PT 제약으로 outside→inside 정책이 차단될 경우 **outside ACL에 VPN 트래픽 명시 permit**

---

## 7. 설계 한계 및 범위

* 본 LAB은 Packet Tracer 환경 제약(ASA의 NAT 예외 지원 제한)을 고려하여,

  * “인터넷 통신(NAT 포함)”과 “VPN 검증”을 동시에 완벽히 구현하기보다,
  * 우선 **HQ–Branch VPN(사설↔사설)** 통신 성립 및 증빙에 집중하였다.
* 실장비/가상장비(ASAv, IOSv 등) 환경에서는

  * NAT 예외, 정책 옵션, 디버깅 가시성이 더 풍부하므로
  * 동일 설계를 더 정석적으로 구현 가능하다.

---
