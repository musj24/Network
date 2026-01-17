
---

# LAB6 – 운영 관점 네트워크 구성 (NTP / Syslog)

## 목표

본 LAB은 **LAB5까지 완성된 캠퍼스 네트워크 설계**를 기반으로,
단순한 구성 단계를 넘어 **운영 관점**에서 필수적인 요소를 추가·검증하는 것을 목표로 한다.

구체적인 목표는 다음과 같다.

* 중앙 NTP 서버 기반 시간 동기화 구성
* 중앙 Syslog 서버를 통한 로그 수집 구조 구현
* 네트워크 장애 및 이벤트 발생 시 운영 가시성 확보
* “구성하는 네트워크”에서 “운영 가능한 네트워크”로의 확장

---

## 사용 기술

* NTP (Centralized Time Synchronization)
* Syslog (Centralized Log Collection)
* Loopback Interface (운영 식별자 개념)
* ACL (기존 LAB5 정책 유지)


---

## 설계 및 구성 시 고려 사항

### 2.1 설계 전제

* **LAB5까지의 토폴로지 및 정책을 변경하지 않는다.**
* 라우팅(OSPF), 이중화(HSRP), 보안(ACL/Port-Security) 구성은 그대로 유지한다.
* 본 LAB에서는 **운영 가시성 확보와 관리 효율성**에 집중한다.
* Packet Tracer 환경 제약을 인지하고, 실장비 기준 설계 의도를 함께 고려한다.

---

### 2.2 시간 동기화 설계 (NTP)

#### 설계 목표

* 모든 네트워크 장비의 로그 타임스탬프를 일관되게 유지
* 장애 및 이벤트 분석 시 시간 기준 혼선 방지

#### 설계 방식

* 서버 VLAN에 **중앙 NTP 서버** 배치
* Distribution Switch, HQ Router, Access Switch 전반에 NTP 설정
* 각 장비에서 NTP 서버를 참조하도록 구성


NTP Server : 192.168.50.4


#### 설계 의도

* 로그 및 이벤트 분석 시 **시간 정합성 확보**
* 장비별 로컬 시간 편차 제거
* 운영 환경에서 필수적인 기본 구성 요소 반영

> Packet Tracer 환경에서는 `ntp source-interface` 명령어가 지원되지 않아서, 사용하지 않았다.
> 실제 장비 환경에서는 Loopback 인터페이스를 NTP source로 사용하는 것이 일반적이다.

---

### 2.3 운영 식별자 설계 (Loopback Interface)

#### 설계 목표

* 네트워크 장비를 논리적으로 식별 가능한 고정 주소 제공
* 운영 트래픽(NTP/Syslog)의 일관된 출발지 개념 정립

#### 설계 방식

* 각 주요 네트워크 장비(DSW/HQ)에 Loopback 인터페이스 구성
* Loopback 주소는 **관리 전용 대역**으로 분리
* 사용자/서버 트래픽과 논리적으로 분리된 주소 체계 유지

#### 설계 의도

* 물리 인터페이스 장애와 무관한 안정적인 식별자 제공
* 운영 트래픽과 사용자 트래픽의 역할 분리
* 실무 환경에서의 관리 주소 설계 개념 반영

---

### 2.4 로그 수집 설계 (Syslog)

#### 설계 목표

* 네트워크 이벤트를 중앙에서 수집 및 확인
* 장애 발생 시 원인 분석을 위한 로그 기반 확보

#### 설계 방식

* 서버 VLAN에 **중앙 Syslog 서버** 구성
* Distribution Switch, HQ Router, Access Switch 전반에서 Syslog 전송
* Syslog Severity는 Packet Tracer 지원 범위 내에서 설정


Syslog Server : 192.168.50.4


#### 설계 의도

* 분산된 장비 로그를 단일 지점에서 관리
* 링크 다운/업, 라우팅 변경 등 주요 이벤트 가시화
* 운영 관점에서의 네트워크 관리 흐름 반영

> Packet Tracer 환경에서는 Syslog severity 세분화 및
> sequence number 관련 IOS 명령어가 제한적으로 지원된다.
> 본 LAB에서는 Syslog 수집 동작 자체의 검증에 초점을 두었다.

---

## 트러블슈팅 포인트

### – NTP

* NTP 서버 IP 접근 가능 여부 확인
* NTP 서비스 활성화 여부 확인
* `show ntp status`를 통한 동기화 상태 확인
* Packet Tracer 환경에서 시간 표시 오차 발생 가능성 인지

---

### – Syslog

* Syslog 서버 서비스 활성화 여부 확인
* `logging host` 설정 확인
* 장비에서 로컬 로그 생성 여부(`show logging`) 확인
* Packet Tracer 환경에서 L2 Access Switch 이벤트가 서버로 전달되지 않는 제한 인지
* Distribution / HQ 장비 이벤트를 통한 Syslog 수집 검증

---

### – ACL 연계

* 서버 VLAN ACL이 Syslog/NTP 트래픽을 차단하지 않는지 확인
* 운영 트래픽과 사용자 트래픽 정책 간 충돌 여부 점검
* 기존 LAB5 ACL 정책과의 일관성 유지 여부 확인

---

## 검증 요소

* 모든 네트워크 장비에서 `show ntp status` 확인
* Syslog 서버에서 Distribution/HQ 장비 이벤트 로그 수신 확인
* 인터페이스 down/up 또는 OSPF 이벤트 발생 시 Syslog 로그 생성 여부 확인
* NTP 및 Syslog 구성 후 기존 트래픽 정책 정상 동작 여부 확인

---

## 설계 판단 및 한계

* 원래 Route Summarization을 수행하려 했으나, 단일 Area 0 기반 OSPF 구조이기 때문에 하지 않았다.
* 본 LAB에서는 운영 요소(NTP/Syslog)를 **기능 구현보다 설계 의도 중심**으로 반영하였다.
* Packet Tracer 환경 제약으로 일부 IOS 운영 명령어는 구현되지 않았으나,
  실장비 환경 기준의 설계 방향을 함께 고려하였다.
* 본 LAB을 통해 캠퍼스 네트워크 설계를 **운영 가능한 구조로 확장**하는 것을 목표로 하였다.

---


