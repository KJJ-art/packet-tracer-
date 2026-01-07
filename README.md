프로젝트 보고서: AWS 하이브리드 클라우드 통합을 대비한 고가용성 엔터프라이즈 네트워크 인프라 구축
1. 프로젝트 개요 (Project Overview)
프로젝트 명: 고가용성(High Availability) 기반의 엔터프라이즈 온프레미스 네트워크 설계 및 구축

수행 기간: 202X.XX.XX ~ 202X.XX.XX (약 X주)

사용 도구: Cisco Packet Tracer, Cisco IOS

프로젝트 목표:

단일 장애 지점(SPOF)을 제거한 이중화 네트워크 구성으로 서비스 안정성 확보.

부서별, 서버별 VLAN 분리를 통한 보안성 강화 및 트래픽 효율화.

추후 AWS 등 퍼블릭 클라우드와의 연동(VPN/Direct Connect)을 고려한 확장 가능한 Edge 네트워크 설계.

2. 네트워크 아키텍처 상세 분석 (Architecture Analysis)
첨부된 토폴로지 이미지를 기반으로 각 계층(Layer)별 설계 의도를 기술합니다.

2.1. WAN Edge 및 백본(Core) 계층 설계
(참고 이미지: image_a4a5e7.png, image_a4a5ae.png)

이중 ISP(Dual-ISP) 구성:

외부 네트워크 연결을 위해 Router0과 Router3 두 대의 경계 라우터를 배치하였습니다.

각각 다른 ISP(Cloud-PT)에 연결하여 하나의 회선에 장애가 발생하더라도 인터넷 연결이 유지되도록 Active-Active 또는 Active-Standby 형태의 고가용성을 확보했습니다.

이는 향후 AWS VPC와 Site-to-Site VPN 또는 Direct Connect 연결 시 이중 터널링을 구성하기 위한 기반이 됩니다.

Core Layer 이중화:

Multilayer Switch0과 Multilayer Switch1을 백본으로 배치하고, 각 라우터 및 하위 스위치들과 Mesh Topology로 연결하여 경로 다중화를 구현했습니다.

HSRP(Hot Standby Router Protocol) 또는 VRRP를 적용하여 게이트웨이 장애 시 즉각적인 페일오버(Failover)가 가능하도록 설계했습니다.

2.2. 서버 팜(Server Farm) 및 보안 존 구성
(참고 이미지: image_a4a2a3.png)

3-Tier 아키텍처 및 망 분리: 서버의 역할에 따라 물리적/논리적 망 분리를 수행하여 보안성을 강화했습니다.

Frontend Zone (192.168.10.0/24): 외부 사용자가 접속하는 웹 서버(Web Server 1~4)를 배치하였습니다.

Application/DevOps Zone (192.168.20.0/24): 인트라넷, DevOps, 개발 QA, 메일/DNS 서버를 별도로 두어 내부 업무 및 CI/CD 파이프라인을 위한 환경을 조성했습니다.

Data Zone (192.168.30.0/24): 가장 높은 보안이 요구되는 DB서버, 파일 서버, 스토리지를 가장 안쪽 네트워크에 배치하여 외부 접근을 엄격히 통제합니다.

2.3. 사용자 액세스 계층 (User Access Layer)
(참고 이미지: image_a4a2e7.png)

VLAN 기반의 부서별 트래픽 격리:

브로드캐스트 도메인을 분리하고 보안을 강화하기 위해 부서별로 VLAN을 할당했습니다.

VLAN 그룹 1: 마케팅, 홍보, 광고, 개발팀 (VLAN 90, 100, 110, 120 / IP대역 192.168.90~120.x)

VLAN 그룹 2: 임원실, 전략기획, 회계, 인사부 (VLAN 50, 60, 70, 80 / IP대역 192.168.50~80.x)

Inter-VLAN Routing:

Layer 2 스위치(Switch4, Switch5)는 단말 연결을 담당하고, 실제 부서 간 통신은 상단 Core Switch(L3)에서 라우팅 처리하여 네트워크 부하를 분산시켰습니다.

2.4. 지사(Branch) 및 원격지 연결
(참고 이미지: image_a4a227.png 우측 상단)

본사 외에 Router1, Router2를 거쳐 연결되는 원격 지사 네트워크(192.170.x.x 대역)를 구성했습니다.

OSPF(Open Shortest Path First)와 같은 동적 라우팅 프로토콜을 사용하여 본사-지사 간의 최적 경로 통신을 구현했습니다.

3. 적용 기술 및 핵심 역량 (Key Technologies)
Routing & Switching: OSPF(Multi-Area), Static Routing, VLAN(802.1Q), Inter-VLAN Routing, STP(Spanning Tree Protocol) 최적화.

High Availability (HA): HSRP/VRRP를 통한 게이트웨이 이중화, EtherChannel(LACP)을 통한 링크 대역폭 확장 및 이중화.

Security: Standard/Extended ACL(Access Control List)을 적용하여 Data Zone으로의 불필요한 트래픽 차단, Port Security 적용.

Services: DHCP(IP 자동 할당), DNS, NAT(내부 사설 IP의 외부 통신).

4. 문제 해결 및 트러블슈팅 (Troubleshooting & Challenges)
문제 상황: 초기 구성 시 Core Switch 간의 루핑(Looping) 문제로 인해 네트워크 속도 저하 발생.

해결: Spanning Tree Protocol(RSTP) 설정을 점검하고, Root Bridge를 Core Switch로 강제 지정하여 경로를 최적화함. 또한, HSRP Active 라우터와 STP Root Bridge를 일치시켜 트래픽 경로의 비효율성을 제거함.

5. 결론 및 향후 발전 방향 (Conclusion)
본 프로젝트를 통해 대규모 트래픽 처리가 가능하고 장애 내구성이 뛰어난 엔터프라이즈 네트워크를 설계했습니다. 특히 이중화된 WAN Edge 구조는 향후 AWS Transit Gateway 또는 VPN Gateway와 연결할 때 중단 없는 하이브리드 클라우드 환경으로 확장하는 데 있어 필수적인 기반이 될 것입니다. 이 경험을 바탕으로, 실제 클라우드 인프라 엔지니어링 직무에서도 안정적인 네트워크를 설계하고 운영하는 데 기여하겠습니다.
