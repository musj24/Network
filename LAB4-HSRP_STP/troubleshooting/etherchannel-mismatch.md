## EtherChannel Formation Issue

### 증상
- Port-channel suspended 상태
- EC-5-CANNOT_BUNDLE2 에러 발생

### 원인
- Physical interface와 Port-channel 간
  trunk allowed VLAN 불일치

### 해결방안
- Port-channel 인터페이스 우선 구성
- 이후 member interface를 channel-group에 추가
