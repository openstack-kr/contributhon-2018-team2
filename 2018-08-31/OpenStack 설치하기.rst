
===============================
2018-08-31 : OpenStack 설치하기
===============================

오늘은 선릉역 근처에 있는 공개 SW 개발자 센터에서 스터디를 진행하였습니다! :)

오늘 공부한 것들을 최대한 제가 이해한 방향으로 작성해보았습니다.

-----------------
1. Cafe24 Server
-----------------

지난 번에 신청한 Cafe24에서 서버를 받았다!!! (감사합니다..)

IP는 Cafe24로 로그인을 진행한 이후 ``나의 서비스 관리`` > ``서버 IP`` 에서 확인할 수 있고,

``서버 임시 접속 정보`` > ``정보 보기`` 를 누르면 서버에 접속하는 계정과 비밀번호를 알 수 있다.

**세팅일로부터 2주 후에는 확인이 안되니, 그 전에 정보를 잘 저장해 놓거나, 변경하면 된다!**

그럼 IP와 세팅된 임시 패스워드로 접속한다.

``$ ssh root@ip``

패스워드를 쳐서 접속을 한 이후 기존의 임시 패스워드를 변경하기 위해 아래의 명령어를 입력해준다.

``$ passwd root``

새로운 비밀번호로 서버에 접속하면 첫 번째 단계를 성공한 것이다.

아 그리고 혹시 저처럼 서버로 이것저것 해보다가 터트릴 수 있습니다. 조심하세요..

------------------------
2. VirtualBox & Vagrant
------------------------

CentOS 서버에서 vm을 설치하여 그 곳에서 devstack을 설치하려고 한다.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
2.1 VirtualBox & Vagrant 정의
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

설치하기 전에 우리가 설치하게될 것들이 무엇인지 간단하게 정의를 찾아보았다.

* VirtualBox

일반적으로 물리적인 시스템 OS 위에 논리적인 가상 OS 를 올려서 독립적인 동작이 가능하도록 하는 시스템 차원의 가상 머신

* Vagrant

가상 머신을 이용한 개발환경 설정을 자동화해주는 도구.

쉽게 만들고 쉽게 버릴 수 있으며 다시 그 상태를 쉽게 복원하는 "Code as a Infrastructure" 오픈소스 프로젝트.

즉 ``VirtualBox`` 를 이용하면, 손쉽게 VM환경을 구축할 수 있지만 번거로운 작업이기 때문에 이를 자동화 해주기 위해 개발된 것이 ``Vagrant`` 이다.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
2.2 VirtualBox & Vagrant 설치
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

이제 아래의 명령어들을 차례대로 입력하면서 설치를 진행하면 된다.

``$ wget -O /etc/yum.repos.d/virtualbox.repo http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo``

``$ yum update -y``

``$ yum install VirtualBox-5.2``

``$ rpm -ivh https://releases.hashicorp.com/vagrant/2.1.4/vagrant_2.1.4_x86_64.rpm``

만약 다운로드가 느리다면 ``$ wget https://releases.hashicorp.com/vagrant/2.1.4/vagrant_2.1.4_x86_64.rpm`` 을 한 후 ``$ rpm vagrant_2.1.4_x86_64.rpm`` 를 입력하면 다운로드가 진행된다.

``$ mkdir contributhon``

``$ cd contributon/``

``$ vim Vagrantfile``

Vagrantfile에는 아래와 같은 정보를 입력한다::

Vagrant.configure("2") do |config|
config.vm.box = "ubuntu/xenial64"
config.vm.provider "virtualbox" do |vb|
vb.memory = "6144"
vb.cpus = "6"
end
config.vm.network "forwarded_port", guest: 80, host: 8080
end

+) ``config.vm.network "forwarded_port", guest: 80, host: 8080`` 의 의미를 찾아보았다. 
- guest의 80이란 vm의 80 포트를 의미하여 host의 8080은 말그대로 호스트의 8080 포트를 의미한다.
- 즉 vm의 80 포트를 host의 8080 포트에 연결한다는 뜻이다.
- openstack을 설치한 이후 ip:8080을 하면 openstack dashboard에 접속할 수 있다.

``$ vagrant up`` : 설정한 정보대로 vm이 생성된다.

``$ vagrant ssh`` : vm에 접속한다.

여기까지해서 접속했다면 vm 설치는 성공이다!!!

--------------------------------
3. devstack Install
--------------------------------

이제 devstack을 설치할 차례이다.

~~~~~~~~~~~~~~~~~
3.1 devstack 정의
~~~~~~~~~~~~~~~~~

마찬가지로 devstack이 무엇인지 간단하게 찾아보았다.

* `devstack 이란? <https://www.slideshare.net/ianychoi/openstack-devstack-install-1-allinone>`_
- OpenStack 개발 환경을 구성하기 위한 스크립트
- OpenStack의 구성 확인 및 테스트 용도로 사용

~~~~~~~~~~~~~~~~~
3.2 devstack 설치
~~~~~~~~~~~~~~~~~

