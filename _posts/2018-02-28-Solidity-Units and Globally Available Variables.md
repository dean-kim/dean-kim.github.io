---
layout: post
title:  "Solidity Units and Globally Available Variables"
date:   2018-02-28 18:43:59
author: Dean Kim
categories: BlockChain
tags:	BlockChain Solidity Smart_Contracts
cover:  "/assets/instacode.png"
---

블록체인을 공부하며 Smart Contract 작성에 사용되는 Solidity 공식문서를 번역한 글입니다.
제가 이해한 수준으로 기술된 내용도 있어 원본과 내용이 조금 다를 수도 있습니다.
혹시 제가 잘못 이해한 내용이 있다면 말씀해주세요 :)

# Units and Globally Available Variables

## Ether Units

리터럴 번호는 <tt style="color: #FF0000">`wei`</tt>, <tt style="color: #FF0000">`finney`</tt>, <tt style="color: #FF0000">`szabo`</tt> 또는 <tt style="color: #FF0000">`ether`</tt>의 접미사를 사용하여 Ether의 부속 항목 사이에서 변환할 수 있습니다. 
여기서, 후위가 없는 Ether 통화 번호는 Wei로 가정됩니다. <tt style="color: #FF0000">`2 ether == 2000 finney`</tt>는 <tt style="color: #FF0000">`true`</tt>로 평가합니다.

## Time Units

문자 숫자가 <tt style="color: #FF0000">`seconds`</tt>, <tt style="color: #FF0000">`minutes`</tt>, <tt style="color: #FF0000">`hours`</tt>, <tt style="color: #FF0000">`days`</tt>, <tt style="color: #FF0000">`weeks`</tt> 그리고 <tt style="color: #FF0000">`years`</tt> 단위와 같은 접미사는 초 단위가 기본 단위이고 시간 단위가 다음과 같은 방법으로 순수하게 간주되는 시간 단위간에 변환하는 데 사용할 수 있습니다.
* <tt style="color: #FF0000">`1 == 1 seconds`</tt>
* <tt style="color: #FF0000">`1 minutes == 60 seconds`</tt>
* <tt style="color: #FF0000">`1 hours == 60 minutes`</tt>
* <tt style="color: #FF0000">`1 days == 24 hours`</tt>
* <tt style="color: #FF0000">`1 weeks == 7 days`</tt>
* <tt style="color: #FF0000">`1 years == 365 days`</tt>

