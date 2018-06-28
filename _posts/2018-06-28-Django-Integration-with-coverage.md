---
layout: post
title:  "Django Integration with coverage.py"
date:   2018-06-28 17:21:00
author: Dean Kim
categories: Django
tags:	Django TDD Test Coverage
cover:  "/assets/instacode.png"
---

# Django Integration with coverage.py
- 참고 : [공식문서](https://coverage.readthedocs.io/en/coverage-4.5.1a/)

회사의 업무와 개인 프로젝트에서 개발하는 방법론 <tt style="color: #FF0000">`TDD(Test-Driven Development)`</tt>를 사용하고 있습니다.

작성한 테스트에서 빠진 테스트가 있는가와 관계 있는 <tt style="color: #FF0000">`Code Coverage`</tt>와 제가 사용한 <tt style="color: #FF0000">`coverage.py`</tt>에 대해 내용을 정리했습니다.

## Code Coverage?

<tt style="color: #FF0000">`Code Coverage`</tt>에 대해 [위키](https://ko.wikipedia.org/wiki/%EC%BD%94%EB%93%9C_%EC%BB%A4%EB%B2%84%EB%A6%AC%EC%A7%80)에는 다음과 같이 정의되어 있습니다. 

코드 커버리지(Code Coverage)는 소프트웨어의 테스트를 논할 때 얼마나 테스트가 충분한가를 나타내는 지표중 하나다. 
말 그대로 코드가 얼마나 커버되었는가이다. 소프트웨어 테스트를 진행했을 때 코드 자체가 얼마나 실행되었냐는 것이다.

코드의 구조를 이루는 것은 크게 구문(Statement), 조건(Condition), 결정(Decision)이다. 
이러한 구조를 얼마나 커버했느냐에 따라 코드커버리지의 측정기준은 나뉘게 된다. 
일반적으로 많이 사용되는 커버리지는 구문(Statement)커버리지이며, 실행 코드라인이 한번 이상 실행되면 충족된다. 
조건(Condition)커버리지는 각 내부 조건이 참 혹은 거짓을 가지면 충족된다. 
결정(Decision) 커버리지는 각 분기의 내부 조건자체가 아닌 이러한 조건으로 인해 전체 결과가 참 혹은 거짓이면 충족된다. 
그리고 조건과 결정을 복합적으로 고려하는 MC/DC 커버리지 또한 있다. 
커버리지를 측정하는 법은 사람이 로그를 찍어가거나 디버거를 이용하여 볼수는 있으나 매우 힘든 과정이다. 
시중에는 많은 코드커버리지 측정 도구가 나와 있으며, 대표적인 도구로 DT10, LDRA, VectorCAST, CodeScroll Controller Tester 라는 도구가 있다.

## coverage.py

저는 회사 제품 개발과 개인 프로젝트에서 <tt style="color: #FF0000">`Python`</tt>을 사용하는 관계로 <tt style="color: #FF0000">`Code Coverage`</tt> 측정에 <tt style="color: #FF0000">`coverage.py`</tt>를 사용했습니다.
<tt style="color: #FF0000">`coverage.py`</tt>에 대한 자세한 내용은 [공식문서](http://coverage.readthedocs.io/en/latest/index.html)를 참고하시기 바랍니다.

### Install

설치는 간단합니다. 프로젝트 루트에서 다음의 명령으로 간단하게 설치합니다.
~~~~
pip install coverage
~~~~

### Command

<tt style="color: #FF0000">`coverage.py`</tt>에는 다음과 같은 명령어가 있습니다.

* <tt style="color: #FF0000">`run`</tt>: Python 프로그램을 실행하고 실행 데이터를 수집합니다.
* <tt style="color: #FF0000">`report`</tt>: 커버리지 결과를 보여줍니다.
* <tt style="color: #FF0000">`html`</tt>: 커버리지 결과가 있는 주석이 달린 HTML 목록을 생성합니다.
* <tt style="color: #FF0000">`xml`</tt>: 커버리지 결과가 포함된 XML 보고서를 생성합니다.
* <tt style="color: #FF0000">`annotate`</tt>: 소스 파일에 커버리지 결과에 주석을 답니다.
* <tt style="color: #FF0000">`erase`</tt>: 이전에 수집된 데이터를 삭제합니다.
* <tt style="color: #FF0000">`combine`</tt>: 다수의 데이터 파일을 합니다.
* <tt style="color: #FF0000">`debug`</tt>: 진단 정보를 얻습니다.

작성된 테스트와 코드가 있다면 다음의 명령어로 커버리지 데이터를 수집합니다.

~~~~
coverage run --source='.' manage.py test <django project name>
~~~~

수집된 커버리지 데이터를 보려면 다음의 명령어를 실행합니다.

~~~~
coverage report
~~~~

다음의 형태로 결과가 나옵니다.

~~~~
Name                                                                    Stmts   Miss  Cover
-------------------------------------------------------------------------------------------
refrigerator_manager/__init__.py                                            0      0   100%
refrigerator_manager/apps/__init__.py                                       0      0   100%
...
...
refrigerator_manager/apps/items/models.py                                  41     11    73%
refrigerator_manager/apps/items/serializers.py                             10      4    60%
refrigerator_manager/apps/items/views.py                                   29     11    62%
efrigerator_manager/urls.py                                                9      1    89%
-------------------------------------------------------------------------------------------
TOTAL                                                                     221     36    84%
~~~~

<tt style="color: #FF0000">`Miss`</tt>에 나오는 수치는 테스트 코드 작동 중 실행되지 않은 구문의 숫자를 나타냅니다.
작성한 코드 중에는 굳이 테스트를 실행하지 않아도 되는 구문도 있고, 실수로 누락된 구문이 있을 수 있습니다.
그래서 실행되지 않은 구문을 정확하게 알면 테스트 작성에 도움이 되는데요, 실행되지 않은 구문에 대한 안내는 <tt style="color: #FF0000">`report`</tt> 생성 시 옵션을 추가하면 가능합니다.
다음의 명령어로 다시 확인해 보겠습니다.

~~~~
Name                                                                    Stmts   Miss  Cover   Missing
-----------------------------------------------------------------------------------------------------
refrigerator_manager/__init__.py                                            0      0   100%
refrigerator_manager/apps/__init__.py                                       0      0   100%
refrigerator_manager/apps/core/__init__.py                                  0      0   100%
...
...
refrigerator_manager/apps/items/models.py                                  35      9    74%   30, 40, 48-56
refrigerator_manager/apps/items/serializers.py                              4      1    75%   12
refrigerator_manager/apps/items/views.py                                   29     11    62%   41-67
-----------------------------------------------------------------------------------------------------
TOTAL                                                                     210     32    85%
~~~~

위의 결과 중 <tt style="color: #FF0000">`Missing`</tt>의 숫자는 해당 코드에서 테스트 작동 시 누락된 구문을 의미합니다.

테스트 코드와 코드를 확인하고 테스트에 포함되어야 하는 구문이면 테스트를 추가하고, 테스트가 필요없는 구문이라면 커버리지 측정에서 제외시킵니다.

### Excluding code or file from coverage.py

#### Simple Way

커버리지 측저에서 제외시킬 코드 구문 옆에 <tt style="color: #FF0000">`# pragma: no cover`</tt>를 추가합니다.

다음의 예시를 참고하세요.

~~~~
# in <some file>.py
from django.contrib import admin

# Register your models here.
from .models import Place


admin.site.register(Place) # pragma: no cover
~~~~

이렇게 작성을 하면 <tt style="color: #FF0000">`# pragma: no cover`</tt> 이 주석이 있는 구문은 커버리지 측정 시 제외됩니다.

#### Advanced Way

위의 방법이 쉽지만 커버리지 측정 시 제외시킬 코드마다 주석을 붙이는 것은 비효율적이기도 하고, 미관상(?!) 좋지 않기에 제외시킬 구문과 파일을 하나의 설정 파일에서 관리하는 방법 사용을 추천드립니다.

커버리지 측정에 제외시킬 코드와 파일이 있는 경우 프로젝트 루트에 <tt style="color: #FF0000">`.coveragerc`</tt> 파일을 생성하고 제외시킬 코드와 파일을 정의합니다.

예를 들어 <tt style="color: #FF0000">`def __repr__`</tt> 구문을 커버리지 측정 과정에서 제외하고 싶다면 다음과 같이 작성합니다.

~~~~
# in .coveragerc
[report]
exclude_lines = def __repr__
~~~~

다수의 구문을 제외하려면 다음과 같이 작성 가능합니다.

~~~~
# in .coveragerc
[report]
exclude_lines =
    pragma: no cover
    def __repr__
    if self.debug:
    if settings.DEBUG
~~~~

구문이 아닌 파일을 제외시킬 경우 다음과 같이 작성합니다.

~~~~
# in .coveragerc
[run]
omit =
    # omit anything in a .local directory anywhere
    */.local/*
    # omit everything in /usr
    /usr/*
    # omit this single file
    utils/tirefire.py
~~~~

처음 사용해본 것이라 부족한 내용도 있습니다. 혹시 틀린 내용이 있다면 편하게 말씀해주세요. :)
읽어주셔서 감사합니다.