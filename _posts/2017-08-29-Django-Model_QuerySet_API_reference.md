---
layout: post
title:  "Django Models_QuerySet_API_reference"
date:   2017-08-29 10:43:59
author: Dean Kim
categories: Django
tags:	Django Models Models_QuerySet_API_reference
cover:  "/assets/instacode.png"
---

# QuerySet API reference
- 원본 : [공식문서](https://docs.djangoproject.com/en/1.11/ref/models/querysets/)


이 문서는 <tt style="color: #FF0000">`QuerySet`</tt> API의 세부 사항을 설명합니다. [모델](https://docs.djangoproject.com/en/1.11/topics/db/models/) 및 [데이터베이스 쿼리](https://docs.djangoproject.com/en/1.11/topics/db/queries/) 가이드에 제시된 내용을 토대로 작성되었으므로 이 문서를 읽기 전에 해당 문서를 읽고 이해하고 싶을 것입니다.

이 레퍼런스 전반에 걸쳐 우리는 [데이터베이스 쿼리 가이드](https://docs.djangoproject.com/en/1.11/topics/db/queries/)에 제시된 [예제 웹 로그 모델](https://docs.djangoproject.com/en/1.11/topics/db/queries/#queryset-model-example)을 사용할 것입니다.


## When QuerySets are evaluated

내부적으로 <tt style="color: #FF0000">`QuerySet`</tt>은 실제로 데이터베이스에 도달하지 않고 구성, 필터링, 슬라이스 및 일반적으로 전달될 수 있습니다. 쿼리 세트를 평가할 때까지 실제로 데이터베이스 활동이 발생하지 않습니다.

다음과 같은 방법으로 <tt style="color: #FF0000">`QuerySet`</tt>을 평가할 수 있습니다.

* <tt style="color: #FF0000">`Iteration`</tt>. <tt style="color: #FF0000">`QuerySet`</tt>은 반복 가능하며 처음 반복할 때 QuerySet을 실행합니다. 예를 들어, 데이터베이스의 모든 항목의 제목을 인쇄합니다.
~~~~
for e in Entry.objects.all():
    print(e.headline)
~~~~
참고 : 적어도 하나의 결과가 존재하는지 확인하기만 하면됩니다. <tt style="color: #FF0000">`exists()`</tt>를 사용하는 것이 더 효율적입니다.

* <tt style="color: #FF0000">`Slicing`</tt>. [Limiting QuerySet](https://docs.djangoproject.com/en/1.11/topics/db/queries/#limiting-querysets)에서 설명했듯이, <tt style="color: #FF0000">`QuerySet`</tt>은 Python의 배열 슬라이싱 구문을 사용하여 슬라이스할 수 있습니다. 평가되지 않은 <tt style="color: #FF0000">`QuerySet`</tt>을 슬라이스하면 다른 평가되지 않은 <tt style="color: #FF0000">`QuerySet`</tt>이 반환되지만 Django는 슬라이스 구문의 "step"매개 변수를 사용하면 데이터베이스 쿼리를 실행하고 목록을 반환합니다. 평가된 <tt style="color: #FF0000">`QuerySet`</tt>t을 슬라이스하면 목록도 반환됩니다.
<br>또한 평가되지 않은 <tt style="color: #FF0000">`QuerySet`</tt>을 슬라이스하는 경우 평가되지 않은 또 다른 <tt style="color: #FF0000">`QuerySet`</tt>이 반환되지만 더 많은 필터를 추가하거나 순서를 수정하는 것은 허용되지 않습니다. 이는 SQL로 잘 변환되지 않으며 명확한 의미도 가지지 않기 때문입니다.

* <tt style="color: #FF0000">`Pickling/Caching`</tt>. QuerySets를 [pickling QuerySets](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#pickling-querysets) 할 때 관련된 내용에 대한 자세한 내용은 다음 섹션을 참조하십시오. 이 섹션에서 중요한 점은 결과가 데이터베이스에서 읽혀진다는 것입니다.

* <tt style="color: #FF0000">`repr()`</tt>. <tt style="color: #FF0000">`QuerySet`</tt>은 <tt style="color: #FF0000">`repr()`</tt>을 호출할 때 평가됩니다. 이것은 Python 인터프리터 인터프리터에서 편리하기 때문에 대화식으로 API를 사용할 때 결과를 즉시 볼 수 있습니다.

* <tt style="color: #FF0000">`len()`</tt>. <tt style="color: #FF0000">`QuerySet`</tt>은 <tt style="color: #FF0000">`len()`</tt>을 호출 할 때 평가됩니다. 예상 한대로 결과 목록의 길이를 반환합니다.
<br><br>참고 : 집합의 레코드 수를 결정할 필요가 있고 실제 개체가 필요하지 않은 경우 SQL의 <tt style="color: #FF0000">`SELECT COUNT(*)`</tt>를 사용하여 데이터베이스 수준에서 카운트를 처리하는 것이 훨씬 효율적입니다. Django는 정확하게 <tt style="color: #FF0000">`count()`</tt> 메소드를 제공합니다.

* <tt style="color: #FF0000">`list()`</tt>. QuerySet의 list()를 호출하여 QuerySet을 강제적으로 구합니다. 예 :
~~~~
entry_list = list(Entry.objects.all())
~~~~
* <tt style="color: #FF0000">`bool()`</tt>. <tt style="color: #FF0000">`bool()`</tt>, <tt style="color: #FF0000">`or`</tt>, <tt style="color: #FF0000">`and`</tt> 또는 <tt style="color: #FF0000">`if`</tt> 문을 사용하는 것과 같이 부울 컨텍스트에서 <tt style="color: #FF0000">`QuerySet`</tt>을 테스트하면 쿼리가 실행됩니다. 적어도 하나의 결과가 있으면 <tt style="color: #FF0000">`QuerySet`</tt>은 <tt style="color: #FF0000">`True`</tt>이고, 그렇지 않으면 <tt style="color: #FF0000">`False`</tt>입니다. 예 :
~~~~
if Entry.objects.filter(headline="Test"):
   print("There is at least one Entry with the headline Test")
~~~~
참고 : 최소한 하나의 결과만 존재하는지 (실제 객체가 필요하지는 않은지) 확인하려는 경우 <tt style="color: #FF0000">`exists()`</tt>를 사용하는 것이 더 효율적입니다.


### Pickling QuerySets

<tt style="color: #FF0000">`QuerySet`</tt>을 <tt style="color: #FF0000">`pickle`</tt>하면 pickling전에 메모리에 모든 결과가 로드됩니다. pickling은 일반적으로 캐싱의 선구자로 사용되며 캐싱된 queryset이 다시 로드될 때 결과가 이미 사용되고 사용할 준비가 되기를 원합니다. (데이터베이스 읽기는 캐싱의 목적을 무의미한 시간이 걸릴 수 있습니다). 즉, <tt style="color: #FF0000">`QuerySet`</tt>을 실행 취소할 때 현재 데이터베이스에 있는 결과가 아닌 쿼리된 결과가 포함됩니다.

나중에 데이터베이스에서 <tt style="color: #FF0000">`QuerySet`</tt>을 재작성하기 위해 필요한 정보만을 pickle 경우 <tt style="color: #FF0000">`QuerySet`</tt>의 <tt style="color: #FF0000">`query`</tt> 속성을 선택하십시오. 다음과 같은 코드를 사용하여 원래의 <tt style="color: #FF0000">`QuerySet`</tt>을 다시 로드할 수 있습니다 (결과가 로드되지 않음).
~~~~
>>> import pickle
>>> query = pickle.loads(s)     # Assuming 's' is the pickled string.
>>> qs = MyModel.objects.all()
>>> qs.query = query            # Restore the original 'query'.
~~~~
<tt style="color: #FF0000">`query`</tt> 속성은 불투명한 객체입니다. 이는 쿼리 작성의 내부를 나타내며 공개 API의 일부가 아닙니다. 그러나 여기에 설명된대로 속성의 내용을 pickle 및 unpickle하는 것은 안전하고 완벽하게 지원됩니다.

You can’t share pickles between versions

<tt style="color: #FF0000">`QuerySet`</tt>의 pickle은 생성하는데 사용된 Django의 버전에서만 유효합니다. Django 버전 N을 사용하여 pickle을 생성하면 장고 버전 N + 1에서 pickle을 읽을 수 있다는 보장이 없습니다. pickle은 장기 보관 전략의 일부로 사용되어서는 안됩니다.

자동으로 손상된 객체와 같이 pickle 호환성 오류를 진단하기가 어려울 수 있기 때문에 Python과 다른 Django 버전의 쿼리 세트를 unpickle하려고하면 <tt style="color: #FF0000">`RuntimeWarning`</tt>이 발생합니다.


## QuerySet API

다음은 <tt style="color: #FF0000">`QuerySet`</tt>의 정식 선언입니다.

<tt style="color: #FF0000">`class QuerySet(model=None, query=None, using=None)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/query/#QuerySet)

일반적으로 <tt style="color: #FF0000">`QuerySet`</tt>과 상호 작용할 때 [필터를 연결](https://docs.djangoproject.com/en/1.11/topics/db/queries/#chaining-filters)하여 사용합니다. 이 작업을 수행하기 위해 대부분의 <tt style="color: #FF0000">`QuerySet`</tt> 메소드는 새로운 queryset을 반환합니다. 이러한 방법은 이 섹션의 뒷부분에서 자세히 설명합니다.

<tt style="color: #FF0000">`QuerySet`</tt> 클래스에는 인트로 스펙트에 사용할 수 있는 두 개의 공용 속성이 있습니다.

<b>ordered</b>

<tt style="color: #FF0000">`QuerySet`</tt>이 ordered면 <tt style="color: #FF0000">`True`</tt>입니다. 즉, 모델에 <tt style="color: #FF0000">`order_by()`</tt>절이 있거나 기본 순서가 있습니다. 그렇지 않으면 <tt style="color: #FF0000">`False`</tt>.

<b>db</b>

이 쿼리가 지금 실행될 경우 사용할 데이터베이스입니다.


<b>Note</b>

<tt style="color: #FF0000">`GeoQuerySet`</tt>과 같은 특수 쿼리 서브 클래스가 내부 쿼리 상태를 재구성 할 수 있도록 <tt style="color: #FF0000">`QuerySet`</tt>에 대한 <tt style="color: #FF0000">`query`</tt> 매개 변수가 존재합니다. 매개 변수의 값은 해당 u 리 상태의 불투명 한 표현이며 공용 API의 일부가 아 U니다. 간단히 말하면 : 당신이 물어볼 필요가 있다면, 그것을 사용할 필요가 없습니다.


### Methods that return new QuerySets

Django는 <tt style="color: #FF0000">`QuerySet`</tt>에 의해 리턴된 결과의 타입이나 SQL 쿼리가 실행되는 방식을 변경하는 다양한 <tt style="color: #FF0000">`QuerySet`</tt> 세분화 메소드를 제공합니다.

#### filter()

<tt style="color: #FF0000">`filter(**kwargs)`</tt>

지정된 검색 매개 변수와 일치하는 객체가 포함된 새 <tt style="color: #FF0000">`QuerySet`</tt>을 반환합니다.

조회 매개 변수 (**kwargs)는 아래 [Field lookups](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#id4)에 설명된 형식이어야 합니다. 여러 매개 변수는 기본 SQL문에서 <tt style="color: #FF0000">`AND`</tt>를 통해 조인됩니다.

더 복잡한 쿼리 (예 : <tt style="color: #FF0000">`OR`</tt> 문을 사용하는 쿼리)를 실행해야하는 경우 [Q object](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.Q)를 사용할 수 있습니다.

#### exclude()

<tt style="color: #FF0000">`exclude(**kwargs)`</tt>

지정된 조회 매개 변수와 일치하지 않는 객체가 포함된 새 <tt style="color: #FF0000">`QuerySet`</tt>을 반환합니다.

조회 매개 변수 (**kwargs)는 아래 [Field lookups](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#id4)에 설명된 형식이어야합니다. 여러 매개 변수는 기본 SQL 문에서 <tt style="color: #FF0000">`AND`</tt>를 통해 조인되며 모든 것은 <tt style="color: #FF0000">`NOT()`</tt>으로 묶입니다.

이 예제에서는 <tt style="color: #FF0000">`pub_date`</tt>가 2005-1-3보다 늦고 <tt style="color: #FF0000">`headline`</tt>이 "Hello"인 모든 항목을 제외합니다.
~~~~
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3), headline='Hello')
~~~~
SQL문으로는 다음과 같습니다.
~~~~
SELECT ...
WHERE NOT (pub_date > '2005-1-3' AND headline = 'Hello')
~~~~
이 예제에서는 <tt style="color: #FF0000">`pub_date`</tt>가 2005-1-3보다 늦거나 모든 제목이 "Hello"인 모든 항목을 제외합니다.
~~~~
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3)).exclude(headline='Hello')
~~~~
SQL문으로는 다음과 같습니다.
~~~~
SELECT ...
WHERE NOT pub_date > '2005-1-3'
AND NOT headline = 'Hello'
~~~~
두 번째 예제는 더 제한적입니다.

더 복잡한 쿼리 (예 : <tt style="color: #FF0000">`OR`</tt> 문을 사용하는 쿼리)를 실행해야하는 경우 [Q object](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.Q)를 사용할 수 있습니다.

#### annotate()

<tt style="color: #FF0000">`annotate(*args, **kwargs)`</tt>

제공된 [query expressions](https://docs.djangoproject.com/en/1.11/ref/models/expressions/) 목록으로 <tt style="color: #FF0000">`QuerySet`</tt>의 각 개체에 주석을 첨부합니다. 표현식은 간단한 값, 모델의 필드에 대한 참조 (또는 모든 관련 모델), 또는 <tt style="color: #FF0000">`QuerySet`</tt>의 개체의 개체와 관련하여 계산된 집계식 (평균, 합계 등)이 될 수 있습니다.

<tt style="color: #FF0000">`annotate()`</tt>의 각 인수는 반환되는 <tt style="color: #FF0000">`QuerySet`</tt>의 각 객체에 추가되는 주석입니다.

Django가 제공하는 집계 함수는 아래의 [Aggregation Functions(집계 함수)](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#id5)에 설명되어 있습니다.

키워드 인수를 사용하여 지정된 주석은 이 키워드를 주석의 별칭으로 사용합니다. 익명 인수에는 집계 함수의 이름과 집계중인 모델 필드를 기반으로 별칭이 생성됩니다. 단일 필드를 참조하는 집계식만 익명 인수가 될 수 있습니다. 다른 모든 것은 키워드 인수여야 합니다.

예를 들어 블로그 목록을 조작하는 경우 각 블로그에서 몇 개의 항목이 작성되었는지 확인할 수 있습니다.
~~~~
>>> from django.db.models import Count
>>> q = Blog.objects.annotate(Count('entry'))
# The name of the first blog
>>> q[0].name
'Blogasaurus'
# The number of entries on the first blog
>>> q[0].entry__count
42
~~~~
<tt style="color: #FF0000">`Blog`</tt> 모델은 <tt style="color: #FF0000">`entry__count`</tt> 속성 자체를 정의하지 않지만 키워드 인수를 사용하여 집계 함수를 지정하면 주석의 이름을 제어할 수 있습니다.
~~~~
>>> q = Blog.objects.annotate(number_of_entries=Count('entry'))
# The number of entries on the first blog, using the name provided
>>> q[0].number_of_entries
42
~~~~
집계에 대한 자세한 내용은 [집계에 대한 항목 가이드](https://docs.djangoproject.com/en/1.11/topics/db/aggregation/)를 참조하십시오.

#### order_by()

<tt style="color: #FF0000">`order_by(*fields)`</tt>

기본적으로 <tt style="color: #FF0000">`QuerySet`</tt>에 의해 반환된 결과는 모델 <tt style="color: #FF0000">`Meta`</tt>의 <tt style="color: #FF0000">`ordering`</tt> 옵션에 의해 주어진 순서 튜플에 의해 정렬됩니다. <tt style="color: #FF0000">`order_by`</tt> 메소드를 사용하여 <tt style="color: #FF0000">`QuerySet`</tt> 단위로 이 값을 겹쳐 쓸 수 있습니다.

예:
~~~~
Entry.objects.filter(pub_date__year=2005).order_by('-pub_date', 'headline')
~~~~
위의 결과는 <tt style="color: #FF0000">`pub_date`</tt>가 내림차순으로 정렬된 다음 <tt style="color: #FF0000">`headline`</tt>이 오름차순으로 정렬됩니다. <tt style="color: #FF0000">`-pub_date`</tt>앞에 있는 (-) 기호는 내림차순을 나타냅니다. 오름차순이 내포되어 있습니다. 무작위로 order하려면 다음과 같이 <tt style="color: #FF0000">`"?"`</tt>를 사용하십시오.
~~~~
Entry.objects.order_by('?')
~~~~
참고 : <tt style="color: #FF0000">`order_by('?')`</tt> 쿼리는 사용하는 데이터베이스 백엔드에 따라 값이 비싸고 느릴 수 있습니다.

다른 모델의 필드로 정렬하려면 모델 관계를 쿼리할 때와 같은 구문을 사용하십시오. 즉, 필드 이름과 이중 밑줄 (__), 새 모델의 필드 이름 등이 포함됩니다. 원하는 모델을 추가할 수 있습니다. 예 :
~~~~
Entry.objects.order_by('blog__name', 'headline')
~~~~
Django는 다른 모델과 관계가 있는 필드로 정렬을 시도하면 관련 모델의 기본 순서를 사용하거나 <tt style="color: #FF0000">`Meta.ordering`</tt>이 지정되지 않은 경우 관련 모델의 기본 키순으로 정렬합니다. 예를 들어, <tt style="color: #FF0000">`Blog`</tt> 모델에는 기본 순서가 지정되어 있지 않기 때문에 :
~~~~
Entry.objects.order_by('blog')
~~~~
…is identical to:
~~~~
Entry.objects.order_by('blog__id')
~~~~
<tt style="color: #FF0000">`Blog`</tt>가 <tt style="color: #FF0000">`ordering = ['name']`</tt> 인 경우 첫 번째 쿼리 세트는 다음과 같습니다.
~~~~
Entry.objects.order_by('blog__name')
~~~~
관련 필드의 <tt style="color: #FF0000">`_id`</tt>를 참조하여 JOIN의 비용을 들이지 않고 관련 필드에서 쿼리 세트를 주문할 수도 있습니다.
~~~~
# No Join
Entry.objects.order_by('blog_id')

# Join
Entry.objects.order_by('blog__id')
~~~~
표현식에서 <tt style="color: #FF0000">`asc()`</tt> 또는 <tt style="color: #FF0000">`desc()`</tt>를 호출하여 [쿼리식](https://docs.djangoproject.com/en/1.11/ref/models/expressions/)으로 정렬할 수도 있습니다.
~~~~
Entry.objects.order_by(Coalesce('summary', 'headline').desc())
~~~~
<tt style="color: #FF0000">`distinct()`</tt>를 사용하는 경우 관련 모델의 필드로 ordering할 때는 주의하십시오. 관련 모델 순서가 예상된 결과를 어떻게 바꿀 수 있는지에 대한 설명은 [distinct()](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.query.QuerySet.distinct)의 설명을 참조하십시오.

<b>Note</b>

결과를 정렬하기 위해 다중 값 필드를 지정할 수 있습니다 (예 : <tt style="color: #FF0000">`ManyToManyField`</tt> 필드 또는 <tt style="color: #FF0000">`ForeignKey`</tt> 필드의 역 관계).

이 경우를 생각해보십시오.
~~~~
class Event(Model):
   parent = models.ForeignKey(
       'self',
       on_delete=models.CASCADE,
       related_name='children',
   )
   date = models.DateField()

Event.objects.order_by('children__date')
~~~~
여기에는 잠재적으로 각 이벤트에 대한 여러 ordering 데이터가 있을 수 있습니다. 여러 자식이 있는 각 이벤트는 <tt style="color: #FF0000">`order_by()`</tt>가 생성한 새 <tt style="color: #FF0000">`QuerySet`</tt>으로 여러 번 반환됩니다. 즉, <tt style="color: #FF0000">`QuerySet`</tt>에서 <tt style="color: #FF0000">`order_by()`</tt>를 사용하면 예상했던 것보다 더 많은 항목을 반환할 수 있습니다.
따라서 다중 값 필드를 사용하여 결과를 정렬할 때는 주의해야 합니다. ordering하는 품목마다 하나의 ordering 데이터만 있다는 것을 확신할 수 있다면 이 접근법은 문제를 나타내지 않아야 합니다. 그렇지 않은 경우 결과가 예상한 결과인지 확인하십시오.

<b>Note End</b>

순서가 대소 문자를 구분해야하는지 여부를 지정할 방법이 없습니다. 대소 문자 구분과 관련하여 Django는 데이터베이스 백엔드가 정상적으로 명령하지만 결과를 ordering합니다.

대소 문자가 일치하는 순서를 얻을 수 있도록 <tt style="color: #FF0000">`Lower`</tt>로 소문자로 변환된 필드로 정렬할 수 있습니다.
~~~~
Entry.objects.order_by(Lower('headline').desc())
~~~~
기본 순서가 아니더라도 쿼리에 어떤 순서도 적용하지 않으려면 매개 변수없이 <tt style="color: #FF0000">`order_by()`</tt>를 호출하십시오.

<tt style="color: #FF0000">`QuerySet`</tt>이 어떤 식으로든 정렬된 경우 <tt style="color: #FF0000">`True`</tt>가 되는 <tt style="color: #FF0000">`QuerySet.ordered`</tt> 특성을 검사하여 쿼리가 정렬되었는지 여부를 알 수 있습니다.

각 <tt style="color: #FF0000">`order_by()`</tt> 호출은 이전 순서를 지웁니다. 예를 들어 이 검색어는 <tt style="color: #FF0000">`headline`</tt>이 아닌 <tt style="color: #FF0000">`pub_date`</tt>에 의해 정렬됩니다.
~~~~
Entry.objects.order_by('headline').order_by('pub_date')
~~~~

<b>Warning</b>
ordering은 자유로운 운영이 아닙니다. ordering에 추가하는 각 필드는 데이터베이스 비용을 발생시킵니다. 추가하는 각 외래 키에는 암시적으로 모든 기본 순서가 포함됩니다.

조회에 지정된 순서가 없는 경우, 결과는 지정되지 않은 순서로 데이터베이스에서 리턴됩니다. 특정 순서는 결과의 각 객체를 고유하게 식별하는 필드 세트로 정렬할 때만 보장됩니다. 예를 들어 <tt style="color: #FF0000">`name`</tt> 필드가 고유하지 않은 경우에는 이름순으로 정렬하여 동일한 이름의 객체가 항상 같은 순서로 표시되는 것은 아닙니다.
<b>Warning End</b>

#### reverse()

<tt style="color: #FF0000">`reverse()`</tt>

<tt style="color: #FF0000">`reverse()`</tt> 메서드를 사용하여 쿼리 세트의 요소가 반환되는 순서를 바꿉니다. <tt style="color: #FF0000">`reverse()`</tt>를 다시 호출하면 순서가 정상 방향으로 복원됩니다.

queryset에서 "마지막"다섯 항목을 검색하려면 다음을 실행하면 됩니다.
~~~~
my_queryset.reverse()[:5]
~~~~
이것은 파이썬에서 시퀀스의 끝에서부터 슬라이싱하는 것과 완전히 같지 않음에 유의하십시오. 위의 예제는 마지막 항목을 먼저 반환하고, 마지막에서 두 번째 항목을 반환합니다. 파이썬 시퀀스에서는 seq [-5:]를 보면, 다섯 번째 마지막 항목이 먼저 보입니다. Django는 SQL에서 효율적으로 수행 할 수 없으므로 마지막부터 슬라이싱하는 액세스 모드를 지원하지 않습니다.

또한 <tt style="color: #FF0000">`reverse()`</tt>는 일반적으로 정의된 순서가 있는 <tt style="color: #FF0000">`QuerySet`</tt>에서만 호출되어야합니다 (예 : 기본 순서를 정의하는 모델을 쿼리하거나 <tt style="color: #FF0000">`order_by()`</tt>를 사용할 때). 주어진 <tt style="color: #FF0000">`QuerySet`</tt>에 대해 그러한 정렬이 정의되어 있지 않으면 <tt style="color: #FF0000">`reverse()`</tt>를 호출하면 실제 효과가 없습니다. <tt style="color: #FF0000">`reverse()`</tt>를 호출하기 전에 정렬이 정의되지 않았으며 나중에 정의되지 않습니다.

#### distint()

<tt style="color: #FF0000">`distinct(*fields)`</tt>

SQL 쿼리에서 <tt style="color: #FF0000">`SELECT DISTINCT`</tt>를 사용하는 새 <tt style="color: #FF0000">`QuerySet`</tt>을 반환합니다. 이렇게 하면 조회 결과에서 중복행이 제거됩니다.

기본적으로 <tt style="color: #FF0000">`QuerySet`</tt>은 중복행을 제거하지 않습니다. 실제로 <tt style="color: #FF0000">`Blog.objects.all()`</tt>과 같은 간단한 쿼리는 결과 행이 중복될 가능성을 소개하고 있지 않기 때문에 거의 문제가 되지 않습니다. 그러나 쿼리가 여러 테이블에 걸쳐있는 경우 <tt style="color: #FF0000">`QuerySet`</tt>을 평가할 때 중복 결과를 얻을 수 있습니다. <tt style="color: #FF0000">`distinct()`</tt>를 사용할 때입니다.

<b>Note</b>
<tt style="color: #FF0000">`order_by()`</tt> 호출에 사용된 모든 필드는 SQL <tt style="color: #FF0000">`SELECT`</tt> 열에 포함됩니다. <tt style="color: #FF0000">`distinct()`</tt>와 함께 사용하면 예기치 않은 결과가 발생할 수 있습니다. 관련 모델의 필드별로 order하면 해당 필드가 선택한 열에 추가되고 그렇지 않으면 중복 행이 구별되는 것처럼 보일 수 있습니다. 추가 열은 반환된 결과에 표시되지 않으므로 (순서 지정을 지원할 때만 있음) 때로는 뚜렷하지 않은 결과가 반환되는 것처럼 보입니다.

마찬가지로, <tt style="color: #FF0000">`values()`</tt> 쿼리를 사용하여 선택된 열을 제한하면 모든 <tt style="color: #FF0000">`order_by()`</tt> (또는 기본 모델 순서)에 사용된 열이 여전히 관련되어 결과의 고유성에 영향을 줄 수 있습니다.

여기에서 <tt style="color: #FF0000">`distinct()`</tt>를 사용한다면 관련 모델에 의한 ordering에 주의해야 한다는 것입니다. 마찬가지로, <tt style="color: #FF0000">`distinct()`</tt>와 <tt style="color: #FF0000">`values()`</tt>를 함께 사용할 때 <tt style="color: #FF0000">`values()`</tt> 호출이 아닌 필드로 정렬할 때는 주의해야 합니다.
<br><b>Note End</b>

PostgreSQL에서만 <tt style="color: #FF0000">`DISTINCT`</tt>가 적용되어야 하는 필드의 이름을 지정하기 위해 위치 인수 (<tt style="color: #FF0000">`*fields`</tt>)를 전달할 수 있습니다. 이것은 <tt style="color: #FF0000">`SELECT DISTINCT ON`</tt> SQL 쿼리로 변환됩니다. 차이점은 다음과 같습니다. 일반 <tt style="color: #FF0000">`distinct()`</tt> 호출의 경우 데이터베이스는 고유한 행을 판별할 때 각 행의 각 필드를 비교합니다. 지정된 필드 이름이 있는 <tt style="color: #FF0000">`distinct()`</tt> 호출의 경우 데이터베이스는 지정된 필드 이름만 비교합니다.

<b>Note</b>
필드 이름을 지정할 때 <tt style="color: #FF0000">`QuerySet`</tt>에 <tt style="color: #FF0000">`order_by()`</tt>를 제공해야하며 <tt style="color: #FF0000">`order_by()`</tt>의 필드는 <tt style="color: #FF0000">`distinct()`</tt>의 필드로 시작해야합니다.

예를 들어, <tt style="color: #FF0000">`SELECT DISTINCT ON (a)`</tt>은 a 열의 각 값에 대한 첫 번째 행을 제공합니다. order을 지정하지 않으면 임의의 행이 생깁니다.
<b>Note End</b>

예제 (첫 번째 이후는 PostgreSQL에서만 작동합니다) :
~~~~
>>> Author.objects.distinct()
[...]

>>> Entry.objects.order_by('pub_date').distinct('pub_date')
[...]

>>> Entry.objects.order_by('blog').distinct('blog')
[...]

>>> Entry.objects.order_by('author', 'pub_date').distinct('author', 'pub_date')
[...]

>>> Entry.objects.order_by('blog__name', 'mod_date').distinct('blog__name', 'mod_date')
[...]

>>> Entry.objects.order_by('author', 'pub_date').distinct('author')
[...]
~~~~

<b>Note</b>
<tt style="color: #FF0000">`order_by()`</tt>는 정의된 기본 관련 모델 순서를 사용합니다. <tt style="color: #FF0000">`DISTINCT ON`</tt> 표현식이 <tt style="color: #FF0000">`ORDER BY`</tt> 절의 시작 부분에있는 표현식과 일치하는지 확인하기 위해 <tt style="color: #FF0000">`_id`</tt> 또는 참조된 필드로 명시적으로 정렬해야 할 수도 있습니다. 예를 들어, <tt style="color: #FF0000">`Blog`</tt> 모델이 <tt style="color: #FF0000">`name`</tt>순으로 <tt style="color: #FF0000">`ordering`</tt>을 정의한 경우 :
~~~~
Entry.objects.order_by('blog').distinct('blog')
~~~~
<tt style="color: #FF0000">`blog__name`</tt>에 의해 쿼리가 정렬되어 <tt style="color: #FF0000">`DISTINCT ON`</tt>식이 일치하지 않기 때문에 작동하지 않습니다. 두 표현식이 일치하는지 확인하려면 관계 _id 필드 (이 경우 <tt style="color: #FF0000">`blog_id`</tt>) 또는 참조된 필드 (<tt style="color: #FF0000">`blog__pk`</tt>)로 명시적으로 정렬해야합니다.
<b>Note End</b>

#### values()

<tt style="color: #FF0000">`values(*fields, **expressions)`</tt>

iterable로 사용될 경우 모델 인스턴스가 아닌 dictionaries를 반환하는 <tt style="color: #FF0000">`QuerySet`</tt>을 반환합니다.

해당 dictionaries represents 각각은 모델 오브젝트의 속성 이름에 해당하는 키로 오브젝트를 나타냅니다.

이 예에서는 <tt style="color: #FF0000">`values()`</tt>의 dictionaries을 일반 모델 객체와 비교합니다.
~~~~
# This list contains a Blog object.
>>> Blog.objects.filter(name__startswith='Beatles')
<QuerySet [<Blog: Beatles Blog>]>

# This list contains a dictionary.
>>> Blog.objects.filter(name__startswith='Beatles').values()
<QuerySet [{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}]>
~~~~
<tt style="color: #FF0000">`values()`</tt> 메소드는 <tt style="color: #FF0000">`SELECT`</tt>가 제한되어야하는 필드 이름을 지정하는 선택적 위치 인수, <tt style="color: #FF0000">`*fields`</tt>를 취합니다. 필드를 지정하면 각 dictionary에 지정한 필드의 필드 키/값만 포함됩니다. 필드를 지정하지 않으면 각 dictionary에는 데이터베이스 테이블의 모든 필드에 대한 키와 값이 포함됩니다.

예 :
~~~~
>>> Blog.objects.values()
<QuerySet [{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}]>
>>> Blog.objects.values('id', 'name')
<QuerySet [{'id': 1, 'name': 'Beatles Blog'}]>
~~~~
<tt style="color: #FF0000">`values()`</tt> 메서드는 선택적 키워드 인수인 <tt style="color: #FF0000">`**expressions`</tt>를 사용하여 <tt style="color: #FF0000">`annotate()`</tt>에 전달됩니다.
~~~~
>>> from django.db.models.functions import Lower
>>> Blog.objects.values(lower_name=Lower('name'))
<QuerySet [{'lower_name': 'beatles blog'}]>
~~~~
<tt style="color: #FF0000">`values()`</tt> 절내의 집계는 동일한 <tt style="color: #FF0000">`values()`</tt> 절내의 다른 인수 앞에 적용됩니다. 다른 값으로 그룹화해야 하는 경우 이전 <tt style="color: #FF0000">`values()`</tt> 절에 대신 추가하십시오. 예 :
~~~~
>>> from django.db.models import Count
>>> Blog.objects.values('author', entries=Count('entry'))
<QuerySet [{'author': 1, 'entries': 20}, {'author': 1, 'entries': 13}]>
>>> Blog.objects.values('author').annotate(entries=Count('entry'))
<QuerySet [{'author': 1, 'entries': 33}]>
~~~~

언급 할 가치가 있는 몇 가지 미묘한 점 :
* <tt style="color: #FF0000">`ForeignKey`</tt>인 <tt style="color: #FF0000">`foo`</tt>라는 필드가 있는 경우 실제 값을 저장하는 숨겨진 모델 속성의 이름이므로 default <tt style="color: #FF0000">`values()`</tt> 호출은 <tt style="color: #FF0000">`foo_id`</tt>라는 dictionary 키를 반환합니다 (foo 속성은 관련 모델을 참조 함). <tt style="color: #FF0000">`values()`</tt>를 호출하고 필드 이름을 전달할 때 <tt style="color: #FF0000">`foo`</tt> 또는 <tt style="color: #FF0000">`foo_id`</tt>를 전달할 수 있으며 동일한 것을 다시 얻을 수 있습니다 (dictionary 키는 전달한 필드 이름과 일치합니다).

예 :

~~~~
>>> Entry.objects.values()
<QuerySet [{'blog_id': 1, 'headline': 'First Entry', ...}, ...]>

>>> Entry.objects.values('blog')
<QuerySet [{'blog': 1}, ...]>

>>> Entry.objects.values('blog_id')
<QuerySet [{'blog_id': 1}, ...]>
~~~~

* <tt style="color: #FF0000">`distinct()`</tt>와 함께 <tt style="color: #FF0000">`values()`</tt>를 사용할 때 순서가 결과에 영향을 줄 수 있음에 유의하십시오. 자세한 내용은 [distinct()](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.query.QuerySet.distinct)의 내용을 참조하십시오.
* <tt style="color: #FF0000">`extra()`</tt> 호출 후에 <tt style="color: #FF0000">`values()`</tt> 절을 사용하면 <tt style="color: #FF0000">`extra()`</tt>의 <tt style="color: #FF0000">`select`</tt> 인수로 정의된 모든 필드가 명시적으로 <tt style="color: #FF0000">`values()`</tt> 호출에 포함되어야합니다. <tt style="color: #FF0000">`values()`</tt> 호출 후에 만들어진 <tt style="color: #FF0000">`extra()`</tt> 호출은 여분의 선택된 필드를 무시하게됩니다.
* <tt style="color: #FF0000">`values()`</tt> 뒤에 <tt style="color: #FF0000">`only()`</tt> 및 <tt style="color: #FF0000">`defer()`</tt>를 호출하면 의미가 없으므로 <tt style="color: #FF0000">`NotImplementedError`</tt>가 발생합니다.

소수의 사용 가능한 필드에서 값만 필요하므로 모델 인스턴스 객체의 기능이 필요하지 않을 때 유용합니다. 사용할 필드만 선택하는 것이 더 효율적입니다.

마지막으로, <tt style="color: #FF0000">`values()`</tt> 호출 후에 <tt style="color: #FF0000">`filter()`</tt>, <tt style="color: #FF0000">`order_by()`</tt> 등을 호출 할 수 있습니다. 이는 두 호출이 동일함을 의미합니다.

~~~~
Blog.objects.values().order_by('id')
Blog.objects.order_by('id').values()
~~~~

Django를 만든 사람들은 모든 SQL에 영향을 주는 메소드를 먼저 배치하고 (선택적으로) <tt style="color: #FF0000">`values()`</tt>와 같은 출력에 영향을 미치는 메소드를 따르는 것을 선호하지만 실제로는 상관 없습니다. 이것은 당신의 개인주의를 과시하는 당신의 기회입니다.

<tt style="color: #FF0000">`OneToOneField`</tt>, <tt style="color: #FF0000">`ForeignKey`</tt> 및 <tt style="color: #FF0000">`ManyToManyField`</tt> 속성을 통해 역방향 관계가 있는 관련 모델의 필드를 참조할 수도 있습니다.

~~~~
>>> Blog.objects.values('name', 'entry__headline')
<QuerySet [{'name': 'My blog', 'entry__headline': 'An entry'},
     {'name': 'My blog', 'entry__headline': 'Another entry'}, ...]>
~~~~

<b>Warning</b>
<tt style="color: #FF0000">`ManyToManyField`</tt> 특성 및 역방향 관계에는 여러 관련 행이 있을 수 있으므로 결과 집합 크기에 배율 효과가 있을 수 있습니다. <tt style="color: #FF0000">`values()`</tt> 쿼리에 이러한 필드가 여러 개 포함되는 경우 특히 유용합니다. 이 경우 모든 가능한 조합이 반환됩니다.
<b>Warning End</b>

#### values_list()

#### dates()

#### datetimes()

#### none()

#### all()

#### union()

#### intersection()

#### difference()

#### select_related()

#### prefetch_related()

<tt style="color: #FF0000">`prefetch_related(*lookups)`</tt>

지정된 lookups마다 관련 객체를 단일 배치에서 자동으로 검색하는 <tt style="color: #FF0000">`QuerySet`</tt>을 리턴합니다.

이것은 <tt style="color: #FF0000">`select_related`</tt>와 비슷한 목적을 가지고 있습니다. 둘 다 관련 객체에 액세스하여 발생하는 데이터베이스 쿼리의 대홍수를 막을 수 있도록 설계되었지만 전략은 전혀 다릅니다.

<tt style="color: #FF0000">`select_related`</tt>는 SQL 조인을 작성하고 SELECT 문에 관련 오브젝트의 필드를 포함시켜 작동합니다. 이러한 이유로 <tt style="color: #FF0000">`select_related`</tt>는 동일한 데이터베이스 쿼리에서 관련 객체를 가져옵니다. 그러나 '많은'관계에 참여함으로써 생기는 훨씬 더 큰 결과 집합을 피하기 위해 <tt style="color: #FF0000">`select_related`</tt>는 외래 ​​키와 일대일 단일 값 관계로 제한됩니다.

한편, <tt style="color: #FF0000">`prefetch_related`</tt>는 각 관계에 대해 별도의 조회를 수행하고 Python에서 'joining'을 수행합니다. 이를 통해 <tt style="color: #FF0000">`select_related`</tt>가 지원하는 외래 키 및 일대일 관계 외에 <tt style="color: #FF0000">`select_related`</tt>를 사용하여 수행 할 수 없는 다대다 및 다대일 객체를 prefetch 할 수 있습니다. 또한 <tt style="color: #FF0000">`GenericRelation`</tt> 및 <tt style="color: #FF0000">`GenericForeignKey`</tt>의 prefetching을 지원하지만 동일한 결과 세트로 제한되어야합니다. 예를 들어, <tt style="color: #FF0000">`GenericForeignKey`</tt>가 참조하는 오브젝트 prefetch는 조회가 하나의 <tt style="color: #FF0000">`ContentType`</tt>으로 제한되는 경우에만 지원됩니다.

예를 들어, 다음 모델이 있다고 가정 해보십시오.

~~~~
from django.db import models

class Topping(models.Model):
    name = models.CharField(max_length=30)

class Pizza(models.Model):
    name = models.CharField(max_length=50)
    toppings = models.ManyToManyField(Topping)

    def __str__(self):              # __unicode__ on Python 2
        return "%s (%s)" % (
            self.name,
            ", ".join(topping.name for topping in self.toppings.all()),
        )
~~~~

and run:

~~~~
>>> Pizza.objects.all()
["Hawaiian (ham, pineapple)", "Seafood (prawns, smoked salmon)"...
~~~~

이 문제는 <tt style="color: #FF0000">`Pizza.__ str __()`</tt>이 <tt style="color: #FF0000">`self.toppings.all()`</tt>을 요청할 때마다 데이터베이스를 쿼리해야하므로 <tt style="color: #FF0000">`Pizza.objects.all()`</tt>이 Topping 테이블에 피자 <tt style="color: #FF0000">`QuerySet`</tt>의 모든 아이템에 대한 쿼리를 실행해야 합니다.

<tt style="color: #FF0000">`prefetch_related`</tt>를 사용하여 단 두 개의 쿼리로 줄일 수 있습니다.

~~~~
>>> Pizza.objects.all().prefetch_related('toppings')
~~~~

이것은 각 <tt style="color: #FF0000">`pizza`</tt>에 대한 <tt style="color: #FF0000">`self.toppings.all()`</tt>을 의미합니다. 이제는 <tt style="color: #FF0000">`self.toppings.all()`</tt>이 호출될 때마다 항목에 대한 데이터베이스로 이동하지 않고 단일 쿼리로 채워진 prefetch된 <tt style="color: #FF0000">`QuerySet`</tt> 캐시에서 해당 항목을 찾습니다.

즉, 모든 관련 토핑이 단일 쿼리에서 페치되고 관련 결과의 미리 채워진 캐시가 있는 <tt style="color: #FF0000">`QuerySet`</tt>을 만드는데 사용됩니다. 이러한 <tt style="color: #FF0000">`QuerySet`</tt>은 <tt style="color: #FF0000">`self.toppings.all()`</tt> 호출에서 사용됩니다.

<tt style="color: #FF0000">`prefetch_related()`</tt>의 추가 쿼리는 <tt style="color: #FF0000">`QuerySet`</tt>이 평가되기 시작하고 기본 쿼리가 실행된 후에 실행됩니다.

반복 가능한 모델 인스턴스가 있는 경우, <tt style="color: #FF0000">`prefetch_related_objects()`</tt> 함수를 사용하여 해당 인스턴스에서 관련 속성을 prefetch 할 수 있습니다.

기본 <tt style="color: #FF0000">`QuerySet`</tt>의 결과 캐시와 지정된 모든 관련 객체는 메모리에 완전히 로드됩니다. 이렇게하면 <tt style="color: #FF0000">`QuerySet`</tt>의 일반적인 동작이 변경됩니다. <tt style="color: #FF0000">`QuerySet`</tt>은 데이터베이스에서 쿼리가 실행된 후에도 일반적으로 모든 객체를 메모리에 로드하지 않으려 고합니다.

<b>Note</b>
<tt style="color: #FF0000">`QuerySets`</tt>의 경우와 마찬가지로 항상 다른 데이터베이스 쿼리를 암시하는 후속 체인 메서드는 이전에 캐시된 결과를 무시하고 새 데이터베이스 쿼리를 사용하여 데이터를 검색합니다. 그래서, 다음과 같이 쓰면 :

~~~~
>>> pizzas = Pizza.objects.prefetch_related('toppings')
>>> [list(pizza.toppings.filter(spicy=True)) for pizza in pizzas]
~~~~

... 그 다음에 <tt style="color: #FF0000">`pizza.toppings.all()`</tt>이 선취되었다는 사실이 도움이되지 않습니다. <tt style="color: #FF0000">`prefetch_related('toppings')`</tt>는 <tt style="color: #FF0000">`pizza.toppings.all()`</tt>을 암시했지만 <tt style="color: #FF0000">`pizza.toppings.filter()`</tt>는 새롭고 다른 쿼리입니다. prefetched 캐시는 여기에서 도움이 되지 않습니다. 실제로 사용하지 않은 데이터베이스 쿼리를 수행했으므로 성능이 저하됩니다. 따라서 이 기능을 신중하게 사용하십시오!

[related managers](https://docs.djangoproject.com/en/1.11/ref/models/relations/#django.db.models.fields.related.RelatedManager)에서 데이터베이스 변경 메소드 [add()](https://docs.djangoproject.com/en/1.11/ref/models/relations/#django.db.models.fields.related.RelatedManager.add), [remove()](https://docs.djangoproject.com/en/1.11/ref/models/relations/#django.db.models.fields.related.RelatedManager.remove), [clear()](https://docs.djangoproject.com/en/1.11/ref/models/relations/#django.db.models.fields.related.RelatedManager.clear) 또는 [set()](https://docs.djangoproject.com/en/1.11/ref/models/relations/#django.db.models.fields.related.RelatedManager.set)을 호출하면 관계에 대한 prefetched 캐시가 지워집니다.
<b>Note End</b>

또한 일반적인 조인 구문을 사용하여 관련 필드의 관련 필드를 수행 할 수 있습니다. 위의 예제에 추가 모델이 있다고 가정해 보겠습니다.

~~~~
class Restaurant(models.Model):
    pizzas = models.ManyToManyField(Pizza, related_name='restaurants')
    best_pizza = models.ForeignKey(Pizza, related_name='championed_by')
~~~~

다음은 모두 합법적입니다.

~~~~
>>> Restaurant.objects.prefetch_related('pizzas__toppings')
~~~~

이렇게하면 식당에 속한 모든 피자와 그 피자에 속하는 모든 토핑을 미리 가져옵니다. 이렇게하면 총 3개의 데이터베이스 쿼리가 생성됩니다. 하나는 레스토랑용이고 다른 하나는 피자용이고 다른 하나는 토핑용입니다.

~~~~
>>> Restaurant.objects.prefetch_related('best_pizza__toppings')
~~~~

이것은 각 레스토랑의 best 피자와 best 피자의 모든 토핑을 가져올 것입니다. 이것은 3개의 데이터베이스 쿼리에서 수행될 것입니다. 하나는 레스토랑용이고 다른 하나는 '최고의 피자'용이고 다른 하나는 토핑용입니다.

물론 <tt style="color: #FF0000">`select_related`</tt>를 사용하여 쿼리 수를 2로 줄이면 <tt style="color: #FF0000">`best_pizza`</tt> 관계를 가져올 수도 있습니다.

~~~~
>>> Restaurant.objects.select_related('best_pizza').prefetch_related('best_pizza__toppings')
~~~~

prefetch는 메인 쿼리 (<tt style="color: #FF0000">`select_related`</tt>가 필요로 하는 조인을 포함) 후에 실행되기 때문에 <tt style="color: #FF0000">`best_pizza`</tt> 객체가 이미 페치된 것을 감지할 수 있으며, 페치를 다시 건너뜁니다.

<tt style="color: #FF0000">`prefetch_related`</tt> 호출을 연결하면 prefetch된 조회가 누적됩니다. <tt style="color: #FF0000">`prefetch_related`</tt> 동작을 지우려면 <tt style="color: #FF0000">`None`</tt>을 매개 변수로 전달하십시오.

~~~~
>>> non_prefetched = qs.prefetch_related(None)
~~~~

<tt style="color: #FF0000">`prefetch_related`</tt>를 사용할 때 주목해야 할 한 가지 차이점은 쿼리에 의해 생성된 객체가 관련된 다른 객체간에 공유될 수 있다는 것입니다. 즉, 단일 Python 모델 인스턴스가 반환되는 객체 트리의 둘 이상의 지점에 나타날 수 있습니다. 이것은 일반적으로 외래 키 관계에서 발생합니다. 일반적으로 이 동작은 문제가 되지 않으며 실제로 메모리와 CPU 시간을 절약할 수 있습니다.

<tt style="color: #FF0000">`prefetch_related`</tt>가 <tt style="color: #FF0000">`GenericForeignKey`</tt> 관계의 prefetch를 지원하는 동안 쿼리의 수는 데이터에 따라 달라집니다. <tt style="color: #FF0000">`GenericForeignKey`</tt>는 여러 테이블의 데이터를 참조할 수 있으므로 모든 항목에 대해 하나의 쿼리가 아니라 테이블당 하나의 쿼리가 필요합니다. 관련 행이 아직 반입되지 않은 경우 <tt style="color: #FF0000">`ContentType`</tt> 테이블에 대한 추가 쿼리가 있을 수 있습니다.

대부분의 경우 <tt style="color: #FF0000">`prefetch_related`</tt>는 'IN'연산자를 사용하는 SQL 쿼리를 사용하여 구현됩니다. 즉, 큰 <tt style="color: #FF0000">`QuerySet`</tt>의 경우 데이터베이스에 따라 SQL 쿼리를 파싱하거나 실행할 때 자체적인 성능 문제가 발생할 수 있는 큰 'IN'절이 생성될 수 있습니다. 유스 케이스를 위해 항상 프로파일하십시오!

<tt style="color: #FF0000">`iterator()`</tt>를 사용하여 쿼리를 실행하면 <tt style="color: #FF0000">`prefetch_related()`</tt> 호출은 무시될 것이므로 두 최적화가 함께 이해할 수 없으므로 주의해야합니다.

[Prefetch](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.Prefetch) 오브젝트를 사용하여 prefetch 조작을 추가로 제어할 수 있습니다.

가장 간단한 형식인 <tt style="color: #FF0000">`Prefetch`</tt>는 전통적인 문자열 기반 조회와 동일합니다.

~~~~
>>> from django.db.models import Prefetch
>>> Restaurant.objects.prefetch_related(Prefetch('pizzas__toppings'))
~~~~

선택적 queryset 인수를 사용하여 사용자 정의 <tt style="color: #FF0000">`queryset`</tt>을 제공할 수 있습니다. 이것은 queryset의 기본 순서를 변경하는 데 사용할 수 있습니다.
~~~~

>>> Restaurant.objects.prefetch_related(
...     Prefetch('pizzas__toppings', queryset=Toppings.objects.order_by('name')))
~~~~

또는 가능한 경우 <tt style="color: #FF0000">`select_related()`</tt>를 호출하여 쿼리 수를 더 줄입니다.

~~~~
>>> Pizza.objects.prefetch_related(
...     Prefetch('restaurants', queryset=Restaurant.objects.select_related('best_pizza')))
~~~~

선택적인 <tt style="color: #FF0000">`to_attr`</tt> 인수를 사용하여 사용자 정의 속성에 prefetched 결과를 지정할 수도 있습니다. 결과는 목록에 직접 저장됩니다.

이렇게하면 다른 <tt style="color: #FF0000">`QuerySet`</tt>으로 동일한 관계를 여러 번 prefetch 할 수 있습니다. 예 :

~~~~
>>> vegetarian_pizzas = Pizza.objects.filter(vegetarian=True)
>>> Restaurant.objects.prefetch_related(
...     Prefetch('pizzas', to_attr='menu'),
...     Prefetch('pizzas', queryset=vegetarian_pizzas, to_attr='vegetarian_menu'))
~~~~

사용자 정의 <tt style="color: #FF0000">`to_attr`</tt>로 작성된 조회는 다른 조회에 의해 평상시처럼 계속 탐색될 수 있습니다.

~~~~
>>> vegetarian_pizzas = Pizza.objects.filter(vegetarian=True)
>>> Restaurant.objects.prefetch_related(
...     Prefetch('pizzas', queryset=vegetarian_pizzas, to_attr='vegetarian_menu'),
...     'vegetarian_menu__toppings')
~~~~

관련 관리자의 캐시에 필터링된 결과를 저장하는 것보다 모호하지 않으므로 prefetch 결과를 필터링할 때 <tt style="color: #FF0000">`to_attr`</tt>을 사용하는 것이 좋습니다.

~~~~
>>> queryset = Pizza.objects.filter(vegetarian=True)
>>>
>>> # Recommended:
>>> restaurants = Restaurant.objects.prefetch_related(
...     Prefetch('pizzas', queryset=queryset, to_attr='vegetarian_pizzas'))
>>> vegetarian_pizzas = restaurants[0].vegetarian_pizzas
>>>
>>> # Not recommended:
>>> restaurants = Restaurant.objects.prefetch_related(
...     Prefetch('pizzas', queryset=queryset))
>>> vegetarian_pizzas = restaurants[0].pizzas.all()
~~~~

사용자 지정 prefetching은 forward <tt style="color: #FF0000">`ForeignKey`</tt> 또는 <tt style="color: #FF0000">`OneToOneField`</tt>와 같은 단일 관련 관계에서도 작동합니다. 일반적으로 이러한 관계에는 <tt style="color: #FF0000">`select_related()`</tt>를 사용하려고 하지만 커스텀 <tt style="color: #FF0000">`QuerySet`</tt>으로 prefetch하는 것이 유용할 경우가 많습니다 :
* 관련 모델에 대한 추가 prefetch를 수행하는 <tt style="color: #FF0000">`QuerySet`</tt>을 사용하려고합니다.
* 관련 오브젝트의 서브 세트만 prefetch하려고 합니다.
* [deferred fields](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.query.QuerySet.defer)와 같은 성능 최적화 기술을 사용하려고 합니다.

~~~~
>>> queryset = Pizza.objects.only('name')
>>>
>>> restaurants = Restaurant.objects.prefetch_related(
...     Prefetch('best_pizza', queryset=queryset))
~~~~

<b>Note</b>
조회 순서가 중요합니다.

다음 예를 생각해보십시오.
~~~~
>>> prefetch_related('pizzas__toppings', 'pizzas')
~~~~
이것은 <tt style="color: #FF0000">`'pizzas__toppings'`</tt>에 이미 필요한 모든 정보가 포함되어 있으므로 순서가 없더라도 두 번째 인수 <tt style="color: #FF0000">`'pizzas'`</tt>는 실제로 중복됩니다.
~~~~
>>> prefetch_related('pizzas__toppings', Prefetch('pizzas', queryset=Pizza.objects.all()))
~~~~
이전에 조회한 쿼리 세트를 다시 정의하려는 시도로 인해 <tt style="color: #FF0000">`ValueError`</tt>가 발생합니다. 암시적 쿼리 세트는 <tt style="color: #FF0000">`'pizzas__toppings'`</tt>검색의 일부로 <tt style="color: #FF0000">`'pizzas'`</tt>를 트래버스하기 위해 만들어졌습니다.
~~~~
>>> prefetch_related('pizza_list__toppings', Prefetch('pizzas', to_attr='pizza_list'))
~~~~

<tt style="color: #FF0000">`'pizza_list__toppings'`</tt>이 처리 중일 때 <tt style="color: #FF0000">`'pizza_list'`</tt>가 존재하지 않기 때문에 <tt style="color: #FF0000">`AttributeError`</tt>가 발생합니다.

이 고려 사항은 <tt style="color: #FF0000">`Prefetch`</tt> 객체의 사용에만 국한되지 않습니다. 일부 고급 기술에서는 추가 질의를 만들지 않기 위해 특정 순서로 조회를 수행해야 할 수도 있습니다. 따라서 항상 <tt style="color: #FF0000">`prefetch_related`</tt> 인수를 신중하게 order하는 것이 좋습니다.
<b>Note End</b>

#### extra()

<tt style="color: #FF0000">`extra(select=None, where=None, params=None, tables=None, order_by=None, select_params=None)`</tt>

때로는 Django 쿼리 문법 자체로 복잡한 <tt style="color: #FF0000">`WHERE`</tt> 절을 쉽게 표현할 수 없는 경우가 있습니다. 이러한 엣지의 경우 Django는 <tt style="color: #FF0000">`QuerySet`</tt>에 의해 생성된 SQL에 특정 절을 삽입하기 위한 후크인 <tt style="color: #FF0000">`extra()`</tt> <tt style="color: #FF0000">`QuerySet`</tt> 수정자를 제공합니다.

<b>Use this method as a last resort</b>
이것은 미래의 어느 시점에서 우리가 비추천을 지향하는 오래된 API입니다. 다른 쿼리 세트 방법을 사용하여 쿼리를 표현할 수 없는 경우에만 사용하십시오. 그것을 사용해야하는 경우, <tt style="color: #FF0000">`extra()`</tt>를 제거할 수 있도록 QuerySet API를 개선할 수 있도록 [QuerySet.extra 키워드](https://code.djangoproject.com/query?status=assigned&status=new&keywords=~QuerySet.extra)를 사용하여 티켓을 사용 케이스 (기존 티켓 목록을 먼저 확인하십시오)에 제출하십시오. 우리는 더 이상 이 방법에 대한 버그를 개선하거나 수정하지 않습니다.

예를 들어, 다음과 같이 <tt style="color: #FF0000">`extra()`</tt>를 사용합니다.
~~~~
>>> qs.extra(
...     select={'val': "select col from sometable where othercol = %s"},
...     select_params=(someparam,),
... )
~~~~
변환하면
~~~~
>>> qs.annotate(val=RawSQL("select col from sometable where othercol = %s", (someparam,)))
~~~~
[RawSQL](https://docs.djangoproject.com/en/1.11/ref/models/expressions/#django.db.models.expressions.RawSQL)을 사용하는 주된 이점은 필요한 경우 <tt style="color: #FF0000">`output_field`</tt>를 설정할 수 있다는 것입니다. 주된 단점은 원시 SQL에서 queryset의 테이블 별칭을 참조하면 Django가 별칭을 변경할 가능성이 있습니다 (예 : 쿼리 세트가 다른 쿼리에서 하위 쿼리로 사용되는 경우).

<b>END</b>

<b>Warning</b>
<tt style="color: #FF0000">`extra()`</tt>를 사용할 때마다 매우 조심해야 합니다. 사용자가 사용할 때마다 SQL 주입 공격으로부터 보호하기 위해 <tt style="color: #FF0000">`params`</tt>를 사용하여 사용자가 제어할 수 있는 모든 매개 변수를 이스케이프 처리해야합니다. [SQL injection protection](https://docs.djangoproject.com/en/1.11/topics/security/#sql-injection-protection)에 대해 자세히 읽어보십시오.
<b>End</b>

정의에 따르면 이러한 추가 조회는 다른 데이터베이스 엔진(SQL 코드를 명시적으로 작성하고 있기 때문에)으로 이식할 수 없으며 DRY 원칙을 위반할 수 있으므로 가능하면 피해야합니다.

params, select, where 또는 tables 중 하나 이상을 지정하십시오. 인수는 필요하지 않지만 적어도 하나는 사용해야합니다.

<b>select</b>

<tt style="color: #FF0000">`select`</tt> 인수를 사용하면 <tt style="color: #FF0000">`SELECT`</tt> 절에 추가 필드를 넣을 수 있습니다. 해당 속성을 계산하는 데 사용할 속성 이름을 SQL 절로 맵핑하는 사전이어야합니다.

예시:
~~~~
Entry.objects.extra(select={'is_recent': "pub_date > '2006-01-01'"})
~~~~

결과적으로 각 <tt style="color: #FF0000">`Entry`</tt> 객체에는 Entry의 <tt style="color: #FF0000">`pub_date`</tt>가 2006년 1월 1일보다 큰지 여부를 나타내는 부울값인 <tt style="color: #FF0000">`is_recent`</tt>라는 추가 속성이 있습니다.

Django는 주어진 SQL 스니펫을 SELECT 문에 직접 삽입하므로 위 예제의 결과 SQL은 다음과 같습니다.
~~~~
SELECT blog_entry.*, (pub_date > '2006-01-01') AS is_recent
FROM blog_entry;
~~~~

다음 예제는 보다 고급입니다. 각 결과 <tt style="color: #FF0000">`Blog`</tt> 오브젝트에게 <tt style="color: #FF0000">`entry_count`</tt> 속성, 즉 연관된 <tt style="color: #FF0000">`Entry`</tt> 오브젝트의 정수를 제공하는 부속 조회를 수행합니다.
~~~~
Blog.objects.extra(
    select={
        'entry_count': 'SELECT COUNT(*) FROM blog_entry WHERE blog_entry.blog_id = blog_blog.id'
    },
)
~~~~

이 특별한 경우에는 쿼리에 이미 <tt style="color: #FF0000">`FROM`</tt> 절에 <tt style="color: #FF0000">`blog_blog`</tt> 테이블이 포함되어 있다는 사실을 이용합니다.

위 예제의 결과 SQL은 다음과 같습니다.
~~~~
SELECT blog_blog.*, (SELECT COUNT(*) FROM blog_entry WHERE blog_entry.blog_id = blog_blog.id) AS entry_count
FROM blog_blog;
~~~~

Django의 <tt style="color: #FF0000">`select`</tt> 절에는 서브 쿼리를 중심으로 대부분의 데이터베이스 엔진에 필요한 괄호가 필요하지 않습니다. 또한 일부 MySQL 버전과 같은 일부 데이터베이스 백엔드는 하위 쿼리를 지원하지 않습니다.

드문 경우지만 <tt style="color: #FF0000">`extra(select = ...)`</tt>에서 SQL 파편에 매개 변수를 전달하고자 할 수 있습니다. 이를 위해 <tt style="color: #FF0000">`select_params`</tt> 매개 변수를 사용하십시오. <tt style="color: #FF0000">`select_params`</tt>는 시퀀스이고 <tt style="color: #FF0000">`select`</tt> 속성은 dictionary이므로 매개 변수가 추가 선택 조각과 올바르게 일치하도록 주의를 기울여야 합니다. 이 상황에서는 일반적인 Python dictionary뿐만 아니라 <tt style="color: #FF0000">`select`</tt> 값에 [collections.OrderedDict](https://docs.python.org/3/library/collections.html#collections.OrderedDict)를 사용해야합니다.

예를 들면 다음과 같습니다.
~~~~
Blog.objects.extra(
    select=OrderedDict([('a', '%s'), ('b', '%s')]),
    select_params=('one', 'two'))
~~~~

선택 문자열에서 리터럴 <tt style="color: #FF0000">`%s`</tt>을 사용해야하는 경우 시퀀스 <tt style="color: #FF0000">`%%s`</tt>를 사용하십시오.




#### defer()

#### only()

#### using()

#### select_fot_update()

#### raw()



### Methods that do not return QuerySets

### Field lookups

필드 검색은 SQL <tt style="color: #FF0000">`WHERE`</tt> 절의 내용을 지정하는 방법입니다. 그것들은 <tt style="color: #FF0000">`QuerySet`</tt> 메소드 <tt style="color: #FF0000">`filter()`</tt>, <tt style="color: #FF0000">`exclude()`</tt> 및 <tt style="color: #FF0000">`get()`</tt>에 대한 키워드 인수로 지정됩니다.

[모델 및 데이터베이스 쿼리 설명서](https://docs.djangoproject.com/en/1.11/topics/db/queries/#field-lookups-intro)를 참조하십시오.

장고의 내장 검색은 다음과 같습니다. 모델 필드에 대한 [사용자 정의 lookups](https://docs.djangoproject.com/en/1.11/howto/custom-lookups/)를 작성할 수도 있습니다.

검색 유형이 제공되지 않는 경우 (<tt style="color: #FF0000">`Entry.objects.get(id=14)`</tt>에서와 같이) 조회 유형은 <tt style="color: #FF0000">`exact`</tt>하다고 가정합니다.

#### exact

정확히 일치. 비교를 위해 제공된 값이 <tt style="color: #FF0000">`None`</tt>이면 SQL <tt style="color: #FF0000">`NULL`</tt>로 해석됩니다 (자세한 내용은 [isnull](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#std:fieldlookup-isnull) 참조).

예시:
~~~~
Entry.objects.get(id__exact=14)
Entry.objects.get(id__exact=None)
~~~~

SQL:
~~~~
SELECT ... WHERE id = 14;
SELECT ... WHERE id IS NULL;
~~~~

<b>MySQL comparisons</b>
MySQL에서 데이터베이스 테이블의 "데이터 정렬"설정은 <tt style="color: #FF0000">`exact`</tt>한 비교가 대소 문자를 구분하는지 여부를 결정합니다. 데이터베이스 설정이며 Django 설정이 아닙니다. 대소 문자를 구별하는 비교를 사용하도록 MySQL 테이블을 구성하는 것이 가능하지만, 약간의 절충이 필요합니다. 이에 대한 자세한 내용은 [데이터베이스](https://docs.djangoproject.com/en/1.11/ref/databases/) 설명서의 데이터 [정렬 단원](https://docs.djangoproject.com/en/1.11/ref/databases/#mysql-collation)을 참조하십시오.
<b>END</b>

#### iexact

대소 문자를 구분하지 않는 정확한 일치 비교를 위해 제공된 값이 <tt style="color: #FF0000">`None`</tt>이면 SQL <tt style="color: #FF0000">`NULL`</tt>로 해석됩니다 (자세한 내용은 [isnull](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#std:fieldlookup-isnull) 참조).

예시:
~~~~
Blog.objects.get(name__iexact='beatles blog')
Blog.objects.get(name__iexact=None)
~~~~

SQL:
~~~~
SELECT ... WHERE name ILIKE 'beatles blog';
SELECT ... WHERE name IS NULL;
~~~~

첫 번째 쿼리는 <tt style="color: #FF0000">`'Beatles Blog', 'beatles blog', 'BeAtLes BLoG'`</tt>등과 일치합니다.

<b>SQLite users</b>
SQLite 백엔드 및 유니 코드 (비 ASCII) 문자열을 사용할 때 문자열 비교에 대한 [데이터베이스 참고](https://docs.djangoproject.com/en/1.11/ref/databases/#sqlite-string-matching)를 기억하십시오. SQLite는 유니 코드 문자열에 대해 대소 문자를 구분하지 않습니다.
<b>END</b>

#### contains

대소문자를 구별하는 containment test.

예시:
~~~~
Entry.objects.get(headline__contains='Lennon')
~~~~

SQL:
~~~~
SELECT ... WHERE headline LIKE '%Lennon%';
~~~~

이것은 <tt style="color: #FF0000">`'Lennon honored today'`</tt>헤드 라인과 일치하지만 <tt style="color: #FF0000">`'lennon honored today'`</tt>은 아닙니다.

<b>SQLite users</b>
SQLite는 대소 문자를 구별하는 <tt style="color: #FF0000">`LIKE`</tt> 문을 지원하지 않습니다. SQLite에서 <tt style="color: #FF0000">`contains`</tt>는 <tt style="color: #FF0000">`icontains`</tt>와 같은 동작을 포함합니다. 자세한 정보는 [데이터베이스 참고 사항](https://docs.djangoproject.com/en/1.11/ref/databases/#sqlite-string-matching)을 참조하십시오.
<b>END</b>

#### icontains

대소문자를 구별하지 않는 containment test.

예시:
~~~~
Entry.objects.get(headline__icontains='Lennon')
~~~~

SQL:
~~~~
SELECT ... WHERE headline ILIKE '%Lennon%';
~~~~

<b>SQLite users</b>
SQLite 백엔드 및 유니 코드 (비 ASCII) 문자열을 사용할 때 문자열 비교에 대한 [데이터베이스 참고](https://docs.djangoproject.com/en/1.11/ref/databases/#sqlite-string-matching)를 기억하십시오.
<b>END</b>

#### in

주어진 list

예시:
~~~~
Entry.objects.filter(id__in=[1, 3, 4])
~~~~

SQL:
~~~~
SELECT ... WHERE id IN (1, 3, 4);
~~~~

queryset을 사용하여 리터럴 값 목록을 제공하는 대신 값 목록을 동적으로 평가할 수도 있습니다.
~~~~
inner_qs = Blog.objects.filter(name__contains='Cheddar')
entries = Entry.objects.filter(blog__in=inner_qs)
~~~~

이 queryset는 subselect 문으로 평가됩니다.
~~~~
SELECT ... WHERE blog.id IN (SELECT id FROM ... WHERE NAME LIKE '%Cheddar%')
~~~~

<tt style="color: #FF0000">`__in`</tt> 조회에 값으로 <tt style="color: #FF0000">`values()`</tt> 또는 <tt style="color: #FF0000">`values_list()`</tt>의 결과로 <tt style="color: #FF0000">`QuerySet`</tt>을 전달하면 결과에서 하나의 필드 만 추출해야합니다. 예를 들어 다음과 같이 작동합니다 (블로그 이름 필터링).
~~~~
inner_qs = Blog.objects.filter(name__contains='Ch').values('name')
entries = Entry.objects.filter(blog__name__in=inner_qs)
~~~~

이 예제에서는 내부 쿼리가 두 개 필드 값을 추출하려고 시도하기 때문에 예외가 발생합니다. 단 하나만 예상됩니다.
~~~~
# Bad code! Will raise a TypeError.
inner_qs = Blog.objects.filter(name__contains='Ch').values('name', 'id')
entries = Entry.objects.filter(blog__name__in=inner_qs)
~~~~

<b>Performance considerations</b>
중첩된 쿼리 사용에 주의하고 데이터베이스 서버의 성능 특성을 이해해야합니다 (의심스러운 경우 벤치 마크!). 일부 데이터베이스 백엔드, 특히 MySQL은 중첩된 쿼리를 잘 최적화하지 않습니다. 이 경우 값 목록을 추출한 다음 두 번째 쿼리로 전달하는 것이 더 효율적입니다. 즉, 하나가 아닌 두 개의 쿼리를 실행합니다.
~~~~
values = Blog.objects.filter(
        name__contains='Cheddar').values_list('pk', flat=True)
entries = Entry.objects.filter(blog__in=list(values))
~~~~
첫 번째 쿼리를 강제 실행하려면 Blog <tt style="color: #FF0000">`QuerySet`</tt>에서 <tt style="color: #FF0000">`list()`</tt>를 호출하십시오. 이를 사용하지 않으면 [QuerySets이 게으르기](https://docs.djangoproject.com/en/1.11/topics/db/queries/#querysets-are-lazy) 때문에 중첩된 쿼리가 실행됩니다.
<b>END</b>

#### gt

Greater than.

예시:
~~~~
Entry.objects.filter(id__gt=4)
~~~~

SQL:
~~~~
SELECT ... WHERE id > 4;
~~~~

#### gte

Greater than or equal to.

#### lt

Less than.

#### lte

Less than or equal to.

#### startswith

대소문자 구분 Starts-with.

예시:
~~~~
Entry.objects.filter(headline__startswith='Lennon')
~~~~

SQL:
~~~~
SELECT ... WHERE headline LIKE 'Lennon%';
~~~~

SQLite는 대소 문자를 구별하는 <tt style="color: #FF0000">`LIKE`</tt> 문을 지원하지 않습니다. SQLite에서 <tt style="color: #FF0000">`startswith`</tt>는 <tt style="color: #FF0000">`istartswith`</tt>와 같은 동작을 포함합니다.

#### istartswith

대소문자 구분하지 않는 start-with

예시:
~~~~
Entry.objects.filter(headline__istartswith='Lennon')
~~~~

SQL:
~~~~
SELECT ... WHERE headline ILIKE 'Lennon%';
~~~~

<b>SQLite users</b>
SQLite 백엔드 및 유니 코드 (비 ASCII) 문자열을 사용할 때 문자열 비교에 대한 [데이터베이스 참고](https://docs.djangoproject.com/en/1.11/ref/databases/#sqlite-string-matching)를 기억하십시오.
<b>END</b>

#### endswith

대소문자 구분 end-with

예시:
~~~~
Entry.objects.filter(headline__endswith='Lennon')
~~~~

SQL:
~~~~
SELECT ... WHERE headline LIKE '%Lennon';
~~~~

<b>SQLite users</b>
SQLite는 대소 문자를 구별하는 <tt style="color: #FF0000">`LIKE`</tt> 문을 지원하지 않습니다. <tt style="color: #FF0000">`endswith`</tt>는 SQLite에 대한 <tt style="color: #FF0000">`iendswith`</tt>와 같은 역할을 합니다. 자세한 내용은 [데이터베이스 노트 문서](https://docs.djangoproject.com/en/1.11/ref/databases/#sqlite-string-matching)를 참조하십시오.
<b>END</b>

#### iendswith

대소문자 구별 없는 ends-with

예시:
~~~~
Entry.objects.filter(headline__iendswith='Lennon')
~~~~

SQL:
~~~~
SELECT ... WHERE headline ILIKE '%Lennon'
~~~~

<b>SQLite users</b>
SQLite 백엔드 및 유니 코드 (비 ASCII) 문자열을 사용할 때 문자열 비교에 대한 [데이터베이스 참고](https://docs.djangoproject.com/en/1.11/ref/databases/#sqlite-string-matching)를 기억하십시오.
<b>END</b>

#### range

Range test (inclusive).

예시:
~~~~
import datetime
start_date = datetime.date(2005, 1, 1)
end_date = datetime.date(2005, 3, 31)
Entry.objects.filter(pub_date__range=(start_date, end_date))
~~~~

SQL:
~~~~
SELECT ... WHERE pub_date BETWEEN '2005-01-01' and '2005-03-31';
~~~~

SQL에서 BETWEEN을 사용할 수 있는 곳이면 어느 곳이나 <tt style="color: #FF0000">`range`</tt>를 쓸 수 있습니다. - 날짜, 숫자 및 문자까지 사용할 수 있습니다.

<b>Warning</b>
날짜가 있는 <tt style="color: #FF0000">`DateTimeField`</tt>를 필터링하는 것은 마지막 날의 항목을 포함하지 않습니다. 왜냐하면 경계가 "주어진 날짜의 0am"으로 해석되기 때문입니다. <tt style="color: #FF0000">`pub_date`</tt>가 <tt style="color: #FF0000">`DateTimeField`</tt> 인 경우 위의 표현식이 이 SQL로 변환됩니다.
~~~~
SELECT ... WHERE pub_date BETWEEN '2005-01-01 00:00:00' and '2005-03-31 00:00:00';
~~~~
일반적으로 날짜와 시간대를 섞어서는 안됩니다.
<b>END</b>

#### date

datetime 필드의 경우, 값을 날짜로 변환합니다. chaining additional 필드 조회를 허용합니다. 날짜 값을 받습니다.

예시:
~~~~
Entry.objects.filter(pub_date__date=datetime.date(2005, 1, 1))
Entry.objects.filter(pub_date__date__gt=datetime.date(2005, 1, 1))
~~~~

(관련 쿼리의 구현이 다른 데이터베이스 엔진마다 다르기 때문에 이 lookup에는 동등한 SQL 코드 조각이 포함되어 있지 않습니다.)

[USE_TZ](https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-USE_TZ)가 <tt style="color: #FF0000">`True`</tt>이면 필터링 전에 필드가 현재 시간대로 변환됩니다.

#### year

date 및 datetime 필드의 경우 정확히 일치하는 연도입니다. chaining additional 필드 조회를 허용합니다. 정수 연도를 취합니다.

예시:
~~~~
Entry.objects.filter(pub_date__year=2005)
Entry.objects.filter(pub_date__year__gte=2005)
~~~~

SQL:
~~~~
SELECT ... WHERE pub_date BETWEEN '2005-01-01' AND '2005-12-31';
SELECT ... WHERE pub_date >= '2005-01-01';
~~~~

(정확한 SQL 구문은 각 데이터베이스 엔진마다 다릅니다.)

[USE_TZ](https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-USE_TZ)가 <tt style="color: #FF0000">`True`</tt>이면 필터링하기 전에 datetime 필드가 현재 시간대로 변환됩니다.

#### month

date 및 datetime 필드의 경우 정확히 일치하는 month입니다. chaining additional 필드 조회를 허용합니다. 1 (1월)에서 12 (12월) 사이의 정수를 사용합니다.

예시:
~~~~
Entry.objects.filter(pub_date__month=12)
Entry.objects.filter(pub_date__month__gte=6)
~~~~

SQL:
~~~~
SELECT ... WHERE EXTRACT('month' FROM pub_date) = '12';
SELECT ... WHERE EXTRACT('month' FROM pub_date) >= '6';
~~~~

(정확한 SQL 구문은 각 데이터베이스 엔진마다 다릅니다.)

[USE_TZ](https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-USE_TZ)가 <tt style="color: #FF0000">`True`</tt>이면 필터링하기 전에 datetime 필드가 현재 시간대로 변환됩니다. 이렇게 하려면 [데이터베이스에 표준 시간대 정의](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#database-time-zone-definitions)가 필요합니다.

#### day

#### week

#### week_day

#### time

#### hour

#### minute

#### second

#### isnull

각각 <tt style="color: #FF0000">`IS NULL`</tt> 및 <tt style="color: #FF0000">`IS NOT NULL`</tt>의 SQL 쿼리에 해당하는 <tt style="color: #FF0000">`True`</tt> 또는 <tt style="color: #FF0000">`False`</tt>를 취합니다.

예시:
~~~~
Entry.objects.filter(pub_date__isnull=True)
~~~~

SQL:
~~~~
SELECT ... WHERE pub_date IS NULL;
~~~~

#### search

#### regex

대소 문자를 구분하는 정규식 일치.

정규식 구문은 사용중인 데이터베이스 백엔드 구문입니다. 정규 표현식을 지원하지 않는 SQLite의 경우, 이 기능은 (파이썬) 사용자 정의 REGEXP 함수에 의해 제공되므로 일반 표현식 구문은 파이썬의 <tt style="color: #FF0000">`re`</tt> 모듈의 구문입니다.

예시:
~~~~
Entry.objects.get(title__regex=r'^(An?|The) +')
~~~~

SQL:
~~~~
SELECT ... WHERE title REGEXP BINARY '^(An?|The) +'; -- MySQL

SELECT ... WHERE REGEXP_LIKE(title, '^(An?|The) +', 'c'); -- Oracle

SELECT ... WHERE title ~ '^(An?|The) +'; -- PostgreSQL

SELECT ... WHERE title REGEXP '^(An?|The) +'; -- SQLite
~~~~

정규식 구문을 전달하기 위해 원시 문자열(예: <tt style="color: #FF0000">`'foo'`</tt> 대신 <tt style="color: #FF0000">`r'foo'`</tt>)을 사용하는 것이 좋습니다.

#### iregex

대 / 소문자를 구분하지 않는 정규식 일치.

예시:
~~~~
Entry.objects.get(title__iregex=r'^(an?|the) +')
~~~~

SQL:
~~~~
SELECT ... WHERE title REGEXP '^(an?|the) +'; -- MySQL

SELECT ... WHERE REGEXP_LIKE(title, '^(an?|the) +', 'i'); -- Oracle

SELECT ... WHERE title ~* '^(an?|the) +'; -- PostgreSQL

SELECT ... WHERE title REGEXP '(?i)^(an?|the) +'; -- SQLite
~~~~

















### Aggregation functions


## Query-related tools

이 절에서는 다른 곳에서 문서화되지 않은 쿼리 관련 도구에 대한 참조 자료를 제공합니다.

### Q() objects

<tt style="color: #FF0000">`class Q`</tt> [source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/query_utils/#Q)

<tt style="color: #FF0000">`F`</tt> 객체와 같은 <tt style="color: #FF0000">`Q()`</tt> 객체는 데이터베이스 관련 작업에 사용할 수 있는 Python 객체의 SQL 표현식을 캡슐화합니다.

일반적으로 <tt style="color: #FF0000">`Q() object`</tt>를 사용하면 조건을 정의하고 다시 사용할 수 있습니다. 이렇게하면 <tt style="color: #FF0000">`| (OR) 및 & (AND)`</tt> 연산자를 사용하여 [복잡한 데이터베이스 조회를 구성](https://docs.djangoproject.com/en/1.11/topics/db/queries/#complex-lookups-with-q)할 수 있습니다; 특히 <tt style="color: #FF0000">`QuerySets`</tt>에서 <tt style="color: #FF0000">`OR`</tt>을 사용하는 것이 가능하지 않습니다.

### Prefetch() objects

<tt style="color: #FF0000">`class Prefetch(lookup, queryset=None, to_attr=None)`</tt> [source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/query/#Prefetch)

<tt style="color: #FF0000">`Prefetch()`</tt> 객체는 <tt style="color: #FF0000">`prefetch_related()`</tt>의 동작을 제어하는데 사용할 수 있습니다.

<tt style="color: #FF0000">`lookup`</tt> 인수는 <tt style="color: #FF0000">`prefetch_related()`</tt>에 전달된 문자열 기반 조회와 동일하게 작동하고 따르는 관계를 설명합니다. 예 :
~~~~
>>> from django.db.models import Prefetch
>>> Question.objects.prefetch_related(Prefetch('choice_set')).get().choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
# This will only execute two queries regardless of the number of Question
# and Choice objects.
>>> Question.objects.prefetch_related(Prefetch('choice_set')).all()
<QuerySet [<Question: What's up?>]>
~~~~

<tt style="color: #FF0000">`queryset`</tt> 인수는 지정된 lookup에 대한 기본 <tt style="color: #FF0000">`QuerySets`</tt>을 제공합니다. 이는 prefetch operation을 추가로 필터링하거나 prefetched relation에서 <tt style="color: #FF0000">`select_related()`</tt>를 호출하여 쿼리의 수를 더 줄이는 데 유용합니다.
~~~~
>>> voted_choices = Choice.objects.filter(votes__gt=0)
>>> voted_choices
<QuerySet [<Choice: The sky>]>
>>> prefetch = Prefetch('choice_set', queryset=voted_choices)
>>> Question.objects.prefetch_related(prefetch).get().choice_set.all()
<QuerySet [<Choice: The sky>]>
~~~~

<tt style="color: #FF0000">`to_attr`</tt> 인수는 prefetch operation의 결과를 사용자 정의 속성으로 설정합니다.
~~~~
>>> prefetch = Prefetch('choice_set', queryset=voted_choices, to_attr='voted_choices')
>>> Question.objects.prefetch_related(prefetch).get().voted_choices
<QuerySet [<Choice: The sky>]>
>>> Question.objects.prefetch_related(prefetch).get().choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
~~~~

<b>Note</b>
<tt style="color: #FF0000">`to_attr`</tt>을 사용할 때 prefetched result가 목록에 저장됩니다. 이렇게하면 캐시된 결과를 <tt style="color: #FF0000">`QuerySets`</tt> 인스턴스 내에 저장하는 기존의 <tt style="color: #FF0000">`prefetch_related()`</tt> 호출에 비해 속도가 크게 향상 될 수 있습니다.
<b>END</b>

### prefetch_related_objects()

<tt style="color: #FF0000">`prefetch_related_objects(model_instances, *related_lookups)`</tt> [source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/query/#prefetch_related_objects)

<b>New in Django 1.10