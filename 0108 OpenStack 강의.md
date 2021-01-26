##  01/08 OpenStack 강의



```
1Day

Cloud Computing 이해(On-premise vs. Cloud)
-사용자 요청에 따라 공유된 자원이나 데이타를 인터넷 기반으로 제공하는 기술로
여러 유형의 서비스를 사용한 만큼 지불하는 종량과금제로 제공되는 computing

클라우드 서비스 유형
-IaaS/PaaS/SaaS

클라우드 배치 유형
-Public/Private/hybrid/community Cloud

오픈스택이란?

https://wiki.openstack.org/wiki/ReleaseNotes/Juno/ko
Core Service 이해
-compute
-image
-object storage
-block storage
-network
-dashboard
-Identity
-orchestration

Day2 (13:00~ )
----------------------------------------------------------
1.컨트롤러 준비작업
   os update,/etc/hosts,ntp server 구축,centos 최적화(filrewalld/NetworkManager/SELinux),repository 추가

2.오픈스택 설치(packstack on centos)
vi /etc/chrony.conf
----------------------------------------------------
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server 2.kr.pool.ntp.org iburst
server 127.127.1.0 

allow 10.0.0.0/24
-----------------------------------------------------
vi openstack.txt
-----------------------------------------------------
326 CONFIG_KEYSTONE_ADMIN_PW=abc123
1185 CONFIG_PROVISION_DEMO=n
11 CONFIG_DEFAULT_PASSWORD=abc123
46 CONFIG_CEILOMETER_INSTALL=n
 50 CONFIG_AODH_INSTALL=n
873 CONFIG_NEUTRON_OVS_BRIDGE_IFACES=br-ex:ens33
----------------------------------------------------------------------------

3.packstack을 이용한 all-in-one 구성

4.오픈스택 서비스 사용하기

Horizon 접속
Horizon 메뉴
Openstack 용어 정의
프로젝트/사용자 /Flavor 생성 

--------------------------------------------------------------------------
네트워크/라우터
Floating IP용: ext1->subext1->10.0.0.0/24,gw: 10.0.0.2, dns:10.0.0.2,dhcp X, 사용 IP pool(10.0.0.210,10.0.0.220),외부네트워크
Fixed IP 용: int1->subint1->192.168.0.0/24,gw:192.168.0.254,dns:10.0.0.2,dhcp 활성화)
router1 생성
외부 네트워크과 router간 연결: 게이트웨이 설정
내부 네트워크와 router간 연결: 인터페이스 추가
--------------------------------------------------------------------------

보안그룹/Floating IP 생성
Keypair 생성
이미지
인스턴스 시작
볼륨/vm snapshot 생성
Swift 사용하기
manual 설치(https://docs.openstack.org/install-guide/)
------------------------------------------------------
Overview
Example architecture
Networking
Environment
  Security
  Host networking
  Network Time Protocol (NTP)
  OpenStack packages
  SQL database
  Message queue
  Memcached

5.OpenStack CLI로 관리하기 
Identity 서비스 (Keystone)
                 -서비스 특징 
                 -서비스 구조
                 -CLI로 관리

https://docs.openstack.org/keystone/rocky/install/index-rdo.html
```