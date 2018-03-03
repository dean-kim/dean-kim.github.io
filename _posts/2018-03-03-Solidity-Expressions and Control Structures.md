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

# Expressions and Control Structures

## Input Parameters and Output Parameters

Javascript와 마찬가지로 함수는 입력으로 매개 변수를 사용할 수 있습니다. Javascript 나 C와는 달리, 임의의 수의 매개 변수를 출력으로 반환할 수도 있습니다.

### Input Parameters

입력 매개 변수는 변수와 동일한 방식으로 선언됩니다. 
예외로, 사용되지 않는 매개 변수는 변수 이름을 생략 할 수 있습니다. 
예를 들어 계약에서 두 개의 정수로 하나의 외부 호출을 수락한다고 가정하면 다음과 같이 작성할 수 있습니다.

~~~~
pragma solidity ^0.4.16;

contract Simple {
    function taker(uint _a, uint _b) public pure {
        // do something with _a and _b.
    }
}
~~~~

### Output Parameters

<tt style="color: #FF0000">`return`</tt> 키워드 다음에 동일한 syntex로 출력 매개 변수를 선언할 수 있습니다. 예를 들어 두 개의 결과를 반환하기를 원한다고 가정합시다. 두 개의 정수의 합과 곱을 구하면 다음과 같이 쓸 수 있습니다.

~~~~
pragma solidity ^0.4.16;

contract Simple {
    function arithmetics(uint _a, uint _b)
        public
        pure
        returns (uint o_sum, uint o_product)
    {
        o_sum = _a + _b;
        o_product = _a * _b;
    }
}
~~~~

