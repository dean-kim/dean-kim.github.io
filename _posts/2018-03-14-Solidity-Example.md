---
layout: post
title:  "Solidity by Example"
date:   2018-03-14 20:43:59
author: Dean Kim
categories: BlockChain
tags:	BlockChain Solidity Example
cover:  "/assets/instacode.png"
---

블록체인을 공부하며 Smart Contract 작성에 사용되는 Solidity 공식문서를 번역한 글입니다.
제가 이해한 수준으로 기술된 내용도 있어 원본과 내용이 조금 다를 수도 있습니다.
혹시 제가 잘못 이해한 내용이 있다면 말씀해주세요 :)

학습의 목적으로 번역을 해서 아래의 예제 코드를 설명하는 주석을 공식문서보다 자세하게 작성하였습니다.
이에 대해서도 잘못 작성된 내용은 말씀해주세요 :)

# Solidity by Example

## Voting

아래의 계약은 확실히 복잡하지만 Solidity의 특징들을 보여주는 쇼케이스입니다. 그것은 투표 계약을 이행합니다. 
물론, 전자 투표의 주요 문제는 올바른 사람에게 투표권을 부여하는 방법과 조작을 방지하는 방법입니다. 
우리는 여기에서 모든 문제를 해결하지는 않겠지만, 투표 계산이 자동적이면 완전하게 투명하도록 투표를 위임할 수 있는 방법을 보여줄 것입니다.

발의된 안건당 하나의 계약서를 작성하여 각 옵션에 대한 짧은 이름을 제공하는 것이 아이디어입니다. 그러면 의장을 맡고 있 계약 작성자는 각 주소에 개별적으로 투표권을 부여합니다.

주소가 있는 사람들은 자신을 투표하거나 신뢰할 수 있는 사람에게 투표를 위임할 수 있습니다.

투표 시간이 끝나면 <tt style="color: #FF0000">`winningProposal()`</tt>은 가장 많은 득표 수를 가진 제안을 반환합니다.

~~~~
pragma solidity ^0.4.16;