이러한 단위를 사용하여 캘린더 계산을 수행할 경우 주의해야 합니다. [윤초(leap second)](http://terms.naver.com/entry.nhn?docId=855977&cid=42346&categoryId=42346)으로 인해 매해가 365일과 같지 않고 심지어 하루도 매일이 24시간과 같지 않기 때문입니다.
윤초를 예측할 수 없기 때문에 정확한 달력 라이브러리는 외부 오라클에 의해 업데이트되어야 합니다.

이러한 접미어는 변수에 적용할 수 없습니다. 예를 들어 며칠 내에 일부 입력 변수를 해석하려면 다음과 같은 방법으로 수행할 수 있습니다. :

~~~~
function f(uint start, uint daysAfter) public {
    if (now >= start + daysAfter * 1 days) {
      // ...
    }
}
~~~~

## Special Variables and Functions

글로벌 네임 스페이스에는 항상 존재하는 특수 변수와 함수가 있으며 주로 블록 체인에 대한 정보를 제공하는 데 사용됩니다.

### Block and Transaction Properties

* <tt style="color: #FF0000">`block.blockhash(uint blockNumber) returns (bytes32)`</tt>: 주어진 블록의 해시 - 현재를 제외한 256개의 가장 최근 블록에서만 작동합니다.
* <tt style="color: #FF0000">`block.coinbase`</tt> (<tt style="color: #FF0000">`address`</tt>): 현재 블록 채굴자의 주소
* <tt style="color: #FF0000">`block.difficulty`</tt> (<tt style="color: #FF0000">`uint`</tt>): 현재 블록 난이도
* <tt style="color: #FF0000">`block.gaslimit`</tt> (<tt style="color: #FF0000">`uint`</tt>): 현재 블록 gas limit
* <tt style="color: #FF0000">`block.number`</tt> (<tt style="color: #FF0000">`uint`</tt>): 현재 블록 넘버
* <tt style="color: #FF0000">`block.timestamp`</tt> (<tt style="color: #FF0000">`uint`</tt>): [유닉스 에폭](https://ko.wikipedia.org/wiki/%EC%9C%A0%EB%8B%89%EC%8A%A4_%EC%8B%9C%EA%B0%84) 이후의 현재 블록 타임 스탬프
* <tt style="color: #FF0000">`msg.data`</tt> (<tt style="color: #FF0000">`bytes`</tt>): complete calldata
* <tt style="color: #FF0000">`msg.gas`</tt> (<tt style="color: #FF0000">`uint`</tt>): 남은 gas
* <tt style="color: #FF0000">`msg.sender`</tt> (<tt style="color: #FF0000">`address`</tt>): 메시지 발신자(current call)
* <tt style="color: #FF0000">`msg.sig`</tt> (<tt style="color: #FF0000">`bytes4`</tt>): calldata의 처음 4바이트(즉, 함수 식별자)
* <tt style="color: #FF0000">`msg.value`</tt> (<tt style="color: #FF0000">`uint`</tt>): 메시지와 함께 전송된 wei
* <tt style="color: #FF0000">`now`</tt> (<tt style="color: #FF0000">`uint`</tt>): 현재 블록 타임 스탬프 (<tt style="color: #FF0000">`block.timestamp`</tt>의 별칭)
* <tt style="color: #FF0000">`tx.gasprice`</tt> (<tt style="color: #FF0000">`uint`</tt>): 트랜잭션에 사용되는 gas 비용
* <tt style="color: #FF0000">`tx.origin`</tt> (<tt style="color: #FF0000">`address`</tt>): 거래의 발신자(full call chain)

<b>Note</b>
<br><tt style="color: #FF0000">`msg.sender`</tt> 및 <tt style="color: #FF0000">`msg.value`</tt>를 포함하여 <tt style="color: #FF0000">`msg`</tt>의 모든 구성원 값은 모든 외부 함수 호출에 대해 변경될 수 있습니다. 여기에는 라이브러리 함수에 대한 호출이 포함됩니다.
<br><b>Note End</b>

<b>Note</b>
<br>자신이 하는 일을 모르는 경우 <tt style="color: #FF0000">`block.timestamp`</tt>, <tt style="color: #FF0000">`now`</tt> 및 <tt style="color: #FF0000">`block.blockhash`</tt>를 임의성의 소스로 사용하지 마십시오.

타임 스탬프와 블록 해시는 채굴자에 의해 영향을 받을 수 있습니다. 채굴자 커뮤니티의 나쁜 행위자는 선택한 해시에 카지노 지불 함수를 실행하고 돈을 받지 못한 경우 다른 해시를 다시 시도 할 수 있습니다.

현재 블록 타임 스탬프는 마지막 블록의 타임 스탬프보다 엄격히 커야하지만, 정식 체인에서 두 개의 연속 블록의 타임 스탬프 사이의 어딘가에 위치한다는 것만을 보장합니다.
<br><b>Note End</b>

<b>Note</b>
<br>블록 해시는 확장성을 이유로 모든 블록에서 사용할 수 없습니다. 가장 최근의 256블록의 해시에만 액세스 할 수 있으며 다른 모든 값은 0입니다.
<br><b>Note End</b>

### Error Handling

<tt style="color: #FF0000">`assert(bool condition)`</tt>:
<br>조건이 부합하지 않으면 내부 오류용으로 사용됩니다.

<tt style="color: #FF0000">`require(bool condition)`</tt>:
<br>조건이 충족되지 않는 경우 던집니다. 입력 또는 외부 구성 요소의 오류에 사용됩니다.

<tt style="color: #FF0000">`revert()`</tt>:
<br>실행을 중단하고 상태 변경을 되돌리기

### Mathematical and Cryptographic Functions

<tt style="color: #FF0000">`addmod(uint x, uint y, uint k) returns (uint)`</tt>:
<br>임의의 정확도로 덧셈을 수행하고 <tt style="color: #FF0000">`2**256`</tt>으로 래핑되지 않는 경우 <tt style="color: #FF0000">`(x + y) % k`</tt>를 계산합니다. <tt style="color: #FF0000">`k! = 0`</tt>를 버전 0.5.0부터 시작하는 것으로 가정합니다.

<tt style="color: #FF0000">`mulmod(uint x, uint y, uint k) returns (uint)`</tt>:
<br>임의의 정확도로 곱셈을 수행하고 <tt style="color: #FF0000">`2**256`</tt>으로 래핑되지 않는 경우 <tt style="color: #FF0000">`(x * y) % k`</tt>를 계산합니다. <tt style="color: #FF0000">`k! = 0`</tt>를 버전 0.5.0부터 시작하는 것으로 가정합니다.

<tt style="color: #FF0000">`keccak256(...) returns (bytes32)`</tt>:
<br>[(tightly packed) arguments](https://solidity.readthedocs.io/en/latest/abi-spec.html#abi-packed-mode)의 Ethereum-SHA-3(Keccak-256)해시를 계산하십시오

<tt style="color: #FF0000">`sha256(...) returns (bytes32)`</tt>:
<br>[(tightly packed) arguments](https://solidity.readthedocs.io/en/latest/abi-spec.html#abi-packed-mode)의 SHA-256해시를 계산하십시오

<tt style="color: #FF0000">`sha3(...) returns (bytes32)`</tt>:
<br><tt style="color: #FF0000">`keccak256`</tt>의 별칭

<tt style="color: #FF0000">`ripemd160(...) returns (bytes20)`</tt>:
<br>[(tightly packed) arguments](https://solidity.readthedocs.io/en/latest/abi-spec.html#abi-packed-mode)의 RIPEMD-160 hash해시를 계산하십시오

<tt style="color: #FF0000">`ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)`</tt>:
<br>타원 곡선 시그니터로부터 공개 키와 관련된 주소를 복구하거나 오류 발생시 0을 반환합니다 ([예제 사용법](https://ethereum.stackexchange.com/questions/1777/workflow-on-signing-a-string-with-private-key-followed-by-signature-verificatio)).

위의 "tightly packed"는 인수가 패딩없이 연결됨을 의미합니다. 즉, 다음 사항이 모두 동일함을 의미합니다.

~~~~
keccak256("ab", "c")
keccak256("abc")
keccak256(0x616263)
keccak256(6382179)
keccak256(97, 98, 99)
~~~~

패딩이 필요한 경우 명시 적 유형 변환을 사용할 수 있습니다. <tt style="color: #FF0000">`keccak256("\x00\x12")`</tt>은 <tt style="color: #FF0000">`keccak256(uint16(0x12))`</tt>과 동일합니다.

상수는 저장하는데 필요한 최소 바이트 수를 사용하여 압축됩니다. 예를 들어, <tt style="color: #FF0000">`keccak256(0) == keccak256(uint8(0))`</tt> 및 <tt style="color: #FF0000">`keccak256(0x12345678) == keccak256(uint32(0x12345678))`</tt>을 의미합니다.

<tt style="color: #FF0000">`sha256`</tt>, <tt style="color: #FF0000">`ripemd160`</tt> 또는 프라이빗 블록 체인의 <tt style="color: #FF0000">`ecrecover`</tt>에 대해 Out-of-Gas를 실행하는 것일 수 있습니다. 
그 이유는 이러한 것들이 소위 프리 컴파일된 계약으로 구현되고 이러한 계약은 첫 번째 메시지를 받은 후에만 존재합니다(계약 코드는 하드 코드되었지만). 
존재하지 않는 계약서에 대한 메시지는 더 비쌉니다. 따라서 실행은 Out-of-Gas 오류로 실행됩니다. 
이 문제를 해결할 수 있는 방법은 먼저 예를 들어 1 Wei를 실제 계약서에서 사용하기 전에 각 계약서에 적용해야합니다. 이것은 공식 또는 test net에서 문제가 되지 않습니다.

### Address Related

<tt style="color: #FF0000">`<address>.balance`</tt> (<tt style="color: #FF0000">`uint256`</tt>):
<br>[Address](https://solidity.readthedocs.io/en/latest/types.html#address)의 잔액(Wei)

<tt style="color: #FF0000">`<address>.transfer(uint256 amount)`</tt>:
<br>[Address](https://solidity.readthedocs.io/en/latest/types.html#address)에 주어진 Wei의 양을 보내고, 실패 시에 던지며, 2300의 가스를 보내고, 조절할 수 없음.

<tt style="color: #FF0000">`<address>.send(uint256 amount) returns (bool)`</tt>:
<br>[Address](https://solidity.readthedocs.io/en/latest/types.html#address)에 주어진 Wei의 양을 보내고, 실패하면 <tt style="color: #FF0000">`False`</tt>를 반환하고, 2300개의 가스를 전달함. 조절할 수 없음.

<tt style="color: #FF0000">`<address>.call(...) returns (bool)`</tt>:
<br>low-level <tt style="color: #FF0000">`CALL`</tt>을 발행하고, 실패시 <tt style="color: #FF0000">`false`</tt>를 반환하며, 사용 가능한 모든 가스를 전달하고, 조정 가능함.

<tt style="color: #FF0000">`<address>.callcode(...) returns (bool)`</tt>:
<br>low-level <tt style="color: #FF0000">`CALLCODE`</tt>를 발행하고, 실패시 <tt style="color: #FF0000">`false`</tt>를 반환하며, 사용 가능한 모든 가스를 전달하고, 조정 가능함.

<tt style="color: #FF0000">`<address>.delegatecall(...) returns (bool)`</tt>:
<br>low-level <tt style="color: #FF0000">`DELEGATECALL`</tt>을 발행하고, 실패시 <tt style="color: #FF0000">`false`</tt>를 반환하며, 사용 가능한 모든 가스를 전달하고, 조정 가능함.

자세한 내용은 [Address](https://solidity.readthedocs.io/en/latest/types.html#address) 섹션을 참조하십시오.

<b>Warning</b>
<br><tt style="color: #FF0000">`send`</tt> 사용시 몇 가지 위험이 있습니다. 
호출 스택 깊이가 1024(호출자가 항상 강제 할 수 있음)이면 전송이 실패하고 수신자가 가스를 다 소모하면 실패합니다. 
따라서 안전한 Ether 전송을하기 위해서는 항상 <tt style="color: #FF0000">`send`</tt>의 반환값을 확인하거나 <tt style="color: #FF0000">`transfer`</tt>를 사용하십시오.
받는 사람이 돈을 인출하는 패턴을 사용하십시오.
<br><b>Warning End</b>

<b>Note</b>
<br><tt style="color: #FF0000">`callcode`</tt> 사용은 권장되지 않으며 나중에 제거될 것입니다.
<br><b>Note End</b>

### Contract Related

<tt style="color: #FF0000">`this`</tt> <b>(current contract’s type)</b>:
<br>명시적으로 [Address](https://solidity.readthedocs.io/en/latest/types.html#address) 변환이 가능한 현재 계약

<tt style="color: #FF0000">`selfdestruct(address recipient)`</tt>:
<br>현재 계약을 파기하고, 주어진 [Address](https://solidity.readthedocs.io/en/latest/types.html#address)로 자금을 보낸다.

<tt style="color: #FF0000">`suicide(address recipient)`</tt>:
<br><tt style="color: #FF0000">`selfdestruct`</tt>의 별칭