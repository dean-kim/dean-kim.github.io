---
title:  "Solidity Smart Contract Practice with Truffle"
date:   2018-05-16 20:09:00
author: Dean Kim
categories: BlockChain
tags:	BlockChain Solidity Smart Contract Practice Truffle
cover:  "/assets/instacode.png"
---

예전에 작성했던 Smart Contract를 Truffle을 이용해 작성해보도록 하겠습니다.

이번에 사용하는 [Truffle](http://truffleframework.com/)은 이더리움의 dApp 개발에 사용하는 framework입니다. 
Smart Contract 작성과 컴파일, 배포, 테스트 등을 도와주는 framework입니다.

# Truffle install

Truffle을 설치하려면 <tt style="color: #FF0000">`node.js`</tt>가 필요합니다. 먼저 node.js를 설치합니다. 
[node.js 설치페이지](https://nodejs.org/ko/)에서 LTS 버전으로 설치해주세요.
Truffle 설치에는 node.js가 5.0 버전 이상이 필요한데 현재(2018.05.16 기준) 8.11.1 버전이니 버전 문제는 없습니다.

node.js 설치가 완료되면 터미널에서 다음의 명령어로 Truffle을 설치합니다.

~~~~
npm install -g truffle
~~~~

여기까지하면 truffle의 설치가 완료되었습니다.

그리고 작성할 Smart Contract의 테스트할 환경을 제공해주는 <tt style="color: #FF0000">`Ganache`</tt>를 설치해줍니다.
[Ganache 설치페이지](https://github.com/trufflesuite/ganache/releases)에서 본인의 OS에 맞는 설치파일을 다운로드한 후 설치합니다.

이제 Truffle을 이용해서 Smart Contract를 작성하고 테스트하고 배포할 환경이 갖추어 졌습니다.

## Init

먼저 Smart Contract를 작성할 디렉토리를 생성합니다.
저는 이전에 작성했던 NoShow라는 이름으로 진행할 예정이기에 동일한 이름으로 디렉토리를 만들겠습니다.
만들고 해당 디렉토리로 이동합니다.

~~~~
mkdir NoShow
cd NoShow
~~~~

터미널에서 다음의 명령어로 Truffle 프로젝트의 기본 구조를 만듭니다.

~~~~
truffle init
~~~~

그러면 다음과 같은 메세지가 나오면서 성공적으로 설치가 끝났다는 메세지가 나옵니다.

~~~~
Downloading...
Unpacking...
Setting up...
Unbox successful. Sweet!

Commands:

  Compile:        truffle compile
  Migrate:        truffle migrate
  Test contracts: truffle test
~~~~

위의 과정으로 디렉토리에 다음과 같은 디렉토리와 파일들이 생성되었습니다.

```
.
├── contracts
├── migrations
├── test
├── truffle-config.js
└── truffle.js
```

각각이 어떤 역할들을 하는지 살펴보겠습니다. 내용은 [공식페이지](http://truffleframework.com/docs/)를 참고했습니다.

* <tt style="color: #FF0000">`contracts`</tt>

Solidity contract들이 위치하는 디렉토리입니다.

* <tt style="color: #FF0000">`migrations`</tt>

스크립트 가능한 배포 파일의 디렉터리입니다.
마이그레이션은 Ethereum네트워크에 계약을 배포하는데 도움이 되는 JavaScript파일입니다. 
이러한 파일은 배포 작업을 준비하는 역할을 담당하며 배포 요구 사항이 시간에 따라 변경될 것이라는 가정 하에 작성됩니다. 
프로젝트가 진행됨에 따라 블록 체인에서 이러한 발전을 위해 새로운 마이그레이션 스크립트를 만들 수 있습니다. 
이전에 실행된 마이그레이션 기록은 특별 마이그레이션 계약을 통해 on-chain으로 기록됩니다.

* <tt style="color: #FF0000">`test`</tt>

애플리케이션 및 contract들의 테스트를 위한 테스트 파일용 디렉토리입니다.

* <tt style="color: #FF0000">`truffle.js(truffle-config.js)`</tt>
 
Truffle configuration file 입니다. 프로젝트 디렉토리의 루트에 있습니다.

## Configuration

Truffle은 프로젝트 디렉토리의 루트에 있는 <tt style="color: #FF0000">`truffle.js`</tt>에 설정된 내용을 바탕으로 실행환경을 조성합니다.
그래서 다음과 같이 <tt style="color: #FF0000">`truffle.js`</tt>를 작성합니다.

~~~~
# in truffle.js
module.exports = {
    networks: {
        development: {
            host: '127.0.0.1',
            port: 7545,
            network_id: '*',
        }
    }
};
~~~~

작성한 내용을 살펴보겠습니다.

* <tt style="color: #FF0000">`networks`</tt>

마이그레이션 중에 배포할 수 있는 네트워크와 각 네트워크와 상호작용할 때 특정한 트랜잭션 매개 변수(gas price, address 등)를 지정합니다.
네트워크 개체는 네트워크 이름으로 연결되며 네트워크 매개 변수를 정의하는 해당 개체를 포함합니다. 네트워크 구성이 설정되지 않으면 Truffle은
이 contract를 배포할 수 없습니다.

만약 다른 네트워크를 연결하도록 Truffle을 구성하려면 네트워크를 추가하고, 네트워크 id를 지정하면 됩니다.

네트워크 이름은 특정 네트워크에서 마이그레이션을 실행할 때와 같이 사용자 인터페이스 용도로 사용됩니다.

여기서는 개발용으로 사용할 목적이라 <tt style="color: #FF0000">`development`</tt>를 사용하겠습니다.

로컬에서 사용하기에 <tt style="color: #FF0000">`host`</tt>는 <tt style="color: #FF0000">`127.0.0.1`</tt>을 사용합니다. localhost가 가능한지 파악필요~
<tt style="color: #FF0000">`port`</tt>는 Ganache의 기본 설정인 <tt style="color: #FF0000">`7545`</tt>를 사용합니다.
<tt style="color: #FF0000">`network_id`</tt>는 이더리움 네트워크를 식별하는 ID 입니다. 만약 live network를 사용한다면 <tt style="color: #FF0000">`1`</tt>을 사용합니다.
여기서는 개발용으로 사용할 예정이므로 <tt style="color: #FF0000">`*`</tt>을 설정합니다.

## Create Contract with TDD

### Create Contract file and test file

Contract는 터미널에서 다음의 명령으로 생성합니다.

~~~~
truffle create contract NoShow
~~~~

<tt style="color: #FF0000">`contrack`</tt> 디렉토리를 확인해보면 <tt style="color: #FF0000">`NoShow.sol`</tt>이 잘 생성되었음을 확인할 수 있습니다.

생성된 파일을 열고 다음을 작성합니다.

~~~~
# in contracts/NoShow.sol
pragma solidity ^0.4.4;


contract NoShow {
}
~~~~

그리고 <tt style="color: #FF0000">`test`</tt> 디렉토리에 <tt style="color: #FF0000">`TestNoshow.sol`</tt>을 생성합니다.

### Start TDD



<tt style="color: #FF0000">``</tt>