/// @title 위임투표.
contract Ballot {
    // 변수에 사용될 새로운 복합 유형을 struct로 선언합니다.
    // 한 사람의 투표자를 나타냅니다.
    struct Voter {
        uint weight; // 부호가 없는 정수유형, weight는 위임의 누적입니다. 
        bool voted;  // 부울유형, 만약 true이면 그 사람은 이미 투표한 상태입니다.
        address delegate; // address 유형, 위임된 사람
        uint vote;   // 부호가 없는 정수유형, 투표한 제안의 인덱스입니다.
    }

    // 하나의 제안에 속하는 변수들의 유형을 struct로 선언합니다.
    struct Proposal {
        bytes32 name;   // bytes32 유형, 짧은 이름(최대 32바이트)
        uint voteCount; // 부호가 없는 정수유형, 누적된 투표 수
    }

    address public chairperson; // address 유형, 의장(계약의 생성자)

    // mapping 유형, 이것은 각 가능한 주소에 대해 `Voter` struct를 저장하는 상태 변수를 선언합니다.
    mapping(address => Voter) public voters;

    // 동적으로 크기가 지정된 `Proposal` struct의 배열인 proposals를 선언합니다.
    Proposal[] public proposals;

    /// `proposalNames` 중 하나를 선택하기 위한 새로운 투표 용지를 만드십시오.
    // 계약의 이름과 동일한 이름의 함수는 생성자입니다.
    // 이 생성자 함수는 이름이 proposalNames 이며 동적으로 크기가 지정된 bytes32 배열 유형인 parameter를 받습니다.
    // 또한 이 함수의 가시성은 public으로 외부에서 접근할 수 있습니다.
    function Ballot(bytes32[] proposalNames) public {
        chairperson = msg.sender; // 이 함수를 호출한 사람이 의장이 됩니다.(address 유형)
        voters[chairperson].weight = 1; // 위에 선언한 mapping 유형의 voters 중 키값을 의장의 주소로 의장의 weight에 1을 할당합니다.

        // 제공된 제안서 이름 각각에 대해 새 제안서 개체를 만들어 배열 끝에 추가합니다.
        for (uint i = 0; i < proposalNames.length; i++) {
            // `Proposal ({...})`은 임시 Proposal 객체를 만들고 `proposals.push(...)`는 `proposals`의 끝에 추가합니다.
            proposals.push(Proposal({
                name: proposalNames[i],
                voteCount: 0
            }));
        }
    }

    // 투표자에게 이 안건에 대한 투표권을 부여합니.
    // 의장만이 호출할 수 있습니다.
    // giveRightToVote 함수는 address 유형의 voter를 parameter로 받습니다.
    // 이 함수의 가시성은 public으로 선언되어 외부에서 접근가능합니다.
    function giveRightToVote(address voter) public {
        // `require`의 인수가`false`로 평가되면, 실행을 중단하고, 모든 상태와 이더 잔액을 실행이전으 되돌립니다. 
        // 함수가 잘못 호출되는 경우에는 이것을 사용하는 것이 좋습니다. 
        // 그러나 조심하십시오, 이것은 현재 제공된 모든 gas를 소비할 것입니다(이것은 앞으로 바뀔 예정입니다).
        require(
            // 이 함수의 호출자가 의장과 동일한지 평가합니다.
            (msg.sender == chairperson) &&
            // 그리고 and 조건으로 voters(앞서 mapping 유형으로 선언)에서 키값 voter(함수의 인자로 받은 address 유형의 parameter)의 attr voted에 할당된 값이 존재하는지 평가합니다.('!'은 not의 의미) 
            !voters[voter].voted &&
            // 그리고 and 조건으로 voters(앞서 mapping 유형으로 선언)에서 키값 voter(함수의 인자로 받은 address 유형의 parameter)의 attr weight에 할당된 값이 0인지 평가합니다.
            (voters[voter].weight == 0)
        );
        // require이 true로 평가되면 voters(앞서 mapping 유형으로 선언)에서 키값 voter(함수의 인자로 받은 address 유형의 parameter)의 attr weight에 1을 할당합니다.
        voters[voter].weight = 1;
    }

    /// 투표권을 위임하는 함수를 작성합니다.
    // delegate 함수는 address 유형의 to를 parameter로 받습니다.
    // 이 함수의 가시성은 public으로 선언되어 외부에서 접근가능합니다.
    function delegate(address to) public {
        // reference를 지정합니다.
        // 지정된 값으로 초기화된 새 임시 메모리 struct를 만들고 storage로 복사합니다.
        Voter storage sender = voters[msg.sender];
        // sender의 attr voted를 평가합니다.(voted는 부울 유형)
        require(!sender.voted);

        // Self-delegation는 허용되지 않습니다.
        require(to != msg.sender);

        // 'to'가 위임하는 한 위임은 계속 추진됩니다. 
        // 일반적으로 이러한 루프는 너무 위험하기 때문에 너무 길면 블록에서 사용할 수 있는 것보다 더 많은 gas가 필요할 수 있습니다. 
        // 이 경우 위임은 실행되지 않지만 다른 상황에서는 이러한 루프로 인해 계약서가 완전히 "고착"될 수 있습니다.
        // address(0)은 트랜잭션은 새 계약을 작성합니다. 여기서 0은 주소가 0이라는 의미가 아니라 보낸 사람의 주소와 보낸 거래 수("nonce")에서 파생된 주소입니다.
        while (voters[to].delegate != address(0)) {
            to = voters[to].delegate;

            // 위임에 루프가 있음을 알았습니다. 허용되지 않았습니다. Self-delegation인지 평가합니다. 
            require(to != msg.sender);
        }

        // 'sender'는 참조이므로 `voters[msg.sender].voted`를 sender.voted로 수정합니다.
        // 투표권을 위임했으므로 attr voted에 true를 할당합니다.
        sender.voted = true;
        // 'sender'는 참조이므로 `voters[msg.sender].delegate`를 sender.delegate로 수정합니다.
        // 투표권을 위임한 사람의 주소를 arrt delegate에 할당합니다.
        sender.delegate = to;
        // delegate_ 라는 storage에 voters[to]를 저장합니다.
        Voter storage delegate_ = voters[to];
        // delegate_의 attr voted를 평가합니다.
        if (delegate_.voted) {
            // 만약 위임받은 사람이 이미 투표한 경우라면 투표수를 증가시킵니다.
            proposals[delegate_.vote].voteCount += sender.weight;
        } else {
            // 만약 위임받은 사람이 투표하지 않았으면 위임하는 사람의 weight를 위임받은 사람에게 더합니다.
            delegate_.weight += sender.weight;
        }
    }

    /// 안건 `proposals[proposal].name`에 투표하는 함수입니다.(위임된 투표권 행사 포함입니다.)
    //  vote 함수는 uint 유형의 proposal을 parameter로 받습니다.
    //  이 함수의 가시성은 public으로 선언되어 외부에서 접근가능합니다.
    function vote(uint proposal) public {
        // reference를 지정합니다.
        // 지정된 값으로 초기화된 새 임시 메모리 struct를 만들고 storage로 복사합니다.
        Voter storage sender = voters[msg.sender];
        // sender의 attr voted를 평가합니다. (false이어야 다음 구문이 실행됩니다.)
        require(!sender.voted);
        // 투표를 했으니 이제 voted를 true로 변환합니다.
        sender.voted = true;
        // 해당 안건에 투표했으니 sender의 attr vote에 함수 vote에 인수로 넣었던 proposal(uint 유형)을 할당합니다.
        sender.vote = proposal;

        // If `proposal` is out of the range of the array,
        // this will throw automatically and revert all
        // changes.
        // (?)`proposal '이 배열의 범위를 벗어나면 자동으로 던지고 모든 변경 사항을 되돌립니다.
        // struct Proposal로 이루어진 proposals 라는 배열에서 vote 함수의 인수로 넣은 proposal을 키값으로 하는 Proposal의 voteCount에 sender의 weight를 더합니다.
        proposals[proposal].voteCount += sender.weight;
    }

    /// 이전에 투표한 것들을 근거로 어떤 제안이 가장 많이 투표받았는지 계산하는 함수입니다.
    //  winningProposal 함수는 public으로 선언되어 외부에서 접근가능하며 view라는 함수의 Modifier로 상태 수정을 허용하지 않습니다.
    //  또한 winningProposal 함수는 uint 유형의 winningProposal_를 반환합니다.
    function winningProposal() public view
            returns (uint winningProposal_)
    {
        // uint 유형의 winningVoteCount 변수에 0을 할당합니다.
        uint winningVoteCount = 0;
        // struct Proposal로 이루어진 proposals 라는 배열의 요소를 도는 for loop 입니다.
        for (uint p = 0; p < proposals.length; p++) {
            // struct Proposal로 이루어진 proposals 라는 배열의 요소의 attr voteCount가 winningVoteCount보다 큰 경우 winningVoteCount에 voteCount를 할당합니다.
            // winningProposal_에 해당 proposal을 할당합니다.
            if (proposals[p].voteCount > winningVoteCount) {
                winningVoteCount = proposals[p].voteCount;
                winningProposal_ = p;
            }
        }
    }

    // winningProposal() 함수를 호출하여 제안 배열에 포함된 제안의 인덱스를 가져온 다음 가장 많은 득표한 제안의 이름을 반환합니다.
    // winnerName 함수는 public으로 선언되어 외부에서 접근가능하며 view라는 함수의 Modifier로 상태 수정을 허용하지 않습니다.
    // 또한 winnerName 함수는 bytes32 유형의 winnerName_를 반환합니다.
    function winnerName() public view
            returns (bytes32 winnerName_)
    {
        winnerName_ = proposals[winningProposal()].name;
    }
}
~~~~

