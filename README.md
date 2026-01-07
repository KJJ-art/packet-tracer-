# AWS Hybrid Cloud Readiness Enterprise Network
> **고가용성(High Availability) 및 망 분리(Security)를 고려한 엔터프라이즈 네트워크 인프라 구축 프로젝트**
---

## 1. 프로젝트 개요
본 프로젝트는 단일 장애 지점(SPOF)을 제거하고, 향후 **AWS 하이브리드 클라우드** 연동을 대비하여 **이중화(Redundancy)**와 **보안(Security)**이 강화된 기업형 네트워크를 설계 및 구축한 시뮬레이션입니다.

- **수행 기간:** 2026.1.2 ~ 2026.1.7
- **사용 툴:** Cisco Packet Tracer 8.x
- **핵심 목표:** `Dual-ISP 구축`, `HSRP 게이트웨이 이중화`, `VLAN 기반 망 분리`, `3-Tier 보안 존`

<br>

## 2. 네트워크 아키텍처

### 2.1 전체 토폴로지 (Full Topology)
> **설계 의도:** Core Layer와 Distribution Layer의 명확한 구분 및 이중화 링크 구성

![Server Farm](server_farm.png)

### 2.2 계층별 상세 설계

#### WAN Edge & Core Layer
- **Dual-ISP 구성:** `Router0`과 `Router3`를 통해 서로 다른 ISP로 연결, Active-Active/Standby 구성을 통해 인터넷 가용성 확보.
- **Backbone 이중화:** L3 Switch 간 Mesh 연결 및 EtherChannel(LACP) 적용으로 대역폭 확장 및 경로 다중화.

#### Server Farm & Security Zone
![Server Farm](./images/server_farm.png)

보안 등급에 따라 서버 구역을 3단계로 물리적/논리적으로 분리하였습니다.
1.  **DMZ/Frontend (192.168.10.0/24):** 외부 접근이 필요한 웹 서버
2.  **Application (192.168.20.0/24):** 내부 업무, DevOps, DNS/Mail
3.  **Data/Backend (192.168.30.0/24):** DB, 스토리지 (가장 높은 보안 수준, ACL 적용)

#### User Access Layer (VLAN)
![Access Layer](./images/user_access.png)

부서별 업무 특성에 맞춰 VLAN을 분리하여 브로드캐스트 트래픽을 제어하고 보안을 강화했습니다.

| VLAN ID | 부서명 (Group) | IP Subnet | 비고 |
| :---: | :--- | :--- | :--- |
| **90~120** | 개발, 마케팅, 홍보 | `192.168.90.0/24` ~ | 실무 부서 |
| **50~80** | 임원, 기획, 인사, 회계 | `192.168.50.0/24` ~ | 관리/보안 부서 |

<br>

## 3. 핵심 적용 기술 (Key Technologies)

* **Routing Protocols:**
    * `OSPF (Multi-Area)`: 본사 및 지사 간 효율적인 라우팅 정보 교환
    * `Static Routing`: ISP 연결 및 특정 경로 고정
* **High Availability:**
    * `HSRP/VRRP`: 게이트웨이 장애 시 1초 이내 Failover
    * `STP (RSTP)`: L2 루프 방지 및 수렴 속도 개선
* **Security:**
    * `Standard/Extended ACL`: DB 존으로의 비인가 접근 차단
    * `Port Security`: 스위치 포트별 MAC 주소 제한

<br>

## 4. 트러블슈팅
### Issue: 비대칭 라우팅 및 루핑 발생
* **현상:** Core Switch 간 링크 연결 시 트래픽이 한쪽으로 쏠리거나 STP Block 포트로 인해 통신 지연 발생.
* **해결:**
    1.  STP Root Bridge를 `Multilayer Switch0` (Active)으로 강제 지정.
    2.  HSRP Active 라우터와 STP Root Bridge를 일치시켜 경로 최적화 수행.

<br>

## 5. 결론
본 프로젝트를 통해 엔터프라이즈급 네트워크의 핵심인 **가용성**과 **보안성**을 확보하는 구조를 설계했습니다. 특히 이중화된 WAN Edge는 향후 AWS VPC와의 **Site-to-Site VPN** 또는 **Direct Connect** 연결 시 중단 없는 하이브리드 클라우드 환경을 제공하는 기반이 됩니다.

