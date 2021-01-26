## OpenStack Day-5

### compute1으로 VM 만들기

* compute1 파일 새로 만들고 vmware 복사해서 가져온 후 VMware에서 새로운 virtual machine 생성

* VMware 에서 2GB로 생성하고  number of core => 2개로 설정

* 생성 후  vi 편집기로 ip 수정 

  ``` vi /etc/sysconfig/network-scripts/ifcfg-ens33
  # vi /etc/sysconfig/network-scripts/ifcfg-ens33
  ```

  ip를 10.0.0.101로 바꾸고 uuid 주석처리

  xshell로 새로만든 후 이름 compute1과 ip 10.0.0.101로 생성

  hostnamectl set-hostname compute1으로 이름 변경

##  Neutron

방화벽 베이스 2가지 

DNS(Domain name service)

* Virtual Networking ingrastructure(VNI) 가상 네트워크 환경을 위한 네트워크 측면의 모든 것을 관리하고 오픈스택 환경에서 Physical Networking Infrastructure(PNI) 물리 네트워크 환경의 측면에서 레이어에 접속하는 것을 관리한다.
* 오픈스택 네트워킹은 방화벽, 로드밸렌서, 그리고 가상 사설 네트워크 등의 서비스들을 포함한 고급 가상 네트워크 토폴리지들을 tenanats가 생성 하도록 한다.

![image-20200114112825372](C:\Users\HPE\AppData\Roaming\Typora\typora-user-images\image-20200114112825372.png)

#### router 정보 확인

``` # ip netns
# ip netns
```

### ML2 (Modular Layer2)

* TypeDriver
* MechanismDriver

물리적인 망과 다이렉트로 연결하는 방식 Flat 방식

![image-20200114112745076](C:\Users\HPE\AppData\Roaming\Typora\typora-user-images\image-20200114112745076.png)

tenant Network => 내부 네트워크



Provider Network => 공인 ip를 바로 subnet으로 받아서 물리적으로 direct 통신

-----



![image-20200114113156498](C:\Users\HPE\AppData\Roaming\Typora\typora-user-images\image-20200114113156498.png)

alive 상태가 xxx로 되면 프로세스는 작동하나 서비스가 제공이 안되고 있다는 뜻으로 time 동기화 설정을 해줘야한다.



----

#### dvr( distributed virtual router)

외부 네트워크 브릿지 => br-ex

dvr 형태로 구성하게 되면 컴퓨트 노드가 여러개가 되었을 때 분배되어서 제공(br-ex를 타고 나가지 않고 로컬 내의 br-ex를 타고 통신)



linuxbridge-agent Vs opencv-bridge

linuxbridge를 사용하게 되면 컨트롤러를 타고 나갈 필요 없이 물리적 네트워크 통신으로 로컬내에서 바로 통신할 수 있게 된다.





* Ip  : Fixed ip => 사설 ip,    Floating ip => router(외부통신용, 공인 ip)



floating ip로 요청이 들어오면 DNAT(Destination)를 타고 floating ip를 fixed ip 로 변경 후 인스턴스로 들어가게 됨.

나갈 때는 SNAT 테이블을 타고 SIP가 floating ip로 변경 되어 나가게 된다.



Ceph (블록기반=>RBD, object기반, file 기반=>CephFS  스토리지)  Vs. chef(자동화)





iscsi protocol 기반의 LVM을 사용하는 것이 Cinder 다.

* iSCSI(Internet Small Computer System Interface)



iSCSI Initiator = NOVA

iSCSI Targer = Cinder

