---
layout: post
title:  "Solidity Miscellaneous"
date:   2018-03-12 20:43:59
author: Dean Kim
categories: BlockChain
tags:	BlockChain Solidity Miscellaneous
cover:  "/assets/instacode.png"
---

블록체인을 공부하며 Smart Contract 작성에 사용되는 Solidity 공식문서를 번역한 글입니다.
제가 이해한 수준으로 기술된 내용도 있어 원본과 내용이 조금 다를 수도 있습니다.
혹시 제가 잘못 이해한 내용이 있다면 말씀해주세요 :)

# Miscellaneous

## Layout of State Variables in Storage

정적으로 크기가 지정된 변수(매핑 및 동적 크기의 배열 유형을 제외한 모든 항목)는 위치 <tt style="color: #FF0000">`0`</tt>부터 시작하여 저장소에 연속적으로 배치됩니다. 
32바이트 미만인 여러 항목은 가능한 경우 다음 규칙에 따라 단일 저장소 슬롯에 압축됩니다.

* 저장 슬롯의 첫번째 항목은 lower-order로 정렬되어 저장됩니다.
* 기본적인 형태는 그것들을 저장하는데 필요한 바이트들만 사용한다.
* 기본 유형이 저장소 슬롯의 나머지 부분에 맞지 않으면 다음 저장소 슬롯으로 이동합니다.
* Struct 및 배열 데이터는 항상 새 슬롯을 시작하고 전체 슬롯을 차지합니다(하지만 struct 또는 배열 내부의 항목은 이러한 규칙에 따라 꽉 채워집니다).

<b>Warning</b>
<br>32바이트보다 작은 요소를 사용하면 계약서의 가스 사용량이 더 높아질 수 있습니다. 
이는 EVM이 한 번에 32바이트로 작동하기 때문입니다. 
따라서 요소가 그보다 작으면 EVM은 요소 크기를 32 바이트에서 원하는 크기로 줄이기 위해 더 많은 작업을 사용해야합니다.

컴파일러가 여러 요소를 하나의 저장소 슬롯에 압축하여 여러 읽기 또는 쓰기를 단일 작업으로 결합하기 때문에 저장소 값을 처리하는 경우 축소된 인수를 사용하는 것이 유용합니다. 
함수 인수 또는 메모리 값을 처리할 때 컴파일러에서 이러한 값을 압축하지 않으므로 고유한 이점은 없습니다.

마지막으로, EVM에서 이를 최적화할 수 있도록 저장소 변수와 struct 멤버를 순서대로 배열하여 단단히 묶으십시오. 
예를 들어 <tt style="color: #FF0000">`uint128`</tt>, <tt style="color: #FF0000">`uint256`</tt>, <tt style="color: #FF0000">`uint128`</tt> 대신 <tt style="color: #FF0000">`uint128`</tt>, <tt style="color: #FF0000">`uint128`</tt>, <tt style="color: #FF0000">uint256``</tt> 순서로 저장소 변수를 선언하는 경우 전자는 두 개의 저장소 슬롯을 차지하므로 후자는 세 개를 차지합니다.
<br><b>Warning End</b>

structs와 배열의 요소는 마치 명시적으로 주어진 것처럼 서로 저장됩니다.

예측할 수 없는 크기로 인해 매핑 및 동적 크기 배열 유형은 Keccak-256 해시 계산을 사용하여 값 또는 배열 데이터의 시작 위치를 찾습니다. 이러한 시작 위치는 항상 전체 스택 슬롯입니다.

매핑 또는 동적 배열 자체는 위의 규칙에 따라(또는 맵핑이나 배열 배열에 대한 매핑에 대해 이 규칙을 반복적으로 적용하여) 일부 위치 <tt style="color: #FF0000">`p`</tt>에서 저장소의(채워지지 않은) 슬롯을 차지합니다. 
동적 배열의 경우 이 슬롯은 배열의 요소 수를 저장합니다(바이트 배열과 문자열은 예외입니다, 아래 참조). 
매핑의 경우, 슬롯은 사용되지 않습니다(하지만 서로 동일한 매핑이 서로 다른 해시 분포를 사용하기 위해 필요합니다). 
배열 데이터는 <tt style="color: #FF0000">`keccak256(p)`</tt>에 있고, 매핑 키 <tt style="color: #FF0000">`k`</tt>에 해당하는 값은 <tt style="color: #FF0000">`keccak256(k. p)`</tt>에 있으며 <tt style="color: #FF0000">`.`</tt>으로 연결되어있습니다. 
값이 다시 기본이 아닌 유형인 경우 <tt style="color: #FF0000">`keccak256(k. p)`</tt>의 오프셋을 추가하여 위치를 찾습니다.

<tt style="color: #FF0000">`bytes`</tt> 및 <tt style="color: #FF0000">`string`</tt>은 길이가 짧은 동일한 슬롯에 데이터를 저장합니다. 
특히 : 데이터의 길이가 최대 <tt style="color: #FF0000">`31`</tt>바이트 인 경우 상위 바이트에 저장되고(왼쪽 정렬) 최하위 바이트에 <tt style="color: #FF0000">`length * 2`</tt>가 저장됩니다. 
길이가 더 길면 기본 슬롯에 <tt style="color: #FF0000">`length * 2 + 1`</tt>이고 데이터는 <tt style="color: #FF0000">`keccak256(slot)`</tt>에 평소대로 저장됩니다.

