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
모든 식별자(계약 이름, 함수 이름 및 변수 이름)는 ASCII character set으로 제한됩니다. 문자열 변수에 UTF-8로 인코딩된 데이터를 저장할 수 있습니다.
<b>Note End</b>

<b>Warning</b>
비슷한 모양의(또는 심지어 같은) 문자는 다른 코드 포인트를 가질 수 있고 다른 바이트 배열로 인코딩되기 때문에 유니 코드 텍스트를 사용할 때 주의하십시오.
<b>Warnig End</b>

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

이 부분은 Structure of a Contract 부분의 번역이 완료되면 이어서 작성하겠습니다 :)