이제 아래의 명령어들을 차례대로 입력하면서 설치를 진행하면 된다.

* `devstack install 방법 <https://docs.openstack.org/devstack/latest/>`_ 에서 차례대로 진행하면 되는데, 공부한 기록을 남기기 위하여 따로 아래에 작성했다. 

``$ sudo useradd -s /bin/bash -d /opt/stack -m stack`` : 개별의 stack user를 생성한다.

``$ echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack`` : devstack은 system상으로 많은 변화를 만들것이니, sudo 권한을 준다.

``$ sudo su - stack`` :  생성한 stack 으로 사용자를 변경한다.

``$ git clone https://git.openstack.org/openstack-dev/devstack`` : github에 있는 devstack을 clone한다.

``$ cd devstack``

``$ sudo ifconfig``

위의 명령어를 입력하면 엄청나게 긴 글이 나온다 ::

br-ex     Link encap:Ethernet  HWaddr 8a:a2:fd:f3:1d:4b
inet addr:172.24.4.1  Bcast:0.0.0.0  Mask:255.255.255.0
inet6 addr: fe80::88a2:fdff:fef3:1d4b/64 Scope:Link
inet6 addr: 2001:db8::2/64 Scope:Global
UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
RX packets:27 errors:0 dropped:0 overruns:0 frame:0
TX packets:12 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1
RX bytes:1572 (1.5 KB)  TX bytes:1256 (1.2 KB)

enp0s3    Link encap:Ethernet  HWaddr 02:93:23:4d:82:b3
inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
inet6 addr: fe80::93:23ff:fe4d:82b3/64 Scope:Link
UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
RX packets:1106840 errors:0 dropped:0 overruns:0 frame:0
TX packets:341418 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:1103625143 (1.1 GB)  TX bytes:24041347 (24.0 MB)

lo        Link encap:Local Loopback
inet addr:127.0.0.1  Mask:255.0.0.0
inet6 addr: ::1/128 Scope:Host
UP LOOPBACK RUNNING  MTU:65536  Metric:1
RX packets:682153 errors:0 dropped:0 overruns:0 frame:0
TX packets:682153 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1
RX bytes:213601831 (213.6 MB)  TX bytes:213601831 (213.6 MB)

virbr0    Link encap:Ethernet  HWaddr 52:54:00:f0:23:1b
inet addr:192.168.122.1  Bcast:192.168.122.255  Mask:255.255.255.0
UP BROADCAST MULTICAST  MTU:1500  Metric:1
RX packets:0 errors:0 dropped:0 overruns:0 frame:0
TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

여기서 ``enp0s3`` 를 보면 ``inet addr:10.0.2.15`` 로  ubuntu가 10.0.2.15 ip로 설정된 것을 볼 수 있다.

이 ip는 아래 local.conf의 HOST_IP에 작성해주면 된다.

``$ vim local.conf`` : local.conf 파일을 생성한다.

여기서 localrc, local.conf의 차이를 말씀해 주셨는데, localrc는 옛날 버전이며 local.conf는 최신 버전이라고 한다.
local.conf만 생성했다고 해서 localrc가 생성되지 않는 것이 아니라 local.conf안에 localrc가 포함되어있다.

local.config 내용 :

[[local|localrc]]

HOST_IP=10.0.2.15

ADMIN_PASSWORD=secret

DATABASE_PASSWORD=$ADMIN_PASSWORD

RABBIT_PASSWORD=$ADMIN_PASSWORD

SERVICE_PASSWORD=$ADMIN_PASSWORD

``local.conf`` 를 위와 같이 입력하고, 저장을 해준다.

그럼 이제 ``$ ./stack.sh`` 를 입력하여 devstack을 설치해준다!!!

devstack 설치는 20~30분 정도가 소요된다.

~~~~~~~~~~~~~~~~~~~~~~~~
3.2.1 잠깐 쉬어가는 타임
~~~~~~~~~~~~~~~~~~~~~~~~

잠깐 설치를 진행하는 동안 문서를 작성하는 방법에 대해서, 오늘 스터디를 한 내용을 github에 올리는 방법에 대해서 설명해 주셨다.

보통 문서를 작성할 때는 markdown을 많이 활용한다.

하지만 openstack에서는 sphinx를 사용하는데, 

sphinx란 Python 코드 내에 들어간 docstring을 자동으로 문서화해주고 아주 간단한 설정으로 쉽게 문서를 작성할 수 있게 하는 도구이다.

이 문서를 작성하는 문법을 공부할 때 아래의 `오픈스택 문서 <https://github.com/openstack/openstack-manuals/tree/master/doc>`_ 를 참고하여 공부하면 좋다. 

문법을 공부하고 문서를 작성했다면, 해당 문서를 우리 팀의 github에 올려야한다.

일단 github에 들어가면 `openstack team1 <https://github.com/openstack-kr/contributhon-2018-team1/>`_ 오른쪽 위에 ``fork`` 라는 버튼이 보일 것이다.

