---
title:  "Docker Make Custom Image"
date:   2018-04-19 20:13:00
author: Dean Kim
categories: Docker
tags:	Docker Image Centos Pyenv Python PIP
cover:  "/assets/instacode.png"
---

Docker 공부를 하며 저만의 Base Image를 만들고 Docker hub에 push하는 과정을 정리한 포스팅입니다.
이 글은 Docker hub 계정과 repo 있음과 docker가 설치되었음을 전제로 작성되었습니다.

# Make Custom Docker Image

학습 목적으로 만드는 만큼 불필요한 요소도 있지만 경험을 하는데 의의를 두고 있습니다. 불필요한 것들도 있다고 느끼실 수도 있지만 특별한 목적으로 설치한 것이 아니니 이해부탁드립니다. :) 

우선 터미널에서 docker hub에 로그인 합니다.

~~~~
docker login
~~~~

위의 명령어를 실행시키고 username과 password를 차례로 입력하면 터미널에서 docker hub에 접근할 수 있습니다.

로그인이 성공하면 docker hub에서 centos를 pull 합니다.

~~~~
docker pull centos:latest // :latest는 tag입니다. 가장 최신 버전을 받겠다는 의미입니다.
~~~~

받은 Image를 확인합니다.

~~~~
docker images
~~~~

docker hub의 centos를 pull 받았기 때문에 centos라는 이름으로 Image가 존재할 것입니다.
이 Image에 접속해서 필요한 것들을 설치하도록 하겠습니다.

다음의 명령어로 container로 들어갑니다.

~~~~
docker run -t -i centos bin/bash
~~~~

container에 들어왔으면 다음의 명령어로 먼저 git을 설치합니다.

~~~~
yum install -y git // y 옵션은 설치하는 과정에서 y/n을 묻는 질문에서 자동으로 y를 입력하고 설치를 진행합니다.
~~~~

소스 코드에서 소프트웨어를 빌드하고 컴파일할 수 있게 해주는 CentOS 개발 도구를 설치합니다.

~~~~
yum -y groupinstall development
~~~~

CentOS는 안정성을 기본으로 하는 RHEL(Red Hat Enterprise Linux)에서 파생되었습니다. 
이 때문에 테스트되고 안정된 버전의 응용 프로그램이 시스템 및 다운로드 가능한 패키지에서 가장 많이 발견되므로 CentOS에서는 Python 2만 찾을 수 있습니다.

대신 Python 3의 최신 업스트림 안정 버전을 설치하고자 하므로 IUS를 설치해야합니다. IUS는 업스트림 안정화 인라인을 의미합니다. 
커뮤니티 프로젝트인 IUS는 일부 최신 버전의 선택 소프트웨어에 대해 Red Hat Package Manager(RPM) 패키지를 제공합니다.

다음의 명령어로 RPM 패키지를 설치합니다.

~~~~
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
~~~~

의존성 패키지들을 설치합니다.

~~~~
yum install -y zlib-devel bip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel xz
~~~~

다음의 명령어로 pyenv를 설치합니다.

~~~~
curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
~~~~

pyenv 설치 시 PATH를 자동으로 추가하지 않기 때문에 다음의 명령어로 bash_profile에 추가해줘야 합니다.

~~~~
echo 'export PATH="$HOME/.pyenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
source ~/.bash_profile
~~~~

pyenv를 이용해 python 3.6.3을 설치합니다.

~~~~
pyenv install 3.6.3
~~~~

설치한 python 3.6.3을 사용하도록 다음의 명령어를 입력합니다.

~~~~
pyenv global 3.6.3
~~~~

다음의 명령어로 python 버전을 확인합니다.

~~~~
python --version
~~~~

설치가 잘 되었음을 확인하고 container에서 나갑니다.

~~~~
exit
~~~~

터미널에서 다음의 명령어로 commit을 합니다.

~~~~
docker commit -m "custom image init" -a "deankim" {CONTAINER ID입력} deankim/centos-pyenv:basic
~~~~

위의 명령어에서 m 옵션은 commit 메시지, a 옵션은 작성자입니다.

CONTAINER ID는 docker ps -a의 명령어로 확인하실 수 있습니다.

deankim/centos-pyenv는 docker hub 상의 제 repo 주소입니다.

:basic은 이 Image에 대한 tag입니다.

commit이 완료되면 repo로 push합니다.

~~~~
docker push deankim/centos-pyenv:basic
~~~~

docker hub 페이지 로그인 후 repo를 보시면 Image를 확인하실 수 있습니다.

처음 만들어 본 Image라 centos에 관련된 개발도구나 rpm, 의존성 패키지에 대해서 하나하나 자세히 알지는 못한 것이 아쉽지만 처음 Image를 만들어
본 것에 의의를 두고 기회가 될 때 하나씩 알아가도록 하겠습니다.

읽어주셔서 감사합니다. :)