### Possible Improvements

현재 모든 참가자에게 투표권을 부여하기 위해 많은 거래가 필요합니다. 더 나은 방법을 생각해 볼 수 있습니까?

## Blind Auction

이 섹션에서는 Ethereum에 대해 얼마나 쉽게 완전한 블라인드 경매 계약을 만들 수 있는지를 보여줄 것입니다. 
모든 사람이 입찰가를 볼 수 있는 공개 경매부터 시작하여 입찰 기간이 끝날 때까지 실제 입찰가를 볼 수 없는 블라인드 경매로 이 계약을 연장합니다.

### Simple Open Auction

다음 간단한 경매 계약의 일반적인 개념은 모든 사람이 입찰 기간 중에 입찰을 보낼 수 있다는 것입니다. 
입찰가는 입찰자를 입찰에 묶기 위해 이미 money / ether를 전송하는 것을 포함합니다. 
가장 높은 입찰가가 발생하면 이전에 가장 높은 입찰자가 돈을 돌려받습니다. 
입찰 기간이 끝난 후 수혜자가 돈을 받을 수 있도록 수동으로 계약서를 호출해야합니다. 
계약서는 스스로 활성화할 수 없습니다.

~~~~
pragma solidity ^0.4.21;

contract SimpleAuction {
    // 이 경매의 parameter 입니다. 시간은 절대 유닉스 타임스탬프(1970.01.01 이후의 초 또는 시간 단위(초) 입니다.
    // address 유형의 beneficiary라는 변수를 public으로 선언했습니다.
    address public beneficiary;
    // uint 유형의 auctionEnd라는 변수를 public으로 선언했습니다.
    uint public auctionEnd;

    // 경매의 현재 상태를 나타냅니다.
    // address 유형의 highestBidder라는 변수를 public으로 선언했습니다. 현재 경매에서 최고가를 제시한 사람의 주소입니다.
    address public highestBidder;
    // uint 유형의 highestBid라는 변수를 public으로 선언했습니다. 현재 경매에서 제시된 최고가 입니다.
    uint public highestBid;

    // 최고가가 갱신되어 이전 입찰가의 인출 허용
    // mapping 유형의 pendingReturns를 선언합니다.
    mapping(address => uint) pendingReturns;

    // 마지막에 true로 설정하면 변경이 허용되지 않습니다.
    bool ended;

    // 어떤 변화에 따라 발생하는 이벤트를 선언합니다.
    // HighestBidIncreased 라는 이벤트는 인자로 address 유형의 bidder와 uint 유형의 amount를 받습니다.
    event HighestBidIncreased(address bidder, uint amount);
    // AuctionEnded 라는 이벤트는 인자로 address 유형의 winner와 uint 유형의 amount를 받습니다.
    event AuctionEnded(address winner, uint amount);

    // 다음은 세 개의 슬래시로 인식할 수 있는 natspec 주석입니다. 사용자에게 거래 확인을 요청하면 표시됩니다.
    // natspec 주석은 슬래시 3개(///), 여러 줄로 표기 시 (/** ... */)를 사용합니다. 함수 호출 시 사용자에게 보여지는 내용으로 사용합니다.

    /// Create a simple auction with `_biddingTime`
    /// seconds bidding time on behalf of the
    /// beneficiary address `_beneficiary`.
    /// address 유형의 _beneficiary를 계약 선언에서 선언한 beneficiary 변수에 할당하고,
    /// uint 유형의 _biddingTime(입찰 시간, 초)에 현재 시간을 더해 계약 선언에서 선언한 auctionEnd에 할당해
    /// simple auction을 만듭니다.
    //  아래의 SimpleAuction는 계약 이름과 같은, 생성자의 역할을 합니다.
    function SimpleAuction(
        uint _biddingTime,
        address _beneficiary
    ) public {
        beneficiary = _beneficiary;
        auctionEnd = now + _biddingTime;
    }

    /// Bid on the auction with the value sent
    /// together with this transaction.
    /// The value will only be refunded if the
    /// auction is not won.
    /// 이 거래와 함께 보낸 값으로 경매에 입찰하십시오. 낙찰되지 않은 경우에만 환불됩니다.
    // bid 함수는 public으로 선언되어 외부에서 접근이 가능합니다.
    // bid 함수는 payable 이라는 modifier를 사용합니다. 이 payable은 함수 호출 시 Ether를 받을 수 있는 기능을 제공합니다.
    // payable이 없는 함수에 대해서는 transaction이 reject됩니다.
    // payable는 예약어로 함수의 이름으로 사용할 수 없습니다.
    function bid() public payable {
        // 인수가 필요하지 않으며 모든 정보가 이미 트랜잭션의 일부입니다. 
        // 함수를 Ether을 수신하려면 payable 키워드가 필요합니다.

        // 입찰이 끝나면 call을 되돌리도록 require에 now와 auctionEnd를 비교해 판정합니다.
        // now는 uint 유형이며 현재 블록의 타임 스탬프를 말합니다.(block.timestamp의 별칭입니다.)
        require(now <= auctionEnd);

        // 입찰가가 높지 않으면 돈을 되돌려 줍니다.
        // require에 입찰가(msg.value)와 현재 최고액(highestBid)를 비교해 판정합니다.
        require(msg.value > highestBid);

        if (highestBid != 0) {
            // highestBidder.send(highestBid)를 사용하여 돈을 보내면 신뢰할 수 없는 계약을 실행할 수 있으므로 보안 위험이 있습니다. 
            // 수령인이 돈을 스스로 인출하도록 하는 것이 더 안전합니다.
            // mapping 유형의 pendingReturns에서 키값 highestBidder(address 유형)에 uint 유형의 highestBid를 더합니다.
            pendingReturns[highestBidder] += highestBid;
        }
        // 새로운 입찰가를 제시한 사람을 교체합니다.
        highestBidder = msg.sender;
        // 새로운 입찰가를 기존 입찰가와 교체합니다.
        highestBid = msg.value;
        // 2018.03.08 v0.4.21에 emit 키워드가 도입되었습니다.
        // 'emit EventName();'으로 이벤트를 명시적으로 호출할 수 있습니다.
        // HighestBidIncreased 라는 이벤트를 호출합니다. 
        // 위에 선언한 이벤트 HighestBidIncreased에서 bidder에 현재 입찰자(address 유형), amount에 현재 입찰가(uint 유형)를 할당합니다.
        // 이벤트가 호출되면 EVM이 블록체인에 트랜잭션 로그에 기록을 하고, 이 로그를 외부(자바스크립트 등)에서 사용할 수 있습니다.
        emit HighestBidIncreased(msg.sender, msg.value);
    }

    /// Withdraw a bid that was overbid.
    /// overbid된 입찰 철회
    //  함수 withdraw는 public으로 선언되어 외부에서 접근할 수 있습니다.
    //  returns 키워드 다음에 온 (bool)은 이 함수의 output의 유형을 의미합니다. 그리고 output의 이름도 지정할 수 있습니다. ex) returns (uint _a)
    //  returns 키워드는 함수 withdraw가 실행된 뒤 output을 받겠다는 의미입니다.
    function withdraw() public returns (bool) {
        // uint 유형의 amount에 mapping 유형의 pendingReturns에 저장된 값을 할당합니다.(저장된 값은 키값이 msg.sender인 pendingReturns에 저장된 값입니다.)
        uint amount = pendingReturns[msg.sender];
        if (amount > 0) {
            // 받는 사람이 `send '가 반환되기 전에 수신 호출의 일부로서 이 함수를 다시 호출할 수 있기 때문에 이것을 0으로 설정하는 것이 중요합니다.
            pendingReturns[msg.sender] = 0;

            // 내장 send 함수를 사용합니다.
            // <address>.send(uint256 amount) 주어진 양의 Wei를 address에 보내고 실패하면 false를 반환합니다.
            if (!msg.sender.send(amount)) {
                // 다시 호출할 필요가 없습니다. amount만 다시 설정하면 됩니다.
                pendingReturns[msg.sender] = amount;
                return false;
            }
        }
        return true;
    }

    /// End the auction and send the highest bid
    /// to the beneficiary.
    /// 경매를 끝내고 최고 입찰액을 수혜자에게 보내십시오.
    //  함수 auctionEnd는 public으로 선언되어 외부에서 접근 가능합니다.
    function auctionEnd() public {
        // 다른 계약(즉, 함수를 호출하거나 Ether을 보내는)과 상호 작용하는 함수를 다음 세 단계로 구조화하는 것이 좋습니다.
        // 1. 점검 조건
        // 2. 행동 수행 (잠재적으로 변화하는 조건)
        // 3. 다른 계약과 상호 작용
        // 이러한 단계가 뒤섞인다면, 다른 계약은 현재 계약으로 되돌아가서 상태를 수정하거나 효과(ether 지불)를 여러 번 수행할 수 있습니다. 
        // 내부적으로 호출된 함수가 외부 계약과의 상호 작용을 포함하는 경우 외부 계약과의 상호 작용으로 간주되어야합니다.

        // 1. Conditions
        // 경매가 아직 끝나지 않았음을 판정
        require(now >= auctionEnd);
        // 이 함수가 이미 호출되었는지 판정
        require(!ended); 

        // 2. Effects
        ended = true;
        // 위에 선언한 AuctionEnded 이벤트를 호출합니다.
        emit AuctionEnded(highestBidder, highestBid);

        // 3. Interaction
        // 내장함수 transfer를 사용합니다. <address>.transfer(uint256 amount) 형식으로 사용합니다.
        // beneficiary의 address로 highestBid(단위 : Wei)를 보냅니다.
        beneficiary.transfer(highestBid);
    }
}
~~~~

