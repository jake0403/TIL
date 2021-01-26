##  01/09 OpenStack 강의 Day2



### 방화벽 보안정책 

#### openstack Security Group 생성

1. 블랙리스트 정책 (특정 포트 막아두는것)
2. 화이트리스트 정책(특정 포트만 허용하는것)



http => 80 port

https => 443 port

ssh => 22 port

ping => ICMP 

* 목적에 맞게 포트들을 열어주는 것이 필요함

------------------------------------------------------------------------------------------------------------------------------



#### CIDR 방식

CIDR(사이더) 기법이란, 기존의 IP 주소 할당방식인 Network Class 방식을 대체하기 위해 개발된 도메인간 라우팅 기법으로 최신 IP 할당 방법이다.

 

CIDR 기법은 사이더 블록이라 불리는 그룹으로 IP 주소들을 그루핑하고 그룹들을 계층적으로 관리함으로써 기존의 Network Class 방식에 비해 유연하게 동작할 수 있도록하며, IP 주소 체계를 보다 효율화한다.

(더 많은 IP 주소를 이용할 수 있도록 한다.)

또한, 계층적 구조이기 때문에 Routing 에 있어서 부담이 최소화되는 장점이 있다.

 

기존의 네트워크 클래스 방식의 기법은 IP 주소에 범위를 정하고 해당 범위대로 Class 를 분류한다.

가령, 나뉘는 클래스의 범위는 다음과 같아진다.

 

\- A Class : 0 ~ 127

\- B Class : 128 ~ 191

\- C Class : 192 ~ 223

\- D Class : 224 ~ 239

\- E Class : 240 ~ 245

 

위와 같은 방식에서, Class 의 범주에 딱맞지않는다면, 호스트 주소공간을 할당하기 위해 클래스를 선택할 때 문제가 발생한다. 즉, 더 높은 클래스를 선택함으로써 남은 주소 블록들을 낭비할것인지, 아니면 낮은 클래스를 사용해서 주소의 부족함을 감수할 것인지에 대한 문제가 생기게 된다.

 

CIDR 은 이에 비해 보다 유연성을 제공하고, 그 덕분에 더 많은 IP 주소 공간의 운용을 가능하도록 한다.

CIDR(사이더)을 이용한 주소 체계는 다음과 같이 지정될 수 있다.

 



![img](https://k.kakaocdn.net/dn/btO4aF/btqxjAhvKqU/docrx6sWDxLP6UXiM9Ydr0/img.png)



 

이는 해당 IP 주소의 첫 24비트를 네트워크 라우팅을 위한 주소로 사용한다는 의미이다.

 



![img](https://k.kakaocdn.net/dn/dpCCak/btqxjBtXunm/ufMLUNAA8HksujI6TDcPWK/img.png)



따라서 해당 주소의 표현은 위와 같이 이진수로 나타낼 수 있고, 네트워크 주소로 표현되는 부분과 네트워크 주소 뒤에 호스트 번호가 포함되게 된다. 

다음은 예시로 나타낸 주소에 대해 IP 주소 체계를 분석해본 것이다.

비트연산을 이용해 Subnet Mask 와 Broadcast Address 를 어렵지않게 구해낼 수 있고, 해당 정보를 바탕으로 Host 의 가용 범위 역시 구해낼 수 있다.



![img](https://k.kakaocdn.net/dn/LNMCJ/btqxhMprVSd/WfUdfWg3vfEYqAtfXddgr0/img.png)



즉, 위의 주소 192.168.0.15/24 의 의미는 192.168.0.1 부터 192.168.0.255 까지의 주소 범위를 의미하는 CIDR 표기법이라 할 수 있다. (앞의 24 bit가 Masking 이기 때문이다.)

이를 192.168.0.15/32 로 표기하면 이는 192.168.0.15 주소 그 자체를 의미한다.

 

CIDR 는 네트워크를 이해하는 데 있어 필수적인 기초개념이고, 생각나지않을 때마다 정리해보면 IP 주소 체계를 이해하는데 도움이 될 수 있다.



출처: https://jins-dev.tistory.com/entry/CIDR-사이더-기법에-대한-정리 [Jins' Dev Inside]

---

### Security Group 에 rule 추가

* 네트워크 보안그룹 생성 후 규칙 추가
  * 룰 (SSH) CIDR 방식 선택 후 CIDR는 0.0.0.0/0 (누구나 허용)
  * 룰 (HTTP) CIDR 방식 선택
  * 룰 (ICMP) =>ping test

### console

* nova web 기반
* CLI



### ssh 인증 방식 2가지

1. password 기반 인증 (default)

2. key 기반 인증 (비대칭 키 or 대칭 키) => 암호화와 복호화가 같으면 대칭 다르면 비대칭

   ssh-rsa(비대칭)

* Key 기반 인증 => Key pair 생성 = 암호화 키 생성

공개키는 서버(인스턴스)에 들어가있고 개인키는 ex)X-shell에 있음 그 두 개가 지문(fingerprint)를 통해서 일치하면 접속이 허용된다.



openstack- compute - 키 페어 - 키 페어 생성