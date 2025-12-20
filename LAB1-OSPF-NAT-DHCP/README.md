LAB1-OSPF-NAT-DHCP

1. 목표
HQ와 Branch로 나뉘어진 환경에서 DHCP 서버를 통해 IP를 부여받고, 공인 주소의 절약을 위해 외부 인터넷과는 PAT를 이용한다.
또한 Branch 내부의 서버는 static NAT로 외부에 공개한다.

2. 사용할 기술
- OSPF
- NAT
- DHCP

3. 고려할 점
- HQ와 ISP를 연결하는 G0/2에 PAT를 설정하되, Branch 내부서버는 별도의 static NAT 주소를 설정한다. (내부 서버의 변환될 ip는 203.1.113.1 로 한다)

- ISP와 인터넷에 연결되는 인터페이스엔 내부 정보를 전달하지 않기위해, ospf passive-interface를 설정한다.
- 또한 DHCP 서버와 연결된 인터페이스에도 마찬가지로 passive-interface를 설정한다.
- 내부와 인터넷을 연결하기 위해, default route를 전파한다.
  
- DHCP 서버는, HQ에 192.168.1.0/24 주소를, Branch엔 192.168.2.0/24 주소를 부여하되, 각기 게이트웨이는 x.1로, 제외하는 주소는 0 - 10까지로 구성한다.
- HQ,Branch와 DHCP서버의 네트워크가 다르기 때문에, HQ와 Branch에서 각기 ip-helper address를 DHCP 서버로 설정한다.


4. 검증할 점
- OSPF 및 DHCP 검증을 위해 HQ 내부 와 Branch 내부 PC간 PING 연결 및 내부 pc들의 Internet(8.8.8.8) 연결
- ISP의 G0/0과 G0/1이 passive-interface 인지
- PAT 변환이 HQ의 G0/2 주소로 되는지
- Branch 내부 서버가 제대로 static 변환이 되는지 


5. 트러블슈팅