### Blind Auction

이전 공개 경매는 다음과 같은 블라인드 경매로 확장됩니다. 
블라인드 경매의 장점은 입찰 기간의 마지막을 향해 어떤 시간 압박도 없다는 것입니다. 
투명한 컴퓨팅 플랫폼에 블라인드 경매를 만드는 것이 모순처럼 들릴 수도 있지만 암호화가 도움이 될 것입니다.

입찰 기간 동안 입찰자는 실제로 입찰를 보내지 않고 해시된 버전만 전송합니다. 
현재 해시값이 동일한 두 개의(충분히 긴) 값을 찾는 것은 실제로 불가능하기 때문에 입찰자는 그 값으로 입찰을 위탁합니다. 
입찰 기간이 끝나면 입찰자는 입찰 금액을 암호화되지 않은 상태로 보내야 하며 계약자는 해시값이 입찰 기간 동안 제공된 값과 동일한지 확인해야 합니다.

또 다른 도전은 경매가 구속력을 가지면 동시에 블라인드로 만드냐하는 것입니다. 
경매에서 이긴 후 입찰자가 돈을 보내지 못하도록 하는 유일한 방법은 입찰자가 입찰과 함께 돈을 발송하도록 하는 것입니다. 
Ethereum에서는 value transfer를 blind할 수 없기 때문에 누구나 그 value를 볼 수 있습니다.

