---
layout: post
title:  "Python TDD-Data_Structure_Stack"
date:   2017-11-13 10:43:59
author: Dean Kim
categories: Algorithm
tags:	Python TDD Algorithm Stack
cover:  "/assets/instacode.png"
---

# Stack
- 설명 : [공식문서](https://ko.wikipedia.org/wiki/%EC%8A%A4%ED%83%9D)

<tt style="color: #FF0000">`Stack`</tt>은 제한적으로 접근할 수 있는 나열 구조입니다. 접근은 항상 목록의 끝에서만 일어납니다.
그래서 한 쪽 끝에서만 자료를 넣거나 뺄 수 있는 선형 구조로 되어있습니다. (LIFO - Last In First Out)
자료를 넣는 것은 <tt style="color: #FF0000">`Push`</tt>, 자료를 꺼내는 것을 <tt style="color: #FF0000">`Pop`</tt>이라고 합니다.
자료를 꺼낼 때에는 가장 최근에 저장한 자료부터 가져오게 됩니다.
자세한 설명은 위의 위키백과 링크를 참조하시면 됩니다.

이 포스트에서는 Python으로 Stack을 작성할 것이며 TDD로 진행합니다. 때문에 Test 코드가 먼저 작성됨을 말씀드립니다.

## 파일 만들기

이 스터디는 python3.6 버전을 사용합니다.
스터디를 진행하는 디렉토리를 만들고, 그 디렉토리에 tesk_stack.py를 만듭니다.

### 테스트 작성 -1

test_stack.py를 열고, 테스트에 필요한 모듈들을 import 합니다.

~~~~
# in test_stack.py
import unittest
import stack

if __name__ == '__main__':
    unittest.main()
~~~~
<tt style="color: #FF0000">`unittest`</tt>는 Python에서 기본적으로 제공하는 테스트를 위한 모듈입니다.
자세한 설명은 [공식문서](https://docs.python.org/3/library/unittest.html)를 참조하세요.
<tt style="color: #FF0000">`stack`</tt>은 앞으로 작성해서 테스트를 진행할 파일입니다.

import한 unittest를 상속해서 test class를 작성합니다.
~~~~
# in test_stack.py
...
import stack

class TestStack(unittest.TestCase):
    def setUp(self):
        pass

    def tearDown(self):
        pass
...
~~~~
저는 TestStack이라는 이름으로 만들었습니다. setUp, tearDown은 테스트 과정을 도와주는 메소드들인데 이 포스팅에서는 직접적인 연관이 없어 pass로 넘기도록 하겠습니다.

여기까지 작성한 후 터미널에서 python3.6 test_stack.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
Traceback (most recent call last):
  File "data_structure/stack/test_stack.py", line 2, in <module>
    import stack
ModuleNotFoundError: No module named 'stack'
~~~~
아직 stack.py를 만들지 않은 상태로 import를 했으니 당연히 오류가 나옵니다.

### stack 파일 작성 -1

stack.py를 만들어줍니다.
~~~~
# in stack.py
class Stack:
    def __init__(self):
        pass
~~~~

만들었으면 터미널에서 다시 python3.6 test_stack.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~

----------------------------------------------------------------------
Ran 0 tests in 0.000s

OK
~~~~

### 테스트 작성 -2

이제 테스트를 작성할 기본 준비가 되었습니다.

먼저 stack 자료형에서 자료를 담을 storage가 있어야 되는데 이 storage가 있는지 확인하는 테스트를 작성하겠습니다.
~~~~
# in test_stack.py
...
    def tearDown(self):
        pass
        
    def test_can_create_storage(self):
        stack_instance = stack.Stack()
        self.assertEqual(stack_instance.storage, [])
...
~~~~
test_can_create_storage라는 메소드를 만들고, stack instance를 stack_instance라는 변수명으로 만듭니다.
instance 내부에 storage라는 이름을 가진 저장소가 리스트 형식으로 존재하는지 테스트하는 구문을 작성합니다.
<tt style="color: #FF0000">`assertEqual(first, second, msg=None)`</tt>은 first, second 인자로 넘어온 값이 같은지를 검사, 같지 
않을 때 기본 메세지를 반환하고 테스트 케이스 실행을 종료합니다. msg인자를 전달하면 테스트 실패 시 msg 인자에 전달한 값이 반환됩니다.

여기까지 작성한 후 터미널에서 python3.6 test_stack.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~

======================================================================
ERROR: test_can_create_storage (__main__.TestStack)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "data_structure/stack/test_stack.py", line 17, in test_can_create_storage
    self.assertEqual(stack_instance.storage, [])
AttributeError: 'Stack' object has no attribute 'storage'

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (errors=1)

~~~~

이전에 작성한 stack.py에서 storage를 작성하지 않아 storage를 찾을 수 없다고 나오는 것을 확인할 수 있습니다.
이제 stack.py를 작성하러 갑니다.

### stack 파일 작성 -2

stack.py를 열어 다음을 추가합니다.
~~~~
# in stack.py
class Stack:
    def __init__(self):
        self.storage = []
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_stack.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.

~~~~
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
~~~~
storage가 리스트 형식으로 잘 만들어졌다는 것을 확인할 수 있습니다.

### 테스트 작성 -3

이제 stack이 갖고 있는 메소드들을 만드는 단계로 넘어갑니다.
앞서 설명했듯이 stack에는 자료를 저장하는 <tt style="color: #FF0000">`Push`</tt>, 자료를 꺼내는 <tt style="color: #FF0000">`Pop`</tt>
2가지의 메소드를 갖고 있습니다.
우선 <tt style="color: #FF0000">`Push`</tt>에 대한 테스트를 작성하겠습니다.
~~~~
# in test_stack.py
...
    def test_can_save_data_by_push_method(self):
        stack_instance = stack.Stack()
        stack_instance.push(1)
        self.assertEqual(len(stack_instance.storage), 1)
...
~~~~
<tt style="color: #FF0000">`assertEqual`</tt>을 이용해서 자료가 잘 들어갔는지 확인하는 로직을 작성합니다.
여기까지 작성한 후 터미널에서 python3.6 test_stack.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
.E
======================================================================
ERROR: test_can_save_data_by_push_method (__main__.TestStack)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "data_structure/stack/test_stack.py", line 22, in test_can_save_data_by_push_method
    stack_instance.push(1)
AttributeError: 'Stack' object has no attribute 'push'

----------------------------------------------------------------------
Ran 2 tests in 0.000s
~~~~
오류 메세지를 확인했으니 이를 해결할 코드를 stack.py에 작성하러 갑니다.

### stack 파일 작성 -3

~~~~
# in stack.py
...
        self.storage = []
        
    def push(self, input_item):
        self.storage.append(input_item)
~~~~
먼저 <tt style="color: #FF0000">`push`</tt> 메소드를 정의하고, input_item을 받아서 storage에 저장하는 로직을 작성합니다.
여기까지 작성한 후 터미널에서 python3.6 test_stack.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
..
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
~~~~
<tt style="color: #FF0000">`Push`</tt> 메소드가 잘 작동하는 것을 확인할 수 있습니다.

### 테스트 작성 -4

자료를 꺼내는 <tt style="color: #FF0000">`pop`</tt> 메소드를 작성해볼까요?
방법은 <tt style="color: #FF0000">`push`</tt>를 작성할 때와 같습니다.
~~~~
# in test_stack.py
...
        self.assertEqual(len(stack_instance.storage), 1)
        
    def test_can_return_data_by_pop_method(self):
        stack_instance = stack.Stack()
        stack_instance.push(1)
        stack_instance.push(2)
        output = stack_instance.pop()
        self.assertEqual(output, 2)
        self.assertEqual(len(stack_instance.storage), 1)
        
    def test_can_return_error_when_try_to_pop_method_empty_storage(self):
        stack_instance = stack.Stack()
        self.assertEqual(stack_instance.pop(), 'storage is empty!')
...
~~~~
위에 작성한 <tt style="color: #FF0000">`push`</tt> 메소드를 이용해서 1, 2를 순서대로 저장한 후 나중에 저장된 2가 반환되는지 확인하는 로직을 작성하고,
반환된 2는 storage에 남아있으면 안되므로 이것을 확인하는 로직도 추가합니다.
그리고 storage가 비어있을 경우에는 storage가 비어있다고 알려주는 것에 대한 오류메세지 처리에 대한 테스트 코드도 작성합니다.
여기서는 storage가 비어있다는 메세지가 필요한 만큼 <tt style="color: #FF0000">`assertEqual`</tt>에 오류 메세지 'storage is empty!'을 넣어
확인하는 테스트를 작성합니다.

여기까지 작성한 후 터미널에서 python3.6 test_stack.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
.E.
======================================================================
ERROR: test_can_return_data_by_pop_method (__main__.TestStack)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "data_structure/stack/test_stack.py", line 30, in test_can_return_data_by_pop_method
    output = stack_instance.pop()
AttributeError: 'Stack' object has no attribute 'pop'

----------------------------------------------------------------------
Ran 3 tests in 0.001s
~~~~
오류 메세지를 확인했으니 이를 해결할 코드를 stack.py에 작성하러 갑니다.

### stack 파일 작성 -4

~~~~
# in stack.py
...
    def pop(self):
        if not len(self.storage) == 0:
            return self.storage.pop()
        elif len(self.storage) == 0:
            return 'storage is empty!'
~~~~
먼저 <tt style="color: #FF0000">`pop`</tt> 메소드를 정의하고, input_item을 받아서 storage에 저장하는 로직을 작성합니다.
그리고 storage가 비어있을 경우 'storage is empty!' 메세지를 반환하도록 작성합니다.
여기까지 작성한 후 터미널에서 python3.6 test_stack.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
....
----------------------------------------------------------------------
Ran 4 tests in 0.000s

OK
~~~~

### 테스트 작성 -5

이제 마지막으로 자료가 얼마나 저장되어 있는지를 확인하는 <tt style="color: #FF0000">`size`</tt> 메소드를 작성합니다.
~~~~
# in test_stack.py
...
        self.assertEqual(stack_instance.pop(), 'storage is empty!')
        
    def test_can_return_size_by_size_method(self):
        stack_instance = stack.Stack()
        stack_length = stack_instance.size()
        self.assertEqual(stack_length, 0)
        stack_instance.push(1)
        stack_length_after_push = stack_instance.size()
        self.assertEqual(stack_length_after_push, 1)
        stack_instance.pop()
        stack_length_after_pop = stack_instance.size()
        self.assertEqual(stack_length_after_pop, 0)
...
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_stack.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
...E.
======================================================================
ERROR: test_can_return_size_by_size_method (__main__.TestStack)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "data_structure/stack/test_stack.py", line 42, in test_can_return_size_by_size_method
    stack_length = stack_instance.size()
AttributeError: 'Stack' object has no attribute 'size'

----------------------------------------------------------------------
Ran 5 tests in 0.001s
~~~~

### stack 파일 작성 -5

이제 마지막입니다.
storage를 리스트 형식으로 정의했으니 storage의 size는 <tt style="color: #FF0000">`len()`</tt>을 사용해 쉽게 확인할 수 있습니다.
~~~~
# in stack.py
...
     def size(self):
        return len(self.storage)
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_stack.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
.....
----------------------------------------------------------------------
Ran 5 tests in 0.000s

OK
~~~~

지금까지 Python의 unittest를 이용해서 <tt style="color: #FF0000">`Stack`</tt>이라는 자료구조를 Python으로 구현해보았습니다.

개발을 공부하며 TDD가 중요하다는 말을 많이 듣기만 했지 실제로는 경험하지 못하고 있었는데 회사의 새로운 기능들과 서비스를 구현하며 조금씩 테스트 기반으로 
개발을 하다보니 테스트의 중요성에 대해 깨닫게 되었습니다.
Python을 배운지 6개월 정도가 되었는데 짧은 시간 동안에 놓친 부분들이 많은 것 같아 알고리즘의 기초가 될 수 있는 자료구조를 TDD 방식으로 작성하면 언어의 
기본도 돌아볼 수 있고, 테스트 코드 작성에 익숙해질 것이라고 생각해서 앞으로 몇 가지의 알고리즘을 TDD로 작성하고자 합니다.

제가 공부한 것이 다른 분들께도 도움이 되기를 바라며 이 포스팅을 마칩니다.
실제 코드들은 작성이 완료되는데로 제 github repo에 공유하겠습니다.

읽어주셔서 감사합니다.