```
Day5
1.Nova 추가
2.Neutron
3.CLI로 Instance 시작
4.Cinder
5.Swift
6.Openstack High Availability Architecture
----------------------------------------------
compute1 추가하기
https://docs.openstack.org/nova/rocky/install/compute-install.html
https://docs.openstack.org/nova/rocky/install/compute-install-rdo.html
yum install openstack-nova-compute -y

   14  cp /etc/nova/nova.conf /etc/nova/nova.conf.old
   15  scp controller:/etc/nova/nova.conf /etc/nova
   16  ls -l /etc/nova/nova.conf
   17  vi /etc/nova/nova.conf
------------------------------------
  1254 my_ip=10.0.0.101
  11017 vncserver_proxyclient_address=10.0.0.101

systemctl enable libvirtd.service openstack-nova-compute.service
systemctl start libvirtd.service 
systemctl start  openstack-nova-compute.service (starting 오류) <--- libvirtd restart 필요할 수 있음
----------------------------------------------------------

controller(starting 오류 해결 )
------------------------------------------------------------------------------
vi /etc/sysconfig/iptables
13번 아래에 추가
-A INPUT -s 10.0.0.101/32 -p tcp -m multiport --dports 5671,5672 -m comment --comment "001 amqp incoming amqp_10.0.0.101" -j ACCEPT
-A INPUT -s 10.0.0.101/32 -p tcp -m multiport --dports 5671,5672 -j ACCEPT
-A INPUT -s 10.0.0.100/32 -p tcp -m multiport --dports 5671,5672 -j ACCEPT

systemctl reload iptables
-----------------------------------------------------------------------------------
[root@controller ~]# . keystonerc_admin 
[root@controller ~(keystone_admin)]# openstack compute service list --service nova-compute
+----+--------------+------------+------+---------+-------+----------------------------+
| ID | Binary       | Host       | Zone | Status  | State | Updated At                 |
+----+--------------+------------+------+---------+-------+----------------------------+
|  7 | nova-compute | controller | nova | enabled | up    | 2020-01-14T01:01:00.000000 |
|  8 | nova-compute | compute1   | nova | enabled | up    | 2020-01-14T01:01:05.000000 |
+----+--------------+------------+------+---------+-------+----------------------------+

vi /etc/nova/nova.conf
   9662 [scheduler]
   9663 discover_hosts_in_cells_interval = 300

[root@controller ~(keystone_admin)]# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'default': 8afd82d6-6a72-4fad-b9b3-d4377c9e806f
Checking host mapping for compute host 'compute1': 7b9f0c79-da34-4845-83e7-4ef0b54e0d50
Creating host mapping for compute host 'compute1': 7b9f0c79-da34-4845-83e7-4ef0b54e0d50
Found 1 unmapped computes in cell: 8afd82d6-6a72-4fad-b9b3-d4377c9e806f
--------------------------------------------------------------------------------------
Install and configure controller node
Prerequisites
Configure networking options
Configure the metadata agent
Configure the Compute service to use the Networking service
Finalize installation
-------------------------------------------------------------
Install and configure compute node
--------------------------------------------------
Install the components
yum install openstack-neutron-linuxbridge ebtables ipset -y

Configure the common component
----------------------------------------------
[root@compute1 ~]# cd /etc/neutron/
[root@compute1 neutron]# ls
conf.d  neutron.conf  plugins  rootwrap.conf
[root@compute1 neutron]# cp neutron.conf neutron.conf.old
[root@compute1 neutron]# scp controller:/etc/neutron/neutron.conf neutron.conf
root@controller's password: 
neutron.conf                                                         100%   71KB   4.8MB/s   00:00    
[root@compute1 neutron]# vi /etc/neutron/neutron.conf
761 #connection=mysql+pymysql://neutron:9e2064f267fd4602@10.0.0.100/neutron
:wq!

Configure networking options

[root@compute1 neutron]# vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini
    146 [linux_bridge]
    147 physical_interface_mappings = provider:ens33

   205 [vxlan]
    206 enable_vxlan = true
    207 local_ip = 10.0.0.101
    208 l2_population = true

    182 [securitygroup]
    183 enable_security_group = true
    184 firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
    185 
[root@compute1 neutron]# modprobe br_netfilter
[root@compute1 neutron]# lsmod|grep br_netfilter
br_netfilter           22256  0 
bridge                151336  1 br_netfilter
[root@compute1 neutron]# sysctl -a|grep nf-call
net.bridge.bridge-nf-call-arptables = 0
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1

Configure the Compute service to use the Networking service
-------------------------------------------------------------------------
[root@compute1 neutron]# systemctl enable neutron-linuxbridge-agent
Created symlink from /etc/systemd/system/multi-user.target.wants/neutron-linuxbridge-agent.service to /usr/lib/systemd/system/neutron-linuxbridge-agent.service.
[root@compute1 neutron]# systemctl start neutron-linuxbridge-agent.service
[root@compute1 neutron]# systemctl status neutron-linuxbridge-agent.service
yum install -y openstack-utils
openstack-status
[root@controller ~(keystone_admin)]# openstack network agent list
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host       | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| 1892bd72-a84a-4bc4-a037-1e2189cb4340 | Metadata agent     | controller | None              | :-)   | UP    | neutron-metadata-agent    |
| 32481bca-4d86-4b29-8c06-b158e6d0ac4e | L3 agent           | controller | nova              | :-)   | UP    | neutron-l3-agent          |
| 335a69a4-197e-4e27-bdfc-e8ff0df7f585 | Open vSwitch agent | controller | None              | :-)   | UP    | neutron-openvswitch-agent |
| 5b489ba0-1290-4e42-b69e-8355fc8716e7 | Metering agent     | controller | None              | :-)   | UP    | neutron-metering-agent    |
| 63333b1a-265b-4ab0-bc50-882fcf066f6c | Linux bridge agent | compute1   | None              | :-)   | UP    | neutron-linuxbridge-agent |
| fa6d6aa9-45bb-4741-9303-ea9e90efa099 | DHCP agent         | controller | nova              | :-)   | UP    | neutron-dhcp-agent        |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+--------
Finalize installation
Linux bridge agent 추가 오류시 
[root@compute1 neutron]# vi linuxbridge-agent.log 
[root@compute1 neutron]# getenforce
Enforcing
[root@compute1 neutron]# setenforce 0
[root@compute1 neutron]# systemctl restart neutron-linuxbridge-agent
[root@compute1 neutron]# vi linuxbridge-agent.log 
[root@compute1 neutron]# vi /etc/selinux/config 
SELINUX=disabled

https://docs.openstack.org/install-guide/launch-instance.html
-------------------------------------------------------------------
openstack project create demo
openstack user create --domain default \
  --project demo --password abc123 demo
openstack role add --project demo --user demo _member_

[root@controller ~(keystone_admin)]# vi keystonerc_demo 
---------------------------------------------------------------------
unset OS_SERVICE_TOKEN
    export OS_USERNAME=demo
    export OS_PASSWORD='abc123'
    export OS_REGION_NAME=RegionOne
    export OS_AUTH_URL=http://10.0.0.100:5000/v3
    export PS1='[\u@\h \W(keystone_demo)]\$ '
    
export OS_PROJECT_NAME=demo
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_IDENTITY_API_VERSION=3
------------------------------------------------------------------

Networking Option 2: Self-service networks
  217  . keystonerc_admin 
  227  openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 m1.nano
  228  openstack flavor list
  229  . keystonerc_demo 
  230  ls .ssh
  231  openstack keypair create --public-key ~/.ssh/id_rsa.pub mykey
  232  openstack keypair list
  233  openstack security group rule create --proto icmp default
  234  openstack security group rule create --proto tcp --dst-port 22 default
  218  openstack network create selfservice
  219  openstack subnet create --network selfservice   --dns-nameserver 8.8.4.4 --gateway 172.16.1.1   --subnet-range 172.16.1.0/24 selfservice
  220  openstack router create router
  221  openstack router add subnet router selfservice
 
  223  openstack router set router --external-gateway ext1
  224  . keystonerc_admin 
  225  openstack port list --router router

  235  openstack image list
  236  ls
yum install -y wget
 wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
  237   2 --file ./cirros-0.3.5-x86_64-disk.img 
  238  openstack image list
  239   openstack network list
penstack server create --flavor m1.nano --image cirros \
  --nic net-id=f86403ab-adfe-4447-af09-ba92a164dcb5 --security-group default \
  --key-name mykey selfservice-instance
# openstack server list

인스턴스 console 접속
--------------------------------------------------------------
 1. openstack console url show selfservice-instance (web기반 novnc protocol로 접속 가능)
 2. virsh list --all
    virsh console 1 ( disconnect는 ^] )
 ------------------------------------------------------------- 
  248  openstack floating ip create ext1
  249  openstack server add floating ip selfservice-instance 10.0.0.216
  255  ip netns exec qrouter-b716d035-eaee-4143-8489-c93dfb7241c7 ssh cirros@10.0.0.216

Block Storage (Cinder)
  258  vgs
  259  pvs
  260  losetup -a
  261  ls -l /var/lib/cinder/cinder-volumes
  262  lvs
  263  lsblk
  264  cinder create --name demo-v1 1
  265  cinder list
  270  nova volume-attach selfservice-instance a4a545cc-1ecf-4bc9-9d40-ab5124bd87ef auto
  271  lsblk

Object Storage Service(swift)
--------------------------------------
  274  swift post demo-c1
  275  swift upload demo-c1 cirros-0.3.5-x86_64-disk.img
  276  swift list demo-c1 --lh
  277  cd /var/tmp
  278  swift download demo-c1 
  279  ls -l
```