다음 계약에서는 가장 높은 입찰가보다 큰 값을 수락하여 이 문제를 해결합니다. 
물론 이것은 공개 단계에서만 확인할 수 있기 때문에 일부 입찰가가 유효하지 않을 수 있으며 이는 의도적으로(값이 높은 전송으로 잘못된 입찰을 하기위한 명시적인 플래그를 제공하기까지 함)제공됩니다, 입찰자는 여러 개의 높거나 낮은 무효 입찰을 배치하여 경쟁을 혼란스럽게 만들 수 있습니다.

~~~~
// 버전 프라그마(0.4.21 버전으로 작성되었음을 의미하며 해당 버전의 컴파일러 사용을 지시합니다.)
pragma solidity ^0.4.21;

// BlindAuction 계약을 선언합니다.
contract BlindAuction {
    // BlindAuction 계약에서 변수로 사용되는 Bid struct(complex type)으로 선언합니다.
    // Bid는 하나의 입찰로 사용됩니다.
    struct Bid {
        // bytes32 유형의 blindedBid를 선언합니다.
        bytes32 blindedBid;
        // uint 유형의 deposit를 선언합니다.
        uint deposit;
    }

    // address 유형의 beneficiary를 public으로 선언합니다.
    // public으로 변수를 선언하면 외부에서 접근이 가능합니다.
    address public beneficiary;
    // uint 유형의 biddingEnd를 public으로 선언합니다.
    uint public biddingEnd;
    // uint 유형의 revealEnd를 public으로 선언합니다.
    uint public revealEnd;
    // bool 유형의 ended를 public으로 선언합니다.
    bool public ended;

    // mapping 유형의 bids를 public으로 선언합니다.
    // address를 key로, value는 Bid의 배열형태로 갖습니다.
    mapping(address => Bid[]) public bids;

    // address 유형의 highestBidder를 public으로 선언합니다.
    address public highestBidder;
    // uint 유형의 highestBid를 public으로 선언합니다.
    uint public highestBid;

    // 이전 입찰가의 인출 허용하는 pendingReturns를 mapping 유형으로 선언합니다.
    mapping(address => uint) pendingReturns;

    // AuctionEnded라는 이벤트를 선언합니다. 이 이벤트는 address 유형의 winner와 uint 유형의 highestBid를 parameter로 받습니다.
    event AuctionEnded(address winner, uint highestBid);

    /// Modifiers are a convenient way to validate inputs to
    /// functions. `onlyBefore` is applied to `bid` below:
    /// The new function body is the modifier's body where
    /// `_` is replaced by the old function body.
    /// modifier는 함수에 대한 입력 값을 검사하는 편리한 방법입니다. 
    /// 아래의 함수 `bid'에 `onlyBefore`을 적용합니다: 새로운 함수 본문은 `_`가 이전 함수 본문을 대체하는 modifier의 본문입니다.
    // modifier onlyBefore를 선언합니다.
    // onlyBefore는 uint 유형의 _time을 인자로 받습니다. 입력된 _time이 now 보다 크면 함수 본문을 실행합니다.
    modifier onlyBefore(uint _time) { require(now < _time); _; }
    // modifier onlyAfter를 선언합니다.
    // onlyAfter는 uint 유형의 _time을 인자로 받습니다. 입력된 _time이 now 보다 작으면 함수 본문을 실행합니다.
    modifier onlyAfter(uint _time) { require(now > _time); _; }

    // 함수 BlindAuction를 선언합니다.
    // 함수 BlindAuction는 uint 유형의 _biddingTime과 _revealTime를 address 유형의 _beneficiary를 parameter로 받습니다.
    // 또한 public으로 선언되어 외부에서 접근이 가능합니다.
    // 위에 선언한 계약과 같은 이름의 BlindAuction 함수는 생성자입니다.
    function BlindAuction(
        uint _biddingTime,
        uint _revealTime,
        address _beneficiary
    ) public {
        // 앞서 선언한 beneficiary에 _beneficiary를 할당합니다.
        beneficiary = _beneficiary;
        // 앞서 선언한 biddingEnd에 블록시간(now)과 parameter로 받은 _biddingTime를 더해 할당합니다.
        // now는 uint 유형이며 현재 블록의 타임 스탬프를 말합니다.(block.timestamp의 별칭입니다.)
        biddingEnd = now + _biddingTime;
        // revealEnd에는 위의 biddingEnd에 parameter로 받은 _revealTime를 더해 할당합니다.
        revealEnd = biddingEnd + _revealTime;
    }

    /// Place a blinded bid with `_blindedBid` = keccak256(value,
    /// fake, secret).
    /// The sent ether is only refunded if the bid is correctly
    /// revealed in the revealing phase. The bid is valid if the
    /// ether sent together with the bid is at least "value" and
    /// "fake" is not true. Setting "fake" to true and sending
    /// not the exact amount are ways to hide the real bid but
    /// still make the required deposit. The same address can
    /// place multiple bids.
    /// `_blindedBid` = keccak256(value, fake, secret)으로 블라인드 입찰을 하십시오. 
    /// 보낸 ether는 공개 단계에서 입찰가가 정확하게 공개된 경우에만 환불됩니다. 
    /// 입찰가와 함께 보낸 ether가 "value"이상이고 "fake"가 사실이 아닌 경우 입찰이 유효합니다. 
    /// "fake"를 true로 설정하고 정확한 금액을 보내지 않으면 실제 입찰가를 숨기고 필요한 입금을 계속하는 방법입니다. 
    /// 동일한 주소는 여러 입찰을 할 수 있습니다.
    // 함수 bid는 bytes32 유형의 _blindedBid를 parameter로 받습니다.
    // 그리고 public으로 선언되어 외부에서 접근이 가능합니다.
    // 또한 modifier payable을 사용합니다. 이 payable은 함수 호출 시 Ether를 받을 수 있는 기능을 제공합니다.
    // 마지막으로 위에 선언한 modifier onlyBefore를 사용합니다. onlyBefore로 parameter biddingEnd를 판별해서 조건이 true가 되면 함수의 본문을 실행시킵니다.
    function bid(bytes32 _blindedBid)
        public
        payable
        onlyBefore(biddingEnd)
    {
        // bid 함수의 본문입니다.
        // mapping 유형 bids에 msg.sender를 키값으로 parameter _blindedBid과 메세지에 담겨져 전달된 value(msg.value)를 struct Bid에 할당합니다.
        bids[msg.sender].push(Bid({
            blindedBid: _blindedBid,
            deposit: msg.value
        }));
    }

    /// Reveal your blinded bids. You will get a refund for all
    /// correctly blinded invalid bids and for all bids except for
    /// the totally highest.
    /// 블라인 입찰을 공개합니다. 완전히 블라인드인 모든 무효 입찰가와 완전히 높은 입찰가를 제외한 모든 입찰가에 대해 환불을 받게됩니다.
    // 함수 reveal을 선언합니다.
    // 함수 reveal은 uint 유형을 갖는 배열 _values과 bool 유형을 갖는 배열 _fake 그리고 bytes32 유형을 갖는 배열 _secret을 parameter로 받습니다.
    // 그리고 public으로 선언되어 외부에서 접근이 가능합니다.
    // modifier onlyAfter와 onlyBefore를 사용합니다.
    function reveal(
        uint[] _values,
        bool[] _fake,
        bytes32[] _secret
    )
        public
        onlyAfter(biddingEnd)
        onlyBefore(revealEnd)
    {
        // 함수 reveal의 본문입니다.
        // uint 유형의 length를 선언하고, mapping 유형의 bids에서 키값이 msg.sender인 value의 length를 할당합니다.
        uint length = bids[msg.sender].length;
        // 함수 reveal의 parameter들의 length와 위에 선언한 length를 비교합니다.
        require(_values.length == length);
        require(_fake.length == length);
        require(_secret.length == length);
        
        // uint 유형의 refund를 선언합니다.
        uint refund;
        // for loop 시작
        for (uint i = 0; i < length; i++) {
            // bid에 mapping 유형의 bids의 값을 할당합니다.
            var bid = bids[msg.sender][i];
            // value, fake, secret에 _values, _fake, _secret 값들을 할당합니다.
            var (value, fake, secret) =
                    (_values[i], _fake[i], _secret[i]);
            // bid의 attr blindedBid이 keccak256(value, fake, secret)와 비교 판정
            if (bid.blindedBid != keccak256(value, fake, secret)) {
                // 입찰은 실제로 공개되지 않았습니다.
                // 환불하지 않습니다.
                continue;
            }
            // uint 유형의 refund에 bid의 attr deposit을 더합니다.
            refund += bid.deposit;
            // fake가 false이고 bid의 attr deposit이 value 이상인지 판별
            if (!fake && bid.deposit >= value) {
                // 함수 placeBid에 msg.sender와 value를 넣어 판별
                if (placeBid(msg.sender, value))
                    // 위의 판별이 true이면 uint 유형의 refund에서 value만큼 차감합니다.
                    refund -= value;
            }
            // 발신자가 동일한 입금을 다시 청구할 수 없게 만듭니다.
            // 0을 bytes32 유형으로 변환
            bid.blindedBid = bytes32(0);
        }
        // msg.sender에게 refund를 보냅니다.
        // 내장함수 transfer를 사용합니다. <address>.transfer(uint256 amount) 형식으로 사용합니다.
        // 여기서 단위는 Wei를 사용합니다.
        msg.sender.transfer(refund);
    }

    // 함수 placeBid를 선언합니다.
    // 이것은 계약 자체(또는 파생된 계약)에서만 호출할 수 있는 "internal"함수입니다.
    // 함수 placeBid는 address 유형의 bidder, uint 유형의 value를 parameter로 받습니다.
    // return 키워드를 사용해서 반환값이 있음을 알려주며 반환값은 bool 유형을 갖는 success라는 변수입니다.
    function placeBid(address bidder, uint value) internal
            returns (bool success)
    {
        // placeBid 함수 본문
        // uint 유형의 value와 highestBid 비교
        if (value <= highestBid) {
            return false;
        }
        // highestBidder가 0인지 판정
        if (highestBidder != 0) {
            // 이전에 가장 높은 입찰자에게 환불합니다.
            // mapping 유형의 pendingReturns에 키값 highestBidder의 value에 highestBid를 더합니다.
            pendingReturns[highestBidder] += highestBid;
        }
        // highestBid에 value를 할당합니다.
        highestBid = value;
        // highestBidder에 bidder를 할당합니다.
        highestBidder = bidder;
        // true를 반환합니다.
        return true;
    }

    /// Withdraw a bid that was overbid.
    /// overbid된 입찰을 철회합니다.
    // 함수 withdraw를 선언합니다.
    // public으로 선언되어 외부에서 접근이 가능합니다.
    function withdraw() public {
        // uint 유형의 amount를 선언합니다.
        // mapping 유형의 pendingReturns에서 키값 msg.sender에 해당하는 value를 amount에 할당합니다.
        uint amount = pendingReturns[msg.sender];
        // uint 유형의 amount가 0보다 큰지 판별합니다.
        if (amount > 0) {
            // 받는 사람이 'transfer'가 반환되기 전에 수신 호출의 일부로 이 함수를 다시 호출 할 수 있으므로 이 값을 0으로 설정하는 것이 중요합니다(위의 conditions -> effects -> interaction에 대한 설명 참조).
            // 반환하려는 값을 0으로 변경합니다.
            pendingReturns[msg.sender] = 0;

            // msg.sender에게 amount를 보냅니다.
            // 내장함수 transfer를 사용합니다. <address>.transfer(uint256 amount) 형식으로 사용합니다.
            // 여기서 단위는 Wei를 사용합니다.
            msg.sender.transfer(amount);
        }
    }

    /// End the auction and send the highest bid
    /// to the beneficiary.
    /// 경매를 끝내고 최고 입찰액을 수혜자에게 보냅니다.
    // 함수 auctionEnd를 선언합니다.
    // public으로 선언되어 외부에서 접근이 가능합니다.
    // modifier onlyAfter를 사용합니다.
    function auctionEnd()
        public
        onlyAfter(revealEnd)
    {
        // ended가 false인지 판별합니다.
        require(!ended);
        // 위에 선언한 AuctionEnded 이벤트를 실행시켜 로그에 남깁니다.
        // AuctionEnded 이벤트는 address 유형의 winner(여기서는 highestBidder)와 uint 유형의 highestBid(여기서는 highestBid)를 parameter로 받습니다.
        emit AuctionEnded(highestBidder, highestBid);
        // ended에 true를 할당합니다.
        ended = true;
        // beneficiary에게 highestBid를 전송합니다.
        // 내장함수 transfer를 사용합니다. <address>.transfer(uint256 amount) 형식으로 사용합니다.
        // 여기서 단위는 Wei를 사용합니다.
        beneficiary.transfer(highestBid);
    }
}
~~~~