이 fork는 OS에서 프로세스를 복제한다는 의미로 (처음 알았다..) 해당 github를 똑같이 복제하여 내 repository로 가져오는 것이다.

이렇게 **복제한 곳에서는 commit을 하더라도, 본래의 github는 변경되지 않는다.**

이렇게 복제된 자신만의 공간에서 문서를 작성하고 수정하고 수정이 끝난 문서들은 ``pull request`` 를 해야한다.

즉 본래의 github에 merge하기 위해 요청을 해야한다.

이 버튼은 fork한 자신의 repository에가면 branch가 있는 버튼 옆에 존재한다.

이 버튼을 눌러 요청을 하고 수락이 되면! 원본 github에 내 글이 올라가게 된다.

*(저도 해본적이 없어서.. 한번 실습을 해보면 더 이해가 빠를 것 같습니다!)*

그리고 멘토님이 당부하셨던건 commit message를 잘 작성하는 방법에 대해서 공부하고, 

commit message를 잘 작성하기 위해서 연습하라고 하셨다.

`좋은 깃(Git) 커밋 메시지 작성하기 <https://tech.ssut.me/2015/06/24/write-a-good-git-commit-message/>`_ 를 참고하여 commit message를 작성하는 방법을 공부하자!

~~~~~~~~~~~~~~~~~~~~~~~~~~~~
3.3 openstack dashboard 접속
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

약 2000초 이후에.. openstack 설치가 완료되었다.

설치가 끝난 이후에는 openstack dashboard로 접속해야한다.

``$ exit`` 를 해 vagrant를 빠져 나온후

``$ sudo ifconfig`` 를 실행한다.::

eth0      Link encap:Ethernet  HWaddr 00:25:90:B5:49:24
inet addr:110.10.129.22  Bcast:110.10.129.127  Mask:255.255.255.128
inet6 addr: fe80::225:90ff:feb5:4924/64 Scope:Link
UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
RX packets:1247717 errors:0 dropped:0 overruns:0 frame:0
TX packets:491484 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:1742171123 (1.6 GiB)  TX bytes:43531987 (41.5 MiB)

lo        Link encap:Local Loopback
inet addr:127.0.0.1  Mask:255.0.0.0
inet6 addr: ::1/128 Scope:Host
UP LOOPBACK RUNNING  MTU:65536  Metric:1
RX packets:99913 errors:0 dropped:0 overruns:0 frame:0
TX packets:99913 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:0
RX bytes:8029129 (7.6 MiB)  TX bytes:8029129 (7.6 MiB)

``eth0`` 에서 ``inet addr`` 를 보면 ip가 있는데 그 ip인 (여기서는 ``110.10.129.22`` )로 openstack dashboard으로 접속할 수 있다.

http://110.10.129.22:8080/ 으로 접속이 되면 성공이다!!!!

그럼 끝!!! 수고하셨습니다!!

====
Tip
====

devstack을 조금 더 편리하게 사용하기 위해서, 몇가지 팁과 공부할 자료를 주셨다.

----------
1. Screen
----------

~~~~~~~~~~~~~~~~~
1.1. Screen 정의
~~~~~~~~~~~~~~~~~

- linux에서 물리적인 터미널을 여러 개의 가상 터미널로 다중화해주는 도구이다. 각 screen으로 생성한 가상 터미널은 **독립적으로 동작하며 사용자 세션이 분리되어도 동작** 한다.

- 이 도구는 백그라운드로 동작하는 다중 터미널을 만들어 백그라운드 작업을 간단히 수행할 수 있고, 중**간에 끊더라도 다시 접속하면 같은 화면을 볼 수 있도록 한다.** 

- 이를 이용해서 시간이 오래 걸리는 도구를 설치할 때에도 screen을 만들어 설치하고 screen을 나와도 설치는 중단되지 않고 실행되게 할 수 있다. 또한 카폐에서 작업을 하다가 집에 가더라도 screen으로 다시 접속하면 내가 작업하던 부분부터 확인할 수 있다. (!!!!!!)

~~~~~~~~~~~~~~~~~
1.2. Screen 설치
~~~~~~~~~~~~~~~~~

``$ yum install screen`` : screen 도구를 설치한다.

``$ screen -S [screen 이름]`` : screen을 원하는 이름으로 생성한다.

* screen에서 빠져나가고 싶을 때 : ``ctrl+a,d``
* screen에 다시 접속하고 싶을 때 : ``$ screen -r [screen 이름]``

``$ screen -list`` : screen list를 확인한다.

``$ screen -X -S [없애고 싶은 세션 숫자] quit`` : screen session 삭제

------------------------------
2. 공부할 때 도움되는 참고글
------------------------------

* `openstack document <https://docs.openstack.org/install-guide/>`_ : openstack 공식 문서
* `openstack network 구축 과정 이해 <https://printf.kr/archives/307>`_
* `devstack으로 multi node 구성하기 <https://nhnent.dooray.com/share/posts/NksDQdLvSA-KRSuJra5jlA>`_
