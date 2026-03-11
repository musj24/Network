
---

# LAB7 – 관리 접근 제어 (Admin-Only SSH to HQ1/HQ2)

## 목표

본 LAB은 **LAB6까지 구성된 운영 가능한 캠퍼스 네트워크**를 기반으로,
운영/관리 관점에서 중요한 **원격 관리 접근 통제(SSH 접근 제어)** 를 구현하는 것을 목표로 한다.

구체적인 목표는 다음과 같다.

* **Admin PC(192.168.50.5)에서만** HQ1/HQ2 라우터에 SSH 접속 허용
* 그 외 사용자/서버/타 VLAN 단말의 SSH 접속은 차단
* “접속 가능/불가능”을 명확히 검증 가능한 형태로 테스트 케이스 구성
* 기존 운영 요소(NTP/Syslog/OSPF/HSRP/ACL 등)와 충돌 없이 적용

---

## 사용 기술

* SSH v2 (Secure Remote Management)
* VTY Access Control (`access-class <10> in`)
* Standard ACL (관리 IP 기반 허용)
* (기존 구성 유지) OSPF / HSRP / ACL 정책 / NTP / Syslog

---

## 설계 및 구성 시 고려 사항

### 2.1 설계 전제

* **LAB6까지의 토폴로지 및 정책을 변경하지 않는다.**
* 라우팅(OSPF), 이중화(HSRP), 운영 요소(NTP/Syslog)는 그대로 유지한다.
* Packet Tracer 환경 제약을 인지하되, 실장비 기준 운영 설계를 우선한다.
* HQ1, HQ2의 enable secret은 각기 HQ1, HQ2로 한다.

---

### 2.2 관리 접근 통제 설계 (Admin-Only SSH)

#### 설계 목표

* 네트워크 장비 관리접근을 **최소 권한** 원칙으로 통제
* 운영자가 아닌 일반 사용자/서버 대역에서 HQ 장비로의 원격 접속 시도를 차단
* 인증/암호화가 필요한 원격 접속은 Telnet이 아닌 SSH로 통일

#### 설계 방식

* HQ1/HQ2 라우터에 SSH 서버 기능 활성화

  * 도메인 설정 + RSA Key 생성
  * SSH 버전 2 강제
* 표준 ACL로 **Admin PC(192.168.50.5)만 허용**
* VTY Line에 `access-class`를 적용하여 **SSH 접속 시도 자체를 인입에서 통제**

**Admin PC:** `192.168.50.5`
**허용 대상 장비:** HQ1, HQ2 (Router)

---

## 구성 요약(핵심 설정)

> 아래 핵심은 두 라우터(HQ1/HQ2)에 공통으로 적용된 구조입니다.

### 3.1 SSH 기본 설정

* `ip domain-name local.intra`
* `crypto key generate rsa`
* `ip ssh version 2`


### 3.2 Admin-Only 접근 제어(ACL + VTY)

* `access-list 10 permit host 192.168.50.5`
* `line vty 0 4`

  * `transport input ssh`
  * `access-class 10 in`

#### 설계 의도

* ACL을 인터페이스에 거는 데이터 트래픽 정책과 달리,
  본 LAB은 **VTY(관리 접속) 자체를 제어**하여 관리면 보안을 강화한다.
* 즉, 사용자 VLAN 간 통신/서비스 영향 없이 “관리 접속만” 안전하게 제한한다.

---

## 트러블슈팅 포인트

### – SSH 접속 실패(정상 IP인데도 접속 불가)

* **VTY에서 SSH만 허용했는지** 확인

  * `show run | section line vty`
  * `transport input ssh` 누락 시 Telnet/기타 입력 허용/불허 혼선 발생 가능
* **RSA 키 미생성/도메인 미설정**으로 SSH가 동작하지 않는 케이스

  * `show ip ssh` 로 SSH 상태 확인
  * `crypto key generate rsa` 미실행 시 SSH 세션 수립 실패 가능 

### – Admin PC만 허용했는데도 다른 PC에서 접속이 됨

* `access-class 10 in`이 **VTY에 적용되지 않았거나** 다른 라인에 적용된 경우

  * `show run | section line vty`로 실제 적용 여부 확인 
* ACL이 너무 넓게 잡힌 경우(예: permit 192.168.50.0/29)

  * `show access-lists 10`으로 ACL 정확성 점검

### – Admin PC(192.168.50.5)에서만 “특정 HQ만” 접속 불가

* 해당 HQ 라우터가 **서버 VLAN(192.168.50.0/29)으로의 리턴 경로**를 정상 학습 중인지 확인

  * `show ip route 192.168.50.0`
  * OSPF 인접/라우팅 누락 시 “접속 시도는 들어오지만 응답이 안 나가” 세션이 안 잡히는 형태 발생 가능


### – ACL과 기존 LAB5 정책 충돌 우려

* 본 LAB은 VTY 접근 통제라서, 일반적인 VLAN 간 ACL과 충돌 가능성은 낮다.
* 다만, “관리 PC가 HQ까지 도달하는 경로” 자체가 LAB5 ACL에 의해 막힐 수 있으므로,
  접속 실패 시에는 **(1) 경로/라우팅 (2) VLAN ACL hit** 순으로 점검한다.

---

## 검증 요소

### 5.1 Admin PC에서 SSH 접속 성공

1. Admin PC(192.168.50.5)에서 HQ1/HQ2로 SSH 시도

   * `ssh -l <user> <HQ1 IP>`
   * `ssh -l <user> <HQ2 IP>`
2. 라우터에서 세션 확인

   * `show ssh`
   * `show users`

**성공 기준**

* HQ1/HQ2 모두 SSH 세션이 수립되고, 라우터에서 접속 사용자/라인이 확인된다.

### 5.2 비인가 단말에서 SSH 접속 차단

* HQ_vlan10/20/30 등 임의 PC에서 HQ1/HQ2로 SSH 시도
* 라우터 ACL hit 확인

  * `show access-lists 10`

**성공 기준**

* 비인가 단말은 SSH 세션이 성립하지 않으며,
* ACL 10이 “Admin host만 permit” 형태로 유지되고(deny는 암묵적), 비인가 시도는 차단된다.


---

## 설계 판단 및 한계

* 본 LAB은 “장비 관리 접근 통제”를 **VTY access-class 기반으로 단순·명확하게 구현**하였다.
* 실무 환경에서는 다음을 추가 고려할 수 있다.

  * 로컬 계정/AAA(TACACS+/RADIUS) 연동
  * 소스 IP 고정(Loopback 기반)


---

