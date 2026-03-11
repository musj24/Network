

# 부팅 및 초기화

실제 L2 스위치 (WS-C2960L-16TS-LL)로 Putty 또는 Tere Term 을 이용한 장비 구성

## 상황
- 중고로 구입한 스위치에 enable password가 걸려있어서 공장 초기화를 수행

## 수행 과정
- 부팅중 부팅중 Mode 스위치를 눌러 복구 모드 진입
- dir flash: 명령어로 파일 목록 확인
- delete flash:config.text 로 설정파일 제거
- delete flash:vlan.dat 로 기존 vlan 파일 제거
- 부팅 완료


