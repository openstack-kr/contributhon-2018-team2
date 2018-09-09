=========================
Contribution History
=========================

-----------------
Week 1
-----------------

~~~~~~~~~~~~~~~~~
1. DevStack 설치
~~~~~~~~~~~~~~~~~

	- Cafe24 호스팅 : Cent OS 6.1 64bit (*116.125.120.18*) 	
	- DevStack과 궁합이 좋은 Ubuntu로 OS 변경 
	- Vagrant를 이용해 Ubuntu 가상 OS 설치
	- `공식문서 <https://docs.openstack.org/devstack/latest/>`_ 참조하여 DevStack 설치	
	- Vagrant에서 8080으로 포트포워딩하여 116.125.120.18:8080으로 접속확인
	- Vagrant 파일 [1]_ ::

            Vagrant.configure("2") do |config|
              config.vm.box = "ubuntu/xenial64"
              config.vm.provider "virtualbox" do |vb|
                  vb.memory = "6144"
                  vb.cpus = "6"
              end
              config.disksize.size = '50GB'
              config.vm.network "forwarded_port", guest: 80, host: 8080
              config.vm.network "forwarded_port", guest: 20, host: 20
              config.vm.network "forwarded_port", guest: 21, host: 21
            end

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
2. git repo 생성 및 rst 문서작성 연습
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	- 컨트리뷰톤 1팀 git repo fork
	- `링크 <http://docutils.sourceforge.net/docs/user/rst/quickref.html>`_ 참조하여 히스토리 문서 작성

-----------------
Week 2
-----------------

~~~~~~~~~~~~~~~~~~~~~~~~~
1. PR 생성 및 코드리뷰
~~~~~~~~~~~~~~~~~~~~~~~~~

	- 생성 repo에 `upstream <https://github.com/openstack-kr/contributhon-2018-team1>`_ remote 추가
	- branch 변경 후 PR 생성 [2]_  
	- git.openstack.org의 openstack-dev/sandbox(연습용 repo)에서 코드리뷰 연습

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
2. 네트워크 생성 및 VM 연결
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	- 네트워크 생성
	- 네트워크 <-> VM 연결
	- 라우터 생성
	- 라우터를 이용해 public <-> private 네트워크 연결
	- 연결된 public ip를 통해 VM에 ssh로 접속 확인

-----------------

.. [1] VM용량을 50Gb로 늘리지 않았을 때 인스턴스가 생성되지 않는 이슈 발생.
.. [2] master branch는 가급적 유지하기 위함