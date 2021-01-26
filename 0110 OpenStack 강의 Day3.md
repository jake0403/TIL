## 01/10 OpenStack 강의 Day3

```
Day3
--------------------------------------------------------------------------
보안그룹/Floating IP 생성
Keypair 생성
이미지

http://download.cirros-cloud.net/
인스턴스 시작

yum install -y openstack-utils

인스턴스 오류시
#openstack-status

neutron-openvswitch-agent  inactive  ---> systemctl start neutron-openvswitch-agent
 

볼륨 생성 후 연결하여 사용
-----------------------------------
lsblk
sudo sh
fdisk /dev/vdb  (partition 생성)
n->기본으로
lsblk
mkfs -t ext4 /dev/vdb1
mkdir /app
mount /dev/vdb1 /app
df -h
cp /etc/p* /app
ls /app
-----------------------------------
 
[root@controller ~]# ip netns
qrouter-8caebfa9-19f5-441c-83c2-19e342ec7978 (id: 1)
qdhcp-597b7e6c-866c-4a67-b485-e714fd1d050b (id: 0)
[root@controller ~]# ip netns exec qrouter-8caebfa9-19f5-441c-83c2-19e342ec7978 /bin/sh
sh-4.2# 
sh-4.2# ip a
sh-4.2# ssh cirros@10.0.0.212
The authenticity of host '10.0.0.212 (10.0.0.212)' can't be established.
RSA key fingerprint is SHA256:TfJccBH9eBIv0GnCcvxVrulw8cpTl+a3zUAyHrheHUc.
RSA key fingerprint is MD5:90:af:dd:93:fe:bb:e6:fe:5a:c4:0e:da:d5:63:9a:d6.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.0.0.212' (RSA) to the list of known hosts.
cirros@10.0.0.212's password:  cubswin:)
$ 
$ lsblk
NAME   MAJ:MIN RM    SIZE RO TYPE MOUNTPOINT
vda    253:0    0      1G  0 disk 
`-vda1 253:1    0 1011.9M  0 part /
vdb    253:16   0      1G  0 disk 
`-vdb1 253:17   0   1023M  0 part /app
$

/vm snapshot 생성
Swift 사용하기

5.OpenStack CLI로 관리하기 
Identity 서비스 (Keystone)
                 -서비스 특징 
                 -서비스 구조
                 -CLI로 관리
#cp keystonerc_admin keystonerc_stack1
# vi keystonerc_stack1 
unset OS_SERVICE_TOKEN
    export OS_USERNAME=stack1
    export OS_PASSWORD='abc123'
    export OS_REGION_NAME=RegionOne
    export OS_AUTH_URL=http://10.0.0.100:5000/v3
    export PS1='[\u@\h \W(keystone_stack1)]\$ '
    
export OS_PROJECT_NAME=pro1
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_IDENTITY_API_VERSION=3

#source keystonerc_stack1
[root@controller ~(keystone_stack1)]#openstack token issue

-----------------------------------------------------------


Day4
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

```
Day4
manual 설치(https://docs.openstack.org/install-guide/)

1.[root@controller ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens33
#UUID=
IPADDR="10.0.0.11"
2.#systemctl restart network
3.#ip a
4.[root@controller ~]# vi /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.0.0.11 controller
10.0.0.31 compute1

#vi /etc/chrony.conf
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server 10.0.0.100 iburst

    8  date
    9  systemctl status chronyd
   10  systemctl restart chronyd
   11  chronyc sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? 10.0.0.100                    0   6     0     -     +0ns[   +0ns] +/-    0ns
^* ec2-13-209-84-50.ap-nort>     2   6    17     1    -53us[ -471us] +/- 2661us
^- dadns.cdnetworks.co.kr        2   6    17     4   -112us[ -530us] +/-   43ms

------------------------------------------------------
Overview
Example architecture
Networking
Environment
  Security
  Host networking
  Network Time Protocol (NTP)
  OpenStack packages
-------------------------------
#yum install python-openstackclient -y
#yum install openstack-selinux -y

  SQL database
------------------------------
#yum install mariadb mariadb-server python2-PyMySQL -y
#vi /etc/my.cnf.d/openstack.cnf
[mysqld]
bind-address = 10.0.0.11

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
-------------------------------------------
#systemctl enable mariadb.service
# systemctl start mariadb.service
# systemctl status mariadb.service
#mysql_secure_installation

all--> Y로 진행

 Message queue
#yum install rabbitmq-server -y
systemctl enable rabbitmq-server.service
systemctl start rabbitmq-server.service
rabbitmqctl add_user openstack RABBIT_PASS
rabbitmqctl set_permissions openstack ".*" ".*" ".*"

  Memcached
yum install memcached python-memcached -y
vi /etc/sysconfig/memcached
OPTIONS="-l 127.0.0.1,::1,controller"
systemctl enable memcached.service
# systemctl start memcached.service

https://docs.openstack.org/keystone/rocky/install/index-rdo.html
Keystone
https://docs.openstack.org/keystone/rocky/install/keystone-install-rdo.html
---------------------------------------------------------------------------------------
[root@controller ~]# mysql -uroot -pabc123
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 11
Server version: 10.1.20-MariaDB MariaDB Server

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE keystone;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
    -> IDENTIFIED BY 'KEYSTONE_DBPASS';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
    -> IDENTIFIED BY 'KEYSTONE_DBPASS';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye

 yum install openstack-keystone httpd mod_wsgi -y
vi /etc/keystone/keystone.conf
742 connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
2829 provider = fernet

su -s /bin/sh -c "keystone-manage db_sync" keystone
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
  --bootstrap-admin-url http://controller:5000/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne

vi /etc/httpd/conf/httpd.conf
96 ServerName controller

ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
systemctl enable httpd.service
systemctl start httpd.service

export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3

openstack project create --domain default \
  --description "Service Project" service
openstack project create --domain default \
  --description "Demo Project" myproject
openstack user create --domain default \
  --password abc123 myuser
openstack role create myrole
openstack role add --project myproject --user myuser myrole

[root@controller ~(keystone_admin)]# vi admin-openrc 
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export PS1='[\u@\h \W(keystone_admin)]\$ '

[root@controller ~(keystone_admin)]# vi demo-openrc 
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=myproject
export OS_USERNAME=myuser
export OS_PASSWORD=abc123
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export PS1='[\u@\h \W(keystone_myuser)]\$ '

--------------------------------------------------------------
Glance
https://docs.openstack.org/glance/rocky/install/

# mysql -u root -pabc123
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 23
Server version: 10.1.20-MariaDB MariaDB Server

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE glance;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
    ->   IDENTIFIED BY 'GLANCE_DBPASS';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
    ->   IDENTIFIED BY 'GLANCE_DBPASS';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye

openstack user create --domain default --password GLANCE_PASS glance
openstack user set --domain default --password GLANCE_PASS glance
openstack role add --project service --user glance admin


openstack service create --name glance   --description "OpenStack Image" image
 
openstack endpoint create --region RegionOne   image public http://controller:9292
openstack endpoint create --region RegionOne   image internal http://controller:9292
openstack endpoint create --region RegionOne   image admin http://controller:9292
----------------------------------------
vi /etc/glance/glance-api.conf
   1901 connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
   3475 [keystone_authtoken]
   3476 www_authenticate_uri  = http://controller:5000
   3477 auth_url = http://controller:5000
   3478 memcached_servers = controller:11211
   3479 auth_type = password
   3480 project_domain_name = Default
   3481 user_domain_name = Default
   3482 project_name = service
   3483 username = glance
   3484 password = GLANCE_PASS

4424 flavor = keystone
2007 [glance_store]
   2008 stores = file,http
   2009 default_store = file
   2010 filesystem_store_datadir = /var/lib/glance/images/
--------------------------------------------------------------------

vi /etc/glance/glance-registry.conf 

   1147 connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance

  1254 [keystone_authtoken]
   1255 www_authenticate_uri = http://controller:5000
   1256 auth_url = http://controller:5000
   1257 memcached_servers = controller:11211
   1258 auth_type = password
   1259 project_domain_name = Default
   1260 user_domain_name = Default
   1261 project_name = service
   1262 username = glance
   1263 password = GLANCE_PASS

 2153 [paste_deploy]
   2154 flavor = keystone
su -s /bin/sh -c "glance-manage db_sync" glance
 

# systemctl enable openstack-glance-api.service \
  openstack-glance-registry.service
# systemctl start openstack-glance-api.service \
  openstack-glance-registry.service

yum install -y openstack-utils

[root@controller ~(keystone_admin)]# openstack-status
== Glance services ==
openstack-glance-api:                   active
openstack-glance-registry:              active
== Keystone service ==

187p
--------------------------
 yum install -y wget
http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
   33  yum install -y wget
   34  wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
   35  ls
   36  file cirros-0.3.5-x86_64-disk.img 
   37  openstack image create "cirros" --file cirros-0.3.5-x86_64-disk.img --disk-format qcow2 --container-format bare --public
   38  openstack image list
   39  ls /var/lib/glance/images/
   40  ls -l /var/lib/glance/images/
   41  glance image-show 6a4fa563-2c55-4baf-9709-6f2f070460bf

all-in-one
   86  scp 10.0.0.11:/root/ci* .
   87  qemu-img info cirros-0.3.5-x86_64-disk.img 
   88  qemu-img convert -O vmdk cirros-0.3.5-x86_64-disk.img cirros-0.3.5-x86_64-disk.vmdk 
   89  ls -l ci*
```