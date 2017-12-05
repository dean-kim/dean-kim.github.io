---
layout: post
title:  "Python TDD-Data_Structure-Graph"
date:   2017-12-03 21:43:59
author: Dean Kim
categories: Algorithm
tags:	Python TDD Algorithm Graph
cover:  "/assets/instacode.png"
---

# Graph
- 설명 : [위키백과](https://en.wikipedia.org/wiki/Graph_(abstract_data_type))

<tt style="color: #FF0000">`Graph`</tt>는 연결할 객체를 나타내는 정점(vertex)과 객체를 연결하는 간선(edge)의 집합으로 구성되어 있습니다.
자세한 내용은 생략하도록 하겠습니다.

이 포스트에서는 Python으로 Queue를 작성할 것이며 TDD로 진행합니다. 때문에 Test 코드가 먼저 작성됨을 말씀드립니다.

## 파일 만들기

이 스터디는 python3.6 버전을 사용합니다.
스터디를 진행하는 디렉토리를 만들고, 그 디렉토리에 test_python_graph.py를 만듭니다.

### 테스트 작성 -1

test_python_graph.py를 열고, 테스트에 필요한 모듈들을 import 합니다.
~~~~
# in test_python_graph.py
import unittest
import python_graph

if __name__ == '__main__':
    unittest.main()
~~~~
<tt style="color: #FF0000">`unittest`</tt>는 Python에서 기본적으로 제공하는 테스트를 위한 모듈입니다.
<tt style="color: #FF0000">`python_graph`</tt>는 앞으로 작성해서 테스트를 진행할 파일입니다.

import한 unittest를 상속해서 test class를 작성합니다.
~~~~
# in test_python_graph.py
...
import python_graph

class TestPythonGraph(unittest.TestCase):
    def setUp(self):
        pass

    def tearDown(self):
        pass
...
~~~~
저는 TestPythonGraph라는 이름으로 만들었습니다. setUp, tearDown은 앞선 Stack 포스트에서 설명했으니 참고하시기 바랍니다.

여기까지 작성한 후 터미널에서 python3.6 test_python_graph.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
Traceback (most recent call last):
  File "data_structure/graph/test_python_graph.py", line 2, in <module>
    import python_graph
ModuleNotFoundError: No module named 'python_graph'
~~~~
아직 python_graph.py를 만들지 않은 상태로 import를 했으니 당연히 오류가 나옵니다.

### python_graph 파일 작성 -1

python_graph.py를 만들어줍니다.
~~~~
# in python_graph.py
class Graph:
    def __init__(self):
        pass
~~~~
만들었으면 터미널에서 다시 python3.6 test_python_graph.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~

----------------------------------------------------------------------
Ran 0 tests in 0.000s

OK
~~~~

### 테스트 작성 -2

이제 테스트를 작성할 기본 준비가 되었습니다.

먼저 graph 자료형에는 <tt style="color: #FF0000">`vertex`</tt>와 <tt style="color: #FF0000">`edge`</tt>의 집합이 있어야 하는데 각각의 집합은 리스트 자료형, 딕셔너리 자료형으로 정의하겠습니다.
이제 위에 언급한 2개의 잡합의 자료형을 확인하는 테스트를 작성합니다.
~~~~
# in test_python_graph.py
...
        pass

    def test_can_create_vertices_and_adj_list(self):
        graph_instance = python_graph.Graph()
        self.assertEqual(graph_instance.vertices, [])
        self.assertEqual(graph_instance.adj_list, {}) 
...
~~~~
test_can_create_vertices_and_adj_list라는 메소드를 만들고, graph instance를 graph_instance라는 변수명으로 만듭니다.
instance 내부에 vertices라는 이름을 가진 저장소가 리스트 형식으로 존재하는지, adj_list라는 이름을 가진 저장소가 딕셔너리 형식으로 존재하는지 테스트하는 구문을 작성합니다.
<tt style="color: #FF0000">`assertEqual(first, second, msg=None)`</tt>은 first, second 인자로 넘어온 값이 같은지를 검사, 같지 
않을 때 기본 메세지를 반환하고 테스트 케이스 실행을 종료합니다. msg인자를 전달하면 테스트 실패 시 msg 인자에 전달한 값이 반환됩니다.

여기까지 작성한 후 터미널에서 python3.6 test_python_graph.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
E
======================================================================
ERROR: test_can_create_vertices_and_adj_list (__main__.TestPythonGraph)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "data_structure/graph/test_python_graph.py", line 14, in test_can_create_vertices_and_adj_list
    self.assertEqual(graph_instance.vertices, [])
AttributeError: 'Graph' object has no attribute 'vertices'

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
~~~~
이전에 작성한 python_graph.py에서 vertices와 adj_list를 작성하지 않아 verices를 찾을 수 없다고 나오는 것을 확인할 수 있습니다.
이제 python_queue.py를 작성하러 갑니다.

### python_graph 파일 작성 -2

python_graph.py를 열어 다음을 추가합니다.
~~~~
# in python_graph.py
class Graph:
    def __init__(self):
        self.vertices = []
        self.adj_list = {}
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_python_graph.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
~~~~
2개의 집합이 리스트형으로 잘 만들어진 것을 확인할 수 있습니다.

### 테스트 작성 -3

이제 vertex를 vertices 집합에 추가하는 <tt style="color: #FF0000">`add_vertex`</tt> 메소드를 테스트하는 코드를 작성하겠습니다.
~~~~
# in test_python_graph.py
...
        self.assertEqual(graph_instance.adj_list, [])
        
    def test_can_add_vertex_in_vertices_and_add_vertex_index_in_adjList(self):
        graph_instance = python_graph.Graph()
        graph_instance.add_vertex('A')
        graph_instance.add_vertex('B')
        graph_instance.add_vertex('C')
        graph_instance.add_vertex('D')
        graph_instance.add_vertex('E')
        self.assertEqual(graph_instance.vertices, ['A', 'B', 'C', 'D', 'E'])
        self.assertEqual(graph_instance.adj_list['A'], [])
...
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_python_graph.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
E.
======================================================================
ERROR: test_can_add_vertex_in_vertices_and_add_vertex_index_in_adjList (__main__.TestPythonGraph)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "data_structure/graph/test_python_graph.py", line 19, in test_can_add_vertex_in_vertices_and_add_vertex_index_in_adjList
    graph_instance.add_vertex('A')
AttributeError: 'Graph' object has no attribute 'add_vertex'

----------------------------------------------------------------------
Ran 2 tests in 0.001s

FAILED (errors=1)
~~~~
add_vertex 메소드를 작성하지 않아 찾을 수 없다는 오류 메세지가 출력됩니다.
이제 python_graph.py를 작성하러 갑시다.

### python_graph 파일 작성 -3

~~~~
# in python_graph.py
...
        self.adj_list = []
        
    def add_vertex(self, vertex):
        self.vertices.append(vertex)
        self.adj_list[vertex] = []
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_python_graph.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
..
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
~~~~
<tt style="color: #FF0000">`add_vertex`</tt> 메소드가 잘 작동하는 것을 확인할 수 있습니다.

### 테스트 작성 -4

데이터를 추가할 때 기존의 자료와 중복되면 안되겠죠?
중복을 방지하기 위해 추가하려는 데이터가 존재하지 않는 경우에만 데이터를 추가하는 테스트를 먼저 작성합니다.
~~~~
# in test_python_graph.py
...
        self.assertEqual(graph_instance.adj_list['A'], [])
    def test_not_add_vertex_in_vertices_when_vertex_already_exist(self):
        graph_instance = python_graph.Graph()
        graph_instance.add_vertex('A')
        graph_instance.add_vertex('B')
        graph_instance.add_vertex('C')
        graph_instance.add_vertex('C')
        self.assertEqual(graph_instance.vertices, ['A', 'B', 'C'])
...
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_python_graph.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
..F
======================================================================
FAIL: test_not_add_vertex_in_vertices_when_vertex_already_exist (__main__.TestPythonGraph)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "data_structure/graph/test_python_graph.py", line 33, in test_not_add_vertex_in_vertices_when_vertex_already_exist
    self.assertEqual(graph_instance.vertices, ['A', 'B', 'C'])
AssertionError: Lists differ: ['A', 'B', 'C', 'C'] != ['A', 'B', 'C']

First list contains 1 additional elements.
First extra element 3:
'C'

- ['A', 'B', 'C', 'C']
?            -----

+ ['A', 'B', 'C']

----------------------------------------------------------------------
Ran 3 tests in 0.001s

FAILED (failures=1)
~~~~
이제 python_graph.py를 작성하러 갑시다.

### python_graph 파일 작성 -4

<tt style="color: #FF0000">`add_vertex`</tt> 메소드에 추가하려는 데이터의 존재 여부를 판단해서 중복되지 않는 경우 추가하는 코드를 작성합니다.
~~~~
# in python_graph.py
...     
    def add_vertex(self, vertex):
        if vertex not in self.vertices:
            self.vertices.append(vertex)
...
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_python_graph.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
...
----------------------------------------------------------------------
Ran 3 tests in 0.001s

OK
~~~~

### 테스트 작성 -5

이제 vertex간의 관계를 adj_list 집합에 추가하는 <tt style="color: #FF0000">`add_edge`</tt> 메소드를 테스트하는 코드를 작성하겠습니다.
~~~~
# in test_python_graph.py
...
        self.assertEqual(graph_instance.adj_list['A'], [])
        
    def test_can_add_edge_in_adj_list(self):
        graph_instance = python_graph.Graph()
        graph_instance.add_vertex('A')
        graph_instance.add_vertex('B')
        graph_instance.add_vertex('C')
        graph_instance.add_edge('A', 'B')
        graph_instance.add_edge('A', 'C')
        self.assertEqual(graph_instance.adj_list['A'], ['B', 'C'])
...
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_python_graph.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
E..
======================================================================
ERROR: test_can_add_edge_in_adj_list (__main__.TestPythonGraph)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "data_structure/graph/test_python_graph.py", line 32, in test_can_add_edge_in_adj_list
    graph_instance.add_edge('A', 'B')
AttributeError: 'Graph' object has no attribute 'add_edge'

----------------------------------------------------------------------
Ran 3 tests in 0.001s

FAILED (errors=1)
~~~~
add_edge 메소드를 작성하지 않아 찾을 수 없다는 오류 메세지가 출력됩니다.
이제 python_graph.py를 작성하러 갑시다.

### python_graph 파일 작성 -5

~~~~
# in python_graph.py
...
        self.adj_list[vertex] = []
        
    def add_edge(self, start, end):
        self.adj_list[start].append(end)
        self.adj_list[end].append(start)
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_python_graph.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
...
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
~~~~
<tt style="color: #FF0000">`add_edge`</tt> 메소드가 잘 작동하는 것을 확인할 수 있습니다.

지금까지 Python의 unittest를 이용해서 <tt style="color: #FF0000">`Graph`</tt>라는 자료구조를 Python으로 구현해보았습니다.

다음 포스트에서는 <tt style="color: #FF0000">`Sorting`</tt> 알고리즘을 TDD로 작성하겠습니다.