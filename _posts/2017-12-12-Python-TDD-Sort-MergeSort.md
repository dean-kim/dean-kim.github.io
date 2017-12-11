---
layout: post
title:  "Python TDD-Sort-Mergesort"
date:   2017-12-12 21:43:59
author: Dean Kim
categories: Algorithm
tags:	Python TDD Algorithm Mergesort
cover:  "/assets/instacode.png"
---

# Merge sort
- 설명 : [위키백과](https://ko.wikipedia.org/wiki/%ED%95%A9%EB%B3%91_%EC%A0%95%EB%A0%AC)

<tt style="color: #FF0000">`Mergesort`</tt>가 어떻게 작동하는지 궁금하신 분들은 다음의 링크에서 애니메이션으로 확인하실 수 있습니다.
코드와 함께 진행하는 모습을 확인할 수 있어서 저처럼 기초가 부족한 분들은 이해하시는데 도움이 되실 겁니다.
[머지소트의 이해를 도와드립니다.](https://visualgo.net/en/sorting)

이 포스트에서는 Python으로 Mergesort를 작성할 것이며 TDD로 진행합니다. 때문에 Test 코드가 먼저 작성됨을 말씀드립니다.

## 파일 만들기

이 스터디는 python3.6 버전을 사용합니다.
스터디를 진행하는 디렉토리를 만들고, 그 디렉토리에 test_python_mergesort.py를 만듭니다.

### 테스트 작성 -1

test_python_mergesort.py를 열고, 테스트에 필요한 모듈들을 import 합니다.
~~~~
# in test_python_mergesort.py
import unittest
import python_mergesort

if __name__ == '__main__':
    unittest.main()
~~~~
<tt style="color: #FF0000">`unittest`</tt>는 Python에서 기본적으로 제공하는 테스트를 위한 모듈입니다.
<tt style="color: #FF0000">`python_mergesort`</tt>는 앞으로 작성해서 테스트를 진행할 파일입니다.

import한 unittest를 상속해서 test class를 작성합니다.
~~~~
# in test_python_mergesort.py
...
import python_mergesort

class TestPythonMergesort(unittest.TestCase):
    def setUp(self):
        pass

    def tearDown(self):
        pass
...
~~~~
저는 TestPythonMergesort 이름으로 만들었습니다. setUp, tearDown은 앞선 Stack 포스트에서 설명했으니 참고하시기 바랍니다.

여기까지 작성한 후 터미널에서 python3.6 test_python_mergesort.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
Traceback (most recent call last):
  File "sort/merge_sort/test_python_mergesort.py", line 2, in <module>
    import python_mergesort
ModuleNotFoundError: No module named 'python_mergesort'
~~~~
아직 python_mergesort.py를 만들지 않은 상태로 import를 했으니 당연히 오류가 나옵니다.

### python_mergesort 파일 작성 -1

python_mergesort.py를 만들어줍니다.
~~~~
# in python_mergesort.py
class Mergrsort:
    def __init__(self):
        pass
~~~~
만들었으면 터미널에서 다시 python3.6 test_python_mergesort.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~

----------------------------------------------------------------------
Ran 0 tests in 0.000s

OK
~~~~

### 테스트 작성 -2

~~~~
# in test_python_mergesort.py
...
    def tearDown(self):
        pass
        
    def test_sort_correctly(self):
        unsorted_collection = [-11, 0, 5, 2, -20, 100, 2]
        sorted_collection = [-20, -11, 0, 2, 2, 5, 100]
        py_merge_sort = python_mergesort.Mergesort()
        result = py_merge_sort.merge_sort(unsorted_collection)
        self.assertEqual(result, sorted_collection, 'unsorted_collection is not sorted')
...
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_python_mergesort.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
E
======================================================================
ERROR: test_sort_correctly (__main__.TestPythonMergesort)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "sort/merge_sort/test_python_mergesort.py", line 15, in test_sort_correctly
    py_merge_sort = python_mergesort.Mergesort()
AttributeError: module 'python_mergesort' has no attribute 'Mergesort'

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
~~~~
python_mergesort class에 merge_sort가 정의되지 않아 오류 메세지가 나옵니다.

### python_mergesort 파일 작성 -2

python_mergesort.py에 merge_sort 메소드를 정의해줍니다.
~~~~
# in python_mergesort.py
...
    def __init__(self):
        pass
        
    def merge_sort(self, collection):
        if len(collection) < 2:
            return collection

        middle = len(collection)//2
        left = self.merge_sort(collection[:middle])
        right = self.merge_sort(collection[middle:])

        i, j, k = 0, 0, 0

        while i < len(left) and j < len(right):
            if left[i] < right[j]:
                collection[k] = left[i]
                i += 1
            else:
                collection[k] = right[j]
                j += 1
            k += 1

        if i == len(left):
            while j < len(right):
                collection[k] = right[j]
                j += 1
                k += 1

        elif j == len(right):
            while i < len(left):
                collection[k] = left[i]
                i += 1
                k += 1

        return collection
        
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_python_mergesort.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
~~~~

### 테스트 작성 -3

정렬하고자 하는 데이터가 하나의 요소만 갖고 있을 경우 넣은 데이터를 그대로 반환해야 합니다.
~~~~
# in test_python_mergesort.py
...
        self.assertEqual(result, sorted_collection, 'unsorted_collection is not sorted')
        
    def test_if_unsorted_collection_has_only_one_element(self):
        unsorted_collection = [0]
        sorted_collection = [0]
        py_merge_sort = python_mergesort.Mergesort()
        result = py_merge_sort.merge_sort(unsorted_collection)
        self.assertEqual(result, sorted_collection, 'error when unsorted_collection has only one element')
...
~~~~
여기까지 작성한 후 터미널에서 python3.6 test_python_mergesort.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
이미 코드에 반영이 되어있기에 오류가 나오지 않을 것이라 예상됩니다.
~~~~
..
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
~~~~
문제 없이 잘 작동합니다.

다음에는 bubble sort에 대한 테스트 코드를 작성해보겠습니다.

읽어주셔서 감사합니다.