그래서 다음 계약 스니펫 :
~~~~
pragma solidity ^0.4.0;

contract C {
  struct s { uint a; uint b; }
  uint x;
  mapping(uint => mapping(uint => s)) data;
}
~~~~

<tt style="color: #FF0000">`data[4][9].b`</tt>의 위치는 <tt style="color: #FF0000">`keccak256(uint256(9) . keccak256(uint256(4) . uint256(1))) + 1`</tt>입니다.

## Layout in Memory

Solidity는 3개의 256비트 슬롯을 예약합니다.

* 0 - 64 : 해싱 방법을 위한 스크래치 공간
* 64 - 96 : 현재 할당된 메모리 크기(일명 free memory pointer)

스크래치 공간은 명령문(즉, 인라인 어셈블리내) 사이에서 사용될 수 있습니다.

Solidity는 항상 새로운 개체를 free memory pointer에 놓고 메모리가 해제되지 않습니다(나중에 변경될 수 있습니다).

<b>Warning</b>
<br>Solidity에는 64바이트보다 큰 임시 메모리 영역이 필요하기 때문에 스크래치 공간에 맞지 않는 연산이 있습니다. 
사용 가능한 메모리가 가리키는 위치에 배치되지만 수명이 짧으면 포인터가 업데이트되지 않습니다. 
메모리가 0일 수도 있고 0가 아닐 수도 있습니다. 이 때문에 free memory가 0이 될 것으로 예상해서는 안됩니다.
<br><b>Warning End</b>

## Layout of Call Data

