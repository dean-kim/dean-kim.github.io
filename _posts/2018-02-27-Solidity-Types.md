---
layout: post
title:  "Solidity Types"
date:   2018-02-27 22:43:59
author: Dean Kim
categories: BlockChain
tags:	BlockChain Solidity Smart_Contracts
cover:  "/assets/instacode.png"
---

블록체인을 공부하며 Smart Contract 작성에 사용되는 Solidity 공식문서를 번역한 글입니다.
제가 이해한 수준으로 기술된 내용도 있어 원본과 내용이 조금 다를 수도 있습니다.
혹시 제가 잘못 이해한 내용이 있다면 말씀해주세요 :)

# Types

Solidity는 정적타입 언어입니다. 즉, 컴파일 시점에 각 변수(state and local)의 유형을 지정해야합니다 (또는 적어도 하단부의 Type Deduction 참조). Solidity는 복합 유형을 형성하기 위해 결합할 수 있는 몇 가지 기본 유형을 제공합니다.

또한 유형은 연산자가 포함된 표현식에서 서로 상호 작용할 수 있습니다. 다양한 연산자에 대한 빠른 참조는 [Order of Precedence of Operators](https://solidity.readthedocs.io/en/latest/miscellaneous.html#order)를 참조하십시오.


## Value Types

다음 유형은 이 유형의 변수가 항상 값으로 전달되기 때문에 값 유형이라고 도합니다. 즉, 함수 인수 또는 할당으로 사용될 때 항상 복사됩니다.

### Booleans

<tt style="color: #FF0000">`bool`</tt> : 가능한 값은 <tt style="color: #FF0000">`true`</tt> 및 <tt style="color: #FF0000">`false`</tt> 상수입니다.

연산자들:
* <tt style="color: #FF0000">`!`</tt> (논리적 부정)
* <tt style="color: #FF0000">`&&`</tt> (논리적 결합 "and")
* <tt style="color: #FF0000">`||`</tt> (논리적 분리 "or")
* <tt style="color: #FF0000">`==`</tt> (일치)
* <tt style="color: #FF0000">`!=`</tt> (불일)

연산자 <tt style="color: #FF0000">`||`</tt> 및 <tt style="color: #FF0000">`&&`</tt> 일반적인 short-circuiting 규칙을 적용합니다. 이것은 식 <tt style="color: #FF0000">`f(x) || g(y)`</tt>, 
<tt style="color: #FF0000">`f(x)`</tt>가 <tt style="color: #FF0000">`true`</tt>로 평가되면 <tt style="color: #FF0000">`g(y)`</tt>는 부작용이 있더라도 평가되지 않습니다.

### Integers

<tt style="color: #FF0000">`int`</tt> / <tt style="color: #FF0000">`uint`</tt> : 다양한 크기의 부호있는 및 부호없는 정수입니다. 
키워드 <tt style="color: #FF0000">`uint8`</tt> ~ <tt style="color: #FF0000">`uint256`</tt>은 <tt style="color: #FF0000">`8`</tt> (부호없는 8 ~ 256 비트) 및 <tt style="color: #FF0000">`int8`</tt> ~ <tt style="color: #FF0000">`int256`</tt> 단계입니다. 
<tt style="color: #FF0000">`uint`</tt> 및 <tt style="color: #FF0000">`int`</tt>는 각각 <tt style="color: #FF0000">`uint256`</tt> 및 <tt style="color: #FF0000">`int256`</tt>의 별칭입니다.

연산자들:
* 비교 연산자 : <tt style="color: #FF0000">`<=`</tt>, <tt style="color: #FF0000">`<`</tt>, <tt style="color: #FF0000">`==`</tt>, <tt style="color: #FF0000">`!=`</tt>, <tt style="color: #FF0000">`>=`</tt>, <tt style="color: #FF0000">`>`</tt> (evaluate to<tt style="color: #FF0000">`bool`</tt>)
* 비트 연산자 : <tt style="color: #FF0000">`&`</tt>, <tt style="color: #FF0000">`|`</tt>, <tt style="color: #FF0000">`^`</tt>(비트 배타적), <tt style="color: #FF0000">`~`</tt>(비트 부정)
* 산술 연산자 : <tt style="color: #FF0000">`+`</tt>, <tt style="color: #FF0000">`-`</tt>, 단항의 <tt style="color: #FF0000">`-`</tt>, 단항의 <tt style="color: #FF0000">`+`</tt>, <tt style="color: #FF0000">`*`</tt>, <tt style="color: #FF0000">`/`</tt>, <tt style="color: #FF0000">`%`</tt>(나머지), <tt style="color: #FF0000">`**`</tt>(거듭제곱), <tt style="color: #FF0000">`<<`</tt>(왼쪽 이동), <tt style="color: #FF0000">`>>`</tt>(오른쪽 이동)

Division은 항상 자릅니다. (EVM의 <tt style="color: #FF0000">`DIV`</tt> opcode(명령 코드)로 컴파일됨) 두 연산자가 [리터럴](https://solidity.readthedocs.io/en/latest/types.html#rational-literals)(또는 리터럴 표현식)인 경우 잘라내지 않습니다.

제로에 의한 나눗셈과 제로에 의한 모듈러스는 런타임 예외를 던집니다.

시프트 연산의 결과는 왼쪽 피연산자의 유형입니다. 표현식 <tt style="color: #FF0000">`x << y`</tt>는 <tt style="color: #FF0000">`x * 2 ** y`</tt>와 같고 <tt style="color: #FF0000">`x >> y`</tt>는 <tt style="color: #FF0000">`x / 2 ** y`</tt>와 같습니다. 즉, 음수 부호 이동이 확장됨을 의미합니다. 음수로 시프트하면 런타임 예외가 발생합니다.

<b>Warning</b>
<br>부호있는 정수 유형의 음수 값 시프트 권한에 의해 생성된 결과는 다른 프로그래밍 언어에 의해 생성된 결과와 다릅니다. 고밀도에서 오른쪽 지도를 나누기로 이동하면 이동된 음수 값이 0으로 반올림됩니다(잘렸음). 다른 프로그래밍 언어에서 음수 값의 시프트 권한은 반올림(음의 무한대 방향)과 함께 나누기와 같이 작동합니다.
<br><b>Warning End</b>

### Fixed Point Numbers

<b>Warning</b>
<br>고정 소수점 수는 아직 Solidity에서 완전히 지원하지 않습니다. 그것들은 선언할 수는 있지만 지정할 수는 없습니다.
<br><b>Warning End</b>

<tt style="color: #FF0000">`fixed`</tt> / <tt style="color: #FF0000">`ufixed`</tt> : 다양한 크기의 부호가 있는 고정 소수점 수. 
키워드 <tt style="color: #FF0000">`ufixedMxN`</tt> 및 <tt style="color: #FF0000">`fixedMxN`</tt>입니다. 
여기서 <tt style="color: #FF0000">`M`</tt>은 유형별로 취한 비트 수를 나타내고 <tt style="color: #FF0000">`N`</tt>은 사용 가능한 소수점 수를 나타냅니다. 
<tt style="color: #FF0000">`M`</tt>은 8로 나눌 수 있고 8비트에서 256비트가 되어야합니다. <tt style="color: #FF0000">`N`</tt>은 0과 80 사이여야 합니다. 
<tt style="color: #FF0000">`ufixed`</tt> 및 <tt style="color: #FF0000">`fixed`</tt>는 각각 <tt style="color: #FF0000">`ufixed128x19`</tt> 및 <tt style="color: #FF0000">`fixed128x19`</tt>의 별명입니다.

연산자들:
* 비교 연산자 : <tt style="color: #FF0000">`<=`</tt>, <tt style="color: #FF0000">`<`</tt>, <tt style="color: #FF0000">`==`</tt>, <tt style="color: #FF0000">`!=`</tt>, <tt style="color: #FF0000">`>=`</tt>, <tt style="color: #FF0000">`>`</tt> (evaluate to<tt style="color: #FF0000">`bool`</tt>)
* 산술 연산자 : <tt style="color: #FF0000">`+`</tt>, <tt style="color: #FF0000">`-`</tt>, 단항의 <tt style="color: #FF0000">`-`</tt>, 단항의 <tt style="color: #FF0000">`+`</tt>, <tt style="color: #FF0000">`*`</tt>, <tt style="color: #FF0000">`/`</tt>, <tt style="color: #FF0000">`%`</tt>(나머지)

<b>Note</b>
<br>부동 소수점 (많은 언어에서 <tt style="color: #FF0000">`float`</tt>과 <tt style="color: #FF0000">`double`</tt>, 더 정확하게 IEEE 754 숫자)과 고정 소수점 숫자 사이의 주요 차이점은 정수에 사용되는 비트 수와 소수 부분 (십진수 도트 뒤의 부분)은 전자는 엄격하게 후자에 정의되어있다. 
일반적으로 부동 소수점에서는 거의 전체 공간이 숫자를 나타내는 데 사용되지만 극소수의 비트만 소수점을 정의합니다.
<br><b>Note End</b>

### Address

<tt style="color: #FF0000">`address`</tt> : 20 바이트값(Ethereum 주소의 크기)을 보유합니다. 주소 유형은 member를 포함하고 있으며 모든 계약서의 기반이 됩니다.

연산자들:
* <tt style="color: #FF0000">`<=`</tt>, <tt style="color: #FF0000">`<`</tt>, <tt style="color: #FF0000">`==`</tt>, <tt style="color: #FF0000">`!=`</tt>, <tt style="color: #FF0000">`>=`</tt> 그리고 <tt style="color: #FF0000">`>`</tt>

<b>Note</b>
<br>버전 0.5.0부터 시작하는 계약은 주소 유형에서 파생되지 않지만 명시 적으로 주소로 변환 될 수 있습니다.
<br><b>Note End</b>

### Member of Addresses

* <tt style="color: #FF0000">`balance`</tt> 그리고 <tt style="color: #FF0000">`transfer`</tt>

빠른 참조는 [Address Related](https://solidity.readthedocs.io/en/latest/units-and-global-variables.html#address-related)에서 확인하세요.

<tt style="color: #FF0000">`balance`</tt> 속성을 사용하여 주소의 잔액을 조회하고 <tt style="color: #FF0000">`transfer`</tt> 함수를 사용하여 Ether(wei 단위)를 주소로 보낼 수 있습니다.

~~~~
address x = 0x123;
address myAddress = this;
if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);
~~~~

<b>Note</b>
<br><tt style="color: #FF0000">`x`</tt>가 계약 주소인 경우 해당 코드(보다 구체적으로 : fallback 함수가 있는 경우)가 <tt style="color: #FF0000">`transfer`</tt>와 함께 실행됩니다 (EVM의 기능이므로 막을 수 없습니다). 
해당 실행에 가스가 부족하거나 어떤식 으로든 실패하는 경우 Ether 전송이 되돌려지고 현재 계약이 예외로 중지됩니다.
<br><b>Note End</b>

* <tt style="color: #FF0000">`send`</tt>

보내기는 <tt style="color: #FF0000">`transfer`</tt>의 하위 수준입니다. 실행이 실패하면 현재 계약은 예외로 중지되지 않고 <tt style="color: #FF0000">`send`</tt>는 <tt style="color: #FF0000">`false`</tt>를 반환합니다.

<b>Warning</b>
<br><tt style="color: #FF0000">`send`</tt>를 사용하는데 위험이 있습니다:
콜 스택 깊이가 1024(호출자가 항상 강제로 설정할 수 있음)인 경우 전송이 실패하고 수신자의 가스가 부족한 경우에도 전송이 실패합니다. 그러므로 안전하게 Ether를 이전하기 위해서는 항상 <tt style="color: #FF0000">`send`</tt>의 반환 값을 확인하거나, <tt style="color: #FF0000">`transfer`</tt>를 사용해야 합니다. 수신자가 돈을 인출하는 패턴을 사용하십시오.
<br><b>Warning End</b>

* <tt style="color: #FF0000">`call`</tt>, <tt style="color: #FF0000">`callcode`</tt> 그리고 <tt style="color: #FF0000">`delegatecall`</tt>

더욱이, ABI를 준수하지 않는 계약과 연계하기 위해서, 함수 <tt style="color: #FF0000">`call`</tt>은 임의의 유형의 인수를 임의로 취하는 기능이 제공됩니다. 이러한 인수는 32바이트로 패딩되고 연결됩니다. 한 가지 예외는 첫 번째 인수가 정확히 4바이트로 인코딩되는 경우입니다. 이 경우 함수 서명을 사용할 수 있도록 패딩되지 않습니다.

~~~~
address nameReg = 0x72ba7d8e73fe8eb666ea66babc8116a41bfb10e2;
nameReg.call("register", "MyName");
nameReg.call(bytes4(keccak256("fun(uint256)")), a);
~~~~

<tt style="color: #FF0000">`call`</tt>은 호출된 함수가 종료되었는지(<tt style="color: #FF0000">`true`</tt>) 또는 EVM 예외가 발생했는지(<tt style="color: #FF0000">`false`</tt>) 나타내는 boolean을 반환합니다. 반환된 실제 데이터에 액세스할 수 없습니다(인코딩 및 크기를 미리 알고 있어야 합니다).

<tt style="color: #FF0000">`.gas()`</tt> 수정자를 사용하여 제공된 가스를 조정할 수 있습니다.

~~~~
namReg.call.gas(1000000)("register", "MyName");
~~~~

마찬가지로 공급된 Ether 값도 제어할 수 있습니다.

~~~~
nameReg.call.value(1 ether)("register", "MyName");
~~~~

마지막으로 이러한 수정자는 결합될 수 있습니다. 그들의 순서는 중요하지 않습니다.

~~~~
nameReg.call.gas(1000000).value(1 ether)("register", "MyName");
~~~~

<b>Note</b>
<br>overloaded 함수에서 가스 또는 값 수정자를 사용할 수 없습니다.

이 문제를 해결하려면 가스 및 값에 대한 특별한 경우를 소개하고 overloaded 해결 시점에 가스 및 값이 존재하는지 다시 확인하십시오.
<br><b>Note End</b>

유사한 방식으로 <tt style="color: #FF0000">`delegatecall`</tt> 함수를 사용할 수 있습니다. 차이점은 지정된 주소의 코드만 사용되며 다른 모든 측면(storage, balance, ...)은 현재 계약에서 가져온 것입니다. 
<tt style="color: #FF0000">`delegatecall`</tt>의 목적은 다른 계약서에 저장된 라이브러리 코드를 사용하는 것입니다. 사용자는 두 계약의 저장소 레이아웃을 사용하여 <tt style="color: #FF0000">`delegatecall`</tt>을 사용하는 것이 적합해야 합니다. 
homestead 이전에는 원래 <tt style="color: #FF0000">`msg.sender`</tt> 및 <tt style="color: #FF0000">`msg.value`</tt> 값에 대한 액세스를 제공하지 않는 <tt style="color: #FF0000">`callcode`</tt>라는 제한된 변형만 사용할 수 있었습니다.

세 함수 <tt style="color: #FF0000">`call`</tt>, <tt style="color: #FF0000">`delegatecall`</tt> 및 <tt style="color: #FF0000">`callcode`</tt>는 매우 low-level의 함수이므로 Solidity의 type-safety를 중단할 때와 같은 최후의 수단으로 사용해야합니다.

<tt style="color: #FF0000">`.gas()`</tt> 옵션은 세 가지 방법 모두에서 사용할 수 있지만 <tt style="color: #FF0000">`.value()`</tt> 옵션은 <tt style="color: #FF0000">`delegatecall`</tt>에서 지원되지 않습니다.

<b>Note</b>
<br>모든 계약은 주소 구성원을 상속하므로 <tt style="color: #FF0000">`this.balance`</tt>를 사용하여 현재 계약의 잔액을 쿼리할 수 있습니다.
<br><b>Note End</b>

<b>Note</b>
<br><tt style="color: #FF0000">`callcode`</tt> 사용은 권장되지 않으며 나중에 제거될 것입니다.
<br><b>Note End</b>

<b>Warning</b>
<br>이 모든 함수들은 low-level의 함수이므로 주의해서 사용해야 합니다. 구체적으로 알 수 없는 계약은 악성일 수 있으며 호출할 경우 해당 계약에 제어 권한을 넘겨주고 다시 계약 상태로 되돌릴 수 있으므로 call이 반환될 때 상태 변수가 변경될 수 있습니다.
<br><b>Warning End</b>

### Fixed-size byte arrays

<tt style="color: #FF0000">`bytes1`</tt>, <tt style="color: #FF0000">`bytes2`</tt>, <tt style="color: #FF0000">`bytes3`</tt>, ..., <tt style="color: #FF0000">`bytes32`</tt>. <tt style="color: #FF0000">`byte`</tt>는 <tt style="color: #FF0000">`bytes1`</tt>의 별칭입니다.

연산자들:
* 비교 연산자 : <tt style="color: #FF0000">`<=`</tt>, <tt style="color: #FF0000">`<`</tt>, <tt style="color: #FF0000">`==`</tt>, <tt style="color: #FF0000">`!=`</tt>, <tt style="color: #FF0000">`>=`</tt>, <tt style="color: #FF0000">`>`</tt> (evaluate to<tt style="color: #FF0000">`bool`</tt>)
* 비트 연산자 : <tt style="color: #FF0000">`&`</tt>, <tt style="color: #FF0000">`|`</tt>, <tt style="color: #FF0000">`^`</tt>(비트 배타적), <tt style="color: #FF0000">`~`</tt>(비트 부정)
* 인덱스 접근 : <tt style="color: #FF0000">`x`</tt>가 <tt style="color: #FF0000">`bytesI`</tt> 유형인 경우 <tt style="color: #FF0000">`0 <= k < I`</tt>인 <tt style="color: #FF0000">`x[k]`</tt>는 <tt style="color: #FF0000">`k`</tt>번째 바이트(읽기 전용)를 반환합니다.

시프팅 연산자는 오른쪽 피연산자로 모든 정수 유형과 함께 작동하지만(왼쪽 피연산자의 유형을 반환합니다), 시프트할 비트 수를 나타냅니다. 음수로 이동하면 런타임 예외가 발생합니다.

Members:
* <tt style="color: #FF0000">`.length`</tt> 바이트 배열의 고정 길이를 반환합니다(읽기 전용).

<b>Note</b>
<br>바이트의 배열을 <tt style="color: #FF0000">`byte[]`</tt>로 사용할 수는 있지만, 호출할 때 정확히 모든 31 바이트의 공간을 낭비하고 있습니다. <tt style="color: #FF0000">`bytes`</tt>를 사용하는 것이 좋습니다.
<br></b>Note End</b>

### Dynamically-sized byte array

<tt style="color: #FF0000">`bytes`</tt>:
동적 크기의 바이트 배열. [Arrays](https://solidity.readthedocs.io/en/latest/types.html#arrays)를 참조하십시오. value-type이 아닙니다!

<tt style="color: #FF0000">`string`</tt>:
동적 크기의 UTF-8로 인코딩 된 문자열은 [Arrays](https://solidity.readthedocs.io/en/latest/types.html#arrays)를 참조하십시오. value-type이 아닙니다!

일반적으로 임의 길이의 원시 바이트 데이터에는 <tt style="color: #FF0000">`bytes`</tt>를 사용하고 임의 길이 문자열 (UTF-8) 데이터에는 <tt style="color: #FF0000">`string`</tt>을 사용합니다. 길이를 특정 바이트 수로 제한 할 수 있으면 훨씬 저렴하기 때문에 항상 <tt style="color: #FF0000">`bytes1`</tt> - <tt style="color: #FF0000">`bytes32`</tt> 중 하나를 사용하십시오.

<b>Note</b>
<br>대소 문자가 혼합된 주소 체크섬 형식은 [EIP-55](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md)에 정의되어 있습니다.
<br><b>Note End</b>

### Rational and Integer Literals

정수 리터럴은 0-9 범위의 숫자 시퀀스로 구성됩니다. 그것들은 십진수로 해석됩니다. 예를 들어, <tt style="color: #FF0000">`69`</tt>는 69를 의미합니다. Solidity에서는 8진 리터럴이 존재하지 않으며 선행 0은 유효하지 않습니다.

소수 자릿수 리터럴은 <tt style="color: #FF0000">`.`</tt>에 의해 형성됩니다. 한쪽에 적어도 하나의 숫자가 있어야합니다. 예로 <tt style="color: #FF0000">`1.`</tt>, <tt style="color: #FF0000">`.1`</tt> 및 <tt style="color: #FF0000">`1.3`</tt>이 포함됩니다.

과학적 표기법도 지원됩니다. 여기서 기수는 분수를 가질 수 있지만 지수는 가질 수 없습니다. 예로 <tt style="color: #FF0000">`2e10`</tt>, <tt style="color: #FF0000">`-2e10`</tt>, <tt style="color: #FF0000">`2e-10`</tt>, <tt style="color: #FF0000">`2.5e1`</tt>을 포함합니다.

숫자 리터럴 표현식은 비 리터럴 유형으로 변환될 때까지 (즉 리터럴이 아닌 표현식과 함께 사용하여) 임의의 정밀도를 유지합니다. 즉, 연산이 넘치지 않고 나눗셈은 숫자의 리터럴 표현에서 잘라내지 않는다는 것을 의미합니다.

예를 들어, <tt style="color: #FF0000">`(2**800 + 1) - 2**800`</tt>은 중급 결과가 machine word size에도 맞지 않더라도 상수 <tt style="color: #FF0000">`1`</tt> (유형 <tt style="color: #FF0000">`uint8`</tt>)이 됩니다. 또한 <tt style="color: #FF0000">`.5 * 8`</tt>의 결과는 정수 <tt style="color: #FF0000">`4`</tt>입니다 (정수가 아닌 정수도 사용됩니다).

정수에 적용할 수 있는 연산자는 피연산자가 정수인 경우 숫자 리터럴 식에도 적용할 수 있습니다. 둘 중 하나라도 소수일 경우 비트 연산이 허용되지 않으며 지수가 소수일 경우 지수가 허용되지 않습니다(비 유리수가 발생할 수 있으므로).

<b>Note</b>
<br>Solidity는 각 유리수에 대해 숫자 상수 유형을 가집니다. 정수 리터럴과 유리수 리터럴은 숫자 리터럴 유형에 속합니다. 또한 모든 숫자 리터럴 표현식(즉, 숫자 리터럴 및 연산자만 포함된 표현식)은 숫자 리터럴 유형에 속합니다. 따라서 숫자 리터럴 표현식 <tt style="color: #FF0000">`1 + 2`</tt>와 <tt style="color: #FF0000">`2 + 1`</tt>은 모두 유리수 3에 대해 동일한 숫자 리터럴 유형에 속합니다.
<br><b>Note End</b>

<b>Warning</b>
<br>이전 버전에서는 정수 리터럴의 부분이 잘리지 않았지만 이제는 유리수로 변환됩니다. 즉, <tt style="color: #FF0000">`5 / 2`</tt>는 <tt style="color: #FF0000">`2`</tt>와 같지 않지만 <tt style="color: #FF0000">`2.5`</tt>로 변환됩니다.
<br><b>Warning End</b>

<b>Note</b>
<br>숫자 리터럴 표현식은 리터럴이 아닌 표현식과 함께 사용되는 즉시 리터럴이 아닌 유형으로 변환됩니다. 다음 예제에서 <tt style="color: #FF0000">`b`</tt>에 지정된 식의 값이 정수로 계산되지만 부분 식 <tt style="color: #FF0000">`2.5 + a`</tt>는 코드가 컴파일되지 않도록 유형 확인을 수행하지 않습니다.
<br><b>Note End</b>

~~~~
uint128 a = 1;
uint128 b = 2.5 + a + 0.5;
~~~~

### String Literals

문자열 리터럴은 큰 따옴표 또는 작은 따옴표(<tt style="color: #FF0000">`"foo"`</tt>또는 <tt style="color: #FF0000">`'bar''`</tt>)로 작성됩니다. 
그것들은 C; 에서처럼 후행하는 0을 의미하지 않습니다. "foo"는 4가 아닌 3바이트를 나타냅니다. 
정수 리터럴과 마찬가지로 유형도 다양하지만 <tt style="color: #FF0000">`bytes1`</tt>, ..., <tt style="color: #FF0000">`bytes32`</tt> (적합하다면 <tt style="color: #FF0000">`bytes`</tt> 및 <tt style="color: #FF0000">`string`</tt>)로 암시적으로 변환할 수 있습니다.

문자열 리터럴은 <tt style="color: #FF0000">`\n`</tt>, <tt style="color: #FF0000">`\xNN`</tt> 및 <tt style="color: #FF0000">`\uNNNN`</tt>과 같은 이스케이프 문자를 지원합니다. 
<tt style="color: #FF0000">`\xNN`</tt>은 16진수 값을 취해 적절한 바이트를 삽입하며 <tt style="color: #FF0000">`\uNNNN`</tt>은 유니코드 코드포인트를 취하여 UTF-8 시퀀스를 삽입합니다.

### Hexadecimal Literals

Hexademical Literals에는 키워드 <tt style="color: #FF0000">`hex`</tt>가 접두사로 사용되며 이중 또는 작은 따옴표 (<tt style="color: #FF0000">`hex"001122FF"`</tt>)로 묶여 있습니다. 
내용은 16 진수 문자열이어야하며 그 값은 해당 값의 2진 표현입니다.

Hexademical Literals는 String Literals처럼 작동하며 동일한 변환 기능 제한이 있습니다.

### Enums

열거형은 Solidity에서 사용자 정의 형식을 만드는 한 가지 방법입니다. 
이들은 명시적으로 모든 정수 유형으로 변환할 수 있지만 암시적 변환은 허용되지 않습니다. 
명시적 변환은 런타임에 값 범위를 확인하고 오류가 발생하면 예외가 발생합니다. Enum에는 최소한 하나의 멤버가 필요합니다.

~~~~
pragma solidity ^0.4.16;

contract test {
    enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }
    ActionChoices choice;
    ActionChoices constant defaultChoice = ActionChoices.GoStraight;

    function setGoStraight() public {
        choice = ActionChoices.GoStraight;
    }

    // enum 유형은 ABI의 일부가 아니므로 Solidity 외부의 모든 사항에 대해
    // "getChoice"의 시그니처가 자동으로 "getChoice() returns (uint8)"로 변경됩니다.
    // 사용된 정수의 유형은 모든 열거형 값들을 담을 수 있을만큼 커야합니다.
    // 만약 값이 더 많다면 'uint16'이 사용됩니다.
    function getChoice() public view returns (ActionChoices) {
        return choice;
    }

    function getDefaultChoice() public pure returns (uint) {
        return uint(defaultChoice);
    }
}
~~~~

### Function Types

함수 유형은 함수 유형입니다. 
함수 유형의 변수는 함수에서 할당할 수 있으며 함수 유형의 함수 매개 변수는 함수를 함수 호출로 전달하거나 함수 호출에서 함수를 반환하는데 사용할 수 있습니다. 
함수 유형에는 내부 및 외부 함수의 두 가지 종류가 있습니다.

내부 함수는 현재 계약의 컨텍스트 외부에서 실행될 수 없기 때문에 현재 계약 내에서 호출 될 수 있습니다 (특히 현재 코드 단위 내부에서는 내부 라이브러리 함수 및 상속된 함수가 포함됨). 
내부 함수를 호출하는 것은 현재 계약의 함수를 내부적으로 호출 할 때와 마찬가지로 항목 레이블로 점프함으로써 실현됩니다.

외부 함수는 주소와 함수 시그니처로 구성되며 외부 함수 호출을 통해 전달되고 반환 될 수 있습니다.

함수 유형은 다음과 같이 표시됩니다.

~~~~
function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]
~~~~

매개 변수 유형과 달리 반환 유형은 비워둘 수 없습니다. 함수 유형이 아무 것도 반환하지 않으면 전체 <tt style="color: #FF0000">`returns (<return types>)`</tt> 부분을 생략해야합니다.

기본적으로 함수 유형은 internal 키워드이므로 <tt style="color: #FF0000">`internal`</tt> 키워드는 생략 할 수 있습니다. 대조적으로, 계약 함수 자체는 기본적으로 public이며, 유형의 이름으로 사용될 때만 기본값이 internal입니다.

현재 계약에서 함수에 액세스하는 방법은 두 가지가 있습니다. 이름으로 직접 지정하거나 <tt style="color: #FF0000">`f`</tt> 또는 <tt style="color: #FF0000">`this.f`</tt>를 사용하는 것입니다. 전자는 내부 함수를, 후자는 외부 함수를 제공합니다.

함수 유형 변수가 초기화되지 않은 경우 이를 호출하면 예외가 발생합니다. 함수에서 <tt style="color: #FF0000">`delete`</tt>를 사용한 후에 함수를 호출하면 마찬가지입니다.

외부 함수 유형이 Solidity의 컨텍스트 외부에서 사용되는 경우 <tt style="color: #FF0000">`function`</tt> 유형으로 처리되며, 함수 유형이 단일 <tt style="color: #FF0000">`bytes24`</tt> 유형으로 함께 주소 뒤에 인코딩됩니다.

현재 계약의 공용 기능은 내부 기능과 외부 기능으로 모두 사용할 수 있습니다. <tt style="color: #FF0000">`f`</tt>를 내부 함수로 사용하려면 f를 사용하고 외부 형식을 사용하려면 <tt style="color: #FF0000">`this.f`</tt>를 사용하십시오.

또한 공용 (또는 외부) 함수에는 [ABI function selector](https://solidity.readthedocs.io/en/latest/abi-spec.html#abi-function-selector)를 반환하는 <tt style="color: #FF0000">`selector`</tt>라는 특수 멤버가 있습니다.

~~~~
pragma solidity ^0.4.16;

contract Selector {
  function f() public view returns (bytes4) {
    return this.f.selector;
  }
}
~~~~

내부 함수 유형을 사용하는 방법을 보여주는 예제 :

~~~~
pragma solidity ^0.4.16;

library ArrayUtils {
  // 내부 함수는 내부 코드가 동일한 코드 컨텍스트에 포함되기 때문에 내부 라이브러리 함수에서 사용할 수 있습니다.
  function map(uint[] memory self, function (uint) pure returns (uint) f)
    internal
    pure
    returns (uint[] memory r)
  {
    r = new uint[](self.length);
    for (uint i = 0; i < self.length; i++) {
      r[i] = f(self[i]);
    }
  }
  function reduce(
    uint[] memory self,
    function (uint, uint) pure returns (uint) f
  )
    internal
    pure
    returns (uint r)
  {
    r = self[0];
    for (uint i = 1; i < self.length; i++) {
      r = f(r, self[i]);
    }
  }
  function range(uint length) internal pure returns (uint[] memory r) {
    r = new uint[](length);
    for (uint i = 0; i < r.length; i++) {
      r[i] = i;
    }
  }
}

contract Pyramid {
  using ArrayUtils for *;
  function pyramid(uint l) public pure returns (uint) {
    return ArrayUtils.range(l).map(square).reduce(sum);
  }
  function square(uint x) internal pure returns (uint) {
    return x * x;
  }
  function sum(uint x, uint y) internal pure returns (uint) {
    return x + y;
  }
}
~~~~

외부 함수 유형을 사용하는 또 다른 예는 다음과 같습니다.

~~~~
pragma solidity ^0.4.20; // should actually be 0.4.21

contract Oracle {
  struct Request {
    bytes data;
    function(bytes memory) external callback;
  }
  Request[] requests;
  event NewRequest(uint);
  function query(bytes data, function(bytes memory) external callback) public {
    requests.push(Request(data, callback));
    emit NewRequest(requests.length - 1);
  }
  function reply(uint requestID, bytes response) public {
    // reply가 신뢰할 수있는 출처에서 오는 것인지 확인합니다.
    requests[requestID].callback(response);
  }
}

contract OracleUser {
  Oracle constant oracle = Oracle(0x1234567); // known contract
  function buySomething() {
    oracle.query("USD", this.oracleResponse);
  }
  function oracleResponse(bytes response) public {
    require(msg.sender == address(oracle));
    // Use the data
  }
}
~~~~

<b>Note</b>
<br>람다 또는 인라인 함수는 계획되었지만 아직 지원되지 않습니다.
<br><b>Note End</b>


## Reference Types

복잡한 유형, 항상 256 비트에 맞지 않는 유형같은 것들은 이미 본 value type보다 신중하게 처리해야합니다. 
그것들을 복사하는 데에 상당한 비용이 들 수 있기 때문에, 우리는 그것들을 <b>memory</b>에 저장할 것인지(유지관리가 되지 않는) 또는 <b>storage</b>(상태 변수를 저장하는 곳)에 저장할지를 생각해야합니다.

### Data location

모든 복합 유형, 즉 배열과 구조체에는 memory 또는 storage에 저장되는지에 대한 추가 주석인 "데이터 위치"가 있습니다. 
컨텍스트에 따라 항상 기본값이 있지만 유형에 <tt style="color: #FF0000">`storage`</tt> 또는 <tt style="color: #FF0000">`memory`</tt>를 추가하여 재정의할 수 있습니다.
함수 매개 변수(반환 매개 변수 포함)의 기본값은 <tt style="color: #FF0000">`memory`</tt>이며 로컬 변수의 기본값은 <tt style="color: #FF0000">`storage`</tt>이고 위치는 상태 변수(명백하게)에 대한 <tt style="color: #FF0000">`storage`</tt>로 강제 설정됩니다.

또한 함수 인수가 저장되는 수정 불가능한 비 지속 영역 인 <tt style="color: #FF0000">`calldata`</tt>라는 세 번째 데이터 위치가 있습니다. 
외부 함수의 함수 매개 변수(반환 매개 변수가 아님)는 <tt style="color: #FF0000">`calldata`</tt>에 강제하고 <tt style="color: #FF0000">`memory`</tt>와 비슷하게 동작합니다.

데이터 위치는 할당이 작동하는 방식을 변경하기 때문에 중요합니다. 
storage와 memory 간의 할당과 상태 변수(다른 상태 변수에서도)는 항상 독립적인 복사본을 만듭니다. 
로컬 저장소 변수에 대한 할당은 참조를 지정하기만하며, 이 참조는 그 동안 해당 변수가 변경되더라도 항상 상태 변수를 가리킵니다. 
반면, 메모리 저장 참조 유형에서 다른 메모리 저장 참조 유형으로의 할당은 사본을 만들지 않습니다.

~~~~
pragma solidity ^0.4.0;

contract C {
    uint[] x; // x의 데이터 위치는 storage입니다. 

    // memoryArray의 데이터 위치는 memory입니다.
    function f(uint[] memoryArray) public {
        x = memoryArray; // 전체 배열을 storage에 복사합니다. 
        var y = x; // 포인터 지정, y의 데이터 위치 storage에 저장
        y[7]; // 8번째 원소를 반환한다.
        y.length = 2; // x에서 y까지 modifies
        delete x; // 배열을 지우고, 또한 y를 modifies.
        // 다음은 작동하지 않습니다. 새로운 임시 / 이름없는 배열을 storage에 만들어야하지만 storage는 "정적으로"할당됩니다.
        // y = memoryArray;
        // 이것은 포인터를 "재설정"하기 때문에 작동하지 않지만 가리킬 수 있는 적절한 위치가 없습니다.
        // delete y;
        g(x); // g를 호출하고, x에 대한 참조를 건네줍니다.
        h(x); // h를 호출하고 memory에 독립적인 임시 복사본을 만듭니다.
    }

    function g(uint[] storage storageArray) internal {}
    function h(uint[] memoryArray) public {}
}
~~~~

#### Summary

##### Forced data location:
* 외부 함수의 매개 변수(반환하지 않음) : calldata
* 상태 변수 : storage

##### Default data location:
* 함수의 매개 변수 (반환됨) : memory
* 다른 모든 지역 변수 : storage

### Arrays

배열은 컴파일 타임 고정 크기를 가질 수도 있고 동적인 크기를 가질 수도 있습니다. 
storage 배열의 경우 요소 유형은 임의적일 수 있습니다(즉, 다른 배열, 매핑 또는 구조체). 
memory 배열의 경우 매핑이 될 수 없으며 공개적으로 표시되는 함수의 인수인 경우 ABI 유형이어야합니다.

고정 사이즈 <tt style="color: #FF0000">`k`</tt>와 요소형 <tt style="color: #FF0000">`T`</tt>의 배열은, <tt style="color: #FF0000">`T[k]`</tt>라고 기록되며 동적 사이즈의 배열은 <tt style="color: #FF0000">`T[]`</tt>로서 기록됩니다. 
예를 들어, 5개의 동적 배열 <tt style="color: #FF0000">`uint`</tt>는 <tt style="color: #FF0000">`uint[][5]`</tt>입니다 (다른 언어와 비교할 때 표기법이 반대로됨을 유의하십시오). 
세 번째 동적 배열의 두 번째 uint에 액세스하려면 <tt style="color: #FF0000">`x[2][1]`</tt>을 사용합니다 (인덱스는 0부터 시작하며 액세스는 선언과 반대되는 방식으로 작동합니다. 즉 <tt style="color: #FF0000">`x[2]`</tt>는 오른쪽에서 유형의 한 수준을 제거한 상태입니다).

<tt style="color: #FF0000">`bytes`</tt> 및 <tt style="color: #FF0000">`string`</tt> 유형의 변수는 특수 배열입니다. 
<tt style="color: #FF0000">`bytes`</tt>는 <tt style="color: #FF0000">`byte[]`</tt>와 비슷하지만 calldata에 밀집되어 있습니다. <tt style="color: #FF0000">`string`</tt>은 <tt style="color: #FF0000">`bytes`</tt>와 같지만 길이 또는 색인 액세스를 허용하지 않습니다(현재).

<tt style="color: #FF0000">`bytes`</tt>는 항상 저렴하기 때문에 항상 <tt style="color: #FF0000">`bytes[]`</tt>보다 선호되어야 합니다.

<b>Note</b>
<br>문자열 <tt style="color: #FF0000">`s`</tt>의 바이트 표현에 액세스하려면 <tt style="color: #FF0000">`bytes(s).length`</tt> / <tt style="color: #FF0000">`bytes(s)[7] = 'x';`</tt>를 사용하십시오. UTF-8 표현의 하위 레벨 바이트에 액세스하고 있지만 개별 문자는 액세스 할 수 없습니다!
<br><b>Note End</b>

배열을 <tt style="color: #FF0000">`public`</tt>으로 표시하고 Solidity가 [getter](https://solidity.readthedocs.io/en/latest/contracts.html#visibility-and-getters)를 생성하도록 할 수 있습니다. 숫자 인덱스는 getter에 대한 필수 매개 변수가 됩니다.

#### Allocating Memory Arrays

가변 길이를 가진 배열을 메모리에 생성하는 것은 <tt style="color: #FF0000">`new`</tt> 키워드를 사용하여 수행 할 수 있습니다. 스토리지 배열과 달리, <tt style="color: #FF0000">`.length`</tt> 멤버에 할당하여 메모리 배열의 크기를 조정할 수는 없습니다.

~~~~
pragma solidity ^0.4.16;

contract C {
    function f(uint len) public pure {
        uint[] memory a = new uint[](7);
        bytes memory b = new bytes(len);
        // Here we have a.length == 7 and b.length == len
        a[6] = 8;
    }
}
~~~~

#### Array Literals / Inline Arrays

배열 리터럴은 표현식으로 작성된 배열이며 즉시 변수에 할당되지 않습니다.

~~~~
pragma solidity ^0.4.16;

contract C {
    function f() public pure {
        g([uint(1), 2, 3]);
    }
    function g(uint[3] _data) public pure {
        // ...
    }
}
~~~~

배열 리터럴의 유형은 고정 크기의 메모리 배열이며 기본 유형은 지정된 요소의 공통 유형입니다. 
<tt style="color: #FF0000">`[1, 2, 3]`</tt>의 유형은 <tt style="color: #FF0000">`uint8[3] memory`</tt>입니다. 이 상수 각각의 유형은 <tt style="color: #FF0000">`uint8`</tt>이기 때문입니다. 
그 때문에 위 예제의 첫 번째 요소를 <tt style="color: #FF0000">`uint`</tt>로 변환해야 했습니다. 
현재 고정 크기 메모리 배열은 동적 크기 메모리 배열에 할당할 수 없습니다. 즉, 다음과 같은 것은 불가능합니다.

~~~~
// This will not compile.

pragma solidity ^0.4.0;

contract C {
    function f() public {
        // The next line creates a type error because uint[3] memory
        // cannot be converted to uint[] memory.
        uint[] x = [uint(1), 3, 4];
    }
}
~~~~

앞으로 이 제한 사항을 제거할 계획이지만 ABI에서 배열이 전달되는 방식 때문에 현재 몇 가지 문제가 발생합니다.

#### Members

<b>length:</b>
<br>배열에는 요소의 수를 보유하는 <tt style="color: #FF0000">`length`</tt> 멤버가 있습니다. 
동적 배열은 <tt style="color: #FF0000">`.length`</tt> 멤버를 변경하여 memory가 아니라 storage에서 크기를 조정할 수 있습니다. 
현재 길이 밖의 요소에 액세스하려고 하면 자동으로 이 작업이 수행되지 않습니다. 
memory 배열의 크기는 일단 생성되면 고정되지만 동적(런타임 매개 변수에 따라 달라질 수 있음)입니다.

<b>push:</b>
<br>동적 저장소 배열과 <tt style="color: #FF0000">`bytes`</tt> (<tt style="color: #FF0000">`string`</tt>이 아님)에는 <tt style="color: #FF0000">`push`</tt>라는 멤버 함수가 있습니다.
이 함수는 배열 끝에 요소를 추가하는 데 사용할 수 있습니다. 이 함수는 새로운 길이를 반환합니다.

<b>Warning</b>
<br>외부 함수에서 배열을 사용할 수는 없습니다.
<br><b>Warning End</b>

<b>Warning</b>
<br>EVM의 제한으로 인해 외부 함수 호출에서 동적 내용을 반환할 수 없습니다. 
<tt style="color: #FF0000">`contract C {function f() returns (uint[]) {...}}`</tt>의 <tt style="color: #FF0000">`f`</tt>가 web3.js에서 호출되면 무언가를 반환하지만 Solidity에서 호출된 경우에는 반환하지 않습니다.

유일한 해결 방법은 큰 정적 크기의 배열을 사용하는 것입니다.
<br><b>Warning End</b>

~~~~
pragma solidity ^0.4.16;

contract ArrayContract {
    uint[2**20] m_aLotOfIntegers;
    // 다음은 한 쌍의 동적 배열이 아니라 쌍의 동적 배열(즉, 길이가 2 인 고정된 크기의 배열)입니다.
    bool[2][] m_pairsOfFlags;
    // newPairs는 memory에 저장됩니다. - 함수 인수의 기본설정

    function setAllFlagPairs(bool[2][] newPairs) public {
        // 스토리지 배열에 대한 할당이 전체 배열을 대체합니다.
        m_pairsOfFlags = newPairs;
    }

    function setFlagPair(uint index, bool flagA, bool flagB) public {
        // 존재하지 않는 색인에 대한 액세스는 예외를 던집니다.
        m_pairsOfFlags[index][0] = flagA;
        m_pairsOfFlags[index][1] = flagB;
    }

    function changeFlagArraySize(uint newSize) public {
        // if the new size is smaller, removed array elements will be cleared
        m_pairsOfFlags.length = newSize;
    }

    function clear() public {
        // these clear the arrays completely
        delete m_pairsOfFlags;
        delete m_aLotOfIntegers;
        // identical effect here
        m_pairsOfFlags.length = 0;
    }

    bytes m_byteData;

    function byteArrays(bytes data) public {
        // byte arrays ("bytes") are different as they are stored without padding,
        // but can be treated identical to "uint8[]"
        m_byteData = data;
        m_byteData.length += 7;
        m_byteData[3] = byte(8);
        delete m_byteData[2];
    }

    function addFlag(bool[2] flag) public returns (uint) {
        return m_pairsOfFlags.push(flag);
    }

    function createMemoryArray(uint size) public pure returns (bytes) {
        // Dynamic memory arrays are created using `new`:
        uint[2][] memory arrayOfPairs = new uint[2][](size);
        // Create a dynamic byte array:
        bytes memory b = new bytes(200);
        for (uint i = 0; i < b.length; i++)
            b[i] = byte(i);
        return b;
    }
}
~~~~

### Structs

Solidity는 다음 예에서 보여지는 struct 유형으로 새 유형을 정의하는 방법을 제공합니다.

~~~~
pragma solidity ^0.4.11;

contract CrowdFunding {
    // Defines a new type with two fields.
    struct Funder {
        address addr;
        uint amount;
    }

    struct Campaign {
        address beneficiary;
        uint fundingGoal;
        uint numFunders;
        uint amount;
        mapping (uint => Funder) funders;
    }

    uint numCampaigns;
    mapping (uint => Campaign) campaigns;

    function newCampaign(address beneficiary, uint goal) public returns (uint campaignID) {
        campaignID = numCampaigns++; // campaignID is return variable
        // Creates new struct and saves in storage. We leave out the mapping type.
        campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
    }

    function contribute(uint campaignID) public payable {
        Campaign storage c = campaigns[campaignID];
        // Creates a new temporary memory struct, initialised with the given values
        // and copies it over to storage.
        // Note that you can also use Funder(msg.sender, msg.value) to initialise.
        c.funders[c.numFunders++] = Funder({addr: msg.sender, amount: msg.value});
        c.amount += msg.value;
    }

    function checkGoalReached(uint campaignID) public returns (bool reached) {
        Campaign storage c = campaigns[campaignID];
        if (c.amount < c.fundingGoal)
            return false;
        uint amount = c.amount;
        c.amount = 0;
        c.beneficiary.transfer(amount);
        return true;
    }
}
~~~~

계약은 크라우드 펀딩 계약의 모든 기능을 제공하지 않지만 struct를 이해하는 데 필요한 기본 개념을 포함합니다. struct 유형은 매핑과 배열 내부에서 사용할 수 있으며 매핑 유형과 배열을 포함할 수 있습니다.

struct 자체는 매핑 멤버의 값 유형이 될 수 있지만 struct가 자체 유형의 멤버를 포함할 수 없습니다. struct의 크기가 유한해야하므로 이 제한이 필요합니다.

모든 함수에서 struct 유형이(기본 storage 데이터 위치의) 로컬 변수에 할당되는 방법에 유의하십시오. 이것은 struct를 복사하지는 않지만 로컬 변수의 멤버에 대한 할당이 실제로 상태에 쓰도록 참조만 저장합니다.

물론 <tt style="color: #FF0000">`campaigns[campaignID].amount = 0`</tt>처럼 로컬 변수에 할당하지 않고 struct의 멤버에 직접 액세스할 수도 있습니다.

### Mappings

매핑 유형은 <tt style="color: #FF0000">`mapping(_KeyType => _ValueType)`</tt>으로 선언됩니다. 
<tt style="color: #FF0000">`_KeyType`</tt>은 매핑, 동적 크기 배열, 계약, 열거형 및 struct를 제외한 거의 모든 유형이 될 수 있습니다. <tt style="color: #FF0000">`_ValueType`</tt>은 실제로 맵핑을 포함하여 모든 유형이 될 수 있습니다.

맵핑은 해시 테이블로 볼 수 있습니다. [해시 테이블](https://ko.wikipedia.org/wiki/%ED%95%B4%EC%8B%9C_%ED%85%8C%EC%9D%B4%EB%B8%94)은 가능한 모든 키가 존재하고 바이트 표현이 모두 0인 값([유형의 기본값](https://solidity.readthedocs.io/en/latest/control-structures.html#default-value))으로 매핑되도록 사실상 초기화됩니다. 
그러나 유사점은 다음과 같이 끝납니다. 실제로 키 데이터가 매핑에 저장되는 것이 아니라, 값을 조회하는 데 사용되는 키 리모컨인 <tt style="color: #FF0000">`keccak256`</tt>해시만 저장됩니다.

이 때문에 매핑에는 키 또는 값이 "set"되는 길이 또는 개념이 없습니다.

매핑은 상태 변수(또는 내부 함수의 storage 참조 유형)에 대해서만 허용됩니다.

<tt style="color: #FF0000">`public`</tt>으로 매핑을 표시하고 Solidity가 [getter](https://solidity.readthedocs.io/en/latest/contracts.html#visibility-and-getters)를 생성하도록 할 수 있습니다. <tt style="color: #FF0000">`_KeyType`</tt>은 getter에 대해 필수 매개 변수가 되고 <tt style="color: #FF0000">`_ValueType`</tt>을 반환합니다.

<tt style="color: #FF0000">`_ValueType`</tt>도 매핑이 될 수 있습니다. getter는 재귀적으로 각 <tt style="color: #FF0000">`_KeyType`</tt>에 대해 하나의 매개 변수를 갖습니다.

~~~~
pragma solidity ^0.4.0;

contract MappingExample {
    mapping(address => uint) public balances;

    function update(uint newBalance) public {
        balances[msg.sender] = newBalance;
    }
}

contract MappingUser {
    function f() public returns (uint) {
        MappingExample m = new MappingExample();
        m.update(100);
        return m.balances(this);
    }
}
~~~~

<b>Note</b>
<br>매핑은 반복 가능하지 않지만, 그 위에 데이터 구조를 구현할 수 있습니다. 예를 들어 [반복 가능한 매핑](https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol)을 참조하십시오.
<br><b>Note End</b>

### Operators Involving LValues

<tt style="color: #FF0000">`a`</tt>가 LValue(즉, 할당 할 수 있는 변수 또는 무언가)이면 다음 연산자를 약자로 사용할 수 있습니다.

<tt style="color: #FF0000">`a += e`</tt>는 <tt style="color: #FF0000">`a = a + e`</tt>와 같습니다. 
<tt style="color: #FF0000">`-=`</tt>, <tt style="color: #FF0000">`*=`</tt>, <tt style="color: #FF0000">`/=`</tt>, <tt style="color: #FF0000">`%=`</tt>, <tt style="color: #FF0000">`|=`</tt>, <tt style="color: #FF0000">`&=`</tt> 및 <tt style="color: #FF0000">`^=`</tt> 연산자가 그에 따라 정의됩니다. 
<tt style="color: #FF0000">`a++`</tt> 및 <tt style="color: #FF0000">`a--`</tt>는 <tt style="color: #FF0000">`a + = 1`</tt> / <tt style="color: #FF0000">`a - = 1`</tt>과 동일하지만 표현식 자체는 여전히 <tt style="color: #FF0000">`a`</tt>의 이전 값을 가집니다. 
반대로 <tt style="color: #FF0000">`--a`</tt>와 <tt style="color: #FF0000">`++a`</tt>는 a에 동일한 효과를 가지지만 변경된 후에는 값을 반환합니다.

#### delete

<tt style="color: #FF0000">`delete a`</tt>는 해당 유형의 초기 값을 <tt style="color: #FF0000">`a`</tt>에 할당합니다. 정수의 경우 <tt style="color: #FF0000">`a = 0`</tt>과 동일하지만 배열에서 사용할 수도 있습니다. 
여기서는 길이가 0인 동적 배열 또는 동일한 길이의 정적 배열을 모든 요소를 재설정하여 할당합니다. 
struct의 경우 모든 멤버가 재설정된 struct를 할당합니다.

<tt style="color: #FF0000">`delete`</tt>는 전체 매핑에 아무런 영향을 주지 않습니다(매핑 키는 임의적이며 일반적으로 알 수 없기 때문에). 
따라서 struct를 삭제하면 매핑이 아닌 모든 멤버가 재설정되고 매핑되지 않은 경우 멤버로 다시 전달됩니다. 
그러나 개별 키와 이 키와 매핑되는 키는 삭제할 수 있습니다.

<tt style="color: #FF0000">`delete a`</tt>는 실제로 <tt style="color: #FF0000">`a`</tt>에 대한 할당과 같이 동작합니다. 즉, <tt style="color: #FF0000">`a`</tt>에 새 객체를 저장합니다.

~~~~
pragma solidity ^0.4.0;

contract DeleteExample {
    uint data;
    uint[] dataArray;

    function f() public {
        uint x = data;
        delete x; // sets x to 0, does not affect data
        delete data; // sets data to 0, does not affect x which still holds a copy
        uint[] storage y = dataArray;
        delete dataArray; // this sets dataArray.length to zero, but as uint[] is a complex object, also
        // y is affected which is an alias to the storage object
        // On the other hand: "delete y" is not valid, as assignments to local variables
        // referencing storage objects can only be made from existing storage objects.
    }
}
~~~~

### Conversions between Elementary Types

#### Implicit Conversions

연산자가 다른 유형에 적용되면 컴파일러는 피연산자 중 하나를 다른 유형으로 암시적으로 변환하려고 시도합니다(할당은 동일함). 
의미적으로 의미가 있고 손실되는 정보가없는 경우 일반적으로 값 유형 간의 암시적 변환이 가능합니다. 
<tt style="color: #FF0000">`uint8`</tt>은 <tt style="color: #FF0000">`uint16`</tt> 및 <tt style="color: #FF0000">`int128`</tt>에서 <tt style="color: #FF0000">`int256`</tt>으로 변환 가능하지만 <tt style="color: #FF0000">`int8`</tt>은 <tt style="color: #FF0000">`uint256`</tt>으로 변환 할 수 없습니다(<tt style="color: #FF0000">`uint256`</tt>은 예를 들어 <tt style="color: #FF0000">`-1`</tt>을 보유할 수 없기 때문입니다). 
게다가 부호없는 정수는 같거나 큰 크기의 바이트로 변환 될 수 있지만 그 반대는 불가능합니다. 
<tt style="color: #FF0000">`uint160`</tt>으로 변환 할 수 있는 모든 유형은 <tt style="color: #FF0000">`address`</tt>로 변환할 수도 있습니다.

#### Explicit Conversions

컴파일러가 암시적 변환을 허용하지 않지만 수행중인 작업을 알고 있는 경우 명시적 유형 변환이 가능할 수 있습니다. 
이렇게하면 예기치 않은 동작이 발생할 수 있으므로 원하는 결과가 나오는지 테스트해야 합니다. 
다음 예제에서 음수 <tt style="color: #FF0000">`int8`</tt>을 <tt style="color: #FF0000">`uint`</tt>로 변환합니다.

~~~~
int8 y = -3;
uint x = uint(y);
~~~~

이 코드 스니펫이 끝날 때 <tt style="color: #FF0000">`x`</tt>는 <tt style="color: #FF0000">`0xfffff..fd`</tt>(64진수 문자)값을 가지며, 이는 256비트의 [2의 보수 표현](https://www.joinc.co.kr/w/Site/Assembly/Documents/Spim/spim-chapter8-1)에서 -3입니다.

형식이 명시적으로 더 작은 형식으로 변환되면 상위 비트가 잘립니다.

~~~~
uint32 a = 0x12345678;
uint16 b = uint16(a); // b will be 0x5678 now
~~~~

### Type Deduction

편의상 항상 변수의 유형을 명시적으로 지정할 필요는 없지만 컴파일러는 변수에 할당된 첫 번째 표현식의 유형에서 자동으로 이를 유추합니다.

~~~~
uint24 x = 0x123;
var y = x;
~~~~

여기에서 <tt style="color: #FF0000">`y`</tt>의 유형은 <tt style="color: #FF0000">`uint24`</tt>입니다. 함수 매개 변수나 반환 매개 변수에서는 <tt style="color: #FF0000">`var`</tt>를 사용할 수 없습니다.

<b>Warning</b>
<br>첫 번째 할당에서만 유형이 추론되므로 다음 스니펫의 루프에서 <tt style="color: #FF0000">`i`</tt>는 <tt style="color: #FF0000">`uint8`</tt> 유형을 가지며 이 유형의 가장 높은 값이 <tt style="color: #FF0000">`2000`</tt>보다 작기 때문에 무한합니다. <tt style="color: #FF0000">`for (var i = 0; i <2000; i ++) {...}`</tt>
<br><b>Warning End</b>