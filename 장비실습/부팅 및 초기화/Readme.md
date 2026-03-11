
## 상황
- 중고로 구입한 스위치에 enable password가 걸려있어서 공장 초기화를 수행

## 수행 과정
- 부팅중 기기 전면부 Mode 스위치를 눌러 복구 모드 진입
- dir flash: 명령어로 파일 목록 확인
- delete flash:config.text 로 설정파일 제거
- delete flash:vlan.dat 로 기존 vlan 파일 제거
- boot 명령어 수행
- 부팅 완료