## Safe Remote Purchase

~~~~
pragma solidity ^0.4.21;

// contract Purchase를 선언 
contract Purchase {
    // uint 유형의 value를 public으로 선언합니다.
    uint public value;
    // address 유형의 seller를 public으로 선언합니다.
    address public seller;
    // address 유형의 buyer를 public으로 선언합니다.
    address public buyer;
    // enum 유형의 State를 선언하며 멤버는 Created, Locked, Inactive로 구성됩니다.
    enum State { Created, Locked, Inactive }
    // State 유형의 state를 public으로 선언합니다.
    State public state;

    // 함수 Purchase를 선언합니다.
    // public으로 선언되어 외부에서 접근이 가능합니다.
    // modifier payable을 사용합니다. 이 payable은 함수 호출 시 Ether를 받을 수 있는 기능을 제공합니다.
    // `msg.value`가 짝수인지 확인합니다.
    // 만약 홀수라면 나눗셈으로 잘라냅니다.
    // 곱셈을 통해 그것이 홀수가 아니라는 것을 확인합니다.
    function Purchase() public payable {
        seller = msg.sender;
        value = msg.value / 2;
        require((2 * value) == msg.value);
    }

    // modifier condition을 선언합니다.
    // modifier condition는 bool 유형의 _condition를 parameter로 받습니다.
    // _condition이 true면 함수 본문을 실행합니다.
    modifier condition(bool _condition) {
        require(_condition);
        _;
    }

    // modifier onlyBuyer를 선언합니다.
    // modifier onlyBuyer는 address 유형의 msg.sender가 address 유형의 buyer와 일치하는지 판별합니다. 일치하면 함수 본문을 실행합니다.
    modifier onlyBuyer() {
        require(msg.sender == buyer);
        _;
    }

    // modifier onlySeller를 선언합니다.
    // modifier onlySeller는 address 유형의 msg.sender가 address 유형의 seller와 일치하는지 판별합니다. 일치하면 함수 본문을 실행합니다.
    modifier onlySeller() {
        require(msg.sender == seller);
        _;
    }

    // modifier inState을 선언합니다.
    // modifier inState는 State 유형의 _state를 parameter로 받습니다.
    // State 유형의 state와 State 유형의 _state가 일치하는지 판별합니다. 일치하면 함수 본문을 실행합니다.
    modifier inState(State _state) {
        require(state == _state);
        _;
    }

    // 이벤트 Aborted를 선언합니다.
    event Aborted();
    // 이벤트 PurchaseConfirmed를 선언합니다.
    event PurchaseConfirmed();
    // 이벤트 ItemReceived를 선언합니다.
    event ItemReceived();

    /// Abort the purchase and reclaim the ether.
    /// Can only be called by the seller before
    /// the contract is locked.
    /// 구매를 중단하고 ether를 되돌립니다. 
    /// 계약이 잠기기 전에 판매자가 호출할 수 있습니다.
    // 함수 abort를 선언합니다.
    // public으로 선언되어 외부에서 접근이 가능합니다.
    // modifier onlySeller와 inState를 사용합니다.
    // modifier inState에 State의 멤버 Created를 넣어 판별합니다.
    function abort()
        public
        onlySeller
        inState(State.Created)
    {
        // 이벤트 Aborted를 호출합니다.
        emit Aborted();
        // state에 State 멤버 Inactive를 할당합니다.
        state = State.Inactive;
        // seller에게 this.balance를 전송합니다.
        // 내장함수 transfer를 사용합니다. <address>.transfer(uint256 amount) 형식으로 사용합니다.
        // 여기서 단위는 Wei를 사용합니다.
        seller.transfer(this.balance);
    }

    /// Confirm the purchase as buyer.
    /// Transaction has to include `2 * value` ether.
    /// The ether will be locked until confirmReceived
    /// is called.
    /// 구매자로 구매를 확인합니다. 트랜잭션은 `2 * value` ether를 포함해야합니다. 
    /// confirmReceived가 호출될 때까지 ether는 잠겨있을 것입니다.
    // 함수 confirmPurchase를 선언합니다.
    // public으로 선언되어 외부에서 접근이 가능합니다.
    // modifier inState와 condition을 사용합니다.
    // modifier inState에 State의 멤버 Created를 넣어 판별합니다.
    // modifier condition에 bool _condition으로 'msg.value == (2 * value)'을 넣어 판정합니다.
    // modifier payable을 사용합니다. 이 payable은 함수 호출 시 Ether를 받을 수 있는 기능을 제공합니다.
    function confirmPurchase()
        public
        inState(State.Created)
        condition(msg.value == (2 * value))
        payable
    {
        // 위에 선언한 이벤트 PurchaseConfirmed를 호출합니다.
        emit PurchaseConfirmed();
        // 위에 public으로 선언한 address 유형의 변수 buyer에 msg.sender를 할당합니다.
        buyer = msg.sender;
        // 위에 public으로 선언한 State 유형의 변수 state에 State의 멤버 Locked를 할당합니다.
        state = State.Locked;
    }

    /// Confirm that you (the buyer) received the item.
    /// This will release the locked ether.
    /// 귀하(구매자)가 물품을 수령했는지 확인하십시오. 잠긴 ether가 방출될 것입니다.
    // 함수 confirmReceived를 선언합니다.
    // modifier onlyBuyer와 inState를 사용합니다.
    // modifier inState에 State의 멤버 Locked를 넣어 판별합니다.
    function confirmReceived()
        public
        onlyBuyer
        inState(State.Locked)
    {
        // 위에 선언한 이벤트 ItemReceived를 호출합니다.
        emit ItemReceived();
        // 먼저 상태를 변경하는 것이 중요합니다. 
        // 그렇지 않으면 아래의 `send`를 사용하여 호출된 계약이 여기에서 다시 호출될 수 있기 때문입니다.
        // 위에 public으로 선언한 State 유형의 변수 state에 State의 멤버 Inactive를 할당합니다.
        state = State.Inactive;

        // 참고 : 실제로 구매자와 판매자가 모두 환불을 차단할 수 있습니다. 철회 패턴을 사용해야 합니다.

        // buyer에게 value를 전송합니다.
        // 내장함수 transfer를 사용합니다. <address>.transfer(uint256 amount) 형식으로 사용합니다.
        // 여기서 단위는 Wei를 사용합니다.
        buyer.transfer(value);
        // seller에게 this.balance를 전송합니다.
        // 내장함수 transfer를 사용합니다. <address>.transfer(uint256 amount) 형식으로 사용합니다.
        // 여기서 단위는 Wei를 사용합니다.
        seller.transfer(this.balance);
    }
}
~~~~