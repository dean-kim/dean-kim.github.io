---
layout: post
title:  "Solidity Security Considerations"
date:   2018-04-03 20:43:59
author: Dean Kim
categories: BlockChain
tags:	BlockChain Solidity Security Considerations
cover:  "/assets/instacode.png"
---

블록체인을 공부하며 Smart Contract 작성에 사용되는 Solidity 공식문서를 번역한 글입니다.
제가 이해한 수준으로 기술된 내용도 있어 원본과 내용이 조금 다를 수도 있습니다.
혹시 제가 잘못 이해한 내용이 있다면 말씀해주세요 :)

학습의 목적으로 번역을 해서 아래의 예제 코드를 설명하는 주석을 공식문서보다 자세하게 작성하였습니다.
이에 대해서도 잘못 작성된 내용은 말씀해주세요 :)

# Security Considerations

예상대로 작동하는 소프트웨어를 만드는 것은 일반적으로 쉽지만 예상하지 못한 방식으로 소프트웨어를 사용할 수 있는지 여부를 확인하는 것은 더 어렵습니다.

Solidity에서는 스마트 계약을 사용하여 토큰 또는 아마도 더 가치있는 것을 처리할 수 ​​있기 때문에 더욱 중요합니다. 
더욱이 스마트 계약의 모든 실행은 공개적으로 이루어지며 그 외에도 소스 코드가 종종 제공됩니다.

당연히 당신은 항상 얼마나 위험한지를 고려해야합니다. 스마트 계약을 공개(따라서 악의적인 행위자에게도 공개됨)된 웹서비스 그리고 심지어 오픈 소스까지 비교할 수 있습니다. 
웹 서비스에만 식료품 목록을 저장하는 경우 너무 신경 쓸 필요는 없겠지만 해당 웹 서비스를 사용하여 은행 계좌를 관리하는 경우 더 신중해야합니다.

이 섹션에는 몇 가지 위험요소와 일반적인 보안 권장 사항이 나열되어 있지만 물론 결코 완전하지는 않습니다. 
또한 스마트 계약 코드에 버그가 없더라도 컴파일러 또는 플랫폼 자체에 버그가 있을 수 있습니다. 
컴파일러의 보안 관련 버그 목록은 컴퓨터에서 읽을 수 있는 알려진 버그 목록에서 찾을 수 있습니다. 
Solidity 컴파일러의 코드 생성기를 다루는 버그 보상 프로그램이 있습니다.

항상 그렇듯이 오픈 소스 문서를 통해 이 섹션을 확장하는 데 도움을 주십시오(특히 몇 가지 예는 손상되지 않을 것입니다)!

## Pitfalls

### Private Information and Randomness

스마트 계약에서 사용하는 모든 것은 공개적으로 볼 수 있으며 로컬 변수 및 상태 변수도 <tt style="color: #FF0000">`private`</tt>로 표시됩니다.

채굴자들이 속임수를 쓸 수 없도록 하려면 스마트 계약에서 난수를 사용하는 것은 매우 까다롭습니다.

### Re-Entrancy

계약(A)에서 다른 계약(B)으로의 모든 상호 작용과 Ether의 양도가 해당 계약(B)에 대한 통제권을 넘깁니다. 
이렇게하면 이 상호 작용이 완료되기 전에 B가 A로 다시 콜백할 수 있습니다. 
예를 들어, 다음 코드는 버그를 포함하고 있습니다(단순한 코드 일뿐 완전한 계약이 아닙니다).

~~~~
pragma solidity ^0.4.0;

// THIS CONTRACT CONTAINS A BUG - DO NOT USE
contract Fund {
    /// Mapping of ether shares of the contract.
    mapping(address => uint) shares;
    /// Withdraw your share.
    function withdraw() public {
        if (msg.sender.send(shares[msg.sender]))
            shares[msg.sender] = 0;
    }
}
~~~~

이 문제는 <tt style="color: #FF0000">`send`</tt>의 일부로 제한된 가스 때문에 심각하지는 않지만 여전히 약점을 드러내고 있습니다. 
이더 전송에는 항상 코드 실행이 포함될 수 있으므로 수신자가 <tt style="color: #FF0000">`withdraw`</tt>하도록 요청하는 계약이 될 수 있습니다. 
이렇게하면 복수의 환불을 받을 수 있으며 기본적으로 계약에서 모든 이더를 회수할 수 있습니다. 
특히 다음 계약을 통해 공격자는 남은 모든 가스를 기본적으로 전달하는 <tt style="color: #FF0000">`call`</tt>을 사용할 때 여러 번 환불할 수 있습니다.

