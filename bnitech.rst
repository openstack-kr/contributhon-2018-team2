[컨트리뷰톤 2018] 오픈스택 2팀: 사전 오프 모임 (8/31 금, 저녁 7시) 내용
+++++++++++++++++++++++++++++++++++++++++++++
카페24 서버 확인
	* 서비스 사용현황
		* 1.234.**.***
		* 서버 임시정보 확인
			* 계정
				* root
			* IPMI 계정
				* b******
			* 임시 비밀번호
				* a*******

    Varant로 설치
	* 언제든지 부시고 재설치 할수 있는 환경

		* 가상환경을 만들어서 오픈스택을 설치한다.
		* 그럼 버리고 다시 생성하기 쉬워진다.
	* 가상환경을 코드로 관리할 수 있게 해준다.
	* VM을 여러개 만들어서 테스트용을 따로 가진다.

오픈스택 설치
	* yum install -y binutils gcc make patch libgomp dkms glibc-headers glibc-devel kernel-headers kernel-devel

		* 필요한 패키지 설치
	* wget https://download.virtualbox.org/virtualbox/5.2.18/VirtualBox-5.2-5.2.18_124319_el6-1.x86_64.rpm
	* rpm 안되서설치

		* 아닐수도

			* wget https://rpmfind.net/linux/centos/6.10/os/x86_64/Packages/SDL-1.2.14-7.el6_7.1.i686.rpm
			* wget https://rpmfind.net/linux/centos/6.10/os/x86_64/Packages/mesa-libGL-11.0.7-4.el6.i686.rpm
			* wget https://rpmfind.net/linux/centos/6.10/os/x86_64/Packages/libXmu-1.1.1-2.el6.x86_64.rpm
		* wget -O /etc/yum.repos.d/virtualbox.repo http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo
		* yum list VirtualBox*
		* yum update
		* yum install VirtualBox-5.2
	* rpm -ivh https://releases.hashicorp.com/vagrant/2.1.4/vagrant_2.1.4_x86_64.rpm


	* 아키텍처

		* PENCIL로 그림

			* 사진1
	* root에 contributon 폴더 만들고

		* 안에서
		* vagrantfile만듬

			* Vagrant.configure("2") do |config|
			*   config.vm.box = "ubuntu/xenial64"
			* end
		* 작성
	* vagrant up
	* vagrant status

		* 상태확인
	* vagrant ssh

		* vatrantfile이 있는곳에서
		* 가상머신 접속
	* vagrant destroy

		* 가상머신 삭제
	* vagrant up 으로 재생성 가능

		* 처음보다 빠르게 된다.
		* 로컬에 기본 파일들이 다운되어있어서
	* ubuntu (가상서버) 에서 작업

		* logout

			* 나가기
	* vagrant 수정

		* config.vm.provider "virtualbox" do |vb|
		*       vb.memory = "6144"
		*       vb.cpus = "6"
		*   end
	* 코드로 인프라를 구성할 수 있다.
	* 조성수님 블로그에서

		* https://nhnent.dooray.com/share/posts/NksDQdLvSA-KRSuJra5jlA
		* virtualbox 네트워크 설명

			* host-only networking 이용
		* vagrant networking 탭

			* Forwarded port

				* https://www.vagrantup.com/docs/networking/forwarded_ports.html
		* vagrantfile 수정

			*  config.vm.network "forwarded_port", guest: 80, host: 8080
			* 추가
		* vagrant reload
		* NAT IP는 접속불가 됨
	* snapshot 지원
	* 프로비져닝도 가능
	* 도커는 컨테이너

		* vagrant는 VM
	* vm 이름도 지정 가능
	* 최종 vagrantfile

		* Vagrant.configure("2") do |config|
		*   config.vm.box = "ubuntu/xenial64"
		*   config.vm.provider "virtualbox" do |vb|
		*       vb.memory = "6144"
		*       vb.cpus = "6"
		*   end
		* config.vm.network "forwarded_port", guest: 80, host: 8080
		* end
	* devstack 설치

		* https://docs.openstack.org/devstack/latest/

			* download devstack 까지 진행
		* git branch -l

			* 브랜치확인
			* git branch -r
		* master 브랜치에서 하면 확임
		* git checkout stable/pike
		* git status

			* 브랜치확인
		* local.conf 파일 만들기

			* [[local|localrc]]
			* HOST_IP=10.0.2.15

				* 기본 지정
			* ADMIN_PASSWORD=secret
			* DATABASE_PASSWORD=$ADMIN_PASSWORD
			* RABBIT_PASSWORD=$ADMIN_PASSWORD
			* SERVICE_PASSWORD=$ADMIN_PASSWORD
		* ./stack.sh

			* 오래걸림
			* 기본적인 모든 서비스 설치
		* 공인아이피로 접속

			* admin
			* secret
	* screen

		* 가상의 화면을 띄운다
		* yum install screen
		* screen -S devstack
		* ctrl+a,d
		* screen -list
		* screen -r 이름
		* screen session 지우는 방법은 아래와 같습니다.

			* screen -X -S [없애고 싶은 세션 숫자] quit
		* 백그라운드로 실행가능해짐
		* 작업하던 환경 유지시켜줌

깃허브 운영
	* rst 파일형식 사용

		* https://www.slideshare.net/ianychoi/pycon-kr-2017-rst-python-openstack

			* RST 문서 작성 설명

		* 개발뿐만아니라 문서화도 중요하다

			* http://git.openstack.org/cgit
			* https://docs.openstack.org/ko_KR/upstream-training/
	* spec

		* 개발하고싶은거 적어두는 곳
	* 저장소에 문서 작성하는법

		* 저장소를 fork한다.
		* 계정선택
		* 복제되어
		* 쓰기권한이 생긴다.
		* $ ssh-keygen -t rsa -b 4096

			* 생성된 키 깃허브에 입력
		* tect.ssut

			* git 작성 참조