출력 매개 변수의 이름은 생략 될 수 있습니다. 
출력 값은 <tt style="color: #FF0000">`return`</tt> 문을 사용하여 지정할 수도 있습니다. <tt style="color: #FF0000">`return`</tt> 문은 여러 값을 반 할 수도 있습니다. 
[Returning Multiple Values](https://solidity.readthedocs.io/en/latest/control-structures.html#multi-return)을 참조하십시오. 반환 매개 변수는 0으로 초기화됩니다. 명시적으로 설정하지 않으면 0으로 유지됩니다.

입력 매개 변수와 출력 매개 변수는 함수 본문의 표현식으로 사용할 수 있습니다. There, they are also usable in the left-hand side of assignment.

## Control Structures

JavaScript의 컨트롤 구조는 대부분 <tt style="color: #FF0000">`switch`</tt> 및 <tt style="color: #FF0000">`goto`</tt>를 제외하고는 Solidity에서 사용할 수 있습니다. 
C 또는 JavaScript에서 알려진 일반적인 의미로 다음과 같은 것들을 사용할 수 있습니다. <tt style="color: #FF0000">`if`</tt>, <tt style="color: #FF0000">`else`</tt>, <tt style="color: #FF0000">while``</tt>, <tt style="color: #FF0000">`do`</tt>, <tt style="color: #FF0000">`for`</tt>, <tt style="color: #FF0000">`break`</tt>, <tt style="color: #FF0000">`continue`</tt>, <tt style="color: #FF0000">`return`</tt>, <tt style="color: #FF0000">`? :`</tt>

조건문에서는 괄호를 생략 할 수 없지만 단일 명령문 본문에서는 중괄호를 생략 할 수 있습니다.

C와 JavaScript 에서처럼 부울이 아닌 유형에서 유형 변환이 없으므로 Solidity에서는 <tt style="color: #FF0000">`if (1) {...}`</tt>이 유효하지 않습니다.

### Returning Multiple Values

함수에 여러 출력 매개 변수가 있는 경우 <tt style="color: #FF0000">`return (v0, v1, ..., vn)`</tt>은 여러 값을 반환 할 수 있습니다. 구성 요소의 수는 출력 매개 변수의 수와 같아야 합니다.

## Function Calls

### Internal Function Calls

다음의 무의미한 예제에서 볼 수 있듯이 현재 계약의 함수를 직접적으로("내부적으로") 호출 할 수도 있고 재귀적으로 호출 할 수도 있습니다.

~~~~
pragma solidity ^0.4.16;

contract C {
    function g(uint a) public pure returns (uint ret) { return f(); }
    function f() internal pure returns (uint ret) { return g(7) + f(); }
}
~~~~

이 함수 호출은 EVM 내부의 단순 점프로 변환됩니다. 
이것은 현재 메모리가 클리어되지 않는다는 것을 의미합니다. 
즉 메모리 참조를 내부적으로 호출되는 함수에 전달하는 것이 매우 효율적입니다. 
동일한 계약의 함만 내부적으로 호출 할 수 있습니다.

### External Function Calls

표현식 <tt style="color: #FF0000">`this.g(8);`</tt> 및 <tt style="color: #FF0000">`c.g(2);`</tt> (여기서 <tt style="color: #FF0000">`c`</tt>는 계약 인스턴스임)도 유효한 함수 호출이지만 이번에는 함수가 점프를 통해 직접 호출하지 않고 메시지 호출을 통해 "외부적으로"호출됩니다. 
실제 계약이 아직 생성되지 않았으므로 <tt style="color: #FF0000">`this`</tt>에 대한 함수 호출은 생성자에서 사용할 수 없습니다.

다른 계약의 함수는 외부적으로 호출되어야 합니다. 외부 호출의 경우 모든 함수 인수를 메모리에 복사해야합니다.

다른 계약의 함수를 호출 할 때 호출 및 가스와 함께 전송된 Wei의 양은 특별한 옵션인 <tt style="color: #FF0000">`.value()`</tt> 및 <tt style="color: #FF0000">`.gas()`</tt>로 각각 지정할 수 있습니다.

~~~~
pragma solidity ^0.4.0;

contract InfoFeed {
    function info() public payable returns (uint ret) { return 42; }
}

contract Consumer {
    InfoFeed feed;
    function setFeed(address addr) public { feed = InfoFeed(addr); }
    function callFeed() public { feed.info.value(10).gas(800)(); }
}
~~~~

<tt style="color: #FF0000">`info`</tt>에 <tt style="color: #FF0000">`payable`</tt> modifier를 사용해야합니다. 그렇지 않으면 `.value()` 옵션을 사용할 수 없기 때문입니다.

<tt style="color: #FF0000">`InfoFeed(addr)`</tt> 표현식은 "주어진 주소의 계약 유형이 <tt style="color: #FF0000">`InfoFeed`</tt>라는 것을 알고" 생성자를 실행하지 않는다는 명시적인 유형 변환을 수행합니다. 
명시적 유형 변환은 매우 주의하여 처리해야 합니다. 유형에 대해 잘 모르는 계약에서 함수를 호출하지 마십시오.

우리는 또한 함수 <tt style="color: #FF0000">`setFeed(InfoFeed _feed) {feed = _feed; }`</tt> 직접 사용할 수 있었습니다. <tt style="color: #FF0000">`feed.info.value(10).gas(800)`</tt>만이(locally) 함수 호출과 함께 보내지는 가스의 값과 양을 설정하고 마지막 괄호만 실제 호출을 수행한다는 것에 주의하십시오.

함수 호출은 호출된 계약이 존재하지 않는 경우 (계정에 코드가 포함되지 않는다는 의미에서) 또는 호출된 계약 자체가 예외를 발생시키거나 가스의 양을 벗어나는 경우 예외를 발생시킵니다.

<b>Warning</b>
<br>다른 계약과의 상호 작용은 잠재적인 위험을 내포하고 있습니다. 
계약의 소스 코드가 미리 알려지지 않은 경우 특히 그렇습니다. 현재의 계약은 요청된 계약에 통제권을 넘겨주며 잠재적으로는 그 무엇을 할 수도 있습니다. 
호출된 계약이 알려진 부모 계약에서 상속받는 경우에도 상속 계약에는 올바른 인터페이스만 있으면 됩니다. 
그러나 계약의 이행은 완전히 임의적일 수 있으므로 위험을 초래할 수 있습니다. 또한 첫 번째 호출이 반환되기 전에 시스템의 다른 계약을 호출하거나 호출 계약에 다시 들어갈 경우를 대비하여 준비하십시오. 
즉, 호출된 계약은 함수를 통해 호출하는 계약의 상태 변수를 변경할 수 있음을 의미합니다. 예를 들어 계약에서 상태 변수를 변경한 후에 외부 함수 호출이 발생하는 방식으로 함수를 작성하면 계약에 재진입 취약점이 발생하지 않도록 할 수 있습니다.
<br><b>Warning End</b>

### Named Calls and Anonymous Function Parameters

함수 호출 인수는 다음 예제에서 볼 수 있듯이 <tt style="color: #FF0000">`{}`</tt>로 묶인 경우 순서에 관계없이 이름으로도 지정할 수 있습니다. 인수 목록은 함수 선언의 매개 변수 목록과 이름이 일치해야하지만 임의의 순서가 될 수 있습니다.

~~~~
pragma solidity ^0.4.0;

contract C {
    function f(uint key, uint value) public {
        // ...
    }

    function g() public {
        // named arguments
        f({value: 2, key: 3});
    }
}
~~~~

### Omitted Function Parameter Names

사용되지 않은 매개 변수의 이름(특히 리턴 매개 변수)은 생략될 수 있습니다. 이러한 매개 변수는 여전히 스택에 있지만 액세스할 수는 없습니다.

~~~~
pragma solidity ^0.4.16;

contract C {
    // omitted name for parameter
    function func(uint k, uint) public pure returns(uint) {
        return k;
    }
}
~~~~

## Creating Contracts via <tt style="color: #FF0000">`new`</tt> 

계약서는 <tt style="color: #FF0000">`new`</tt> 키워드를 사용하여 새 계약서를 작성할 수 있습니다. 생성되는 계약의 전체 코드는 미리 알려져야하므로 재귀 생성 의존 관계가 불가능합니다.

~~~~
pragma solidity ^0.4.0;

contract D {
    uint x;
    function D(uint a) public payable {
        x = a;
    }
}

contract C {
    D d = new D(4); // will be executed as part of C's constructor

    function createD(uint arg) public {
        D newD = new D(arg);
    }

    function createAndEndowD(uint arg, uint amount) public payable {
        // Send ether along with the creation
        D newD = (new D).value(amount)(arg);
    }
}
~~~~

예제에서 볼 수 있듯이 <tt style="color: #FF0000">`.value()`</tt> 옵션을 사용하여 <tt style="color: #FF0000">`D`</tt>의 인스턴스를 생성하면서 Ether을 전달할 수 있지만 가스량을 제한하는 것은 불가능합니다. 
생성이 실패한 경우(스택 부족, 잔액 부족 또는 다른 문제로 인해) 예외가 발생합니다.

## Order of Evaluation of Expressions

표현식의 평가 순서는 지정되지 않습니다(공식적으로 표현식 트리에서 한 노드의 하위 노드가 평가되는 순서는 지정되지 않지만 물론 노드 자체보다 먼저 평가됩니다). 
명령문이 순서대로 실행되고 부울 표현식에 대한 단락이 수행되는 것만 보장됩니다. 자세한 내용은 [연산자 우선 순위](https://solidity.readthedocs.io/en/latest/miscellaneous.html#order)를 참조하십시오.

## Assignment

### Destructuring Assignments and Returning Multiple Values

Solidity는 내부적으로 튜플 유형, 즉 크기가 컴파일 타임에 일정한 잠재적으로 다른 유형의 객체 목록을 허용합니다. 이러한 튜플을 사용하면 동시에 여러 값을 반환하고 여러 변수(또는 일반적으로 LValues)에 동시에 할당할 수 있습니다.

~~~~
pragma solidity ^0.4.16;

contract C {
    uint[] data;

    function f() public pure returns (uint, bool, uint) {
        return (7, true, 2);
    }

    function g() public {
        // Declares and assigns the variables. Specifying the type explicitly is not possible.
        var (x, b, y) = f();
        // Assigns to a pre-existing variable.
        (x, y) = (2, 7);
        // Common trick to swap values -- does not work for non-value storage types.
        (x, y) = (y, x);
        // Components can be left out (also for variable declarations).
        // If the tuple ends in an empty component,
        // the rest of the values are discarded.
        (data.length,) = f(); // Sets the length to 7
        // The same can be done on the left side.
        // If the tuple begins in an empty component, the beginning values are discarded.
        (,data[3]) = f(); // Sets data[3] to 2
        // Components can only be left out at the left-hand-side of assignments, with
        // one exception:
        (x,) = (1,);
        // (1,) is the only way to specify a 1-component tuple, because (1) is
        // equivalent to 1.
    }
}
~~~~

### Complications for Arrays and Structs

Assignment의 의미는 배열 및 structs와 같은 non-value 유형에 대해서는 조금 더 복잡합니다. 
상태 변수에 할당하면 항상 독립 사본이 생성됩니다. 
반면에 지역 변수에 할당하면 기본 유형, 즉 32 바이트에 맞는 정적 유형에 대해서만 독립 사본이 생성됩니다. 
구조체 또는 배열(<tt style="color: #FF0000">`bytes`</tt> 및 <tt style="color: #FF0000">`string`</tt> 포함)이 상태 변수에서 로컬 변수로 지정되면 로컬 변수는 원래 상태 변수에 대한 참조를 보유합니다. 
지역 변수에 대한 두 번째 할당은 상태를 수정하지 않고 참조만 변경합니다. 
지역 변수의 구성원 (또는 요소)에 대한 할당은 상태를 변경합니다.

## Scoping and Declarations

선언된 변수의 초기 기본값은 바이트 표현이 모두 0입니다. 
변수의 "default values"는 유형이 무엇이든간에 일반적인 "zero-state" 입니다. 
예를 들어 <tt style="color: #FF0000">`bool`</tt>의 기본값은 <tt style="color: #FF0000">`false`</tt>입니다. 
<tt style="color: #FF0000">`uint`</tt> 또는 <tt style="color: #FF0000">`int`</tt> 유형의 기본값은 <tt style="color: #FF0000">`0`</tt>입니다. 
정적 크기의 배열 및 <tt style="color: #FF0000">`bytes1`</tt>에서 <tt style="color: #FF0000">`bytes32`</tt>의 경우 각 개별 요소는 해당 유형에 해당하는 기본값으로 초기화됩니다. 
마지막으로, 동적 크기의 배열, 바이트 및 문자열의 경우 기본값은 빈 배열 또는 문자열입니다.

함수내에서 선언된 변수는 선언된 위치에 관계없이 전체 함수의 범위에 포함됩니다(곧 변경 될 예정, 아래 참조). 
이것은 Solidity가 JavaScript에서 scope rules를 상속 받기 때문에 발생합니다. 
이것은 의미론적 블록의 끝까지 선언된 곳에서만 변수가 범위가 지정된 많은 언어와 대조적입니다. 
따라서 다음 코드는 올바르지 않으며 컴파일러에서 <tt style="color: #FF0000">`Identifier already declared`</tt>를 throw합니다.

~~~~
// This will not compile

pragma solidity ^0.4.16;

contract ScopingErrors {
    function scoping() public {
        uint i = 0;

        while (i++ < 1) {
            uint same1 = 0;
        }

        while (i++ < 2) {
            uint same1 = 0;// Illegal, second declaration of same1
        }
    }

    function minimalScoping() public {
        {
            uint same2 = 0;
        }

        {
            uint same2 = 0;// Illegal, second declaration of same2
        }
    }

    function forLoopScoping() public {
        for (uint same3 = 0; same3 < 1; same3++) {
        }

        for (uint same3 = 0; same3 < 1; same3++) {// Illegal, second declaration of same3
        }
    }
}
~~~~

이 외에도 변수가 선언되면 함수의 시작 부분에서 변수가 기본값으로 초기화됩니다. 결과적으로 다음 코드는 잘못 작성되었음에도 불구하고 합법적입니다.

~~~~
pragma solidity ^0.4.0;

contract C {
    function foo() public pure returns (uint) {
        // baz is implicitly initialized as 0
        uint bar = 5;
        if (true) {
            bar += baz;
        } else {
            uint baz = 10;// never executes
        }
        return bar;// returns 5
    }
}
~~~~

### Scoping starting from Version 0.5.0

버전 0.5.0부터 Solidity는 C99 (및 많은 다른 언어)의보다 광범위한 scoping rules로 변경됩니다. 
변수는 선언 직후부터 <tt style="color: #FF0000">`{}`</tt> 블록 끝까지 표시됩니다. 
이 규칙의 예외로 for 루프의 초기화 부분에서 선언된 변수는 for 루프가 끝날 때까지만 볼 수 있습니다.

코드 블록 외부에서 선언된 변수 및 기타 항목 예를 들어 함수, 계약, 사용자 정의 유형 등과 같은 것들은 scoping behaviour를 변경하지 않습니다. 
즉, 선언되기 전에 상태 변수를 사용하고 재귀적으로 함수를 호출 할 수 있습니다.

이러한 규칙은 이미 실험 기능으로 도입되었습니다.

결과적으로 두 변수의 이름은 동일하지만 범위가 서로 다르기 때문에 다음 예는 경고없이 컴파일됩니다. 
0.5.0이 아닌 모드에서는 범위가 동일하며 (<tt style="color: #FF0000">`minimalScoping`</tt> 함수) 컴파일되지 않습니다.

~~~~
pragma solidity ^0.4.0;
pragma experimental "v0.5.0";
contract C {
    function minimalScoping() pure public {
        {
            uint same2 = 0;
        }

        {
            uint same2 = 0;
        }
    }
}
~~~~

C99 범위 지정 규칙의 특별한 예로서, 다음에서 <tt style="color: #FF0000">`x`</tt>에 대한 첫 번째 할당은 실제로 내부 변수가 아닌 외부 변수를 할당한다는 점에 유의하십시오. 
어쨌든 외부 변수가 음영 처리된다는 경고를 받게됩니다.

~~~~
pragma solidity ^0.4.0;
pragma experimental "v0.5.0";
contract C {
    function f() pure public returns (uint) {
        uint x = 1;
        {
            x = 2; // this will assign to the outer variable
            uint x;
        }
        return x; // x has value 2
    }
}
~~~~

## Error handling: Assert, Require, Revert and Exceptions

Solidity는 상태를 되돌리는 예외를 사용하여 오류를 처리합니다. 
이러한 예외는 현재 호출 (및 모든 하위 호출)에서 상태에 대한 모든 변경 사항을 실행 취소하고 호출자에게 오류를 플래그합니다. 
편의 함수 <tt style="color: #FF0000">`assert`</tt>와 <tt style="color: #FF0000">`require`</tt>는 조건을 검사하고 조건이 충족되지 않으면 예외를 throw 하는 데 사용할 수 있습니다. 
<tt style="color: #FF0000">`assert`</tt> 함수는 내부 오류를 테스트하고 불변성을 검사하는 데에만 사용해야 합니다. 
<tt style="color: #FF0000">`require`</tt> 함수는 입력이나 계약 상태 변수와 같은 유효한 조건을 확인하거나 외부 계약에 대한 호출에서 반환 값의 유효성을 검사하는 데 사용해야합니다. 
제대로 사용하면 분석 도구가 계약을 평가하여 실패한 주장에 도달하는 조건과 함수 호출을 식별 할 수 있습니다. 
제대로 작동하는 코드는 실패한 <tt style="color: #FF0000">`assert`</tt>문에 도달해서는 안됩니다. 이런 일이 생기면 계약서에 수정해야하는 버그가 있습니다.

예외를 트리거하는 두 가지 다른 방법이 있습니다. 
<tt style="color: #FF0000">`revert`</tt> 함수를 사용하여 오류를 플래그하고 현재 호출을 되돌릴 수 있습니다. 
앞으로는 <tt style="color: #FF0000">`revert`</tt> 호출에서 오류에 대한 세부 사항을 포함시킬 수도 있습니다. 
<tt style="color: #FF0000">`throw`</tt> 키워드는 <tt style="color: #FF0000">`revert()`</tt>의 대안으로 사용할 수도 있습니다.

<b>Note</b>
<br>버전 0.4.13부터 <tt style="color: #FF0000">`throw`</tt> 키워드는 향후 제공되지 않을 예정이며 향후 단계적으로 제거될 예정입니다.
<br><b>Note End</b>

하위 호출에서 예외가 발생하면 자동으로 예외가 다시 발생합니다(예외가 다시 발생합니다). 
이 규칙의 예외는 <tt style="color: #FF0000">`send`</tt>와 low-level 함수 <tt style="color: #FF0000">`call`</tt>, <tt style="color: #FF0000">`delegatecall`</tt> 및 <tt style="color: #FF0000">`callcode`</tt>입니다. 
예외가 발생하는 경우 "bubbling up"대신 <tt style="color: #FF0000">`false`</tt>을 반환합니다.

<b>Warning</b>
<br>low-level의 <tt style="color: #FF0000">`call`</tt>, <tt style="color: #FF0000">`delegatecall`</tt> 및 <tt style="color: #FF0000">`callcode`</tt>는 EVM 설계의 일부로 호출된 계정이 존재하지 않으면 성공을 반환합니다. 필요하면 호출하기 전에 존재 여부를 확인해야합니다.
<br><b>Warning End</b>

예외를 잡아내는 것은 아직 가능하지 않습니다.

다음 예제에서는 입력 조건을 쉽게 확인하기 위해 <tt style="color: #FF0000">`require`</tt>를 사용하는 방법과 내부 오류 검사에 <tt style="color: #FF0000">`assert`</tt>를 사용하는 방법을 볼 수 있습니다.

~~~~
pragma solidity ^0.4.0;

contract Sharer {
    function sendHalf(address addr) public payable returns (uint balance) {
        require(msg.value % 2 == 0); // Only allow even numbers
        uint balanceBeforeTransfer = this.balance;
        addr.transfer(msg.value / 2);
        // Since transfer throws an exception on failure and
        // cannot call back here, there should be no way for us to
        // still have half of the money.
        assert(this.balance == balanceBeforeTransfer - msg.value / 2);
        return this.balance;
    }
}
~~~~

<tt style="color: #FF0000">`assert`</tt>-style 예외는 다음과 같은 상황에서 생성됩니다.

1. 너무 크거나 음수인 인덱스로 배열에 액세스하는 경우(예로 <tt style="color: #FF0000">`x[i]`</tt>, <tt style="color: #FF0000">`i >= x.length`</tt> 또는 <tt style="color: #FF0000">`i < 0`</tt>).
2. 너무 크거나 음수인 인덱스에서 고정 길이 <tt style="color: #FF0000">`bytesN`</tt>에 액세스하는 경우.
3. 0으로 나눗셈 또는 모듈로 (예 : <tt style="color: #FF0000">`5 / 0`</tt> 또는 <tt style="color: #FF0000">`23 % 0`</tt>).
4. 음수로 이동하는 경우.
5. 너무 크거나 음수인 값을 열거형으로 변환하는 경우
6. 내부 함수 유형의 0으로 초기화된 변수를 호출하는 경우.
7. false로 평가되는 인수로 <tt style="color: #FF0000">`assert`</tt>를 호출하는 경우

다음과 같은 경우 <tt style="color: #FF0000">`require`</tt>-style 예외가 생성됩니다.

1. <tt style="color: #FF0000">`throw`</tt>를 호출.
2. <tt style="color: #FF0000">`false`</tt>로 평가되는 인수와 함께 <tt style="color: #FF0000">`require`</tt>를 호출합니다.
3. 메시지 호출을 통해 함수를 호출했지만 low level의 operation <tt style="color: #FF0000">`call`</tt>, <tt style="color: #FF0000">`send`</tt>, <tt style="color: #FF0000">`delegatecall`</tt> 또는 <tt style="color: #FF0000">`callcode`</tt>가 사용되는 경우를 제외하고는 메시지 호출을 통해 함수를 호출했지만 정상적으로 완료되지 않습니다(즉, 가스가 없거나 일치하는 함수가 없거나 예외 자체가 발생 함). low level operation는 예외를 throw하지 않지만 <tt style="color: #FF0000">`false`</tt>를 반환하여 실패를 나타냅니다.
4. <tt style="color: #FF0000">`new`</tt> 키워드를 사용하여 계약을 생성했지만 계약 생성이 제대로 완료되지 않은 경우(위의 "제대로 완료되지 않음"의 정의 참조)
5. 코드가 없는 계약을 대상으로하는 외부 함수 호출을 수행하는 경우.
6. 귀하의 계약이 <tt style="color: #FF0000">`payable`</tt> modifier (생성자 및 대체 기능 포함)없이 공개 기능을 통해 Ether를 수신하는 경우.
7. 귀하의 계약이 공용 getter 기능을 통해 Ether를 수신하는 경우.
8. <tt style="color: #FF0000">`.transfer()`</tt>가 실패하면.

내부적으로 Solidity는 <tt style="color: #FF0000">`require`</tt>-style 예외에 대해 되돌리기 연산 (명령어 <tt style="color: #FF0000">`0xfd`</tt>)을 수행하고 <tt style="color: #FF0000">`assert`</tt>-style 예외를 throw하기 위해 잘못된 연산(명령어 <tt style="color: #FF0000">`0xfe`</tt>)을 실행합니다. 
두 경우 모두 EVM이 상태에 대한 모든 변경 사항을 되돌릴 수 있습니다. 
되돌리기 위한 이유는 예상되는 효과가 발생하지 않았기 때문에 실행을 계속하기 위한 안전한 방법이 없다는 것입니다. 
우리가 트랜잭션의 원자성을 유지하기를 원하기 때문에 가장 안전한 방법은 모든 변경 사항을 되돌리고 전체 트랜잭션 (또는 적어도 호출)을 적용하지 않고 수행하는 것입니다. 
<tt style="color: #FF0000">`assert`</tt>-style 예외는 호출에 사용할 수 있는 모든 가스를 소비하지만 <tt style="color: #FF0000">`requie`</tt>-style 예외는 메트로 폴리스 릴리스부터 시작하는 가스를 소비하지 않습니다.

<tt style="color: #FF0000">``</tt>