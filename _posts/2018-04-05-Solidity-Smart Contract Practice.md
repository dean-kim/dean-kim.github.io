---
layout: post
title:  "Solidity Smart Contract Practice"
date:   2018-04-05 20:43:59
author: Dean Kim
categories: BlockChain
tags:	BlockChain Solidity Smart Contract Practice
cover:  "/assets/instacode.png"
---

그 동안 Solidity를 학습한 것을 바탕으로 식당 예약을 하는 Smart Contract를 작성해보았습니다.

# Smart Contract about restaurant reservation

이 계약을 만들기에 앞서 구현하려는 Smart Contract에 대해 간략하게 설명하겠습니다.
 
## Scenario

이 Smart Contract가 사용될 시나리오는 다음과 같습니다.

### Scenario-1

1. <tt style="color: #FF0000">`owner`</tt>(식당의 주인)가 Smart Contract를 생성합니다.
2. <tt style="color: #FF0000">`client`</tt>(식당을 예약하고자 하는 사람, 예약자)가 <tt style="color: #FF0000">`reservationFee`</tt>(예약금)을 조건으로 <tt style="color: #FF0000">`reservation`</tt>(예약)을 합니다.
3. <tt style="color: #FF0000">`client`</tt>(예약자)가 식당에 왔음을 확인한 <tt style="color: #FF0000">`owner`</tt>(주인)가 <tt style="color: #FF0000">`reservationFee`</tt>(예약금)를 <tt style="color: #FF0000">`client`</tt>(예약자)에게 반환합니다.

### Scenario-2

1. <tt style="color: #FF0000">`owner`</tt>(식당의 주인)가 Smart Contract를 생성합니다.
2. <tt style="color: #FF0000">`client`</tt>(식당을 예약하고자 하는 사람, 예약자)가 <tt style="color: #FF0000">`reservationFee`</tt>(예약금)을 조건으로 <tt style="color: #FF0000">`reservation`</tt>(예약)을 합니다.
3. <tt style="color: #FF0000">`client`</tt>(예약자)가 식당에 오지 않았음을 확인한 <tt style="color: #FF0000">`owner`</tt>(주인)가 <tt style="color: #FF0000">`reservationFee`</tt>(예약금)의 절반을 <tt style="color: #FF0000">`client`</tt>(예약자)에게 반환하고, 나머지 절반을 패널티의 명목으로 <tt style="color: #FF0000">`owner`</tt>(식당의 주인)이 갖습니다. 

## Code

작성하는 Smart Contract에 대한 설명은 주석으로 추가했습니다.
~~~~
pragma solidity ^0.4.19;