~~~~
pragma solidity ^0.4.0;

// THIS CONTRACT CONTAINS A BUG - DO NOT USE
contract Fund {
    /// Mapping of ether shares of the contract.
    mapping(address => uint) shares;
    /// Withdraw your share.
    function withdraw() public {
        if (msg.sender.call.value(shares[msg.sender])())
            shares[msg.sender] = 0;
    }
}
~~~~

재전송을 방지하려면 아래에 설명된대로 Checks-Effects-Interactions 패턴을 사용할 수 있습니다.

~~~~
pragma solidity ^0.4.11;

contract Fund {
    /// Mapping of ether shares of the contract.
    mapping(address => uint) shares;
    /// Withdraw your share.
    function withdraw() public {
        var share = shares[msg.sender];
        shares[msg.sender] = 0;
        msg.sender.transfer(share);
    }
}
~~~~

재전송은 반복 전송의 영향일 뿐 아니라 다른 계약에 대한 모든 기능의 영향이라는 점에 유의하십시오. 또한 다중 계약 상황을 고려해야 합니다. 호출된 계약으로 사용자가 종속된 다른 계약의 상태를 수정할 수 있습니다.

### Gas Limit and Loops

예를 들어 스토리지 값에 의존하는 루프와 같이 반복 횟수가 고정되어 있지 않은 루프는 신중하게 사용해야합니다. 
블록 가스 한도로 인해 트랜잭션은 일정량의 가스만 소비할 수 있습니다. 
명시적으로 또는 정상적인 작동으로 인해 루프의 반복 횟수가 블록 가스 한도를 초과하여 증가하여 특정 계약 시점에 전체 계약이 지연될 수 있습니다. 
블록 체인에서 데이터를 읽는 경우에만 실행되는 <tt style="color: #FF0000">`constant`</tt> 함수에는 적용되지 않을 수 있습니다. 
그러나 이러한 기능은 체인상의 작업의 일부로 다른 계약에 의해 호출될 수 있으며 이를 중지시킬 수 있습니다. 
귀하의 계약서에 그러한 사례를 명시하십시오.

### Sending and Receiving Ether

* 계약이나 "외부 계좌"는 현재 누군가가 Ether을 보내는 것을 막을 수 없습니다. 
계약에 따라 정기적인 전송에 반응하고 거부할 수 있지만 메시지 호출을 만들지 않고 Ether을 이동할 수 있는 방법이 있습니다. 
한 가지 방법은 계약 주소를 단순히 "채굴"하는 것이고 두 번째 방법은 selfdestruct(x)를 사용하는 것입니다.
* 계약이 Ether를 수신하면(함수 호출없이) 대체 기능이 실행됩니다. 
대체 기능이 없으면 Ether가 거부됩니다(예외가 발생 함). 
폴백 기능 실행 중에 계약은 그 당시에 사용할 수 있는 "gas stipend"(2300 가스)에만 의존할 수 있습니다. 
이 stipend는 어떤 방식으로든 스토리지에 액세스하기에 충분하지 않습니다. 
당신의 계약이 그런 방식으로 Ether을 받을 수 있는지 확인하기 위해 대체 기능의 가스 요구 사항을 확인하십시오(예 : Remix의 "details"섹션 참조).
* <tt style="color: #FF0000">`addr.call.value(x)()`</tt>를 사용하여 수신 계약에 더 많은 가스를 전달할 수 있는 방법이 있습니다. 
이것은 기본적으로 <tt style="color: #FF0000">`addr.transfer(x)`</tt>와 동일하며 나머지 모든 가스를 전달하고 수신자가 보다 비싼 작업을 수행할 수 있는 능력을 열어줍니다(오류 코드만 반환하고 오류를 자동으로 전파하지는 않습니다). 
여기에는 전송 계약에 대한 호출이나 생각하지 못했던 다른 상태 변경이 포함될 수 있습니다. 따라서 정직한 사용자는 물론 악의적인 행위자에게도 큰 유연성을 제공합니다.
* <tt style="color: #FF0000">`address.transfer`</tt>를 사용하여 Ether을 보내려는 경우, 다음 사항에 유의해야 합니다.
<br>1. 수신자가 계약인 경우 폴백 기능이 실행되어 차례로 송신 계약을 콜백할 수 있습니다.
<br>2. Ether 전송은 1024보다 큰 콜 depth로 인해 실패할 수 있습니다. 
발신자가 콜 depth를 완전히 제어하고 있기 때문에 전송이 실패할 수 있습니다.
 이 가능성을 고려하거나 <tt style="color: #FF0000">`send`</tt>를 사용하고 항상 반환 값을 확인하십시오. 
 더 나아가, 수령인이 Ether를 대신 회수할 수있는 패턴을 사용하여 계약서를 작성하십시오.
