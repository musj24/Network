LAB2-Vlan_interRouting

목표: ROAS와 L3 스위치를 이용해 vlan이 나뉘어져 있는 내부 pc들간의 통신을 가능하게 한다.

사용할 기술

OSPF
NAT
DHCP
ROAS


고려할 점
1. 구역은 HQ와 Branch로 나누되, 내부 PC들도 단일 네트워크가 아닌, 다중 네트워크로 분할한다.
2. 내부 PC들은 서로 통신 가능해야 하며, 외부 인터넷 (8.8.8.8)과 통신도 가능해야 한다.
3. Router와 SW간 링크는 트렁크로 구성해야 정상적인 동작이 가능함.
4. native vlan은 99로 설정한다.

검증할 점
1. 내부 PC간 통신이 가능한가?
2. PC와 외부 인터넷(8.8.8.8)이 통신 가능한가?
   

트러블슈팅
1. ROAS의 가상 인터페이스에 encapsulation dot1q가 빠진다면?

[증상]
- VLAN10 PC에서 Default Gateway로 ping 실패
- VLAN20/30 정상

[확인]
- show interfaces trunk
- show ip interface brief
- show run interface g0/0.10

[원인]
- Sub-interface에 dot1q encapsulation 설정 누락

[조치]
- encapsulation dot1q 10 추가

[결과]
- VLAN10 PC 정상 통신 확인

2. L3 스위치의 ip routing이 빠진다면?







* 추가 *
GNS3 기본 Ethernet switch를 사용해 ROAS 트렁크 환경을 구성할 경우,
DHCP ACK 패킷이 클라이언트까지 정상 전달되지 않는 현상을 확인하였다.
debug ip dhcp server packet 상에서는 DORA 과정이 모두 완료되었으나,
이는 GNS3 기본 스위치의 VLAN 태깅 처리 한계로 판단되어
HQ-SW를 L3스위치로 대체하여 수행했다.

* 추가 2 *
지금 사용가능한 GNS3의 스위치 이미지중에, 제대로 LAB을 구성할수 있는것이 없는것 같다.
그래서 PacketTracer로 구현한다.




