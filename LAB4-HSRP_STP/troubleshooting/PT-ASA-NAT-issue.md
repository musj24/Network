## Packet Tracer ASA NAT Issue

### 증상
- HQ에서 발생한 트래픽은 NAT 정상 동작
- Distribution Switch 및 사용자 호스트에서 발생한 트래픽은
  ASA에서 NAT/xlate 생성되지 않음

### 원인
- Packet Tracer ASA는 inside 인터페이스에 직접 연결된 대역의
  소스 IP만 정상적인 inside 트래픽으로 인식함
- 사용자 VLAN(192.168.x.x) 트래픽은 NAT 이전 단계에서 drop

### 해결방안
- HQ 라우터에서 1차 PAT 수행
- 내부 트래픽의 소스를 ASA inside 대역(192.168.60.x)으로 변환
- 이후 ASA NAT 정상 동작 확인

### Note
- 실제 ASA 환경에서는 발생하지 않는 문제
- 시뮬레이터 환경 제약으로 인한 우회 설계
