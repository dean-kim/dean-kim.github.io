---
layout: post
title:  "Solidity Introduction to Smart Contracts"
date:   2018-02-25 15:43:59
author: Dean Kim
categories: BlockChain
tags:	BlockChain Solidity Smart_Contracts
cover:  "/assets/instacode.png"
---

블록체인을 공부하며 Smart Contract 작성에 사용되는 Solidity 공식문서를 번역한 글입니다.
제가 이해한 수준으로 기술된 내용도 있어 원본과 내용이 조금 다를 수도 있습니다.
혹시 제가 잘못 이해한 내용이 있다면 말씀해주세요 :)

# Introduction to Smart Contracts
- 원본 : [공식문서](https://solidity.readthedocs.io/en/latest/introduction-to-smart-contracts.html)

## A Simple Smart Contract

가장 기본적인 예제부터 시작하겠습니다. 지금 당장 모든 것을 이해하지 못해도 괜찮습니다. 나중에 자세히 살펴보겠습니다.

### Storage

~~~~
pragma solidity ^0.4.0;

contract SimpleStorage {
    uint storedData;

    function set(uint x) public {
        storedData = x;
    }

    function get() public constant returns (uint) {
        return storedData;
    }
}
~~~~

첫 번째은 이 코드가 Solidity 버전 0.4.0으로 작성되었다는 것을 의미합니다. 이 버전보다 최신 버전으로 컴파일된다면 오류가 발생할 수 있기에 이를 막고자 작성하는 코드입니다.
<tt style="color: #FF0000">`pragma`</tt>는 일종의 지시문으로 이후에 작성된 `solidity ^0.4.0`에 맞는 컴파일러를 사용하도록 해줍니다.

Solidity에서 계약은 Ethereum 블록 체인의 특정 주소에 있는 코드(의 함수)와 데이터(의 상태)들의 모음입니다. 
<tt style="color: #FF0000">`uint storedData;`</tt>는 <tt style="color: #FF0000">`uint`</tt>(256 비트의 부호가 없는 정수)타입의 <tt style="color: #FF0000">`storedData`</tt>라고 하는 상태 변수를 선언합니다. 
데이터베이스를 관리하는 코드의 함수를 호출하여 쿼리하고 변경할 수 있는 데이터베이스의 단일 슬롯이라고 생각할 수 있습니다. 
Ethereum의 경우, 이것은 항상 소유 계약입니다. 이 경우 <tt style="color: #FF0000">`set`</tt> 및 <tt style="color: #FF0000">`get`</tt> 함수를 사용하여 변수 값을 수정하거나 검색할 수 있습니다.

다른 언어에서 처럼 상태 변수 접근을 위해 사용하는 <tt style="color: #FF0000">`this`</tt>키워드는 필요하지 않습니다.

이 contract code는 (Ethereum에 의해 구축된 인프라를 이용해) 아직 많은 것을 하고 있지는 않으며 다른 사람이 이 번호를 게시하지 못하도록(실현 가능한) 방법 없이는 전 세계 누구나 엑세스할 수 있는 단일 번호를 저장할 수 있습니다.
그래서 다른 사람이 <tt style="color: #FF0000">`set`</tt> 함수를 이용해 다른 값으로 다시 번호를 덮어 쓸 수는 있지만 그 숫자는 블록 체인의 기록에 저장됩니다. 나중에 액세스 제한을 적용하여 번호만 변경할 수 있는 방법을 살펴 보겠습니다.

<b>Note</b>
<br>모든 식별자(계약 이름, 함수 이름 및 변수 이름)는 ASCII character set으로 제한됩니다. 문자열 변수에 UTF-8로 인코딩된 데이터를 저장할 수 있습니다.
<br><b>Note End</b>

<b>Warning</b>
<br>비슷한 모양의(또는 심지어 같은) 문자는 다른 코드 포인트를 가질 수 있고 다른 바이트 배열로 인코딩되기 때문에 유니 코드 텍스트를 사용할 때 주의하십시오.
<br><b>Warnig End</b>

### Subcurrency Example

다음 계약은 가장 간단한 형태의 암호화화폐를 구현합니다. 
아무 것도 없는 상태에서 코인을 생성하는 것은 가능하지만 계약을 만든 사람만이 코인을 발행할 수 있습니다 (다른 발행 계획을 구현하는 것은 간단합니다). 
또한, 누구든지 Ethereum keypair가 있다면 사용자 이름과 암호를 등록할 필요 없이 코인을 보낼 수 있습니다.

~~~~
pragma solidity ^0.4.20; // should actually be 0.4.21

contract Coin {
    // "public" 키워드는 외부에서 변수를 읽을 수 있도록 합니다. 
    address public minter;
    mapping (address => uint) public balances;

    // "event"는 라이트 클라이언트가 변화에 효율적으로 반응할 수 있도록 합니다.
    event Sent(address from, address to, uint amount);

    // 이것은 컨스트럭터(생성자)로 계약이 생성된 후에만 작동 가능합니다.
    function Coin() public {
        minter = msg.sender;
    }

    function mint(address receiver, uint amount) public {
        if (msg.sender != minter) return;
        balances[receiver] += amount;
    }

    function send(address receiver, uint amount) public {
        if (balances[msg.sender] < amount) return;
        balances[msg.sender] -= amount;
        balances[receiver] += amount;
        emit Sent(msg.sender, receiver, amount);
    }
}
~~~~

이 계약은 몇 가지 새로운 개념을 소개하고 있습니다. 자세히 보겠습니다.

<tt style="color: #FF0000">`address public minter;`</tt> 공개적으로 액세스할 수 있는 유형의 address 변수를 선언합니다. 
address 타입은 산술 연산을 허용하지 않는 160 비트 값입니다. 이 타입은 계약 주소나 외부 사람의 Ethereum 주소를 저장하는 데 적합합니다. 
public 키워드는 계약 외부에서 상태 변수의 현재 값에 액세스할 수 있는 함수를 자동으로 생성합니다. 
이 키워드가 없으면 다른 계약에서 변수에 액세스할 수 없으며 오직 코드 작성자만이 계약에 엑세스할 수 있습니다. 컴파일러에 의해 생성된 함수의 코드는 대략 다음과 같습니다.

~~~~
function minter() returns (address) { return minter; }
~~~~

물론 위와 같은 함수를 추가한다고 해서 원하는 데로 작동하지는 않을 것입니다. 
왜냐하면 같은 이름의 함수와 상태 변수를 가져야 하기 때문입니다. 하지만 컴파일러로부터 아이디어를 얻을 수 있습니다.

다음 줄인 <tt style="color: #FF0000">`address public minter;`</tt> 또한 public 상태 변수를 만들지만 더 복잡한 데이터 타입입니다. 
이 타입은 주소를 부호가 없는 정수로 매핑합니다. 매핑은 모든 가능한 키가 존재하도록 사실상 초기화되고 바이트 표현이 모두 0인 값에 매핑되는 해시 테이블로 
볼 수 있습니다. 맵핑의 모든 키 목록이나 모든 값의 목록을 얻을 수도 없기 때문에 이 유추를 너무 확대 해석하지 않는 것이 좋습니다. 
따라서 맵핑에 추가한 내용을 기억하거나 (또는 더 나은 데이터 타입을 사용하거나 목록을 보하는 것이 좋습니다.) 이와 같은 작업이 필요 없는 환경에서 사용하십시오. 
<tt style="color: #FF0000">`public`</tt> 키워드에 의해 생성된 [getter function](https://solidity.readthedocs.io/en/latest/contracts.html#getter-functions) 이 경우 좀 더 복잡합니다. 대략 다음과 같습니다.

~~~~
function balances(address _account ) returns (uint balance) {
    return balances[_account]
}
~~~~

보시다시피, 이 함수를 사용하면 하나의 계정의 잔액을 쉽게 쿼리할 수 있습니다.

<tt style="color: #FF0000">`event Sent(address from, address to, uint amount);`</tt> 행은 소위 "이벤트"라고 불리며 <tt style="color: #FF0000">`send`</tt> 함수의 마지막에 호출됩니다. 
사용자 인터페이스(물론 서버 어플리케이션 포함)는 많은 비용을 들이지 않고 블록 체인에서 호출되는 이벤트를 수신할 수 있습니다. 
그것이 호출 되자마자 리스너는 <tt style="color: #FF0000">`from, to`</tt> 및 <tt style="color: #FF0000">`amount`</tt>를 인수로 받아 트랜잭션을 추적하기 쉽습니다. 이 이벤트를 청취하려면 다음을 사용해야 합니다.

~~~~
Coin.Sent().watch({}, '', function(error, result) {
    if (!error) {
        console.log("Coin transfer: " + result.args.amount +
            " coins were sent from " + result.args.from +
            " to " + result.args.to + ".");
        console.log("Balances now:\n" +
            "Sender: " + Coin.balances.call(result.args.from) +
            "Receiver: " + Coin.balances.call(result.args.to));
    }
})
~~~~

자동 생성된 기능 <tt style="color: #FF0000">`balance`</tt>가 사용자 인터페이스에서 어떻게 호출되는지 주목하십시오.

특별한 함수 <tt style="color: #FF0000">`Coin`</tt>은 계약 생성 중에 실행되는 생성자이며 나중에 호출 할 수 없습니다. 계약을 생성하는 사람의 주소를 영구적으로 저장합니다. 
<tt style="color: #FF0000">`msg`</tt> (<tt style="color: #FF0000">`tx`</tt> 및 <tt style="color: #FF0000">`block`</tt>과 함께)는 블록 체인에 대한 액세스를 허용하는 일부 속성을 포함하는 매직 전역 변수입니다. 
<tt style="color: #FF0000">`msg.sender`</tt>는 항상 이 함수의 호출이 어디서 시작되었는지 알려주는 주소입니다.

마지막으로 실제로 계약으로 끝나고 사용자와 계약자가 모두 호출할 수 있는 함수는 <tt style="color: #FF0000">`mint`</tt>이며 <tt style="color: #FF0000">`send`</tt>입니다. 
<tt style="color: #FF0000">`mint`</tt>가 계약을 만든 계정을 제외한 다른 사람에 의해 호출되면 아무 일도 일어나지 않습니다. 
다른 한편으로는, <tt style="color: #FF0000">`send`</tt>는 다른 사람에게 coin을 보내는 누군가 (이미이 coin들의 일부를 가지고 있음)가 사용할 수 있습니다. 
이 계약을 사용하여 특정 주소로 coin을 보내면 coin을 보낸 사실과 변경된 잔액이 이 특정 coin 계약 데이터 저장소에만 저장되기 때문에 블록 체인 탐색기에서 그 주소를 볼 때 아무 것도 볼 수 없습니다. 
이러한 이벤트를 사용하면 새 동전의 거래와 잔액을 추적하는 블록 체인 탐색기를 만드는 것이 비교적 쉽습니다.


## Blockchain Basics

개념으로서의 블록체인 (blockchains)은 프로그래머에게 이해하기 너무 어렵지 않습니다.(저는 어렵습니다...) 
그 이유는 대부분의 복잡한 작업(채굴, [hashing](https://en.wikipedia.org/wiki/Cryptographic_hash_function), [elliptic-curve cryptography](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography), [peer-to-peer networks](https://en.wikipedia.org/wiki/Peer-to-peer) 등)은 특정 기능과 약속을 제공하기 위한 것일 뿐입니다. 
일단 이러한 기능을 받아들이면 기본 기술에 대해 걱정할 필요가 없습니다. 또는 Amazon의 AWS가 사용하기 위해 내부적으로 어떻게 작동 하는지를 알아야합니까?

### Transactions

블록체인은 전 세계적으로 공유되는 트랜잭션 데이터베이스입니다. 
즉, 모든 사람이 네트워크에 참여하는 것만으로도 데이터베이스를 읽을 수 있다는 것을 의미합니다. 데이터베이스에서 무언가를 변경하려면, 다른 모든 사용자가 수락해야 하는 소위 트랜잭션을 생성해야 합니다. 
트랜잭션이라는 단어는 변경하려는 값(동시에 두 값을 변경하려고한다고 가정)이 전혀 완료되지 않았거나 완전히 적용되었음을 의미합니다. 
또한 거래가 데이터베이스에 적용되는 동안 다른 거래가 이를 변경할 수 없습니다.

예를 들어 전자 통화로 모든 계정의 잔액을 나열하는 테이블을 생각해보십시오. 
한 계정에서 다른 계정으로의 이전이 요청되면 데이터베이스의 트랜잭션 특성으로 인해 한 계정에서 금액을 빼면 항상 다른 계정에 추가됩니다. 
어떤 이유로 든 대상 계정에 금액을 추가할 수 없으면 원본 계정도 수정되지 않습니다.

또한 트랜잭션은 항상 보낸 사람(작성자)이 암호화하여 서명합니다. 
이를 통해 데이터베이스의 특정 수정 사항에 대한 액세스를 보호할 수 있습니다. 
전자 화폐의 예에서 간단한 확인은 계좌로 열쇠를 가지고 있는 사람만 돈을 송금할 수 있도록합니다.

### Blocks

Bitcoin에서 극복해야 할 주요 장애물은 "이중 지출 공격(double-spend attack)"이라고 불리는 것입니다. 만약 두 가지 트랜잭션이 모두 계정을 비우기 원하는 소위 충돌하는 경우 어떻게 됩니까?

이것에 대한 추상적인 대답은 당신이 신경 쓸 필요가 없다는 것입니다. 
트랜잭션의 순서가 선택되며 트랜잭션이 "블록"으로 묶여서 모든 참여 노드간에 실행 및 분배됩니다. 
두 거래가 서로 모순되는 경우 두 번째 거래가 거부되고 차단 대상이되지 않습니다.

이 블록은 시간상 선형 시퀀스를 형성하며 여기서 "블록체인"이라는 단어가 파생되었습니다. 
블록이 규칙적인 간격으로 체인에 추가됩니다. Ethereum의 경우 약 17초마다 발생합니다.

"order selection mechanism"("마이닝"이라고 함)의 일환으로 블록이 수시로 복귀되지만 체인의 "tip"에서만 발생할 수 있습니다. 
상단에 더 많은 블록이 추가될수록 더 적습니다. 
따라서 트랜잭션이 되돌아와 블록체인에서 제거될 수도 있지만 대기 시간이 길어질수록 가능성은 줄어듭니다. 

## The Ethereum Virtual Machine

### Overview

Ethereum 가상 머신 또는 EVM은 Ethereum의 스마트 컨트랙트를 위한 런타임 환경입니다. 
샌드 박스가 아니라 실제로 완전히 격리되어 EVM 내부에서 실행되는 코드는 네트워크, 파일 시스템 또는 기타 프로세스에 액세스할 수 없습니다. 
스마트 컨트랙트은 다른 스마트 컨트랙트에 대한 액세스를 제한합니다.

### Accounts

Ethereum에는 동일한 주소 공간을 공유하는 두 종류의 계정이 있습니다. public-private pair 키(예 : 사람)가 제어하는 <b>external accounts</b> 계정과 함께 저장된 코드로 제어되는 <b>contract accounts</b>.

external accounts의 주소는 공개 키로 결정되며 contract 주소는 계약서가 생성될 때 결정됩니다(작성자 주소와 해당 주소에서 전송된 거래 수, 즉 소위 "nonce").

계정에 코드가 저장되어 있는지 여부에 관계없이 두 유형은 EVM에서 동일하게 처리됩니다.

모든 계정에는 256비트 단어를 저장소라는 256비트 단어로 매핑하는 영구 키 값 <b>storage</b>가 있습니다.

게다가, 모든 계정은 Ether 잔액을 갖고 있고,(정확하게 "Wei"에서) Ether을 포함하는 트랜잭션을 전송하여 수정할 수 있습니다.

### Transactions

거래란 한 계정에서 다른 계정으로 전송되는 메시지입니다(동일 또는 특별 zero-account, 아래 참조). 바이너리 데이터(페이로드)와 Ether를 포함할 수 있습니다.

대상 계정에 코드가 포함되어 있으면 해당 코드가 실행되고 페이로드가 입력 데이터로 제공됩니다.

대상 계정이 zero-account(주소가 <tt style="color: #FF0000">`0`</tt>인 계정)인 경우 트랜잭션은 <b>new contract</b>을 작성합니다. 
이미 언급했듯이 계약서의 주소는 0번지가 아니라 보낸 사람의 주소와 보낸 거래 수("nonce")입니다. 
이러한 계약 생성 트랜잭션의 페이로드는 EVM 바이트 코드로 간주되어 실행됩니다. 
이 실행 결과는 계약 코드로 영구 저장됩니다. 즉, 계약서를 작성하려면 계약서의 실제 코드를 보내지 않고 해당 코드를 반환하는 코드를 보내야합니다.

### GAS

생성시, 각 거래(transaction)에는 일정 금액의 가스가 부과됩니다. 
이 목적은 거래를 실행하고 이 실행 비용을 지불하는 데 필요한 작업량을 제한하는 것입니다. EVM이 트랜잭션을 실행하는 동안 특정 규칙에 따라 가스가 점차적으로 고갈됩니다.

가스 가격은 거래를 생성한 사람이 정한 값으로, 보내는 계정에서 <tt style="color: #FF0000">`gas_price * gas`</tt>를 먼저 지불해야합니다. 실행 후 일부 가스가 남은 경우 동일한 방식으로 환불됩니다.

어떤 지점에서 가스가 다 소모된 경우(즉, 음수인 경우), 가스 외부 예외가 발생하여 현재 호출 프레임에서 상태에 대한 모든 수정 사항을 되돌립니다.

### Storage, Memory and the Stack

각 계정에는 storage(저장소)라고 하는 영구 메모리 영역이 있습니다. 
저장소는 256비트 단어를 256비트 단어로 매핑하는 키 값 저장소입니다. 
계약내에서 스토리지를 열거할 수는 없으며 스토리지를 수정하는데 비교적 많은 비용이 소요됩니다. 
계약은 자신의 스토리지와는 별도로 스토리지를 읽거나 쓸 수 없습니다.

두 번째 메모리 영역은 메모리라고하며 계약 중 하나는 각 메시지 호출에 대해 새로 지워진 인스턴스를 가져옵니다. 
메모리는 선형이며 바이트 단위로 주소 지정이 가능하지만 읽기는 256비트의 폭으로 제한되며 쓰기는 8비트 또는 256비트 폭으로 제한될 수 있습니다. 
메모리는 이전에 변경되지 않은 메모리 단어(즉, 단어 내의 오프셋)에 액세스(읽기 또는 쓰기)할 때 단(256 비트)로 확장됩니다. 
확장시 가스 비용이 지불되어야합니다. 메모리는 커질수록 비용이 많이 들고(2차로 크기가 조정됨) 더 커집니다.

EVM은 register machine이 아니라 스택 머신이므로 모든 계산은 스택이라고하는 영역에서 수행됩니다. 
최대 크기는 1024이고 요소는 256비트입니다. 스택에 대한 액세스는 다음과 같은 방식으로 최상단으로 제한됩니다. 
최상위 16개 요소 중 하나를 스택 맨 위로 복사하거나 최상위 요소를 그 아래에 있는 16개 요소 중 하나와 바꿀 수 있습니다. 
다른 모든 연산은 스택의 최상위 2개(또는 연산에 따라 하나 이상) 요소를 가져와서 결과를 스택으로 푸시합니다. 
물론 스택 요소를 저장소나 메모리로 이동할 수도 있지만 스택의 맨 처음을 제거하지 않고 스택에서 더 깊은 임의의 요소에 액세스하는 것은 불가능합니다.

### Instruction Set

컨센서스 문제를 야기할 수 있는 잘못된 구현을 피하기 위해 EVM의 명령어 세트는 최소한으로 유지됩니다. 
모든 명령어는 기본 데이터 유형인 256비트 단어에 대해 작동합니다. 
일반적인 산술, 비트, 논리 및 비교 연산이 있습니다. 조건적 및 무조건 부 점프가 가능합니다.
또한 계약은 번호 및 타임 스탬프와 같이 현재 블록의 관련 속성에 액세스할 수 있습니다.

### Message Calls

계약서는 다른 계약서를 호출하거나 메시지 호출 수단을 통해 Ether를 계약되지 않은 계좌로 보낼 수 있습니다. 
메시지 호출은 소스, 대상, 데이터 페이로드, Ether, 가스 및 리턴 데이터가 있다는 점에서 트랜잭션과 유사합니다. 
사실, 모든 트랜잭션은 최상위 메시지 호출로 구성되며 이 메시지 호출은 추가 메시지 호출을 생성할 수 있습니다.

계약에 따라 내부 메시지 호출로 남아있는 가스의 양과 보유하고자 하는 금액을 결정할 수 있습니다. 
내부 호출(또는 다른 예외)에서 out-of-gas 예외가 발생하면 스택에 넣은 오류 값으로 신호를 보냅니다. 
이 경우 call과 함께 보내는 가스만 소모됩니다. 
Solidity에서 호출하는 계약은 이러한 상황에서 기본적으로 수동 예외를 발생시키므로 예외가 호출 스택을 "버블 업"합니다.

이미 말한 것처럼 호출된 계약(호출자와 동일할 수 있음)은 새로 삭제된 메모리 인스턴스를 수신하고 호출 페이로드에 액세스할 수 있습니다.
이 호출 페이로드는 calldata라고하는 별도의 영역에서 제공됩니다. 실행이 끝나면 호출자가 미리 할당한 호출자의 메모리 위치에 저장될 데이터를 반환할 수 있습니다.

호출은 depth가 1024로 제한되어 있으므로 복잡한 작업의 경우 반복 호출보다 루프가 우선되어야합니다.

### Delegatecall / Callcode and Libraries

<b>delegatecall</b>이라는 메시지 호출의 특수한 변형이 있습니다.
이 호출은 대상 주소의 코드가 호출 계약의 컨텍스트에서 실행되고 <tt style="color: #FF0000">`msg.sender`</tt>와 <tt style="color: #FF0000">`msg.value`</tt>가 변경되지 않는다는 점을 제외하고는 메시지 호출과 동일합니다.

이는 계약이 런타임에 다른 주소의 코드를 동적으로 로드할 수 있음을 의미합니다. 저장소, 현재 주소 및 잔액은 여전히 호출 계약을 참조하며 코드는 호출된 주소에서만 가져옵니다.

이를 통해 계약 저장소에 적용할 수 있는 재사용 가능한 라이브러리 코드(예: 복잡한 데이터 구조를 구현하기 위한 목적)의 "라이브러리"기능을 Solidity로 구현할 수 있습니다.

### Logs

블록 레벨까지 모든 방법으로 매핑되는 특별히 인덱싱된 데이터 구조에 데이터를 저장할 수 있습니다. 
로그라는 이 기능은 이벤트를 구현하기 위해 Solidity에서 사용합니다. 
계약은 로그 데이터를 만든 후에는 액세스할 수 없지만 블록체인 외부에서 효율적으로 액세스할 수 있습니다. 
로그 데이터의 일부는 [블룸 필터](https://en.wikipedia.org/wiki/Bloom_filter)에 저장되기 때문에 효율적이고 암호학적으로 안전한 방법으로 이 데이터를 검색할 수 있으므로 전체 블럭체인("라이트 클라이언트")을 다운로드하지 않는 네트워크 피어는 여전히 이러한 로그를 찾을 수 있습니다 .

### Create

계약서는 특수한 연산 코드를 사용하여 다른 계약을 생성할 수도 있습니다(즉, 단순히 제로 주소를 호출하는 것이 아닙니다). 
이러한 생성 호출과 일반 메시지 호출간의 유일한 차이점은 페이로드 데이터가 실행되고 결과가 코드로 저장되고 호출자/작성자가 스택에서 새 계약의 주소를 수신한다는 것입니다.

### Self-destruct

코드가 블록 체인에서 제거될 수 있는 유일한 가능성은 해당 주소의 계약서에서 <tt style="color: #FF0000">`selfdestruct`</tt> 작업을 수행할 때 뿐입니다. 
해당 주소에 저장된 나머지 Ether가 지정된 대상으로 보내진 다음 저장소에서 코드가 제거됩니다.

<b>Warning</b>
<br>계약의 코드에 <tt style="color: #FF0000">`selfdestruct`</tt>에 대한 호출이 없더라도 <tt style="color: #FF0000">`delegatecall`</tt> 또는 <tt style="color: #FF0000">`callcode`</tt>를 사용하여 해당 작업을 수행할 수 있습니다.
<br>Warning End</b>

<b>Note</b>
<br>Ethereum 클라이언트는 오래된 계약을 제거할 수도 있고 구현하지 않을 수도 있습니다. 또한 아카이브 노드는 계약 저장 및 코드를 무기한 보관하도록 선택할 수 있습니다.
<br><b>Note End</b>

<b>Note</b>
<br>현재 외부 계정은 상태에서 삭제할 수 없습니다.
<br><b>Note End</b>