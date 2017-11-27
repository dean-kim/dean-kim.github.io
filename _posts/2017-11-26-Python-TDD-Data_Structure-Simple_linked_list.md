---
layout: post
title:  "Python TDD-Data_Structure-Simple_linked_list"
date:   2017-11-26 20:43:59
author: Dean Kim
categories: Algorithm
tags:	Python TDD Algorithm Simple linked list
cover:  "/assets/instacode.png"
---

# Simple linked list
- 설명 : [위키백과](https://ko.wikipedia.org/wiki/%EC%97%B0%EA%B2%B0_%EB%A6%AC%EC%8A%A4%ED%8A%B8)

<tt style="color: #FF0000">`Linked List`</tt>는 각 노드가 데이터와 포인터를 가지고 한 줄로 연결되어 있는 바식으로 데이터를 저장하는 자료 구조입니다.
이름에서 알 수 있듯이 데이터를 담고 있는 노드들이 연결되어 있는데, 노드의 포인터가 다음이나 이전의 노드와의 연결을 담당합니다.

<tt style="color: #FF0000">`Linked List`</tt>는 연결 방식에 따라 구분할 수 있습니다.
* <tt style="color: #FF0000">`Simple(Singly) linked list`</tt>
* <tt style="color: #FF0000">`Doubly linked list`</tt>
* <tt style="color: #FF0000">`Multiply linked list`</tt>
* <tt style="color: #FF0000">`Circular linked list`</tt>
연결 방식에 따른 <tt style="color: #FF0000">`Linked List`</tt>들에 대한 설명은 [위키백과_영어버전](https://en.wikipedia.org/wiki/Linked_list)에서 확인 하실 수 있습니다.

이 포스트에서는 Python으로 Simple(Singly) linked list를 작성할 것이며 TDD로 진행합니다. 때문에 Test 코드가 먼저 작성됨을 말씀드립니다.

Simple(Singly) linked list는 정보를 저장하는 노드와 바로 다음의 노드를 가리키는 링크 하나로 구성되어 있습니다.

Simple(Singly) linked list는 노드정의, 리스트 생성, 노드삽입, 노드삭제, 리스트 출력의 5개의 테스트를 기반으로 작성될 예정입니다.

## 파일 만들기

이 스터디는 python3.6 버전을 사용합니다.
스터디를 진행하는 디렉토리를 만들고, 그 디렉토리에 test_simple_linked_list.py를 만듭니다.

### 테스트 작성 -1

test_queue.py를 열고, 테스트에 필요한 모듈들을 import 합니다.
처음에는 queue를 import하려고 했으나 Python에 queue가 있는 관계로 python_queue라는 이름으로 진행하겠습니다.
~~~~
# in test_simple_linked_list.py
import unittest
import simple_linked_list

if __name__ == '__main__':
    unittest.main()
~~~~
<tt style="color: #FF0000">`unittest`</tt>는 Python에서 기본적으로 제공하는 테스트를 위한 모듈입니다.
자세한 설명은 [공식문서](https://docs.python.org/3/library/unittest.html)를 참조하세요.
<tt style="color: #FF0000">`simple_linked_list`</tt>는 앞으로 작성해서 테스트를 진행할 파일입니다.

import한 unittest를 상속해서 test class를 작성합니다.
~~~~
# in test_simple_linked_list.py
...
import simple_linked_list

class NodeTest(unittest.TestCase):
    def setUp(self):
        pass

    def tearDown(self):
        pass
...
~~~~
저는 NodeTest라는 이름으로 만들었습니다. Node의 생성 여부를 테스트하는 테스트입니다. setUp, tearDown은 앞선 Stack 포스트에서 설명했으니 참고하시기 바랍니다.

여기까지 작성한 후 터미널에서 python3.6 test_simple_linked_list.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
Traceback (most recent call last):
  File "data_structure/simple_linked_list/test_simple_linked_list.py", line 2, in <module>
    import simple_linked_list
ModuleNotFoundError: No module named 'simple_linked_list'
~~~~
아직 python_queue.py를 만들지 않은 상태로 import를 했으니 당연히 오류가 나옵니다.

### simple_linked_list 파일 작성 -1

simple_linked_list.py를 만들어 줍니다.
~~~~
# in simple_linked_list.py
class Node:
    def __init__(self, data, next=None):
        pass
~~~~

만들었으면 터미널에서 다시 python3.6 test_simple_linked_list.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~

----------------------------------------------------------------------
Ran 0 tests in 0.000s

OK
~~~~

### 테스트 작성 -2

먼저 simple_linked_list 자료형에서 자료를 담을 <tt style="color: #FF0000">`node`</tt>가 있어야 되는데 이 node가 있는지 확인하는 테스트를 작성하겠습니다.

node에는 데이터 저장하는 곳과 다음 node를 가리키는 포인터를 저장하는 곳이 있어야 합니다.

저는 임의로 저장하는 곳의 <tt style="color: #FF0000">`key`</tt>를 <tt style="color: #FF0000">`data`</tt>로, 다음 node를 가리키는 곳의<tt style="color: #FF0000">`key`</tt>를 <tt style="color: #FF0000">`next`</tt>로 해서 만들어 보겠습니다.
~~~~
# in test_simple_linked_list.py
...
    def test_can_create_node_instance(self):
        node_instance = simple_linked_list.Node(3)
        node = {"data": 3, "next": None}
        self.assertEqual(node_instance.data, node["data"])
        self.assertEqual(node_instance.next, None)
...
~~~~
test_can_create_node_instance라는 메소드를 만들고, node instance를 node_instance라는 변수명으로 만듭니다.
simple_linked_list의 Node class 를 이용해서 node_instance를 생성하고, data로 3을 넘겨줍니다. 만들어진 node_instance는 1개 이므로 다음 node를 가리키는 값은 None이 됩니다.
assertEqual을 이용해서 node_instance의 data와 next에 올바른 값이 들어갔는지 확인하는 구문을 작성합니다.
<tt style="color: #FF0000">`assertEqual(first, second, msg=None)`</tt>은 first, second 인자로 넘어온 값이 같은지를 검사, 같지 
않을 때 기본 메세지를 반환하고 테스트 케이스 실행을 종료합니다. msg인자를 전달하면 테스트 실패 시 msg 인자에 전달한 값이 반환됩니다.

여기까지 작성한 후 터미널에서 python3.6 test_simple_linked_list.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
E
======================================================================
ERROR: test_can_create_node_instance (__main__.NodeTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "data_structure/simple_linked_list/test_simple_linked_list.py", line 22, in test_can_create_node_instance
    self.assertEqual(node_instance.data, node["data"])
AttributeError: 'Node' object has no attribute 'data'

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
~~~~
이전에 작성한 simple_linked_list.py에서 Node class에서 data와 next를 작성하지 않아 찾을 수 없다고 나오는 것을 확인할 수 있습니다.
이제 simple_linked_list.py를 작성하러 갑니다.

### simple_linked_list 파일 작성 -2

~~~~
# in simple_linked_list.py
class Node:
    def __init__(self, data, next=None):
        self.data = data
        self.next = next
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_simple_linked_list.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
~~~~

### 테스트 작성 -3

simple linked list의 첫 번체 node는 head라고 칭합니다. simple linked list를 생성할 때 head가 생성여부를 테스트 합니다.
~~~~
# in test_simple_linked_list.py
...
    def test_can_create_linked_list_head(self):
        linked_list_instance = simple_linked_list.LinkedList()
        self.assertEqual(linked_list_instance.head, None)
...
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_simple_linked_list.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
E.
======================================================================
ERROR: test_can_create_linked_list_head (__main__.LinkedListTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "data_structure/simple_linked_list/test_simple_linked_list.py", line 34, in test_can_create_linked_list_head
    linked_list_instance = simple_linked_list.LinkedList()
AttributeError: module 'simple_linked_list' has no attribute 'LinkedList'

----------------------------------------------------------------------
Ran 2 tests in 0.001s

FAILED (errors=1)
~~~~
LinkedList class를 작성하지 않아 찾을 수 없다는 메세지가 출력됩니다 simple_linked_list.py로 돌아가 LinkedList class를 작성하고, 처음 생성 시 head를 생성하도록 작성합니다.

### simple_linked_list 파일 작성 -3

~~~~
# in simple_linked_list.py
...
    def __init__(self, data, next=None):
        pass
        
class LinkedList:
    def __init__(self):
        self.head = None
        self.size = 0
~~~~
size는 LinkedList에 몇 개의 자료가 있는 가를 보여주는 요소입니다.
여기까지 작성한 후 터미널에서 python3.6 test_simple_linked_list.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
..
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
~~~~

### 테스트 작성 -4

이번에는 LinkedList에 자료가 잘 들어가는지에 대한 테스트입니다.
자료를 저장할 때는 새로운 node를 생성하고, 생성된 node의 next에 기존의 node를 저장합니다.
~~~~
# in test_simple_linked_list.py
...
    def test_can_insert_data_linked_list(self):
        linked_list_instance = simple_linked_list.LinkedList()
        linked_list_instance.insert(3)
        self.assertEqual(linked_list_instance.head.data, 3)
        self.assertEqual(linked_list_instance.head.next, None)
...
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_simple_linked_list.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
.E.
======================================================================
ERROR: test_can_insert_data_linked_list (__main__.LinkedListTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "data_structure/simple_linked_list/test_simple_linked_list.py", line 39, in test_can_insert_data_linked_list
    linked_list_instance.insert(3)
AttributeError: 'LinkedList' object has no attribute 'insert'

----------------------------------------------------------------------
Ran 3 tests in 0.006s

FAILED (errors=1)
~~~~
LinkedList에 insert가 없다는 메세지가 나오는군요, simple_linked_list.py에 insert를 작성하러 갑시다.

### simple_linked_list 파일 작성 -4

~~~~
# in simple_linked_list.py
...
        self.size = 0
        
    def insert(self, data):
        new_node = Node(data)
        new_node.next = self.head
        self.head = new_node
        self.size += 1
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_simple_linked_list.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
...
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
~~~~

### 테스트 작성 -5

이번에는 data를 삭제하는 <tt style="color: #FF0000">`delete`</tt> 메소드에 대한 테스트를 작성하겠습니다.
특정 데이터를 삭제할 경우 해당 데이터를 담고 있는 node와 다음 node와의 관계성을 고려해서 삭제하는 로직을 만들어야 합니다.
이 포스팅에서는 simple linked list를 알고 있다는 전제로 작성하는 관계로 이에 대한 자세한 설명은 포스트 상단의 위키백과를 참조하시기 바랍니다.
일단 <tt style="color: #FF0000">`1, 2, 3, 4`</tt>를 <tt style="color: #FF0000">`insert`</tt> 메소드로 저장한 후 <tt style="color: #FF0000">`3`</tt>를 <tt style="color: #FF0000">`delete_node`</tt> 메소드로 삭제하는 테스트를 작성합니다.
<tt style="color: #FF0000">`1, 2, 3, 4`</tt>는 <tt style="color: #FF0000">`4-3-2-1`</tt>의 형태로 저장되며 <tt style="color: #FF0000">`3`</tt> 삭제 시 <tt style="color: #FF0000">`4-2-1`</tt>의 형태가 되어야 합니다.
그렇다면 <tt style="color: #FF0000">`3`</tt>을 삭제하면서 <tt style="color: #FF0000">`4`</tt>의 <tt style="color: #FF0000">`next`</tt>는 <tt style="color: #FF0000">`3`</tt>에서 <tt style="color: #FF0000">`2`</tt>로 변경되어야 합니다.
~~~~
# in test_simple_linked_list.py
...
        self.size += 1
        
    def test_can_delete_data_in_linked_list(self):
        linked_list_instance = simple_linked_list.LinkedList()
        linked_list_instance.insert(1)
        linked_list_instance.insert(2)
        linked_list_instance.insert(3)
        linked_list_instance.insert(4)
        self.assertEqual(linked_list_instance.head.next.data, 3)
        linked_list_instance.delete_node(3)
        self.assertEqual(linked_list_instance.head.next.data, 2)
        self.assertEqual(linked_list_instance.size, 3)
...
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_simple_linked_list.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
.E..
======================================================================
ERROR: test_can_delete_data_in_linked_list (__main__.LinkedListTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "data_structure/simple_linked_list/test_simple_linked_list.py", line 53, in test_can_delete_data_in_linked_list
    linked_list_instance.delete_node(3)
AttributeError: 'LinkedList' object has no attribute 'delete_node'

----------------------------------------------------------------------
Ran 4 tests in 0.001s

FAILED (errors=1)
~~~~
LinkedList에 delete_node가 없다는 메세지가 나오는군요, simple_linked_list.py에 delete_node를 작성하러 갑시다.

### simple_linked_list 파일 작성 -5

~~~~
# in simple_linked_list.py
...
        self.size += 1
        
    def delete_node(self, data):
        current_node = self.head
        found = False
        prev_node = None

        while current_node != None and found is False:
            if current_node.data == data:
                found = True
            else:
                prev_node = current_node
                current_node = current_node.next

        if current_node is None:
            print(data + "not found")

        elif prev_node == None:
            self.head = current_node.next
        else:
            prev_node.next = current_node.next
        
        del current_node
        self.size -= 1
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_simple_linked_list.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
....
----------------------------------------------------------------------
Ran 4 tests in 0.000s

OK
~~~~

### 테스트 작성 -6

마지막으로 작성된 simple linked list를 출력하는 테스트를 작성하겠습니다.
~~~~
# in test_simple_linked_list.py
...
    def test_can_print_linked_list(self):
        linked_list_instance = simple_linked_list.LinkedList()
        linked_list_instance.insert(1)
        linked_list_instance.insert(2)
        linked_list_instance.insert(3)
        linked_list_instance.insert(4)
        self.assertEqual(linked_list_instance.print_linked_list(), '4>>3>>2>>1')
...
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_simple_linked_list.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
...E.
======================================================================
ERROR: test_can_print_linked_list (__main__.LinkedListTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "data_structure/simple_linked_list/test_simple_linked_list.py", line 61, in test_can_print_linked_list
    self.assertEqual(linked_list_instance.print_linked_list(), '4>>3>>2>>1')
AttributeError: 'LinkedList' object has no attribute 'print_linked_list'

----------------------------------------------------------------------
Ran 5 tests in 0.001s

FAILED (errors=1)
~~~~
LinkedList에 print_linked_list가 없다는 메세지가 나오는군요, simple_linked_list.py에 print_linked_list를 작성하러 갑시다.

### simple_linked_list 파일 작성 -6

작성된 simple linked list가 어떤 순서로 연결되어 있는지 보여주는 print_linked_list 메소드를 작성하겠습니다.
각 데이터 사이에는 '>>'를 넣어 각 node가 어떻게 연결되어 있는지 파악할 수 있도록 작성하겠습니다.
~~~~
# in simple_linked_list.py
...
        self.size -= 1
        
    def print_linked_list(self):
        temp = self.head
        result = []
        link_arrow = ">>"
        while temp:
            result.append(str(temp.data))
            temp = temp.next
        return link_arrow.join(result)
        
    def size(self):
        return self.size
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_simple_linked_list.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
.....
----------------------------------------------------------------------
Ran 5 tests in 0.000s

OK
~~~~

### 테스트 작성 -7

마지막으로 simple linked list의 크기를 알아보는 size 메소드를 작성하겠습니다.
~~~~
# in test_simple_linked_list.py
...
    def test_can_return_of_linked_list(self):
        linked_list_instance = simple_linked_list.LinkedList()
        linked_list_instance.insert(1)
        linked_list_instance.insert(2)
        linked_list_instance.insert(3)
        linked_list_instance.insert(4)
        self.assertEqual(linked_list_instance.size, 4)
...
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_simple_linked_list.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
......
----------------------------------------------------------------------
Ran 6 tests in 0.000s

OK
~~~~

테스트가 모두 통과했습니다. size는 초기에 정의한 변수 <tt style="color: #FF0000">`size`</tt>를 확인만 하면 되기에 따로 메소드를 작성하지는 않았습니다.
simple linked list를 TDD로 작성하는 과정이 모두 끝났습니다.

지금까지 Python의 unittest를 이용해서 <tt style="color: #FF0000">`Simple linked list`</tt>라는 자료구조를 Python으로 구현해보았습니다.

개발을 공부하며 TDD가 중요하다는 말을 많이 듣기만 했지 실제로는 경험하지 못하고 있었는데 회사의 새로운 기능들과 서비스를 구현하며 조금씩 테스트 기반으로 
개발을 하다보니 테스트의 중요성에 대해 깨닫게 되었습니다.
Python을 배운지 6개월 정도가 되었는데 짧은 시간 동안에 놓친 부분들이 많은 것 같아 알고리즘의 기초가 될 수 있는 자료구조를 TDD 방식으로 작성하면 언어의 
기본도 돌아볼 수 있고, 테스트 코드 작성에 익숙해질 것이라고 생각해서 앞으로 몇 가지의 알고리즘을 TDD로 작성하고자 합니다.

제가 공부한 것이 다른 분들께도 도움이 되기를 바라며 이 포스팅을 마칩니다.
실제 코드들은 작성이 완료되는데로 제 github repo에 공유하겠습니다.

읽어주셔서 감사합니다.

다음에는 Graph에 대해 작성해보겠습니다.