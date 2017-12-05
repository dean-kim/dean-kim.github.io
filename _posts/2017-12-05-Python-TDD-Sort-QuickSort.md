---
layout: post
title:  "Python TDD-Sort-Quicksort"
date:   2017-12-05 21:43:59
author: Dean Kim
categories: Algorithm
tags:	Python TDD Algorithm Quicksort
cover:  "/assets/instacode.png"
---

# Quick sort
- 설명 : [위키백과](https://ko.wikipedia.org/wiki/%ED%80%B5_%EC%A0%95%EB%A0%AC)

<tt style="color: #FF0000">`Quicksort`</tt>가 어떻게 작동하는지 궁금하신 분들은 다음의 링크에서 애니메이션으로 확인하실 수 있습니다.
코드와 함께 진행하는 모습을 확인할 수 있어서 저처럼 기초가 부족한 분들은 이해하시는데 도움이 되실 겁니다.
[퀵소트의 이해를 도와드립니다.](https://visualgo.net/en/sorting)

이 포스트에서는 Python으로 Quicksort를 작성할 것이며 TDD로 진행합니다. 때문에 Test 코드가 먼저 작성됨을 말씀드립니다.

## 파일 만들기

이 스터디는 python3.6 버전을 사용합니다.
스터디를 진행하는 디렉토리를 만들고, 그 디렉토리에 test_python_quicksort.py를 만듭니다.

### 테스트 작성 -1

test_python_quicksort.py를 열고, 테스트에 필요한 모듈들을 import 합니다.
~~~~
# in test_python_quicksort.py
import unittest
import python_quicksort

if __name__ == '__main__':
    unittest.main()
~~~~
<tt style="color: #FF0000">`unittest`</tt>는 Python에서 기본적으로 제공하는 테스트를 위한 모듈입니다.
<tt style="color: #FF0000">`python_quicksort`</tt>는 앞으로 작성해서 테스트를 진행할 파일입니다.

import한 unittest를 상속해서 test class를 작성합니다.
~~~~
# in test_python_quicksort.py
...
import python_quicksort

class TestPythonQuicksort(unittest.TestCase):
    def setUp(self):
        pass

    def tearDown(self):
        pass
...
~~~~
저는 TestPythonQuicksort라는 이름으로 만들었습니다. setUp, tearDown은 앞선 Stack 포스트에서 설명했으니 참고하시기 바랍니다.

여기까지 작성한 후 터미널에서 python3.6 test_python_quicksort.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~
Traceback (most recent call last):
  File "sort/quick_sort/test_python_quicksort.py", line 2, in <module>
    import python_quicksort
ModuleNotFoundError: No module named 'python_quicksort'
~~~~
아직 python_quicksort.py를 만들지 않은 상태로 import를 했으니 당연히 오류가 나옵니다.

### python_quicksort 파일 작성 -1

python_quicksort.py를 만들어줍니다.
~~~~
# in python_quicksort.py
class Quicksort:
    def __init__(self):
        pass
~~~~
만들었으면 터미널에서 다시 python3.6 test_python_quicksort.py를 실행시키면 다음과 같은 메세지가 나오는 것을 확인할 수 있습니다.
~~~~

----------------------------------------------------------------------
Ran 0 tests in 0.000s

OK
~~~~

### 테스트 작성 -2


