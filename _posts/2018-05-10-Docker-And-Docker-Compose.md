title:  "Docker And Docker-Compose"
date:   2018-05-10 18:09:00
author: Dean Kim
categories: Docker
tags:	Docker Docker-Compose
cover:  "/assets/instacode.png"
---

개인 프로젝트를 진행하며 사용한 Docker와 Docker-Compose에 대한 내용을 정리한 글입니다.

작성하며 공식문서([Dockerfile](https://docs.docker.com/engine/reference/builder/#parser-directives), [Docker-Compose](https://docs.docker.com/compose/compose-file/))를 주로 참고했습니다.

# Dockerfile

먼저 프로젝트의 root에 Dockerfile을 만듭니다.

저는 OS는 CentOS, 개발언어는 Python 3.6버전을 기반으로 프로젝트를 진행하려고 합니다.

아래에 작성된 Dockerfile을 보면서 어떤 설정을 사용했는지 보겠습니다.

~~~~
# in Dockerfile
FROM centos/python-36-centos7:latest
USER root

# RUN mkdir -p var/www/refrigerator
# COPY . /var/www/refrigerator

ADD . /var/www/refrigerator/

WORKDIR /var/www/refrigerator
RUN pip install --upgrade pip
RUN pip install -r requirements.txt

EXPOSE 8000

ENTRYPOINT ["/bin/bash"]
~~~~

#### FROM

먼저 CentOS 기반이면서 Python 3.6이 설치된 Base Image를 pull 합니다.

* 형식 : <tt style="color: #FF0000">`FROM <이미지 이름>:<태그>`</tt>

<tt style="color: #FF0000">`FROM`</tt>은 [Docker Hub](https://hub.docker.com/)에서 어떤 Image를 pull 하는지를 설정합니다.
제가 작성한 <tt style="color: #FF0000">`centos/python-36-centos7:latest`</tt> 중에서 <tt style="color: #FF0000">`latest`</tt>는 가장 최신 버전을 pull 하겠다는 태그입니다.

#### USER

* 형식 : <tt style="color: #FF0000">`USER <계정명>`</tt>

<tt style="color: #FF0000">`USER`</tt>는 이 Docker Image를 실행할 user를 지정합니다.
개발하는 과정에서 root 디렉토리에 파일을 생성할 경우 root 계정만 생성할 수 있기에 저는 root로 설정했습니다.
<tt style="color: #FF0000">`USER`</tt>는 뒤에 오는 RUN, ENTRYPOINT 여기서는 사용하지 않았지만 CMD 명령어에 영향을 주며 혹 다른 user로 어떤 설정을 해야할 경우, 중간에 변경할 수 있습니다.

#### RUN

* 형식 : <tt style="color: #FF0000">`RUN <명령>`</tt>

<tt style="color: #FF0000">`RUN`</tt>은 shell command나 어떤 package 설치 등에 사용됩니다.
여기서의 사용은 <tt style="color: #FF0000">`mkdir`</tt> command로 디렉토리를 생성했습니다. 그리고 아래에서 pip를 upgrade하고, requirements.txt에 있는 package를 설치하는데 사용했습니다.

#### COPY

* 형식 : <tt style="color: #FF0000">`COPY <복사할 파일 경로> <이미지에 복사한 파일이 위치할 경로>`</tt>

<tt style="color: #FF0000">`COPY`</tt>는 지정한 경로에 지정한 file을 복사하는 설정입니다.
여기서는 로컬의 프로젝트에 관련된 모든 파일들을 Image의 /var/www/refrigerator 폴더에 복사하는 설정입니다.

#### ADD

* 형식 : <tt style="color: #FF0000">`ADD <복사할 파일 경로> <이미지에 복사한 파일이 위치할 경로>`</tt>

블로그 글을 작성하다가 알게 된 내용이 있어 기존의 코드는 주석처리하고 새로운 설정으로 작성합니다.
<tt style="color: #FF0000">`ADD`</tt>는 <tt style="color: #FF0000">`COPY`</tt>와 비슷하지만 더 많은 기능을 제공합니다.
<tt style="color: #FF0000">`COPY`</tt>는 복사할 디렉토리가 있어야 하지만 <tt style="color: #FF0000">`ADD`</tt>의 경우 디렉토리가 없으면 설정한 경로를 만들어서 파일을 복사합니다.
또한 파일이 위치할 경로에 URL을 사용 가능하며 복사하는 파일이 압축 파일인 경우 압축을 풀어서 복사합니다.
디렉토리 생성과 복사가 한 번에 되니 처음에 작성했던 코드 2줄이 1줄로 줄어들게 됩니다. 그리고 디렉토리를 생성해야 되는 경우 경로 끝에 <tt style="color: #FF0000">`/`</tt>를 꼭 넣어야 합니다.
경로는 절대경로를 사용해야 합니다.

#### WORKDIR

* 형식 : <tt style="color: #FF0000">`WORKDIR <경로>`</tt>

<tt style="color: #FF0000">`WORKDIR`</tt>는 뒤에오는 명령어를 수행하는 디렉토리를 설정합니다.
위의 작성된 설정에서 보면 <tt style="color: #FF0000">`/var/www/refrigerator`</tt> 이 경로에서 다음의 RUN에 있는 명령어들이 실행됩니다.
중간에 디렉토리를 변경해서 다른 경로에서 또 다른 명령들을 수행할 수 있습니다.

#### EXPOSE

* 형식 : <tt style="color: #FF0000">`WORKDIR <포트번호>`</tt>

호스트와 연결할 포트번호 설정입니다. 


# Docker-Compose

Docker-Compose는 다수의 컨테이너를 관리할 수 있는 툴입니다. 
<tt style="color: #FF0000">`docker-compose.yml`</tt> 파일에 구동할 컨테이너들을 정의하고 관리합니다.

프로젝트 root에 <tt style="color: #FF0000">`docker-compose.yml`</tt>을 만듭니다.

~~~~
version: '2'

services:

  db:
    image: "postgres:9.6.1"
    container_name: db
    ports:
     - "5432:5432"
    environment:
     POSTGRES_PASSWORD: "somepassword"
     POSTGRES_USER: "nobody"
     POSTGRES_DB: "DB"

  someservice:
    build: .
    container_name: manager
    ports:
     - "3000:8000"
     - "3030:3030"
    depends_on:
     - db
    volumes:
     - .:/var/www/some_dir
    stdin_open: true  # means -i option in docker run
    tty: true         # means -t option in docker run
~~~~

#### version

Docker-Compose의 version을 기입합니다.

#### services

Docker-Compose에서는 컨테이너를 service로 간주합니다.
services의 아래에 정의되는 각 서비스들은 각각의 컨테이너에 대한 정의로 생각하시면 됩니다.
여기서는 DB 컨테이너와 service에 대한 컨테이너 2개를 정의하겠습니다.

#### db, someservice

DB, service 컨테이너에 대한 정의가 시작되는 부분입니다.
이름은 편하신 것으로 설정하셔도 됩니다.

#### image

이 컨테이너의 Base Image를 정의합니다. 정의된 Base Image를 pull해서 컨테이너를 만들게 됩니다.

#### build

컨테이너를 만들기 위해 작성한 Dockerfile을 통해 컨테이너를 만드는 설정입니다. Dockerfile이 위치한 경로를 작성해주면 됩니다.
여기서는 docker-compose.yml과 같은 경로에 Dockerfile을 작성한 것으로 작성하였습니다.

#### container_name

이 컨테이너에 대한 이름을 정의합니다.

#### ports

이 컨테이너와 호스트와의 연결할 포트에 대한 설정입니다.

#### depends_on

특정 컨테이너에 대한 의존성을 나타냅니다. 이 항목에 있는 컨테이너가 먼저 만들어지고 <tt style="color: #FF0000">`depends_on`</tt>을 갖고 있는 컨테이너가 나중에 만들어집니다.
이 docker-compose.yml의 정의에 따르면 db 컨테이너가 먼저 만들어지고, someservice 컨테이너가 만들어집니다.

#### environment

이 컨테이너에서 사용하는 환경변수에 대한 정의입니다.

#### volumes

컨테이너에 볼륨을 마운트하는 설정입니다.
만약 호스트의 디렉토리와 컨테이너의 디렉토리를 마운트하기 위해서는 <tt style="color: #FF0000">`<호스트 디렉토리 경로>:<컨테이너 디렉토리 경로>`</tt> 형식으로 작성합니다.

#### stdin_open, tty

각각을 true로 설정하면 docker run 명령에서 <tt style="color: #FF0000">`stdin_open`</tt>는 <tt style="color: #FF0000">`-i`</tt> option, <tt style="color: #FF0000">`tty`</tt> <tt style="color: #FF0000">`-t`</tt> option을 사용가능하게 해줍니다.

### 소회

처음으로 Dockerfile과 Docker-Compose를 작성해보았습니다. 물론 설정 항목에 대해 자세하게 숙지하고 있으면 좋겠지만 자주 사용하는 것이 아닌 만큼 금방 잊을 것 같습니다.
하지만 공식문서와 레퍼런스가 많은 만큼 앞으로 새로운 것을 만들 때 큰 어려움은 없을 것 같다는 긍정적인 생각을 해보며 이번 포스팅을 마칩니다. 읽어주셔서 감사합니다.