<br>3. Ether 전송은 또한 수신 계약의 실행이 할당된 양 이상의 가스를 필요로 하기 때문에 실패할 수 있습니다.
(<tt style="color: #FF0000">`require`</tt>, <tt style="color: #FF0000">`assert`</tt>, <tt style="color: #FF0000">`revert`</tt>, <tt style="color: #FF0000">`throw`</tt> 또는 조작이 너무 비싸기 때문에 명시적으로 사용합니다.) "가스가 부족합니다"(OOG). 
전송 또는 반송 값 확인을 사용하여 전송하는 경우 받는 사람이 전송 계약의 진행을 차단할 수 있는 방법을 제공할 수 있습니다. 
여기서도 가장 좋은 방법은 ["send"패턴 대신 "withdraw"패턴을 사용하는 것입니다.](http://solidity.readthedocs.io/en/latest/common-patterns.html#withdrawal-pattern)

### Callstack Depth

외부 함수 호출은 최대 호출 스택 1024를 초과해서 언제든지 실패할 수 있습니다. 
이러한 상황에서는 Solidity가 예외를 throw합니다. 악의적인 행위자는 계약과 상호 작용하기 전에 호출 스택을 높은 값으로 강제 설정할 수 있습니다.

호출 스택이 고갈된 경우 <tt style="color: #FF0000">`.send()`</tt>가 예외를 throw하지 않지만 이 경우에는 <tt style="color: #FF0000">`false`</tt>를 반환합니다. 
저수준 함수 <tt style="color: #FF0000">`.call()`</tt>, <tt style="color: #FF0000">`.callcode()`</tt> 및 <tt style="color: #FF0000">`.delegatecall()`</tt>도 같은 방식으로 작동합니다.

### tx.origin

tx.origin을 인증에 사용하지 마십시오. 다음과 같은 지갑 계약을 체결했다고 가정해 보겠습니다.

~~~~
pragma solidity ^0.4.11;

// THIS CONTRACT CONTAINS A BUG - DO NOT USE
contract TxUserWallet {
    address owner;

    function TxUserWallet() public {
        owner = msg.sender;
    }

    function transferTo(address dest, uint amount) public {
        require(tx.origin == owner);
        dest.transfer(amount);
    }
}
~~~~

이제 누군가가 이 공격 지갑의 주소로 ether를 전송하도록 속이므로 :

~~~~
pragma solidity ^0.4.11;

interface TxUserWallet {
    function transferTo(address dest, uint amount) public;
}

contract TxAttackWallet {
    address owner;

    function TxAttackWallet() public {
        owner = msg.sender;
    }

    function() public {
        TxUserWallet(msg.sender).transferTo(owner, msg.sender.balance);
    }
}
~~~~

지갑에서 인증을 위해 <tt style="color: #FF0000">`msg.sender`</tt>를 확인한 경우 소유자 주소 대신 공격 지갑의 주소가 표시됩니다. 
그러나 <tt style="color: #FF0000">`tx.origin`</tt>을 확인하면 트랜잭션을 시작한 원래 주소를 가져옵니다. 이 주소는 여전히 소유자 주소입니다. 공격 지갑은 즉시 당신의 모든 자금을 소모합니다.

### Minor Details

* <tt style="color: #FF0000">`for (var i = 0; i <arrayName.length; i ++) {...}`</tt>는 값 0을 유지하는 데 필요한 가장 작은 유형이므로 <tt style="color: #FF0000">`i`</tt> 유형은 <tt style="color: #FF0000">`uint8`</tt>이 됩니다. 255개 이상의 요소가 있으면 루프가 종료되지 않습니다.
* 함수의 <tt style="color: #FF0000">`constant`</tt> 키워드는 현재 컴파일러에 의해 적용되지 않습니다. 또한 EVM에 의해 적용되지 않으므로 일정하다고 주장하는 계약 기능이 여전히 상태를 변경시킬 수 있습니다.
* 전체 32 바이트를 차지하지 않는 유형에는 "dirty higher order bits"가 포함될 수 있습니다. 이는 <tt style="color: #FF0000">`msg.data`</tt>에 액세스할 때 특히 중요합니다. 즉, 가단성 위험이 있습니다. <tt style="color: #FF0000">`0xff000001`</tt> 및 <tt style="color: #FF0000">`0x00000001`</tt>의 원시 바이트 인수를 사용하여 함수 <tt style="color: #FF0000">`f(uint8 x)`</tt>를 호출하는 트랜잭션을 생성할 수 있습니다. 
둘 다 계약에 공급되며 <tt style="color: #FF0000">`x`</tt>에 관한한 둘 다 숫자<tt style="color: #FF0000">`1`</tt>처럼 보이지만, <tt style="color: #FF0000">`msg.data`</tt>는 다를 수 있으므로 <tt style="color: #FF0000">`keccak256(msg.data)`</tt>을 사용하면 다른 결과를 얻게됩니다.

## Recommendations

### Restrict the Amount of Ether

smart contract에 저장할 수 있는 Ether(또는 다른 토큰)의 양을 제한하십시오. 소스 코드, 컴파일러 또는 플랫폼에 버그가 있는 경우 이러한 자금이 손실될 수 있습니다. 손실을 줄이려면 Ether의 양을 제한하십시오.

### Keep it Small and Modular

계약을 작고 쉽게 이해할 수 있게 하십시오. 다른 계약이나 라이브러리에서 관련이 없는 기능을 제거하십시오. 
물론 소스 코드의 품질에 대한 일반적인 권장 사항 : 지역 변수의 양, 함수의 길이 등을 제한하십시오. 
다른 사람들이 당신의 의도가 무엇이고 코드가 하는 것과 다른지를 볼 수 있도록 함수를 문서화하십시오.

### Use the Checks-Effects-Interactions Pattern

대부분의 함수는 먼저 몇 가지 검사를 수행합니다(함수를 호출한 사람, 범위 내의 인수, 충분한 Ether를 그들이 보냈는지, 토큰이 있는 사람 등). 이러한 점검이 먼저 수행되어야 합니다.

두 번째 단계로서, 모든 검사들이 통과되면 현재 계약의 상태 변수에 영향을 주어야 합니다. 다른 계약과의 상호 작용은 모든 기능의 마지막 단계여야 합니다.

조기 계약은 일부 효과를 지연시켰고 외부 함수 호출이 오류가 없는 상태로 돌아오기를 기다립니다. 위에 설명된 재진입 문제 때문에 종종 심각한 실수가 됩니다.

알려진 계약을 호출하면 알 수 없는 계약이 호출될 수 있으므로 항상 이 패턴을 적용하는 것이 좋습니다.

### Include a Fail-Safe Mode

시스템을 완전히 분산시켜서 중개자를 제거하는 동안, 특히 새로운 코드의 경우, 일종의 fail-safe(오류 방지) 메커니즘을 포함시키는 것이 좋습니다.

"임의의 이더가 유출 되었습니까?", "토큰의 합계가 계약의 잔액과 같은가요?"또는 이와 유사한 것들을 자기 점검으로 수행하는 스마트 계약서에 기능을 추가할 수 있습니다. 
너무 많은 가스를 사용할 수는 없으므로 오프 체인 계산을 통한 도움이 필요할 수도 있습니다.

자가 점검이 실패하면 계약은 자동으로 일종의 "fail-safe"모드로 전환됩니다. 이 모드는 대부분의 기능을 비활성화하고 고정된 신뢰할 수 있는 제 3자에게 제어권을 넘겨 주거나 계약을 간단한 "give me back my money" 계약으로 바꿉니다.

### Formal Verification

형식 검증을 사용하면 소스 코드가 특정 형식 스펙을 충족시키는 자동 수학적 증명을 수행할 수 있습니다. 명세는 여전히 공식적이지만(소스 코드와 동일), 대개 훨씬 간단합니다.

공식 검증 자체는 당신이 수행한 작업(규격)과 어떻게 했는지(실제 구현) 간의 차이점을 이해하는 데에만 도움이 될 뿐입니다. 그래도 원하는 사양이 맞는지, 의도하지 않은 효과를 놓치지 않았는지 확인해야 합니다.