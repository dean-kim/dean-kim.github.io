---
layout: post
title:  "Solidity Contracts"
date:   2018-03-05 18:43:59
author: Dean Kim
categories: BlockChain
tags:	BlockChain Solidity Smart_Contracts
cover:  "/assets/instacode.png"
---

블록체인을 공부하며 Smart Contract 작성에 사용되는 Solidity 공식문서를 번역한 글입니다.
제가 이해한 수준으로 기술된 내용도 있어 원본과 내용이 조금 다를 수도 있습니다.
혹시 제가 잘못 이해한 내용이 있다면 말씀해주세요 :)

# Contracts

Solidity의 계약은 객체 지향 언어의 클래스와 유사합니다. 
여기에는 상태 변수와 이러한 변수를 수정할 수 있는 함수에 지속적인 데이터를 포함하고 있습니다. 
다른 계약 (인스턴스)에서 함수를 호출하면 EVM 함수 호출이 수행되므로 상태 변수에 액세스 할 수 없도록 컨텍스트가 전환됩니다.

## Creating Contracts

계약은 Ethereum 거래 또는 Solidity 계약을 통해 "외부에서" 생성될 수 있습니다.

[Remix](https://remix.ethereum.org/)와 같은 IDE는 UI 요소를 사용하여 생성 프로세스를 매끄럽게 만듭니다.

Ethereum에서 프로그래밍 방식으로 계약을 생성하는 것은 JavaScript API [web3.js](https://github.com/ethereum/web3.js)를 사용하는 것이 가장 좋습니다. 오늘날 현재 계약 작성을 용이하게하기 위해 [web3.eth.Contract](https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#new-contract)라는 메소드가 있습니다.

계약이 생성되면 생성자 (계약과 이름이 같은 함수)가 한 번 실행됩니다. 생성자는 선택 사항입니다. 하나의 생성자만 허용되며, 이는 오버로드가 지원되지 않음을 의미합니다.

내부적으로 생성자 인수는 계약 코드 뒤에 [ABI로 인코딩](https://solidity.readthedocs.io/en/latest/abi-spec.html#abi)되어 전달되지만 <tt style="color: #FF0000">`web3.js`</tt>를 사용하는 경우에는 신경쓸 필요가 없습니다.

계약이 다른 계약을 작성하려는 경우 작성된 계약의 소스 코드 (and the binary)를 작성자에게 알려야합니다. 이것은 순환 생성 종속성이 불가능함을 의미합니다.

~~~~
pragma solidity ^0.4.16;

contract OwnedToken {
    // TokenCreator is a contract type that is defined below.
    // It is fine to reference it as long as it is not used
    // to create a new contract.
    TokenCreator creator;
    address owner;
    bytes32 name;

    // This is the constructor which registers the
    // creator and the assigned name.
    function OwnedToken(bytes32 _name) public {
        // State variables are accessed via their name
        // and not via e.g. this.owner. This also applies
        // to functions and especially in the constructors,
        // you can only call them like that ("internally"),
        // because the contract itself does not exist yet.
        owner = msg.sender;
        // We do an explicit type conversion from `address`
        // to `TokenCreator` and assume that the type of
        // the calling contract is TokenCreator, there is
        // no real way to check that.
        creator = TokenCreator(msg.sender);
        name = _name;
    }

    function changeName(bytes32 newName) public {
        // Only the creator can alter the name --
        // the comparison is possible since contracts
        // are implicitly convertible to addresses.
        if (msg.sender == address(creator))
            name = newName;
    }

    function transfer(address newOwner) public {
        // Only the current owner can transfer the token.
        if (msg.sender != owner) return;
        // We also want to ask the creator if the transfer
        // is fine. Note that this calls a function of the
        // contract defined below. If the call fails (e.g.
        // due to out-of-gas), the execution here stops
        // immediately.
        if (creator.isTokenTransferOK(owner, newOwner))
            owner = newOwner;
    }
}

contract TokenCreator {
    function createToken(bytes32 name)
       public
       returns (OwnedToken tokenAddress)
    {
        // Create a new Token contract and return its address.
        // From the JavaScript side, the return type is simply
        // `address`, as this is the closest type available in
        // the ABI.
        return new OwnedToken(name);
    }

    function changeName(OwnedToken tokenAddress, bytes32 name)  public {
        // Again, the external type of `tokenAddress` is
        // simply `address`.
        tokenAddress.changeName(name);
    }

    function isTokenTransferOK(address currentOwner, address newOwner)
        public
        view
        returns (bool ok)
    {
        // Check some arbitrary condition.
        address tokenAddress = msg.sender;
        return (keccak256(newOwner) & 0xff) == (bytes20(tokenAddress) & 0xff);
    }
}
~~~~

## Visibility and Getters

Solidity는 두 가지 종류의 함수 호출 (실제 호출("메시지 호출"이라고도 함)과 외부 EVM 호출을 생성하지 않는 내부 호출)을 알고 있으므로 함수와 상태 변수에 대한 네 가지 유형의 Visibility이 있습니다.

함수는 <tt style="color: #FF0000">`external`</tt>, <tt style="color: #FF0000">`public`</tt>, <tt style="color: #FF0000">`internal`</tt> 또는 <tt style="color: #FF0000">`private`</tt>으로 지정할 수 있습니다. 기본값은 <tt style="color: #FF0000">`public`</tt>입니다. 
상태 변수의 경우 <tt style="color: #FF0000">`external`</tt>은 불가능하며 기본값은 <tt style="color: #FF0000">`internal`</tt>입니다.

<tt style="color: #FF0000">`external`</tt>:
<br>외부 함수는 계약 인터페이스의 일부이므로 다른 계약 및 트랜잭션을 통해 호출 할 수 있습니다. 
외부 함수 <tt style="color: #FF0000">`f`</tt>는 내부적으로 호출 할 수 없습니다. 즉 <tt style="color: #FF0000">`f()`</tt>가 작동하지 않지만 <tt style="color: #FF0000">`this.f()`</tt>는 작동합니다. 외부 함수는 대용량 데이터 배열을 수신 할 때 때로는 더 효율적입니다.

<tt style="color: #FF0000">`public`</tt>:
<br>공용 함수는 계약 인터페이스의 일부이며 내부적으로 또는 메시지를 통해 호출 할 수 있습니다. public 상태 변수의 경우 자동 getter 함수(아래 참조)가 생성됩니다.

<tt style="color: #FF0000">`internal`</tt>:
<br>이러한 함수 및 상태 변수는 <tt style="color: #FF0000">`this`</tt>를 사용하지 않고 내부적으로(즉, 현재 계약 또는 계약에서 파생된 계약 내에서만) 액세스 할 수 있습니다.

<tt style="color: #FF0000">`private`</tt>:
<br>private 함수와 상태 변수는 파생된 계약이 아닌 정의된 계약에서만 볼 수 있습니다.

<b>Note</b>
<br>계약 내부에 있는 모든 것은 모든 외부 옵저버에게 공개됩니다. <tt style="color: #FF0000">`private`</tt>로만 설정하면 다른 계약이 정보에 액세스하고 수정할 수 없지만 여전히 블록 체인 외부의 모든 사람들이 볼 수 있습니다.
<br><b>Note End</b>

visibility 지정자는 상태 변수의 유형 뒤에 제공되며 매개 변수 목록과 함수의 반환 매개 변수 목록 사이에 제공됩니다.

~~~~
pragma solidity ^0.4.16;

contract C {
    function f(uint a) private pure returns (uint b) { return a + 1; }
    function setData(uint a) internal { data = a; }
    uint public data;
}
~~~~

다음 예제에서 <tt style="color: #FF0000">`D`</tt>는 <tt style="color: #FF0000">`c.getData()`</tt>를 호출하여 상태 저장소에 있는 <tt style="color: #FF0000">`data`</tt> 값을 검색할 수 있지만 <tt style="color: #FF0000">`f`</tt>를 호출할 수는 없습니다. 
계약 <tt style="color: #FF0000">`E`</tt>는 <tt style="color: #FF0000">`C`</tt>에서 파생되므로 <tt style="color: #FF0000">`compute`</tt>을 호출할 수 있습니다.

~~~~
// This will not compile

pragma solidity ^0.4.0;

contract C {
    uint private data;

    function f(uint a) private returns(uint b) { return a + 1; }
    function setData(uint a) public { data = a; }
    function getData() public returns(uint) { return data; }
    function compute(uint a, uint b) internal returns (uint) { return a+b; }
}

contract D {
    function readData() public {
        C c = new C();
        uint local = c.f(7); // error: member `f` is not visible
        c.setData(3);
        local = c.getData();
        local = c.compute(3, 5); // error: member `compute` is not visible
    }
}

contract E is C {
    function g() public {
        C c = new C();
        uint val = compute(3, 5); // access to internal member (from derived to parent contract)
    }
}
~~~~

### Getter Functions

컴파일러는 모든 <b>public</b> 상태 변수에 대한 getter 함수를 자동으로 만듭니다. 
아래 주어진 계약에서 컴파일러는 인수를 취하지 않는 <tt style="color: #FF0000">`data`</tt>라는 함수를 생성하고 상태 변수 <tt style="color: #FF0000">`data`</tt>의 값인 <tt style="color: #FF0000">`uint`</tt>를 반환합니다. 
상태 변수의 초기화는 선언 시에 수행 될 수 있습니다.

~~~~
pragma solidity ^0.4.0;

contract C {
    uint public data = 42;
}

contract Caller {
    C c = new C();
    function f() public {
        uint local = c.data();
    }
}
~~~~

getter 함수는 외부에서 볼 수 있습니다. 기호가 내부적으로 (즉, <tt style="color: #FF0000">`this`</tt> 없이) 액세스되면 상태 변수로 평가됩니다. 외부적으로 (즉, <tt style="color: #FF0000">`this`</tt>로) 액세스되면 함수로 평가됩니다.

~~~~
pragma solidity ^0.4.0;

contract C {
    uint public data;
    function x() public {
        data = 3; // internal access
        uint val = this.data(); // external access
    }
}
~~~~

다음 예제는 좀 더 복잡합니다.

~~~~
pragma solidity ^0.4.0;

contract Complex {
    struct Data {
        uint a;
        bytes3 b;
        mapping (uint => uint) map;
    }
    mapping (uint => mapping(bool => Data[])) public data;
}
~~~~

다음과 같은 형식의 함수를 생성합니다:

~~~~
function data(uint arg1, bool arg2, uint arg3) public returns (uint a, bytes3, b) {
    a = data[arg1][arg2][arg3].a;
    b = data[arg1][arg2][arg3].b;
}
~~~~

struct에 대한 매핑은 매핑에 키를 제공하는 좋은 방법이 없기 때문에 생략됩니다.

## Function Modifiers

Modifiers를 사용하여 함수의 behaviour를 쉽게 변경할 수 있습니다. 
예를 들어 함수를 실행하기 전에 조건을 자동으로 확인할 수 있습니다. Modifiers는 계약의 상속 가능한 속성이며 파생된 계약에 의해 무시될 수 있습니다.

~~~~
pragma solidity ^0.4.11;

contract owned {
    function owned() public { owner = msg.sender; }
    address owner;

    // This contract only defines a modifier but does not use
    // it: it will be used in derived contracts.
    // The function body is inserted where the special symbol
    // `_;` in the definition of a modifier appears.
    // This means that if the owner calls this function, the
    // function is executed and otherwise, an exception is
    // thrown.
    modifier onlyOwner {
        require(msg.sender == owner);
        _;
    }
}

contract mortal is owned {
    // This contract inherits the `onlyOwner` modifier from
    // `owned` and applies it to the `close` function, which
    // causes that calls to `close` only have an effect if
    // they are made by the stored owner.
    function close() public onlyOwner {
        selfdestruct(owner);
    }
}

contract priced {
    // Modifiers can receive arguments:
    modifier costs(uint price) {
        if (msg.value >= price) {
            _;
        }
    }
}

contract Register is priced, owned {
    mapping (address => bool) registeredAddresses;
    uint price;

    function Register(uint initialPrice) public { price = initialPrice; }

    // It is important to also provide the
    // `payable` keyword here, otherwise the function will
    // automatically reject all Ether sent to it.
    function register() public payable costs(price) {
        registeredAddresses[msg.sender] = true;
    }

    function changePrice(uint _price) public onlyOwner {
        price = _price;
    }
}

contract Mutex {
    bool locked;
    modifier noReentrancy() {
        require(!locked);
        locked = true;
        _;
        locked = false;
    }

    /// This function is protected by a mutex, which means that
    /// reentrant calls from within `msg.sender.call` cannot call `f` again.
    /// The `return 7` statement assigns 7 to the return value but still
    /// executes the statement `locked = false` in the modifier.
    function f() public noReentrancy returns (uint) {
        require(msg.sender.call());
        return 7;
    }
}
~~~~

여러 modifier는 함수에 공백으로 구분된 목록에 지정하여 함수에 적용되며 제시된 순서대로 평가됩니다.

<b>Warning</b>
<br>이전 버전의 Solidity에서는 modifier를 갖는 함수의 <tt style="color: #FF0000">`return`</tt> 문이 다르게 동작했습니다.
<br><b>Warning End</b>

modifier 또는 함수 본문에서 명시적으로 반환하면 현재 modifier 또는 함수 본문만 남습니다. 반환 변수가 지정되고 앞의 modifier에서 "_" 다음 제어 흐름이 계속됩니다.

임의의 표현식은 modifier 인수에 허용되며 이 context에서 함수에서 볼 수 있는 모든 심볼은 modifier에서 볼 수 있습니다. modifier에 도입된 Symbols는 함수에서 표시되지 않습니다(재정의로 변경 될 수 있음).

## Constant State Variables

상태 변수는 <tt style="color: #FF0000">`constant`</tt>로 선언할 수 있습니다. 
이 경우 컴파일 타임에 상수인 식에서 할당해야 합니다. 
저장 용량, 블록 체인 데이터 (예 : <tt style="color: #FF0000">`now`</tt>, <tt style="color: #FF0000">`this.balance`</tt> 또는 <tt style="color: #FF0000">`block.number`</tt>) 또는 실행 데이터 (<tt style="color: #FF0000">`msg.gas`</tt>)에 액세스하거나 외부 계약을 호출하는 표현식은 허용되지 않습니다. 
메모리 할당에 부작용이 있는 식은 허용되지만 다른 메모리 개체에 부작용이 있는 식은 허용되지 않습니다. 
내장 함수 <tt style="color: #FF0000">`keccak256`</tt>, <tt style="color: #FF0000">`sha256`</tt>, <tt style="color: #FF0000">`ripemd160`</tt>, <tt style="color: #FF0000">`ecrecover`</tt>, <tt style="color: #FF0000">`addmod`</tt> 및 <tt style="color: #FF0000">`mulmod`</tt>가 허용됩니다(외부 계약을 호출하더라도).

메모리 할당자에 부작용을 허용하는 이유는 예를 들어 다음과 같은 복잡한 객체를 구성할 수 있어야 한다는 것입니다. 
룩업 테이블. 이 기능은 아직 완전히 사용할 수 없습니다.

컴파일러는 이러한 변수에 대한 저장 영역 슬롯을 예약하지 않으며 모든 occurrence 해당 상수 표현식 (optimizer에 의해 단일 값으로 계산 될 수 있음)으로 대체됩니다.

현재 모든 유형의 상수가 구현되지는 않습니다. 지원되는 유일한 유형은 값 유형 및 문자열입니다.

~~~~
pragma solidity ^0.4.0;

contract C {
    uint constant x = 32**22 + 8;
    string constant text = "abc";
    bytes32 constant myHash = keccak256("abc");
}
~~~~

## Functions

### View Functions

함수는 상태를 수정하지 않겠다고 약속한 경우 <tt style="color: #FF0000">`view`</tt>로 선언할 수 있습니다.

다음 문은 상태를 수정하는 것으로 간주됩니다.

1. 상태 변수에 기록하기.
2. [Emitting events.](https://solidity.readthedocs.io/en/latest/contracts.html#events)
3. [Creating other contracts.](https://solidity.readthedocs.io/en/latest/control-structures.html#creating-contracts)
4. <tt style="color: #FF0000">`selfdestruct`</tt> 사용.
5. call을 통해 Ether를 보냅니다.
6. <tt style="color: #FF0000">`view`</tt> 또는 <tt style="color: #FF0000">`pure`</tt> 표시되지 않은 함수를 호출합니다.
7. low-level calls 사용.
8. 특정 opcode가 포함된 인라인 어셈블리 사용

~~~~
pragma solidity ^0.4.16;

contract C {
    function f(uint a, uint b) public view returns (uint) {
        return a * (b + 42) + now;
    }
}
~~~~

<b>Note</b>
<br>함수 <tt style="color: #FF0000">`constant`</tt>는 <tt style="color: #FF0000">`view`</tt>의 별칭이지만, 더 이상 사용되지 않으며 버전 0.5.0에서 삭제될 예정입니다.
<br><b>Note End</b>

<b>Note</b>
<br>Getter 메소드는 <tt style="color: #FF0000">`view`</tt>로 표시됩니다.
<br><b>Note End</b>

<b>Warning</b>
<br>버전 0.4.17 이전에는 컴파일러가 <tt style="color: #FF0000">`view`</tt>가 상태를 수정하지 않도록 시행하지 않았습니다.
<br><b>Warning End</b>

### Pure Functions

함수를 <tt style="color: #FF0000">`pure`</tt>로 선언 할 수 있는 경우 함수는 상태를 읽거나 수정하지 않을 것을 약속합니다.

위에서 설명한 상태 수정 명령문 목록 외에도 다음 내용이 상태 읽기로 간주됩니다.

1. 상태 변수 읽기.
2. <tt style="color: #FF0000">`this.balance`</tt> 또는 <tt style="color: #FF0000">`<address>.balance`</tt>에 액세스.
3. <tt style="color: #FF0000">`block`</tt>, <tt style="color: #FF0000">`tx`</tt>, <tt style="color: #FF0000">`msg`</tt>의 멤버에 접근한다. (<tt style="color: #FF0000">`msg.sig`</tt>와 <tt style="color: #FF0000">`msg.data`</tt> 제외).
4. <tt style="color: #FF0000">`pure`</tt>로 표시되지 않은 함수를 호출합니다.
5. 특정 opcode가 포함된 인라인 어셈블리 사용

~~~~
pragma solidity ^0.4.16;

contract C {
    function f(uint a, uint b) public pure returns (uint) {
        return a * (b + 42);
    }
}
~~~~

<b>Warning</b>
<br>버전 0.4.17 이전에는 컴파일러가 <tt style="color: #FF0000">`view`</tt>가 상태를 수정하지 않도록 시행하지 않았습니다.
<br><b>Warning End</b>

### Fallback Function

계약에는 명명되지 않은 함수가 하나만 있을 수 있습니다. 
이 함수는 인수를 가질 수 없으며 아무것도 반환 할 수 없습니다. 
다른 함수가 주어진 함수 식별자와 일치하지 않으면 (또는 데이터가 전혀 제공되지 않은 경우) 계약의 호출에서 실행됩니다.

또한 이 function는 계약서에 plain Ether(데이터 없음)를 수신 할 때마다 실행됩니다. 
또한 Ether를 수신하려면 fallback function을 <tt style="color: #FF0000">`payable`</tt>로 표시해야합니다. 그러한 function이 없는 경우, 계약은 정기적인 거래를 통해 Ether를 받을 수 없습니다.

그런 맥락에서, 일반적으로 함수 호출에 사용할 수 있는 가스는 거의 없습니다(정확하게는 2300가스). 
가능한 한 값싼 함수로 대체하는 것이 중요합니다. fallback function을 호출하는 트랜잭션 (내부 호출과 반대)에 필요한 가스는 서명 확인과 같은 이유로 21000개 이상의 가스를 추가로 청구하므로 훨씬 더 높습니다.

특히, 다음 작업은 fallback function에 제공되는 추가 비용보다 많은 가스를 소비합니다.

* 저장소에 쓰기
* 계약서 작성
* 다량의 가스를 소비하는 외부 기능 호출
* Ether 전송

계약을 전개하기 전에 실행 비용이 2300가스 미만인지 확인하기 위해 fallback function을 철저히 테스트하십시오.

<b>Note</b>
<br>fallback function에는 인수가 있을 수 없지만 <tt style="color: #FF0000">`msg.data`</tt>를 사용하여 호출과 함께 제공된 페이로드를 검색할 수 있습니다.
<br><b>Note End</b>

<b>Warning</b>
<br>Ether를 직접 (함수 호출없이, 즉 <tt style="color: #FF0000">`send`</tt> 또는 <tt style="color: #FF0000">`transfer`</tt>를 사용하여) 수신하지만 fallback function을 정의하지 않는 계약은 Ether을 돌려 보내는 예외를 throw합니다 (Solidity v0.4.0 이전에는 다릅니다). 따라서 계약서에 Ether을 받기를 원하면 fallback function을 구현해야합니다.
<br><b>Warning End</b>

<b>Warning</b>
<br>지불 할 수 있는 fallback function이 없는 계약은 코인베이스 거래 (일명 채굴자 블록 보상)의 수취인 또는 <tt style="color: #FF0000">`selfdestruct`</tt> 대상으로 Ether을 수령할 수 있습니다.

계약은 그러한 Ether 전송에 반응할 수 없으므로 계약을 거부할 수도 없습니다. 이것은 EVM의 디자인 선택이며 Solidity는 이를 해결할 수 없습니다.

또한 <tt style="color: #FF0000">`this.balance`</tt>는 계약에서 구현된 일부 manual accounting (즉, fallback function에서 업데이트된 카운터 보유)의 합계보다 높을 수 있음을 의미합니다.
<br><b>Warning End</b>

~~~~
pragma solidity ^0.4.0;

contract Test {
    // This function is called for all messages sent to
    // this contract (there is no other function).
    // Sending Ether to this contract will cause an exception,
    // because the fallback function does not have the `payable`
    // modifier.
    function() public { x = 1; }
    uint x;
}


// This contract keeps all Ether sent to it with no way
// to get it back.
contract Sink {
    function() public payable { }
}

contract Caller {
    function callTest(Test test) public {
        test.call(0xabcdef01); // hash does not exist
        // results in test.x becoming == 1.

        // The following will not compile, but even
        // if someone sends ether to that contract,
        // the transaction will fail and reject the
        // Ether.
        //test.send(2 ether);
    }
}
~~~~

### Function Overloading

계약에는 동일한 이름이지만 다른 인수가 있는 여러 함수가 있을 수 있습니다. 상속된 함수에도 적용됩니다. 다음 예제에서는 계약 <tt style="color: #FF0000">`A`</tt> 범위에서 <tt style="color: #FF0000">`f`</tt> 함수 오버로드를 보여줍니다.

~~~~
pragma solidity ^0.4.16;

contract A {
    function f(uint _in) public pure returns (uint out) {
        out = 1;
    }

    function f(uint _in, bytes32 _key) public pure returns (uint out) {
        out = 2;
    }
}
~~~~

오버로드 된 함수는 외부 인터페이스에도 있습니다. 두 개의 외부에서 볼 수 있는 함수는 Solidity 유형에 따라 다르지만 외부 유형에 따라 다르면 오류입니다.

~~~~
// This will not compile
pragma solidity ^0.4.16;

contract A {
    function f(B _in) public pure returns (B out) {
        out = _in;
    }

    function f(address _in) public pure returns (address out) {
        out = _in;
    }
}

contract B {
}
~~~~

두 가지 <tt style="color: #FF0000">`f`</tt> 함수는 ABI의 주소 유형을 받아들일 때 위의 오버로드 기능을 수행하지만 Solidity 내부에서는 다른 것으로 간주됩니다.

### Overload resolution and Argument matching

오버로드된 함수는 현재 범위의 함수 선언을 함수 호출에서 제공된 인수와 일치시켜 선택합니다. 모든 인수가 암시적으로 예상 유형으로 변환될 수 있는 경우 함수가 오버로드 후보로 선택됩니다. 정확히 하나의 후보자가 없으면 해결이 실패합니다.

<b>Note</b>
<br>반환 매개 변수는 overload resolution을 고려하지 않습니다.
<br><b>Note End</b>

~~~~
pragma solidity ^0.4.16;

contract A {
    function f(uint8 _in) public pure returns (uint8 out) {
        out = _in;
    }

    function f(uint256 _in) public pure returns (uint256 out) {
        out = _in;
    }
}
~~~~

<tt style="color: #FF0000">`250`</tt>을 <tt style="color: #FF0000">`uint8`</tt> 및 <tt style="color: #FF0000">`uint256`</tt> 유형으로 암시적으로 변환할 수 있으므로 <tt style="color: #FF0000">`f(50)`</tt>를 호출하면 유형 오류가 발생합니다. 반면에 <tt style="color: #FF0000">`256`</tt>이 암시적으로 <tt style="color: #FF0000">`uint8`</tt>로 변환될 수 없으므로 <tt style="color: #FF0000">`f(256)`</tt>는 <tt style="color: #FF0000">`f(uint256)`</tt> overload로 resolve됩니다.

## Events

이벤트를 사용하면 EVM 로깅 기능을 편리하게 사용할 수 있으며, 이러한 기능을 사용하여 이벤트를 수신하는 dapp의 사용자 인터페이스에서 JavaScript 콜백을 "호출"할 수 있습니다.

이벤트는 상속 가능한 계약 멤버입니다. 인자를 호출하면 인자가 트랜잭션 로그에 저장됩니다(블록 체인의 특수 데이터 구조). 이러한 로그는 계약서 주소와 연결되며 블록체인에 통합되어 블록에 액세스할 수 있는 한 계속 유지됩니다(Frontier 및 Homestead는 영원히 지속되지만 Serenity와 함께 변경될 수 있음). 
로그 및 이벤트 데이터는 계약내에서 액세스할 수 없습니다(계약을 작성한 계약을 통해서도 액세스할 수 없음).

로그에 대한 SPV 증명이 가능하기 때문에 외부 엔티티가 그러한 증명으로 계약을 제공하면 로그가 실제로 블록체인 내에 있는지 확인할 수 있습니다. 
하지만 계약은 마지막 256 블록해시만 볼 수 있기 때문에 블록헤더를 제공해야한다는 점에 유의하십시오.

최대 3개의 매개 변수가 <tt style="color: #FF0000">`indexed`</tt>된 속성을 수신하여 각 인수가 검색되도록 합니다. 사용자 인터페이스에서 indexed된 인수의 특정값을 필터링할 수 있습니다.

배열(<tt style="color: #FF0000">`string`</tt> 및 <tt style="color: #FF0000">`bytes`</tt> 포함)이 인덱스된 인수로 사용되는 경우 해당 배열의 Keccak-256 해시가 대신 항목으로 저장됩니다.

<tt style="color: #FF0000">`anonymous`</tt> specifier로 이벤트를 선언한 경우를 제외하고 이벤트 서명의 해시는 주제 중 하나입니다. 이는 특정 익명 이벤트를 이름별로 필터링할 수 없음을 의미합니다.

인덱싱되지 않은 모든 인수는 로그의 데이터 부분에 저장됩니다.

<b>Note</b>
<br>SPV?
<br>SPV란? Simplified Payment Verification의 약자, 코인을 받았다는 사실을 전체 블록체인을 다운로드하지 않고도 검증하는 방법.
<br><b>Note End</b>

<b>Note</b>
<br>인덱싱된 인수는 자체 저장되지 않습니다. 값만 검색할 수 있지만 값 자체를 검색하는 것은 불가능합니다.
<br><b>Note End</b>

~~~~
pragma solidity ^0.4.0;

contract ClientReceipt {
    event Deposit(
        address indexed _from,
        bytes32 indexed _id,
        uint _value
    );

    function deposit(bytes32 _id) public payable {
        // Events are emitted using `emit`, followed by
        // the name of the event and the arguments
        // (if any) in parentheses. Any such invocation
        // (even deeply nested) can be detected from
        // the JavaScript API by filtering for `Deposit`.
        emit Deposit(msg.sender, _id, msg.value);
    }
}
~~~~

JavaScript API에서의 사용은 다음과 같습니다.

~~~~
var abi = /* abi as generated by the compiler */;
var ClientReceipt = web3.eth.contract(abi);
var clientReceipt = ClientReceipt.at("0x1234...ab67" /* address */);

var event = clientReceipt.Deposit();

// watch for changes
event.watch(function(error, result){
    // result will contain various information
    // including the argumets given to the `Deposit`
    // call.
    if (!error)
        console.log(result);
});

// Or pass a callback to start watching immediately
var event = clientReceipt.Deposit(function(error, result) {
    if (!error)
        console.log(result);
});
~~~~

### Low-Level Interface to Logs

<tt style="color: #FF0000">`log0`</tt>, <tt style="color: #FF0000">`log1`</tt>, <tt style="color: #FF0000">`log2`</tt>, <tt style="color: #FF0000">`log3`</tt> 및 <tt style="color: #FF0000">`log4`</tt> 함수를 통해 로깅 메커니즘에 대한 저수준 인터페이스에 액세스할 수도 있습니다. 
<tt style="color: #FF0000">`logi`</tt>는 <tt style="color: #FF0000">`bytes32`</tt> 유형의 <tt style="color: #FF0000">`i + 1`</tt> 매개 변수를 취합니다. 
여기서 첫 번째 인수는 로그의 데이터 부분에 사용되며 다른 인수는 주제로 사용됩니다. 
위의 이벤트 호출은 다음과 같은 방식으로 수행 될 수 있습니다.

~~~~
pragma solidity ^0.4.10;

contract C {
    function f() public payable {
        bytes32 _id = 0x420042;
        log3(
            bytes32(msg.value),
            bytes32(0x50cb9fe53daa9737b786ab3646f04d0150dc50ef4e75f59509d83667ad5adb20),
            bytes32(msg.sender),
            _id
        );
    }
}
~~~~

여기서 긴 16 진수는 이벤트의 시그니쳐인 <tt style="color: #FF0000">`keccak256("Deposit(address, hash256, uint256)")`</tt>과 같습니다.

### Additional Resources for Understanding Events

* [Javascript documentation](https://github.com/ethereum/wiki/wiki/JavaScript-API#contract-events)
* [Example usage of events](https://github.com/ethchange/smart-exchange/blob/master/lib/contracts/SmartExchange.sol)
* [How to access them in js](https://github.com/ethchange/smart-exchange/blob/master/lib/exchange_transactions.js)

## Inheritance

Solidity는 다형성을 포함한 코드를 복사하여 다중 상속을 지원합니다.

모든 함수 호출은 가상입니다. 즉, 계약 이름이 명시적으로 지정된 경우를 제외하고 가장 많이 파생된 함수가 호출됩니다.

계약이 여러 계약에서 상속되면 블록체인에 하나의 계약만 생성되고 모든 기본 계약의 코드가 생성된 계약에 복사됩니다.

일반적인 상속 시스템은 파이썬과 매우 유사합니다. 특히 다중 상속에 관한 것입니다.

자세한 내용은 다음 예제에서 제공됩니다.

~~~~
pragma solidity ^0.4.16;

contract owned {
    function owned() { owner = msg.sender; }
    address owner;
}

// Use `is` to derive from another contract. Derived
// contracts can access all non-private members including
// internal functions and state variables. These cannot be
// accessed externally via `this`, though.
contract mortal is owned {
    function kill() {
        if (msg.sender == owner) selfdestruct(owner);
    }
}

// These abstract contracts are only provided to make the
// interface known to the compiler. Note the function
// without body. If a contract does not implement all
// functions it can only be used as an interface.
contract Config {
    function lookup(uint id) public returns (address adr);
}

contract NameReg {
    function register(bytes32 name) public;
    function unregister() public;
 }

// Multiple inheritance is possible. Note that `owned` is
// also a base class of `mortal`, yet there is only a single
// instance of `owned` (as for virtual inheritance in C++).
contract named is owned, mortal {
    function named(bytes32 name) {
        Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
        NameReg(config.lookup(1)).register(name);
    }

    // Functions can be overridden by another function with the same name and
    // the same number/types of inputs.  If the overriding function has different
    // types of output parameters, that causes an error.
    // Both local and message-based function calls take these overrides
    // into account.
    function kill() public {
        if (msg.sender == owner) {
            Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
            NameReg(config.lookup(1)).unregister();
            // It is still possible to call a specific
            // overridden function.
            mortal.kill();
        }
    }
}

// If a constructor takes an argument, it needs to be
// provided in the header (or modifier-invocation-style at
// the constructor of the derived contract (see below)).
contract PriceFeed is owned, mortal, named("GoldFeed") {
   function updateInfo(uint newInfo) public {
      if (msg.sender == owner) info = newInfo;
   }

   function get() public view returns(uint r) { return info; }

   uint info;
}
~~~~

위에서, 우리는 <tt style="color: #FF0000">`mortal.kill()`</tt>을 호출하여 destruction 요청을 "전달"합니다. 다음 예제와 같이 이것이 이루어지는 방식은 문제가 됩니다.

~~~~
pragma solidity ^0.4.0;

contract owned {
    function owned() public { owner = msg.sender; }
    address owner;
}

contract mortal is owned {
    function kill() public {
        if (msg.sender == owner) selfdestruct(owner);
    }
}

contract Base1 is mortal {
    function kill() public { /* do cleanup 1 */ mortal.kill(); }
}

contract Base2 is mortal {
    function kill() public { /* do cleanup 2 */ mortal.kill(); }
}

contract Final is Base1, Base2 {
}
~~~~

<tt style="color: #FF0000">`Final.kill()`</tt>에 대한 호출은 <tt style="color: #FF0000">`Base2.kill`</tt>을 가장 많이 파생된 오버라이드로 호출하지만 이 함수는 기본적으로 <tt style="color: #FF0000">`Base1`</tt>에 대해 알지 못하기 때문에 <tt style="color: #FF0000">`Base1.kill`</tt>을 우회합니다. 이 문제를 해결하는 방법은 <tt style="color: #FF0000">`super`</tt>를 사용하는 것입니다.

~~~~
pragma solidity ^0.4.0;

contract owned {
    function owned() public { owner = msg.sender; }
    address owner;
}

contract mortal is owned {
    function kill() public {
        if (msg.sender == owner) selfdestruct(owner);
    }
}

contract Base1 is mortal {
    function kill() public { /* do cleanup 1 */ super.kill(); }
}


contract Base2 is mortal {
    function kill() public { /* do cleanup 2 */ super.kill(); }
}

contract Final is Base1, Base2 {
}
~~~~

<tt style="color: #FF0000">`Base2`</tt>가 <tt style="color: #FF0000">`super`</tt>의 함수를 호출하면 기본 계약 중 하나에서 이 함수를 호출하지 않습니다. 
오히려 최종 상속 그래프의 다음 기본 계약에서 이 함수를 호출하므로 <tt style="color: #FF0000">`Base1.kill()`</tt>을 호출합니다(최종 상속 시퀀스는 Final, Base2, Base1, mortal, 소유). 
super를 사용할 때 호출되는 실제 함수는 사용된 클래스의 컨텍스트에서 알 수 없지만 해당 유형은 알려져 있습니다. 이것은 일반적인 가상 메소드 검색과 비슷합니다.

### Constructors

생성자는 계약 작성과 동시에 실행되는 계약과 동일한 이름의 선택적 함수입니다. 생성자 함수는 <tt style="color: #FF0000">`public`</tt> 또는 <tt style="color: #FF0000">`internal`</tt>일 수 있습니다.

~~~~
pragma solidity ^0.4.11;

contract A {
    uint public a;

    function A(uint _a) internal {
        a = _a;
    }
}

contract B is A(1) {
    function B() public {}
}
~~~~

<tt style="color: #FF0000">`internal`</tt>로 설정된 생성자는 계약을 [abstract](https://solidity.readthedocs.io/en/latest/contracts.html#abstract-contract)으로 표시합니다.

### Arguments for Base Constructors

파생된 계약은 기본 생성자에 필요한 모든 인수를 제공해야 합니다. 이 작업은 두 가지 방법으로 수행할 수 있습니다.

~~~~
pragma solidity ^0.4.0;

contract Base {
    uint x;
    function Base(uint _x) public { x = _x; }
}

contract Derived is Base(7) {
    function Derived(uint _y) Base(_y * _y) public {
    }
}
~~~~

하나의 방법은 상속 목록(<tt style="color: #FF0000">`Base(7)`</tt>)에 직접 포함됩니다. 
다른 하나는 modifier가 파생 생성자(<tt style="color: #FF0000">`Base(_y * _y)`</tt>)의 헤더의 일부로 호출되는 방식입니다. 
첫 번째 방법은 생성자 인수가 상수이고 계약의 동작을 정의하거나 설명하는 경우에 더 편리합니다. 
베이스의 생성자 인수가 파생된 계약의 인수에 의존하는 경우 두 번째 방법을 사용해야합니다. 
이 어리석은 예제와 같이 두 위치를 모두 사용하면 modifier-style 인수가 우선합니다.

### Multiple Inheritance and Linearization

다중 상속을 허용하는 언어는 여러 가지 문제를 처리해야합니다. 
하나는 [Diamond Problem](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem)입니다. 
Solidity는 파이썬의 경로를 따르고 "[C3 Linearization](https://en.wikipedia.org/wiki/C3_linearization)"를 사용하여 기본 클래스의 [DAG](https://steemit.com/kr/@sjchoi/adk-3-0-dag-directed-acyclic-graph)에서 특정 순서를 강제합니다. 
이는 단조로움의 바람직한 특성을 나타내지만 일부 상속 graphs는 허용하지 않습니다. 
특히, 기본 클래스가 지시문에 주어진 순서는 중요합니다. 
다음 코드에서 Solidity는 "상속 그래프 선형화 불가능"오류를 표시합니다.

~~~~
// This will not compile

pragma solidity ^0.4.0;

contract X {}
contract A is X {}
contract C is A, X {}
~~~~

그 이유는 <tt style="color: #FF0000">`C`</tt>가 <tt style="color: #FF0000">`A`</tt>에 <tt style="color: #FF0000">`X`</tt>를(<tt style="color: #FF0000">`A, X`</tt>순서로 지정하여) 대체하도록 요청했지만 <tt style="color: #FF0000">`A`</tt> 자체가 해결할 수 없는 모순인 <tt style="color: #FF0000">`X`</tt>를 재정의하도록 요청하기 때문입니다.

기억해야할 간단한 규칙은 기본 클래스를 "대부분 기본"에서 "가장 많이 파생된"순서로 지정하는 것입니다.

### Inheriting Different Kinds of Members of the Same Name

상속으로 인해 동일한 이름의 함수 및 한정자와 계약을 체결하면 오류로 간주됩니다. 
이 오류는 동일한 이름의 이벤트 및 modifier, 동일한 이름의 함수 및 이벤트로도 생성됩니다. 
예외적으로 상태 변수 getter는 public 함수를 재정의할 수 있습니다.

## Abstract Contracts

다음 예제에서처럼 함수 중 하나 이상에 구현이 부족한 경우 계약이 abstract으로 표시됩니다(함수 선언 헤더는 <tt style="color: #FF0000">`;`</tt>로 끝납니다).

~~~~
pragma solidity ^0.4.0;

contract Feline {
    function utterance() public returns (bytes32);
}
~~~~

이러한 계약은 컴파일되지 않으며(구현되지 않은 기능과 함께 구현된 기능이 포함되어 있더라도) 기본 계약으로 사용할 수 있습니다.

~~~~
pragma solidity ^0.4.0;

contract Feline {
    function utterance() public returns (bytes32);
}

contract Cat is Feline {
    function utterance() public returns (bytes32) { return "miaow"; }
}
~~~~

계약이 추상 계약을 상속하고 대체를 통해 구현되지 않은 모든 기능을 구현하지 않으면 그 자체가 abstract이 됩니다.

구현이 없는 함수는 구문이 매우 비슷해 보이더라도 [Function Type](https://solidity.readthedocs.io/en/latest/types.html#function-types)과 다릅니다.

구현이 없는 함수의 예 (함수 선언):

~~~~
function foo(address) external returns (address);
~~~~

함수 유형의 예 (변수가 <tt style="color: #FF0000">`function`</tt> 유형인 변수 선언) :

~~~~
function(address) external returns (address) foo;
~~~~

Abstract 계약은 확장성 및 자체 문서화를 제공하고 [Template method](https://en.wikipedia.org/wiki/Template_method_pattern)와 같은 패턴을 용이하게하고 코드 중복을 제거하여 구현의 계약 정의를 분리합니다.

## Interfaces

인터페이스는 abstract 계약과 유사하지만 구현된 기능을 가질 수 없습니다. 추가 제한 사항이 있습니다.

1. 다른 계약 또는 인터페이스를 상속받을 수 없습니다.
2. 생성자를 정의할 수 없습니다.
3. 변수를 정의할 수 없습니다.
4. 구조체를 정의할 수 없습니다.
5. 열거형을 정의할 수 없습니다.
6. 이러한 제한 중 일부는 향후 해제될 수 있습니다.

인터페이스는 기본적으로 계약 ABI가 표현할 수 있는 것으로 제한되며, 정보 손실없이 ABI와 인터페이스 간의 변환이 가능해야합니다.

인터페이스는 자체 키워드로 표시됩니다.

~~~~
pragma solidity ^0.4.11;

interface Token {
    function transfer(address recipient, uint amount) public;
}
~~~~

## Libraries

라이브러리는 계약과 유사하지만 EVM의 <tt style="color: #FF0000">`DELEGATECALL`</tt>(<tt style="color: #FF0000">`CALLCODE`</tt> until Homestead) 기능을 사용하여 특정 주소에 한 번만 배포되고 코드가 다시 사용된다는 것이 그 목적입니다. 
즉, 라이브러리 함수가 호출되면 코드가 호출 계약의 컨텍스트에서 실행됩니다. 즉 <tt style="color: #FF0000">`this`</tt>는 호출하는 계약을 가리키며 특히 호출하는 계약의 저장소에 액세스할 수 있습니다. 
라이브러리는 고립된 소스 코드이기 때문에 명시적으로 제공되는 경우에만 호출 계약의 상태 변수에 액세스 할 수 있습니다(이름을 지정할 방법이 없거나 그렇지 않은 경우). 
라이브러리 함수는 상태를 수정할 수 없는 경우(즉, <tt style="color: #FF0000">`view`</tt> 또는 <tt style="color: #FF0000">`pure`</tt> 함수인 경우) 라이브러리를 무 상태라고 가정하기 때문에 라이브러리 함수는 직접(즉 <tt style="color: #FF0000">`DELEGATECALL`</tt>을 사용하지 않고) 호출할 수 있습니다. 
특히, Solidity의 유형 시스템이 우회되지 않는한 라이브러리를 파괴할 수 없습니다.

라이브러리는 이를 사용하는 계약의 암시적 기본 계약으로 볼 수 있습니다. 
상속 계층에 명시적으로 표시되지 않지만 라이브러리 함수에 대한 호출은 명시적 기본 계약(<tt style="color: #FF0000">`L`</tt>이 라이브러리의 이름 인 경우 <tt style="color: #FF0000">`L.f()`</tt>)의 함수를 호출하는 것처럼 보입니다. 
또한 라이브러리의 기본 기능이 기본 계약인 것처럼 모든 계약에서 라이브러리의 <tt style="color: #FF0000">`internal`</tt> 함수를 볼 수 있습니다. 
물론 내부 함수 호출은 내부 호출 규칙을 사용합니다. 즉 모든 내부 유형을 전달할 수 있으며 메모리 유형은 참조로 전달되고 복사되지 않습니다. 
이를 EVM에서 실현하려면 내부 라이브러리 함수의 코드와 거기에서 호출되는 모든 함수가 컴파일 타임에 호출 계약으로 끌어 들이고 DELEGATECALL 대신 정규 JUMP 호출이 사용됩니다.

다음 예제에서는 라이브러리를 사용하는 방법을 보여줍니다(단, set을 구현하는 고급 예제는 [using for](https://solidity.readthedocs.io/en/latest/contracts.html#using-for)를 참조하십시오).

~~~~
pragma solidity ^0.4.16;

library Set {
  // We define a new struct datatype that will be used to
  // hold its data in the calling contract.
  struct Data { mapping(uint => bool) flags; }

  // Note that the first parameter is of type "storage
  // reference" and thus only its storage address and not
  // its contents is passed as part of the call.  This is a
  // special feature of library functions.  It is idiomatic
  // to call the first parameter `self`, if the function can
  // be seen as a method of that object.
  function insert(Data storage self, uint value)
      public
      returns (bool)
  {
      if (self.flags[value])
          return false; // already there
      self.flags[value] = true;
      return true;
  }

  function remove(Data storage self, uint value)
      public
      returns (bool)
  {
      if (!self.flags[value])
          return false; // not there
      self.flags[value] = false;
      return true;
  }

  function contains(Data storage self, uint value)
      public
      view
      returns (bool)
  {
      return self.flags[value];
  }
}

contract C {
    Set.Data knownValues;

    function register(uint value) public {
        // The library functions can be called without a
        // specific instance of the library, since the
        // "instance" will be the current contract.
        require(Set.insert(knownValues, value));
    }
    // In this contract, we can also directly access knownValues.flags, if we want.
}
~~~~

물론 라이브러리를 사용하기 위해이 방법을 따를 필요는 없습니다. 
구조체 데이터 유형을 정의하지 않고 사용할 수도 있습니다. 
함수는 저장 참조 매개 변수 없이도 작동하며 여러 저장 참조 매개 변수를 가질 수 있습니다.

<tt style="color: #FF0000">`Set.contains`</tt>, <tt style="color: #FF0000">`Set.insert`</tt> 및 <tt style="color: #FF0000">`Set.remove`</tt>에 대한 호출은 모두 외부 계약/라이브러리에 대한 호출(<tt style="color: #FF0000">`DELEGATECALL`</tt>)로 컴파일됩니다. 
라이브러리를 사용하는 경우 실제 외부 함수 호출이 수행됩니다.
<tt style="color: #FF0000">`msg.sender`</tt>, <tt style="color: #FF0000">`msg.value`</tt> 그리고 <tt style="color: #FF0000">`this`</tt> 이 호출에서 값을 유지합니다(<tt style="color: #FF0000">`CALLCODE`</tt>를 사용하기 때문에 Homestead 이전에는 <tt style="color: #FF0000">`msg.sender`</tt>와 <tt style="color: #FF0000">`msg.value`</tt>가 변경되었지만).

다음 예제는 외부 함수 호출의 오버 헤드없이 사용자 정의 유형을 구현하기 위해 라이브러리에서 메모리 유형과 내부 함수를 사용하는 방법을 보여줍니다.

~~~~
pragma solidity ^0.4.16;

library BigInt {
    struct bigint {
        uint[] limbs;
    }

    function fromUint(uint x) internal pure returns (bigint r) {
        r.limbs = new uint[](1);
        r.limbs[0] = x;
    }

    function add(bigint _a, bigint _b) internal pure returns (bigint r) {
        r.limbs = new uint[](max(_a.limbs.length, _b.limbs.length));
        uint carry = 0;
        for (uint i = 0; i < r.limbs.length; ++i) {
            uint a = limb(_a, i);
            uint b = limb(_b, i);
            r.limbs[i] = a + b + carry;
            if (a + b < a || (a + b == uint(-1) && carry > 0))
                carry = 1;
            else
                carry = 0;
        }
        if (carry > 0) {
            // too bad, we have to add a limb
            uint[] memory newLimbs = new uint[](r.limbs.length + 1);
            for (i = 0; i < r.limbs.length; ++i)
                newLimbs[i] = r.limbs[i];
            newLimbs[i] = carry;
            r.limbs = newLimbs;
        }
    }

    function limb(bigint _a, uint _limb) internal pure returns (uint) {
        return _limb < _a.limbs.length ? _a.limbs[_limb] : 0;
    }

    function max(uint a, uint b) private pure returns (uint) {
        return a > b ? a : b;
    }
}

contract C {
    using BigInt for BigInt.bigint;

    function f() public pure {
        var x = BigInt.fromUint(7);
        var y = BigInt.fromUint(uint(-1));
        var z = x.add(y);
    }
}
~~~~

컴파일러는 라이브러리가 배포될 위치를 알 수 없으므로 이러한 주소는 링커에서 최종 바이트 코드로 채워야 합니다(링크 용 명령줄 컴파일러 사용 방법은 [Using the Commandline Compiler](https://solidity.readthedocs.io/en/latest/using-the-compiler.html#commandline-compiler) 참조). 
주소가 컴파일러에 대한 인수로 제공되지 않으면 컴파일 된 16진 코드에는 <tt style="color: #FF0000">`__Set______`</tt> 형식의 자리 표시자가 포함됩니다(여기서 <tt style="color: #FF0000">`Set`</tt>은 라이브러리의 이름임). 
주소는 라이브러리 계약의 주소를 16진수로 인코딩하여 40개의 모든 기호를 바꾸어 수동으로 채울 수 있습니다.

계약과 비교하여 라이브러리에 대한 제한 사항 :

* 상태 변수 없음
* 상속 받거나 상속받을 수 없습니다.
* Ether를 수신할 수 없습니다.

(이들은 나중에 해제될 수 있습니다.)

### Call Protection For Libraries

소개에서 언급했듯이, 라이브러리의 코드가 <tt style="color: #FF0000">`DELEGATECALL`</tt> 또는 <tt style="color: #FF0000">`CALLCODE`</tt> 대신 <tt style="color: #FF0000">`CALL`</tt>을 사용하여 실행되면 <tt style="color: #FF0000">`view`</tt> 또는 <tt style="color: #FF0000">`pure`</tt> 함수가 호출되지 않으면 복귀합니다.

EVM은 계약이 <tt style="color: #FF0000">`CALL`</tt>을 사용하여 호출되었는지 여부를 감지할 수 있는 직접적인 방법을 제공하지 않지만 계약은 <tt style="color: #FF0000">`ADDRESS`</tt> opcode를 사용하여 현재 실행중인 "위치"를 찾습니다. 
생성된 코드는 이 주소를 호출시의 모드를 결정하기 위해 생성시에 사용 된 주소와 비교합니다.

보다 구체적으로, 라이브러리의 런타임 코드는 항상 푸시(push) 명령으로 시작합니다.
이 명령은 컴파일 타임에 20바이트의 0입니다. 
배포 코드가 실행되면 이 상수는 메모리에서 현재 주소로 대체되며 이 수정된 코드는 계약서에 저장됩니다. 
런타임에 배포 시간 주소가 스택에 푸 될 첫 번째 상수가되고 디스패처 코드가 모든 non-view 그리고 non-pure 함수에 대해 현재 상수와 이 상수를 비교합니다.

## Using For

<tt style="color: #FF0000">`using A for B`</tt> 지시문. 라이브러리 함수(라이브러리 <tt style="color: #FF0000">`A`</tt>에서)를 모든 유형(<tt style="color: #FF0000">`B`</tt>)에 연결하는데 사용할 수 있습니다. 
이 함수는 자신이 호출한 객체를 첫 번째 매개 변수로 받습니다(Python의 <tt style="color: #FF0000">self``</tt> 변수와 같습니다).

<tt style="color: #FF0000">`using A for *`</tt>는 라이브러리 <tt style="color: #FF0000">`A`</tt>의 함수가 모든 유형에 연결되는 효과가 있습니다.

두 가지 상황 모두에서, 모든 기능, 심지어 첫번째 매개 변수의 유형이 객체의 유형과 일치하지 않는 경우에도, 모든 기능이 첨부됩니다. 함수가 호출되고 함수 overload resolution이 수행된 지점에서 유형이 검사됩니다.

<tt style="color: #FF0000">`using A for B`</tt> 지시문은 현재 범위에 대해 활성화되어 있습니다. 계약은 현재 계약으로 제한되지만 나중에 모듈을 포함함으로써 라이브러리 함수를 포함한 데이터 유형을 코드를 추가할 필요없이 사용할 수 있습니다.

다음과 같이 [라이브러리](https://solidity.readthedocs.io/en/latest/contracts.html#libraries)에서 예제를 다시 작성해 보겠습니다.

~~~~
pragma solidity ^0.4.16;

// This is the same code as before, just without comments
library Set {
  struct Data { mapping(uint => bool) flags; }

  function insert(Data storage self, uint value)
      public
      returns (bool)
  {
      if (self.flags[value])
        return false; // already there
      self.flags[value] = true;
      return true;
  }

  function remove(Data storage self, uint value)
      public
      returns (bool)
  {
      if (!self.flags[value])
          return false; // not there
      self.flags[value] = false;
      return true;
  }

  function contains(Data storage self, uint value)
      public
      view
      returns (bool)
  {
      return self.flags[value];
  }
}

contract C {
    using Set for Set.Data; // this is the crucial change
    Set.Data knownValues;

    function register(uint value) public {
        // Here, all variables of type Set.Data have
        // corresponding member functions.
        // The following function call is identical to
        // `Set.insert(knownValues, value)`
        require(knownValues.insert(value));
    }
}
~~~~

다음과 같이 기본 유형을 확장할 수도 있습니다.

~~~~
pragma solidity ^0.4.16;

library Search {
    function indexOf(uint[] storage self, uint value)
        public
        view
        returns (uint)
    {
        for (uint i = 0; i < self.length; i++)
            if (self[i] == value) return i;
        return uint(-1);
    }
}

contract C {
    using Search for uint[];
    uint[] data;

    function append(uint value) public {
        data.push(value);
    }

    function replace(uint _old, uint _new) public {
        // This performs the library function call
        uint index = data.indexOf(_old);
        if (index == uint(-1))
            data.push(_new);
        else
            data[index] = _new;
    }
}
~~~~

모든 라이브러리 호출은 실제 EVM 함수 호출입니다. 즉, 메모리 또는 값 유형을 전달하면 <tt style="color: #FF0000">`self`</tt> 변수에서도 복사가 수행됩니다. 복사가 수행되지 않는 유일한 상황은 저장 영역 참조 변수가 사용될 때 뿐입니다.