contract NoShow {
    // 예약하는 손님
    // address 유형이며 public으로 선언해서 외부에서 접근할 수 있습니다.
    address public client;
    // 식당 주인
    // address 유형이며 public으로 선언해서 외부에서 접근할 수 있습니다.
    address public owner;
    // 예약금
    // uint 유형이며 public으로 선언해서 외부에서 접근할 수 있습니다.
    uint public reservationFee;
    // 노쇼가 아닐 경우 환불하기 위한 pendingReturns라는 변수 선언
    // mapping 유형이며 key값으로는 address 유형을 갖고, value는 uint 유형입니다.
    // public으로 선언되어 외부에서 접근할 수 있습니다.
    mapping(address => uint) public pendingReturns;
    // 예약 기록을 남김
    // event는 storage나 memory에 비해 사용하는 비용이 저렴합니다.
    // Smart Contract의 트랜잭션의 로그를 기록하는 용도로 사용합니다.
    // MadeReservation라는 event를 만들고 예약을 한 사람의 주소와 예약금을 기록합니다.
    event MadeReservation(address reserver, uint amount);
    // 지켜지지 않은 예약을 기록
    // event는 storage나 memory에 비해 사용하는 비용이 저렴합니다.
    // Smart Contract의 트랜잭션의 로그를 기록하는 용도로 사용합니다.
    // BreakReservation라는 event를 만들고 예약을 지키지 않은 사람의 주소와 예약금을 기록합니다.
    event BreakReservation(address reserver, uint amount);
    // 지켜진 예약을 기록
    // event는 storage나 memory에 비해 사용하는 비용이 저렴합니다.
    // Smart Contract의 트랜잭션의 로그를 기록하는 용도로 사용합니다.
    // KeepPromise라는 event를 만들고 예약을 지킨 사람의 주소와 예약금을 기록합니다.
    event KeepPromise(address reserver, uint amount);
    // 함수 호출자를 손님으로 한정하는 modifier 선언
    // require로 msg.sender가 owner가 아님을 확인합니다.
    // '_;'는 require의 조건이 true면 함수의 본문을 실행한다는 의미입니다.
    modifier onlyClient {
        require(msg.sender != owner);
        _;
    }
    // 함수 호출자를 식당 주인으로 한정하는 modifier 선언
    // require로 msg.sender가 owner가 맞음을 확인합니다.
    // '_;'는 require의 조건이 true면 함수의 본문을 실행한다는 의미입니다.
    modifier onlyOwner {
        require(msg.sender == owner);
        _;
    }
    // 계약 생성자
    // 이 생성자는 Smart Contract를 배포하는 시점 1회에만 호출됩니다.
    // 생성자의 이름은 Smart Contract의 이름과 반드시 같아야 합니다.
    // 생성자는 보통 상태 변수의 데이터를 초기화하는 코드가 작성됩니다.
    function NoShow(address _client, uint _reservationFee) public {
        // 이 계약을 생성한(생성자를 호출한) 사람의 주소를 owner에 할당합니다.
        owner = msg.sender;
        client = _client;
        reservationFee = _reservationFee;
    }
    // 예약 기능
    // payable은 이 함수에 msg.value가 사용됨을 의미합니다.
    // 위에 선언한 modifier onlyClient를 사용해 이 함수는 예약자만 호출할 수 있도록 합니다.
    // public으로 선언되어 외부에서 접근할 수 있습니다.
    function reservation() public payable onlyClient {
        // 이 함수의 호출자의 주소를 client에 할당합니다.
        client = msg.sender;
        // 이 함수 호출에 담겨온 value를 reservationFee에 할당합니다.
        reservationFee = msg.value;
        // event MadeReservation에 예약자와 예약금을 기록합니다.
        MadeReservation(client, reservationFee);
        // mapping 유형의 pendingReturns에 key 값으로 예약자의 주소를 부여하고 예약금을 value로 할당합니다.
        pendingReturns[client] = reservationFee;
    }
    // 예약이 지켜지지 않았을 경우 실행하는 함수
    // 위에 선언한 modifier onlyOwner를 사용해 이 함수는 주인만 호출할 수 있도록 합니다.
    // public으로 선언되어 외부에서 접근할 수 있습니다.
    function withdraw() public onlyOwner {
        // uint 유형의 amount 변수를 선언하고 예약자의 주소로 pendingReturns에서 예약금을 찾아 할당합니다.
        uint amount = pendingReturns[client];
        // 예약금이 있음을 확인합니다.
        if (amount > 0) {
            // 예약금이 있으면 중복 실행을 할 수 있으므로 이를 막기 위해 0으로 만들어 줍니다.
            pendingReturns[client] = 0;
        }
        // event BreakReservation에 예약자와 예약금을 기록합니다.
        BreakReservation(msg.sender, amount/2);
        // 패널티 명목으로 예약자에게 예약금의 반만 반환합니다.
        client.transfer(amount/2);
        // 보상의 명목으로 예약자의 예약금의 절반을 주인이 갖습니다.
        owner.transfer(amount/2);
    }
    // 예약이 지켜졌을 경우 실행하는 함수
    // 위에 선언한 modifier onlyOwner를 사용해 이 함수는 주인만 호출할 수 있도록 합니다.
    // public으로 선언되어 외부에서 접근할 수 있습니다.
    function clientCome() public onlyOwner {
        // uint 유형의 amount 변수를 선언하고 예약자의 주소로 pendingReturns에서 예약금을 찾아 할당합니다.
        uint amount = pendingReturns[client];
        // 예약금이 있음을 확인합니다.
        if (amount > 0) {
            // 예약금이 있으면 중복 실행을 할 수 있으므로 이를 막기 위해 0으로 만들어 줍니다.
            pendingReturns[client] = 0;
        }
        // event KeepPromise에 예약자와 예약금을 기록합니다.
        KeepPromise(msg.sender, reservationFee);
        // 예약자에게 예약금 전액을 반환합니다.
        client.transfer(reservationFee);
    } 
}
~~~~

Solidity를 처음 공부하며 간단한 계약을 작성하기까지 재미도 있었지만 참고할 만한 자료가 생각보다 많지 않아 어려움이 많았습니다.

물론 그 어려움은 아직은 제가 개발자로 부족한 점이 많아서라고 생각합니다. 저와 비슷한 상태의 다른 분들이 이 기록을 보고 학습을 하시는데 도움이 되었으면 합니다.
 
앞으로 더 공부해서 조금 더 나은 Smart Contract를 만들어보도록 하겠습니다. 읽어주셔서 감사합니다.