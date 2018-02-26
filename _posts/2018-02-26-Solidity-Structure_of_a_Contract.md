---
layout: post
title:  "Solidity Structure of a Contract"
date:   2018-02-26 22:43:59
author: Dean Kim
categories: BlockChain
tags:	BlockChain Solidity Smart_Contracts
cover:  "/assets/instacode.png"
---

블록체인을 공부하며 Smart Contract 작성에 사용되는 Solidity 공식문서를 번역한 글입니다.
제가 이해한 수준으로 기술된 내용도 있어 원본과 내용이 조금 다를 수도 있습니다.
혹시 제가 잘못 이해한 내용이 있다면 말씀해주세요 :)

# Structure of a Contract

솔리디티의 계약은 객체 지향 언어의 클래스와 비슷합니다. 각각의 계약은 State Variables, Functions, Function Modifiers, Events, Struct Types 그리고 Enum Types을 선언할 수 있습니다.
그리고 계약은 다른 계약을 상속할 수 있습니다.

## State Variables

상태 변수는 계약 저장에 영구적으로 저장되는 값입니다.

~~~~
pragma solidity ^0.4.0;

contract SimpleStorage {
    uint storedData; // State variable
    // ...
}
~~~~

유효한 상태 변수 유형은 [Types](https://solidity.readthedocs.io/en/latest/types.html#types) 섹션을, 가시성을 위한 가능한 선택 사항은 [Visibility and Getters](https://solidity.readthedocs.io/en/latest/contracts.html#visibility-and-getters)를 참조하십시오.

## Functions

함수는 계약내에서 실행 가능한 코드 단위입니다.

~~~~
pragma solidity ^0.4.0;

contract SimpleAuction {
    function bid() public payable { // Function
        // ...
    }
}
~~~~

[Function Calls](https://solidity.readthedocs.io/en/latest/control-structures.html#function-calls)는 내부적으로 또는 외부적으로 발생할 수 있으며 다른 계약에 대해 각각 다른 가시성([Visibility and Getters](https://solidity.readthedocs.io/en/latest/contracts.html#visibility-and-getters))을 가지고 있습니다.

## Function Modifiers

Function Modifiers를 사용하여 함수의 의미를 선언적 방식으로 수정할 수 있습니다 (계약 절의 [Function Modifiers](https://solidity.readthedocs.io/en/latest/contracts.html#modifiers) 참조).

~~~~
pragma solidity ^0.4.11;

contract Purchase {
    address public seller;

    modifier onlySeller() { // Modifier
        require(msg.sender == seller);
        _;
    }

    function abort() public onlySeller { // Modifier usage
        // ...
    }
}
~~~~

## Events

이벤트는 EVM이 로깅할 수 있는 기능으로 사용되는 편리한 인터페이스입니다.

~~~~
pragma solidity ^0.4.20; // should actually be 0.4.21

contract SimpleAuction {
    event HighestBidIncreased(address bidder, uint amount); // Event

    function bid() public payable {
        // ...
        emit HighestBidIncreased(msg.sender, msg.value); // Triggering event
    }
}
~~~~

이벤트가 선언되고 dapp 내에서 사용될 수 있는 방법에 대한 정보는 계약 섹션의 [Events](https://solidity.readthedocs.io/en/latest/contracts.html#events)를 참조하십시오.

## Struct Types

구조체는 여러 변수를 그룹화 할 수 있는 사용자 정의 유형입니다 ([구조체](https://solidity.readthedocs.io/en/latest/types.html#structs) 섹션 참조).

~~~~
pragma solidity ^0.4.0;

contract Ballot {
    struct Voter { // Struct
        uint weight;
        bool voted;
        address delegate;
        uint vote;
    }
}
~~~~

## Enum Types

Enum Types은 값의 유한 집합으로 사용자 정의 유형을 만드는 데 사용할 수 있습니다 ([Enums](https://solidity.readthedocs.io/en/latest/types.html#enums) 참조).

