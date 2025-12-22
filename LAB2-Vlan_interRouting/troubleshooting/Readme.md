
트러블슈팅

1.ROAS의 가상 인터페이스에 encapsulation dot1q가 빠진다면?
[증상]

VLAN10 PC에서 Default Gateway로 ping 실패
VLAN20/30 정상
[확인]

show interfaces trunk
show ip interface brief
show run interface g0/0.10
[원인]

Sub-interface에 dot1q encapsulation 설정 누락
[조치]

encapsulation dot1q 10 추가
[결과]

VLAN10 PC 정상 통신 확인

***

2.L3 스위치의 ip routing이 빠진다면?
[증상]

Branch-SW 에서 라우팅 테이블이 올라오지 않는다.

[확인]

show ip route

[원인]

Branch-SW 설정에 ip routing 명령어 누락
[조치]

ip routing 명령어 추가
[결과]

라우팅 테이블이 올라오고, L3 스위치가 라우팅 기능 수행가능
***

3.Branch<->L3 스위치 연결간에, no switchport가 빠진다면?
[증상]

Branch-SW가 라우터와 마주보는 인터페이스에 ip 주소를 부여하지 못한다.
[확인]

ip address
show run
interface GigabitEthernet1/0/24
 no switchport

[원인]

interface g1/0/24에 no switchport 설정 누락 확인.

[조치]

interface g1/0/24에 no switchport 설정
[결과]

인터페이스에 주소 부여 성공 및 패킷전달 가능