Solidity 계약이 배포되고 계정에서 호출될 때 입력 데이터는 [ABI specification](http://solidity.readthedocs.io/en/latest/abi-spec.html#abi) 형식으로 간주됩니다. 
ABI specification에서는 인수가 32바이트의 배수로 채워져 있어야합니다. 내부 함수 호출은 다른 규칙을 사용합니다.

## Internals - Cleaning Up Variables

값이 256비트보다 짧으면 나머지 비트를 정리해야하는 경우도 있습니다. 
Solidity 컴파일러는 남은 비트에서 잠재적인 garbage의 영향을 받을 수 있는 작업전에 남은 비트를 정리하도록 설계되었습니다. 
예를 들어 메모리에 값을 쓰기 전에 메모리 내용을 해시 계산에 사용하거나 메시지 호출의 데이터로 보낼 수 있기 때문에 나머지 비트는 지워야합니다. 
마찬가지로 저장소에 값을 저장하기 전에 나머지 비트를 정리해야합니다. 그렇지 않으면 garbled(깨진) 값을 볼 수 있기 때문입니다.

다른 한편으로, 우리는 바로 다음 작업이 영향을 받지 않으면 비트를 청소하지 않습니다. 
예를 들어, 0이 아닌 값은 <tt style="color: #FF0000">`JUMPI`</tt> 명령어에 의해 <tt style="color: #FF0000">`true`</tt>로 간주되므로 <tt style="color: #FF0000">`JUMPI`</tt>의 조건으로 사용되기 전에 부울 값을 정리하지 않습니다.

Solidity 컴파일러는 위의 설계 원칙외에도 스택에 로드될 때 입력 데이터를 정리합니다.

다른 유형에는 유효하지 않은 값을 정리하기 위한 다른 규칙이 있습니다:

|        Type       |    Valid Values    |                        Invalid Values Mean                        |
|:-----------------:|:------------------:|:-----------------------------------------------------------------:|
| enum of n members | 0 until n - 1      | exception                                                         |
| bool              | 0 or 1             | 1                                                                 |
| signed integers   | sign-extended word | currently silently wraps; in the future exceptions will be thrown |
| unsigned integers | higher bits zeroed | currently silently wraps; in the future exceptions will be thrown |

## Internals - The Optimizer

Solidity Optimizer는 어셈블리에서 작동하므로 다른 언어에서도 사용할 수 있으며 다른 언어에서도 사용할 수 있습니다. 
<tt style="color: #FF0000">`JUMPs`</tt> 및 <tt style="color: #FF0000">`JUMPDESTs`</tt>에서 명령 시퀀스를 기본 블록으로 분할합니다. 
이 블록들 내에서 명령들이 분석되고 스택, 메모리 또는 저장 장치에 대한 모든 변경 사항은 명령어와 기본적으로 다른 표현식에 대한 포인터인 인수 목록으로 구성된 표현식으로 기록됩니다. 
주요 아이디어는 항상 모든 입력에 대해 항상 같은 표현식을 찾아서 표현식 클래스로 결합하는 것입니다. 
옵티마이저는 먼저 이미 알고있는 표현식 목록에서 각각의 새 표현식을 찾습니다. 
이것이 작동하지 않으면 expression은 <tt style="color: #FF0000">`constant + constant = sum_of_constants`</tt> 또는 <tt style="color: #FF0000">`X * 1 = X`</tt>와 같은 규칙에 따라 단순화됩니다. 
반복적으로 수행되기 때문에, 두번째 요소가 항상 하나로 평가될 것을 알고 있는 더 복잡한 표현일 경우에도 두번째 규칙을 적용할 수 있습니다. 
스토리지 및 메모리 위치를 수정한 경우에는 다르지 않은 스토리지 및 메모리 위치에 대한 지식이 삭제되어야 합니다. 
우리가 처음으로 위치 x에 그리고 그 다음 위치 y에 쓰고 양쪽 모두가 입력 변수라면 두번째는 첫번째 값을 덮어 쓸 수 있어서 우리는 y에 쓴 후에 x에 저장된 것이 무엇인지 알 수 없습니다. 
반면 x - y 표현식의 단순화가 0이 아닌 상수로 평가되면 x에 저장된 내용에 대한 지식을 유지할 수 있음을 알 수 있습니다.

이 프로세스가 끝나면 마지막에 스택에 있어야 하는 표현식을 확인하고 메모리 및 스토리지에 대한 수정 목록을 제공합니다. 
이 정보는 기본 블록과 함께 저장되며 이를 연결하는 데 사용됩니다. 
또한 스택, 스토리지 및 메모리 구성에 대한 지식은 다음 블록으로 전달됩니다. 
모든 <tt style="color: #FF0000">`JUMP`</tt> 및 <tt style="color: #FF0000">`JUMPI`</tt> 명령의 대상을 알면 프로그램의 완전한 제어 흐름 그래프를 작성할 수 있습니다. 
우리가 알지 못하는 단 하나의 목표만 있다면(원칙적으로 점프 목표는 입력으로 계산될 수 있습니다), 알려지지 않은 <tt style="color: #FF0000">`JUMP`</tt>의 목표일 수 있으므로 블록의 입력 상태에 대한 모든 지식을 지워야합니다. 
조건이 상수로 평가되는 <tt style="color: #FF0000">`JUMPI`</tt>가 발견되면 무조건 점프로 변환됩니다.

마지막 단계로 각 블록의 코드가 완전히 다시 생성됩니다. 
블록 끝에 있는 스택의 표현식에서 종속성 그래프가 작성되며 이 그래프에 포함되지 않은 모든 연산은 기본적으로 삭제됩니다. 
이제 메모리와 스토리지에 수정 사항을 원래 코드의 순서대로 적용하고(필요하지 않은 수정 사항을 삭제하는) 마지막으로 올바른 위치에 스택에 필요한 모든 값을 생성하는 코드입니다.

이 단계는 각 기본 블록에 적용되며 새로 생성된 코드는 크기가 작은 경우 교체로 사용됩니다. 
기본 블록이 <tt style="color: #FF0000">`JUMPI`</tt>에서 분리되고 분석 중에 조건이 상수로 평가되면 상수 값에 따라 <tt style="color: #FF0000">`JUMPI`</tt>가 대체되므로 다음과 같은 코드가 작성됩니다.

~~~~
var x = 7;
data[7] = 9;
if (data[x] != x + 2)
  return 2;
else
  return 1;
~~~~

에서 컴파일할 수 있는 코드로 단순화

~~~~
data[7] = 9;
return 1;
~~~~

심지어 처음에 지시 사항에 점프가 포함되어 있었음에도 불구하고

## Source Mappings

AST 출력의 일부로 컴파일러는 AST의 해당 노드가 나타내는 소스 코드의 범위를 제공합니다. 
이 도구는 AST를 기반으로 오류를 보고하는 정적 분석 도구와 로컬 변수 및 해당 용도를 강조 표시하는 디버깅 도구에 이르기까지 다양한 목적으로 사용할 수 있습니다.

또한 컴파일러는 바이트 코드에서 명령어를 생성한 소스 코드의 범위까지 매핑을 생성할 수 있습니다. 
이는 바이트 코드 레벨에서 작동하는 정적 분석 도구와 디버거 내부의 소스 코드에서 현재 위치를 표시하거나 중단점 처리를 위해 중요합니다.

두 가지 종류의 소스 매핑은 모두 정수 식별자를 사용하여 소스 파일을 참조합니다. 
이것들은 일반적으로 <tt style="color: #FF0000">`"sourceList"`</tt>라고 불리는 소스 파일의 목록에 결합된 json과 json / npm 컴파일러의 출력의 일부인 일반적인 배열 인덱스입니다.

<b>Note</b>
<br>특정 소스 파일과 관련이 없는 명령어의 경우 소스 매핑은 정수 식별자 <tt style="color: #FF0000">`-1`</tt>을 할당합니다. 컴파일러에서 생성한 인라인 어셈블리 문에서 발생하는 바이트 코드 섹션에서 이 문제가 발생할 수 있습니다.
<br><b>Note End</b>

AST내부의 소스 매핑은 다음 표기법을 사용합니다.

<tt style="color: #FF0000">`s:l:f`</tt>

여기서 <tt style="color: #FF0000">`s`</tt>는 소스 파일의 범위 시작 부분에 대한 바이트 오프셋이고, <tt style="color: #FF0000">`l`</tt>은 소스 범위의 바이트 길이이며, <tt style="color: #FF0000">`f`</tt>는 위에서 언급한 소스 색인입니다.

바이트 코드에 대한 소스 매핑의 인코딩은 더 복잡합니다. 이것은 <tt style="color: #FF0000">`:`</tt>로 구분된 <tt style="color: #FF0000">`s:l:f:j`</tt>의 목록입니다. 
이러한 각 요소는 명령어에 해당합니다. 
즉, 바이트 오프셋을 사용할 수 없지만 명령어 오프셋을 사용해야합니다(푸시 명령어가 1바이트보다 긴 경우). 
필드 <tt style="color: #FF0000">`s`</tt>, <tt style="color: #FF0000">`l`</tt> 및 <tt style="color: #FF0000">`f`</tt>는 상기와 동일하고, <tt style="color: #FF0000">`f`</tt>j는 <tt style="color: #FF0000">`i`</tt>, <tt style="color: #FF0000">`o`</tt> 또는 <tt style="color: #FF0000">`-`</tt> 점프 명령어가 함수로 들어가거나, 함수로부터 복귀하는지, 또는 예를 들어 루프 함수의 일부로서 정규 점프인지 여부를 나타낼 수 있습니다.

특히 바이트 코드에 대한 이러한 소스 매핑을 압축하기 위해 다음 규칙이 사용됩니다.

* 필드가 비어 있으면 앞의 요소 값이 사용됩니다.
* <tt style="color: #FF0000">`:`</tt>가 누락되면 다음 필드는 모두 비어있는 것으로 간주됩니다.

즉, 다음 소스 매핑은 동일한 정보를 나타냅니다.

<tt style="color: #FF0000">`1:2:1;1:9:1;2:1:2;2:1:2;2:1:2`</tt>

<tt style="color: #FF0000">`1:2:1;:9;2:1:2;;`</tt>

## Tips and Tricks

* 배열의 <tt style="color: #FF0000">`delete`</tt>를 사용하여 모든 요소를 삭제합니다.
* struct 요소에 더 짧은 유형을 사용하고 짧은 유형이 함께 그룹화되도록 정렬하십시오. 
이는 여러 개의 <tt style="color: #FF0000">`SSTORE`</tt> 작업을 하나의 <tt style="color: #FF0000">`SSTORE`</tt>로 결합 할 수 있으므로 가스 비용을 절감할 수 있습니다(<tt style="color: #FF0000">`SSTORE`</tt> 비용은 5000 또는 20000입니다. 따라서 최적화를 원할 것입니다). 
확인하려면 가스 가격 견적 도구 (옵티 마이저 사용)를 사용하십시오!
* 상태 변수를 public으로 만드십시오 - 컴파일러는 자동으로 [getters](http://solidity.readthedocs.io/en/latest/contracts.html#visibility-and-getters)를 생성합니다.
* 함수의 시작 부분에 입력 조건이나 상태를 많이 확인하면 [Function Modifiers](http://solidity.readthedocs.io/en/latest/contracts.html#modifiers)를 사용해보십시오.
* 계약서에 <tt style="color: #FF0000">`send`</tt>라는 함수가 있지만 내장 send 함수를 사용하려면 <tt style="color: #FF0000">`address(contractVariable).send(amount)`</tt>를 사용하십시오.
* 단일 할당으로 스토리지 struct 초기화 : <tt style="color: #FF0000">`x = MyStruct ({a : 1, b : 2});`</tt>

<b>Note</b>
<br>스토리지 struct에 단단히 묶인 속성이 있으면 별개의 할당으로 초기화하십시오. <tt style="color: #FF0000">`x.a = 1; x.b = 2;`</tt> 이러한 방식으로 옵티마이저가 한 번에 스토리지를 업데이트하는 것이 더 쉬울 것이므로 할당 비용이 낮아집니다.
<br><b>Note End</b>

## Cheatsheet

### Order of Precedence of Operators

다음은 평가 순서대로 나열된 연산자의 우선 순위입니다.

| Precedence |             Description             |           Operator                                                                                                                                                                                                                                                                                                                                                                                                                    |
|:----------:|:-----------------------------------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| 1          | Postfix increment and decrement     | <tt style="color: #FF0000">`++`</tt>, <tt style="color: #FF0000">`--`</tt>                                                                                                                                                                                                                                                                                                                                                            |
|            | New expression                      | <tt style="color: #FF0000">`new <typename>`</tt>                                                                                                                                                                                                                                                                                                                                                                                      |
|            | Array subscripting                  | <tt style="color: #FF0000">`<array>[<index>]`</tt>                                                                                                                                                                                                                                                                                                                                                                                    |
|            | Member access                       | <tt style="color: #FF0000">`<object>.<member>`</tt>                                                                                                                                                                                                                                                                                                                                                                                   |
|            | Function-like call                  | <tt style="color: #FF0000">`<func>(<args...>)`</tt>                                                                                                                                                                                                                                                                                                                                                                                   |
|            | Parentheses                         | <tt style="color: #FF0000">`(<statement>)`</tt>                                                                                                                                                                                                                                                                                                                                                                                       |
| 2          | Prefix increment and decrement      | <tt style="color: #FF0000">`++`</tt>, <tt style="color: #FF0000">`--`</tt>                                                                                                                                                                                                                                                                                                                                                            |
|            | Unary plus and minus                | <tt style="color: #FF0000">`+`</tt>, <tt style="color: #FF0000">`-`</tt>                                                                                                                                                                                                                                                                                                                                                              |
|            | Unary operations                    | <tt style="color: #FF0000">`delete`</tt>                                                                                                                                                                                                                                                                                                                                                                                              |
|            | Logical NOT                         | <tt style="color: #FF0000">`!`</tt>                                                                                                                                                                                                                                                                                                                                                                                                   |
|            | Bitwise NOT                         | <tt style="color: #FF0000">`~`</tt>                                                                                                                                                                                                                                                                                                                                                                                                   |
| 3          | Exponentiation                      | <tt style="color: #FF0000">`**`</tt>                                                                                                                                                                                                                                                                                                                                                                                                  |
| 4          | Multiplication, division and modulo | <tt style="color: #FF0000">`*`</tt>, <tt style="color: #FF0000">`/`</tt>, <tt style="color: #FF0000">`%`</tt>                                                                                                                                                                                                                                                                                                                         |
| 5          | Addition and subtraction            | <tt style="color: #FF0000">`+`</tt>, <tt style="color: #FF0000">`-`</tt>                                                                                                                                                                                                                                                                                                                                                              |
| 6          | Bitwise shift operators             | <tt style="color: #FF0000">`<<`</tt>, <tt style="color: #FF0000">`>>`</tt>                                                                                                                                                                                                                                                                                                                                                            |
| 7          | Bitwise AND                         | <tt style="color: #FF0000">`&`</tt>                                                                                                                                                                                                                                                                                                                                                                                                   |
| 8          | Bitwise XOR                         | <tt style="color: #FF0000">`^`</tt>                                                                                                                                                                                                                                                                                                                                                                                                   |
| 9          | Bitwise OR                          | <tt style="color: #FF0000">&#124;</tt>                                                                                                                                                                                                                                                                                                                                                                                                |
| 10         | Inequality operators                | <tt style="color: #FF0000">`<`</tt>, <tt style="color: #FF0000">`>`</tt>, <tt style="color: #FF0000">`<=`</tt>, <tt style="color: #FF0000">`>=`</tt>                                                                                                                                                                                                                                                                                  |
| 11         | Equality operators                  | <tt style="color: #FF0000">`==`</tt>, <tt style="color: #FF0000">`!=`</tt>                                                                                                                                                                                                                                                                                                                                                            |
| 12         | Logical AND                         | <tt style="color: #FF0000">`&&`</tt>                                                                                                                                                                                                                                                                                                                                                                                                  |
| 13         | Logical OR                          | <tt style="color: #FF0000">&#124;&#124;</tt>                                                                                                                                                                                                                                                                                                                                                                                          |
| 14         | Ternary operator                    | <tt style="color: #FF0000">`<conditional> ? <if-true> : <if-false>`</tt>                                                                                                                                                                                                                                                                                                                                                              |
| 15         | Assignment operators                | <tt style="color: #FF0000">`=`</tt>, <tt style="color: #FF0000">&#124;=</tt>, <tt style="color: #FF0000">`^=`</tt>, <tt style="color: #FF0000">`&=`</tt>, <tt style="color: #FF0000">`<<=`</tt>, <tt style="color: #FF0000">`>>=`</tt>, <tt style="color: #FF0000">`+=`</tt>, <tt style="color: #FF0000">`-=`</tt>, <tt style="color: #FF0000">`*=`</tt>, <tt style="color: #FF0000">`/=`</tt>, <tt style="color: #FF0000">`%=`</tt>  |
| 16         | Comma operator                      | <tt style="color: #FF0000">`,`</tt>                                                                                                                                                                                                                                                                                                                                                                                                   |
    
### Global Variables

* <tt style="color: #FF0000">`block.blockhash(uint blockNumber) returns (bytes32)`</tt> : 지정된 블록의 해시 - 256개의 가장 최근 블록에만 작동
* <tt style="color: #FF0000">`block.coinbase`</tt>(<tt style="color: #FF0000">`address`</tt>) : 현재 블록 채굴자 주소
* <tt style="color: #FF0000">`block.difficulty`</tt>(<tt style="color: #FF0000">`uint`</tt>) : 현재 블록 난이도
* <tt style="color: #FF0000">`block.gaslimit`</tt>(<tt style="color: #FF0000">`uint`</tt>) : 현재 블록 gaslimit
* <tt style="color: #FF0000">`block.number`</tt>(<tt style="color: #FF0000">`uint`</tt>) : 현재 블록 number
* <tt style="color: #FF0000">`block.timestamp`</tt>(<tt style="color: #FF0000">`uint`</tt>) : 현재 블록 timestamp
* <tt style="color: #FF0000">`gasleft() returns (uint256)`</tt> : 남아있는 gas
* <tt style="color: #FF0000">`msg.data`</tt>(<tt style="color: #FF0000">`bytes`</tt>) : complete calldata
* <tt style="color: #FF0000">`msg.gas`</tt>(<tt style="color: #FF0000">`uint`</tt>) : 남아있는 gas - 버전 0.4.21에서 사용되지 않으며 <tt style="color: #FF0000">`gasleft()`</tt>로 대체됩니다.
* <tt style="color: #FF0000">`msg.sender`</tt>(<tt style="color: #FF0000">`address`</tt>) : sender of the message (current call)
* <tt style="color: #FF0000">`msg.value`</tt>(<tt style="color: #FF0000">`uint`</tt>) : 메세지와 함께 보낸 wei 수
* <tt style="color: #FF0000">`now`</tt>(<tt style="color: #FF0000">`uint`</tt>) : 현재 블록 타임 스탬프 (<tt style="color: #FF0000">`block.timestamp`</tt>의 별칭)
* <tt style="color: #FF0000">`tx.gasprice`</tt>(<tt style="color: #FF0000">`uint`</tt>) : 트랜잭션 비용(gas)
* <tt style="color: #FF0000">`tx.origin`</tt>(<tt style="color: #FF0000">`address`</tt>) : sender of the transaction (full call chain)
* <tt style="color: #FF0000">`assert(bool condition)`</tt> : 조건이 <tt style="color: #FF0000">`false`</tt>이면 실행을 중단하고 상태 변경을 되돌립니다(내부 오류용으로 사용).
* <tt style="color: #FF0000">`require(bool condition)`</tt> : 조건이 <tt style="color: #FF0000">`false`</tt>이면 실행을 중단하고 상태 변경을 되돌립니다(잘못된 구성의 입력 또는 외부 구성 요소의 오류 사용)
* <tt style="color: #FF0000">`revert()`</tt> : 실행을 중단하고 상태 변경을 되돌리기
* <tt style="color: #FF0000">`keccak256(...) returns (bytes32)`</tt> : [(tightly packed) arguments](http://solidity.readthedocs.io/en/latest/abi-spec.html#abi-packed-mode)의 Ethereum-SHA-3 (Keccak-256) 해시를 계산
* <tt style="color: #FF0000">`sha3(...) returns (bytes32)`</tt> : <tt style="color: #FF0000">`keccak256`</tt>의 별칭
* <tt style="color: #FF0000">`sha256(...) returns (bytes32)`</tt> : [(tightly packed) arguments](http://solidity.readthedocs.io/en/latest/abi-spec.html#abi-packed-mode)의 SHA-256 해시를 계산
* <tt style="color: #FF0000">`ripemd160(...) returns (bytes20)`</tt> : [(tightly packed) arguments](http://solidity.readthedocs.io/en/latest/abi-spec.html#abi-packed-mode)의 RIPEMD-160 해시를 계산
* <tt style="color: #FF0000">`ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)`</tt> : 타원 곡선 시그니쳐로부터 공개 키와 연관된 주소를 복구하고, 오류시 0을 리턴한다.
* <tt style="color: #FF0000">`addmod(uint x, uint y, uint k) returns (uint)`</tt> : <tt style="color: #FF0000">`(x + y) % k`</tt> 여기서 임의의 정밀도로 덧셈을 수행하고 <tt style="color: #FF0000">`2**256`</tt>으로 감싸지 않습니다. 버전 0.5.0부터 <tt style="color: #FF0000">`k! = 0`</tt>임을 주장하십시오.
* <tt style="color: #FF0000">`this`</tt>(current contract’s type) : 명시적으로 <tt style="color: #FF0000">`address`</tt> 변환이 가능한 현재 계약
* <tt style="color: #FF0000">`super`</tt> : 상속 계층 구조에서 한 수준 높은 계약
* <tt style="color: #FF0000">`selfdestruct(address recipient)`</tt> : 현재의 계약을 파기하고, 주어진 주소로 자금을 보냅니다.
* <tt style="color: #FF0000">`suicide(address recipient)`</tt> : <tt style="color: #FF0000">`selfdestruct`</tt>의 별칭
* <tt style="color: #FF0000">`<address>.balance`</tt>(<tt style="color: #FF0000">`uint256`</tt>) : balance of the [Address](http://solidity.readthedocs.io/en/latest/types.html#address) in Wei
* <tt style="color: #FF0000">`<address>.send(uint256 amount) returns (bool)`</tt> : 주어진 양의 Wei를 [Address](http://solidity.readthedocs.io/en/latest/types.html#address)에 보내고, 실패하면 <tt style="color: #FF0000">`false`</tt>를 반환한다.
* <tt style="color: #FF0000">`<address>.transfer(uint256 amount)`</tt> : [Address](http://solidity.readthedocs.io/en/latest/types.html#address)에 주어진 양의 Wei를 보내고, 실패를 던집니다.

### Function Visibility Specifiers

~~~~
function myFunction() <visibility specifier> returns (bool) {
    return true;
}
~~~~

* <tt style="color: #FF0000">`public`</tt>: 외부 및 내부에서 볼 수 있음(저장/상태 변수에 대한 [getter 함수](http://solidity.readthedocs.io/en/latest/contracts.html#getter-functions) 생성)
* <tt style="color: #FF0000">`private`</tt>: 현재 계약에만 표시됩니다.
* <tt style="color: #FF0000">`external`</tt>: 외부에서만 볼 수 있습니다(함수에만 해당) - 즉, 메시지 호출만 가능합니다(<tt style="color: #FF0000">`this.func`</tt>를 통해)
* <tt style="color: #FF0000">`internal`</tt>: 내부적으로만 볼 수 있다.

### Modifiers

* <tt style="color: #FF0000">`pure`</tt> for function: 수정 또는 상태 액세스를 허용하지 않습니다. 아직 적용되지 않았습니다.
* <tt style="color: #FF0000">`view`</tt> for function: 상태 수정을 허용하지 않습니다. 아직 적용되지 않았습니다.
* <tt style="color: #FF0000">`payable`</tt> for function: 호출을 통해 Ether를 받을 수 있습니다.
* <tt style="color: #FF0000">`constant`</tt> for state variables: 할당을 허용하지 않습니다(초기화 제외). 스토리지 슬롯을 점유하지 않습니다.
* <tt style="color: #FF0000">`constant`</tt> for function: <tt style="color: #FF0000">`view`</tt>와 같습니다.
* <tt style="color: #FF0000">`anonymous`</tt> for events: 이벤트 시그니쳐를 topic으로 저장하지 않습니다.
* <tt style="color: #FF0000">`indexed`</tt> for event parameter: 매개 변수를 topic으로 저장합니다.

### Reserved Keywords

이 키워드는 Solidity에 예약되어 있습니다. 다음과 같은 구문의 일부가 될 수 있습니다. :

<tt style="color: #FF0000">`abstract`</tt>, <tt style="color: #FF0000">`after`</tt>, <tt style="color: #FF0000">`case`</tt>, <tt style="color: #FF0000">`catch`</tt>, <tt style="color: #FF0000">`default`</tt>, <tt style="color: #FF0000">`final`</tt>,
<tt style="color: #FF0000">`in`</tt>, <tt style="color: #FF0000">`inline`</tt>, <tt style="color: #FF0000">`let`</tt>, <tt style="color: #FF0000">`match`</tt>, <tt style="color: #FF0000">`null`</tt>, <tt style="color: #FF0000">`of`</tt>,
<tt style="color: #FF0000">`relocatable`</tt>, <tt style="color: #FF0000">`static`</tt>, <tt style="color: #FF0000">`switch`</tt>, <tt style="color: #FF0000">`try`</tt>, <tt style="color: #FF0000">`type`</tt>, <tt style="color: #FF0000">`typeof`</tt>

### Language Grammar

~~~~
SourceUnit = (PragmaDirective | ImportDirective | ContractDefinition)*

// Pragma actually parses anything up to the trailing ';' to be fully forward-compatible.
PragmaDirective = 'pragma' Identifier ([^;]+) ';'

ImportDirective = 'import' StringLiteral ('as' Identifier)? ';'
        | 'import' ('*' | Identifier) ('as' Identifier)? 'from' StringLiteral ';'
        | 'import' '{' Identifier ('as' Identifier)? ( ',' Identifier ('as' Identifier)? )* '}' 'from' StringLiteral ';'

ContractDefinition = ( 'contract' | 'library' | 'interface' ) Identifier
                     ( 'is' InheritanceSpecifier (',' InheritanceSpecifier )* )?
                     '{' ContractPart* '}'

ContractPart = StateVariableDeclaration | UsingForDeclaration
             | StructDefinition | ModifierDefinition | FunctionDefinition | EventDefinition | EnumDefinition

InheritanceSpecifier = UserDefinedTypeName ( '(' Expression ( ',' Expression )* ')' )?

StateVariableDeclaration = TypeName ( 'public' | 'internal' | 'private' | 'constant' )? Identifier ('=' Expression)? ';'
UsingForDeclaration = 'using' Identifier 'for' ('*' | TypeName) ';'
StructDefinition = 'struct' Identifier '{'
                     ( VariableDeclaration ';' (VariableDeclaration ';')* )? '}'

ModifierDefinition = 'modifier' Identifier ParameterList? Block
ModifierInvocation = Identifier ( '(' ExpressionList? ')' )?

FunctionDefinition = 'function' Identifier? ParameterList
                     ( ModifierInvocation | StateMutability | 'external' | 'public' | 'internal' | 'private' )*
                     ( 'returns' ParameterList )? ( ';' | Block )
EventDefinition = 'event' Identifier EventParameterList 'anonymous'? ';'

EnumValue = Identifier
EnumDefinition = 'enum' Identifier '{' EnumValue? (',' EnumValue)* '}'

ParameterList = '(' ( Parameter (',' Parameter)* )? ')'
Parameter = TypeName StorageLocation? Identifier?

EventParameterList = '(' ( EventParameter (',' EventParameter )* )? ')'
EventParameter = TypeName 'indexed'? Identifier?

FunctionTypeParameterList = '(' ( FunctionTypeParameter (',' FunctionTypeParameter )* )? ')'
FunctionTypeParameter = TypeName StorageLocation?

// semantic restriction: mappings and structs (recursively) containing mappings
// are not allowed in argument lists
VariableDeclaration = TypeName StorageLocation? Identifier

TypeName = ElementaryTypeName
         | UserDefinedTypeName
         | Mapping
         | ArrayTypeName
         | FunctionTypeName

UserDefinedTypeName = Identifier ( '.' Identifier )*

Mapping = 'mapping' '(' ElementaryTypeName '=>' TypeName ')'
ArrayTypeName = TypeName '[' Expression? ']'
FunctionTypeName = 'function' FunctionTypeParameterList ( 'internal' | 'external' | StateMutability )*
                   ( 'returns' FunctionTypeParameterList )?
StorageLocation = 'memory' | 'storage'
StateMutability = 'pure' | 'constant' | 'view' | 'payable'

Block = '{' Statement* '}'
Statement = IfStatement | WhileStatement | ForStatement | Block | InlineAssemblyStatement |
            ( DoWhileStatement | PlaceholderStatement | Continue | Break | Return |
              Throw | EmitStatement | SimpleStatement ) ';'

ExpressionStatement = Expression
IfStatement = 'if' '(' Expression ')' Statement ( 'else' Statement )?
WhileStatement = 'while' '(' Expression ')' Statement
PlaceholderStatement = '_'
SimpleStatement = VariableDefinition | ExpressionStatement
ForStatement = 'for' '(' (SimpleStatement)? ';' (Expression)? ';' (ExpressionStatement)? ')' Statement
InlineAssemblyStatement = 'assembly' StringLiteral? InlineAssemblyBlock
DoWhileStatement = 'do' Statement 'while' '(' Expression ')'
Continue = 'continue'
Break = 'break'
Return = 'return' Expression?
Throw = 'throw'
EmitStatement = 'emit' FunctionCall
VariableDefinition = ('var' IdentifierList | VariableDeclaration) ( '=' Expression )?
IdentifierList = '(' ( Identifier? ',' )* Identifier? ')'

// Precedence by order (see github.com/ethereum/solidity/pull/732)
Expression
  = Expression ('++' | '--')
  | NewExpression
  | IndexAccess
  | MemberAccess
  | FunctionCall
  | '(' Expression ')'
  | ('!' | '~' | 'delete' | '++' | '--' | '+' | '-') Expression
  | Expression '**' Expression
  | Expression ('*' | '/' | '%') Expression
  | Expression ('+' | '-') Expression
  | Expression ('<<' | '>>') Expression
  | Expression '&' Expression
  | Expression '^' Expression
  | Expression '|' Expression
  | Expression ('<' | '>' | '<=' | '>=') Expression
  | Expression ('==' | '!=') Expression
  | Expression '&&' Expression
  | Expression '||' Expression
  | Expression '?' Expression ':' Expression
  | Expression ('=' | '|=' | '^=' | '&=' | '<<=' | '>>=' | '+=' | '-=' | '*=' | '/=' | '%=') Expression
  | PrimaryExpression

PrimaryExpression = BooleanLiteral
                  | NumberLiteral
                  | HexLiteral
                  | StringLiteral
                  | TupleExpression
                  | Identifier
                  | ElementaryTypeNameExpression

ExpressionList = Expression ( ',' Expression )*
NameValueList = Identifier ':' Expression ( ',' Identifier ':' Expression )*

FunctionCall = Expression '(' FunctionCallArguments ')'
FunctionCallArguments = '{' NameValueList? '}'
                      | ExpressionList?

NewExpression = 'new' TypeName
MemberAccess = Expression '.' Identifier
IndexAccess = Expression '[' Expression? ']'

BooleanLiteral = 'true' | 'false'
NumberLiteral = ( HexNumber | DecimalNumber ) (' ' NumberUnit)?
NumberUnit = 'wei' | 'szabo' | 'finney' | 'ether'
           | 'seconds' | 'minutes' | 'hours' | 'days' | 'weeks' | 'years'
HexLiteral = 'hex' ('"' ([0-9a-fA-F]{2})* '"' | '\'' ([0-9a-fA-F]{2})* '\'')
StringLiteral = '"' ([^"\r\n\\] | '\\' .)* '"'
Identifier = [a-zA-Z_$] [a-zA-Z_$0-9]*

HexNumber = '0x' [0-9a-fA-F]+
DecimalNumber = [0-9]+ ( '.' [0-9]* )? ( [eE] [0-9]+ )?

TupleExpression = '(' ( Expression? ( ',' Expression? )*  )? ')'
                | '[' ( Expression  ( ',' Expression  )*  )? ']'

ElementaryTypeNameExpression = ElementaryTypeName

ElementaryTypeName = 'address' | 'bool' | 'string' | 'var'
                   | Int | Uint | Byte | Fixed | Ufixed

Int = 'int' | 'int8' | 'int16' | 'int24' | 'int32' | 'int40' | 'int48' | 'int56' | 'int64' | 'int72' | 'int80' | 'int88' | 'int96' | 'int104' | 'int112' | 'int120' | 'int128' | 'int136' | 'int144' | 'int152' | 'int160' | 'int168' | 'int176' | 'int184' | 'int192' | 'int200' | 'int208' | 'int216' | 'int224' | 'int232' | 'int240' | 'int248' | 'int256'

Uint = 'uint' | 'uint8' | 'uint16' | 'uint24' | 'uint32' | 'uint40' | 'uint48' | 'uint56' | 'uint64' | 'uint72' | 'uint80' | 'uint88' | 'uint96' | 'uint104' | 'uint112' | 'uint120' | 'uint128' | 'uint136' | 'uint144' | 'uint152' | 'uint160' | 'uint168' | 'uint176' | 'uint184' | 'uint192' | 'uint200' | 'uint208' | 'uint216' | 'uint224' | 'uint232' | 'uint240' | 'uint248' | 'uint256'

Byte = 'byte' | 'bytes' | 'bytes1' | 'bytes2' | 'bytes3' | 'bytes4' | 'bytes5' | 'bytes6' | 'bytes7' | 'bytes8' | 'bytes9' | 'bytes10' | 'bytes11' | 'bytes12' | 'bytes13' | 'bytes14' | 'bytes15' | 'bytes16' | 'bytes17' | 'bytes18' | 'bytes19' | 'bytes20' | 'bytes21' | 'bytes22' | 'bytes23' | 'bytes24' | 'bytes25' | 'bytes26' | 'bytes27' | 'bytes28' | 'bytes29' | 'bytes30' | 'bytes31' | 'bytes32'

Fixed = 'fixed' | ( 'fixed' [0-9]+ 'x' [0-9]+ )

Ufixed = 'ufixed' | ( 'ufixed' [0-9]+ 'x' [0-9]+ )

InlineAssemblyBlock = '{' AssemblyItem* '}'

AssemblyItem = Identifier | FunctionalAssemblyExpression | InlineAssemblyBlock | AssemblyLocalBinding | AssemblyAssignment | AssemblyLabel | NumberLiteral | StringLiteral | HexLiteral
AssemblyLocalBinding = 'let' Identifier ':=' FunctionalAssemblyExpression
AssemblyAssignment = ( Identifier ':=' FunctionalAssemblyExpression ) | ( '=:' Identifier )
AssemblyLabel = Identifier ':'
FunctionalAssemblyExpression = Identifier '(' AssemblyItem? ( ',' AssemblyItem )* ')'
~~~~