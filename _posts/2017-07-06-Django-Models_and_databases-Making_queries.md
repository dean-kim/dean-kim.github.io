---
layout: post
title:  "Django Models_and_databases-Making_queries"
date:   2017-07-06 10:43:59
author: Dean Kim
categories: Django
tags:	Django Models databases queries
cover:  "/assets/instacode.png"
---

# Making queries
- 원본 : [공식문서](https://docs.djangoproject.com/en/1.11/topics/db/queries/)


[data model](https://docs.djangoproject.com/en/1.11/topics/db/models/)을 만들면 Django는 객체를 생성, 검색, 업데이트 및 삭제할 수 있는 데이터베이스 추상화 API를 자동으로 제공합니다. 이 문서는 이 API를 사용하는 방법을 설명합니다. 다양한 모델 조회 옵션에 대한 자세한 내용은 [data model reference](https://docs.djangoproject.com/en/1.11/ref/models/)를 참조하십시오.

이 가이드(및 참조) 전체에서 Weblog 응용 프로그램을 구성하는 다음 모델을 참조할 것입니다.
~~~~
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):              # __unicode__ on Python 2
        return self.headline
~~~~

## Creating objects

Django는 Python 객체에서 데이터베이스 테이블 데이터를 나타내기 위해 직관적인 시스템을 사용합니다. 모델 클래스는 데이터베이스 테이블을 나타내며 클래스의 인스턴스는 데이터베이스 테이블의 특정 레코드를 나타냅니다.

객체를 생성하려면 키워드 인자를 사용하여 모델 클래스에 인스턴스화한 다음 <tt style="color: #FF0000">`save()`</tt>를 호출하여 객체를 데이터베이스에 저장합니다.

모델이 mysite/blog/models.py 파일에 있다고 가정하면 다음과 같은 예가 있습니다.
~~~~
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()
~~~~
이것은 뒤에서 <tt style="color: #FF0000">`INSERT`</tt> SQL 문을 수행합니다. Django는 명시적으로 <tt style="color: #FF0000">`save()`</tt>를 호출할 때까지 데이터베이스에 접근하지 않습니다.

<tt style="color: #FF0000">`save()`</tt> 메소드에는 반환값이 없습니다.

<b>See also</b>
[save()](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model.save)에는 여기에 설명되지 않은 여러 고급 옵션이 필요합니다. 자세한 내용은 [save()](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model.save)에 대한 설명서를 참조하십시오.
단일 단계로 객체를 만들고 저장하려면 [create()](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.query.QuerySet.create) 메소드를 사용하십시오.

## Saving changes to objects

이미 데이터베이스에 있는 오브젝트의 변경 사항을 저장하려면 <tt style="color: #FF0000">`save()`</tt>를 사용하십시오.

이미 데이터베이스에 저장된 Blog 인스턴스 b5가 주어지면 이 예제는 이름을 변경하고 데이터베이스에서 해당 레코드를 업데이트합니다.
~~~~
>>> b5.name = 'New name'
>>> b5.save()
~~~~
이것은 뒤에서 <tt style="color: #FF0000">`UPDATE`</tt> SQL 문을 수행합니다. Django는 명시적으로 save()를 호출할 때까지 데이터베이스에 접근하지 않습니다.

### Saving ForeignKey and ManyToManyField fields

[ForeignKey](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey) 필드를 업데이트하는 것은 정상 필드를 저장하는 것과 똑같은 방식으로 작동합니다. 문제의 필드에 올바른 유형의 개체를 할당하기만하면 됩니다. 이 예는 `Entry` 및 `Blog`의 적절한 인스턴스가 이미 데이터베이스에 저장되었다고 가정하고 `Entry` 인스턴스 항목의 `blog` 속성을 업데이트합니다 (아래에서 해당 항목을 검색할 수 있음).
~~~~
>>> from blog.models import Blog, Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog
>>> entry.save()
~~~~
[ManyToManyField](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField)를 업데이트하는 것은 약간 다르게 작동합니다. 필드에 [add()](https://docs.djangoproject.com/en/1.11/ref/models/relations/#django.db.models.fields.related.RelatedManager.add) 메소드를 사용하여 관계에 레코드를 추가합니다. 이 예제에서는 `Author` 인스턴스 `joe`를 `entry` 객체에 추가합니다.
~~~~
>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
~~~~
한 번에 [ManyToManyField](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField)에 여러 레코드를 추가하려면 다음과 같이 [add()](https://docs.djangoproject.com/en/1.11/ref/models/relations/#django.db.models.fields.related.RelatedManager.add) 호출에 여러 인수를 포함시킵니다.
~~~~
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
~~~~
Django는 잘못된 유형의 객체를 할당하거나 추가하려고 하면 complain할 것입니다.

## Retrieving objects

데이터베이스에서 객체를 검색하려면 모델 클래스의 [Manager](https://docs.djangoproject.com/en/1.11/topics/db/managers/#django.db.models.Manager)를 통해 [QuerySet](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.query.QuerySet)을 생성하십시오.

<tt style="color: #FF0000">`QuerySet`</tt>는 데이터베이스의 객체 컬렉션을 나타냅니다. 0개, 하나 또는 여러 개의 필터를 가질 수 있습니다. 필터는 주어진 매개 변수를 기반으로 query 결과의 범위를 좁힙니다. SQL 용어에서 <tt style="color: #FF0000">`QuerySet`</tt>은 <tt style="color: #FF0000">`SELECT`</tt> 문과 같고 필터는 <tt style="color: #FF0000">`WHERE`</tt> 또는 <tt style="color: #FF0000">`LIMIT`</tt>와 같은 제한 절입니다.

모델의 <tt style="color: #FF0000">`Manager`</tt>를 사용하여 <tt style="color: #FF0000">`QuerySet`</tt>을 얻습니다. 각 모델에는 최소 하나 이상의 <tt style="color: #FF0000">`Manager`</tt>가 있으며 기본적으로 [objects](https://docs.djangoproject.com/en/1.11/ref/models/class/#django.db.models.Model.objects)라고 합니다. 다음과 같이 모델 클래스를 통해 직접 액세스하십시오.
~~~~
>>> Blog.objects
<django.db.models.manager.Manager object at ...>
>>> b = Blog(name='Foo', tagline='Bar')
>>> b.objects
Traceback:
    ...
AttributeError: "Manager isn't accessible via Blog instances."
~~~~

<b>Note</b>
<br><tt style="color: #FF0000">`Managers`</tt>는 모델 인스턴스가 아닌 모델 클래스를 통해서만 액세스할 수 있으므로 "table-level" 작업과 "record-level" 작업을 구분할 수 있습니다.


<tt style="color: #FF0000">`Manager`</tt>는 모델에 대한 <tt style="color: #FF0000">`QuerySets`</tt>의 주요 소스입니다. 예를 들어 <tt style="color: #FF0000">`Blog.objects.all()`</tt>은 데이터베이스의 모든 <tt style="color: #FF0000">`Blog`</tt> 객체를 포함하는 <tt style="color: #FF0000">`QuerySet`</tt>을 반환합니다.

### Retrieving all objects

테이블에서 개체를 검색하는 가장 간단한 방법은 모든 개체를 가져오는 것입니다. 이렇게하려면 [Manager](https://docs.djangoproject.com/en/1.11/topics/db/managers/#django.db.models.Manager)에서 [all()](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.query.QuerySet.all) 메서드를 사용합니다.
~~~~
>>> all_entries = Entry.objects.all()
~~~~
<tt style="color: #FF0000">`all()`</tt> 메서드는 데이터베이스에 있는 모든 객체의 <tt style="color: #FF0000">`QuerySet`</tt>을 반환합니다.

### Retrieving specific objects with filter

<tt style="color: #FF0000">`all()`</tt>에 의해 반환된 <tt style="color: #FF0000">`QuerySet`</tt>은 데이터베이스 테이블의 모든 객체를 설명합니다. 일반적으로 개체의 전체 집합 중 일부만 선택해야합니다.

이러한 하위 집합을 만들려면 필터 조건을 추가하여 초기 <tt style="color: #FF0000">`QuerySet`</tt>을 구체화하십시오. <tt style="color: #FF0000">`QuerySet`</tt>을 구체화하는 가장 일반적인 두 가지 방법은 다음과 같습니다.

<tt style="color: #FF0000">`filter(**kwargs)`</tt>
지정된 검색 매개 변수와 일치하는 객체가 포함된 새 <tt style="color: #FF0000">`QuerySet`</tt>을 반환합니다.
<tt style="color: #FF0000">`exclude(**kwargs)`</tt>
지정된 조회 매개 변수와 일치하지 않는 객체가 포함된 새 <tt style="color: #FF0000">`QuerySet`</tt>을 반환합니다.

lookup 매개 변수 (<tt style="color: #FF0000">`**kwargs`</tt> 위의 함수 정의에 포함된)는 아래의 [Field lookups](https://docs.djangoproject.com/en/1.11/topics/db/queries/#field-lookups)에 설명된 형식이어야합니다.

예를 들어, 2006년부터 블로그 항목의 <tt style="color: #FF0000">`QuerySet`</tt>을 얻으려면 <tt style="color: #FF0000">`filter()`</tt>를 다음과 같이 사용하십시오.
~~~~
Entry.objects.filter(pub_date__year=2006)
~~~~
default manager 클래스는 다음과 같습니다.
~~~~
Entry.objects.all().filter(pub_date__year=2006)
~~~~

#### Chaining filters

<tt style="color: #FF0000">`QuerySet`</tt>을 정제한 결과는 <tt style="color: #FF0000">`QuerySet`</tt> 그 자체이므로 상세 검색을 함께 연결할 수 있습니다. 예 :
~~~~
>>> Entry.objects.filter(
...     headline__startswith='What'
... ).exclude(
...     pub_date__gte=datetime.date.today()
... ).filter(
...     pub_date__gte=datetime(2005, 1, 30)
... )
~~~~
이것은 데이터베이스의 모든 항목에 대한 초기 <tt style="color: #FF0000">`QuerySet`</tt>을 가져와서 필터를 추가한 다음 제외 항목을 추가한 다음 다른 필터를 추가합니다. 마지막 결과는 2005년 1월 30일과 현재 날짜 사이에 발행된 "What"으로 시작하는 제목이 있는 모든 항목을 포함하는 <tt style="color: #FF0000">`QuerySet`</tt>입니다.

#### Filtered QuerySets are unique

<tt style="color: #FF0000">`QuerySet`</tt>을 구체화할 때마다 이전 <tt style="color: #FF0000">`QuerySet`</tt>에 바인딩된 새로운 <tt style="color: #FF0000">`QuerySet`</tt>을 얻게됩니다. 각 상세 검색은 저장되고 사용되며 재사용될 수 있는 별개의 고유한 <tt style="color: #FF0000">`QuerySet`</tt>을 생성합니다.

예:
~~~~
>>> q1 = Entry.objects.filter(headline__startswith="What")
>>> q2 = q1.exclude(pub_date__gte=datetime.date.today())
>>> q3 = q1.filter(pub_date__gte=datetime.date.today())
~~~~

이 세 가지 <tt style="color: #FF0000">`QuerySet`</tt>은 별개입니다. 첫 번째는 "What"로 시작하는 제목을 포함하는 모든 항목을 포함하는 <tt style="color: #FF0000">`base QuerySet`</tt>입니다. 두 번째는 첫 번째 레코드의 하위 집합이며 <tt style="color: #FF0000">`pub_date`</tt>가 현재 또는 미래의 레코드를 제외하는 추가 조건이 있습니다. 세 번째는 첫 번째 레코드의 하위 집합이며 <tt style="color: #FF0000">`pub_date`</tt>가 현재 또는 미래의 레코드만 선택하는 추가 조건이 있습니다. 초기 <tt style="color: #FF0000">`QuerySet(q1)`</tt>은 정제 프로세스의 영향을 받지 않습니다.

#### QuerySets are lazy

<tt style="color: #FF0000">`QuerySet`</tt> are lazy - <tt style="color: #FF0000">`QuerySet`</tt>을 생성하는 행위는 어떤 데이터베이스 활동도 포함하지 않는다. 하루 종일 필터를 함께 쌓을 수 있으며, 장고는 <tt style="color: #FF0000">`QuerySet`</tt>이 평가될 때까지 실제로 query를 실행하지 않습니다. 이 예제를 살펴보십시오.
~~~~
>>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.date.today())
>>> q = q.exclude(body_text__icontains="food")
>>> print(q)
~~~~
이 데이터베이스는 세 번의 데이터베이스 히트와 비슷하지만 실제로 마지막 행 <tt style="color: #FF0000">`(print(q))`</tt>에서 데이터베이스에 한 번만 도달합니다. 일반적으로 <tt style="color: #FF0000">`QuerySet`</tt>의 결과는 사용자가 "요청"할 때까지 데이터베이스에서 가져오지 않습니다. 그렇게하면 <tt style="color: #FF0000">`QuerySet`</tt>은 데이터베이스에 액세스하여 평가됩니다. 평가가 수행되는 시기에 대한 자세한 내용은 [When QuerySets are evaluated](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#when-querysets-are-evaluated)를 참조하십시오.

### Retrieving a single object with get()

[filter()](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.query.QuerySet.filter)는 단일 객체만이 query와 일치하는 경우에도 항상 <tt style="color: #FF0000">`QuerySet`</tt>을 제공합니다. 이 경우 단일 요소를 포함하는 <tt style="color: #FF0000">`QuerySet`</tt>이 됩니다.

query와 일치하는 객체가 하나 뿐인 경우 객체를 직접 반환하는 <tt style="color: #FF0000">`Manager`</tt>에서 <tt style="color: #FF0000">`get()`</tt> 메서드를 사용할 수 있습니다.
~~~~
>>> one_entry = Entry.objects.get(pk=1)
~~~~
<tt style="color: #FF0000">`filter()`</tt>와 마찬가지로 <tt style="color: #FF0000">`get()`</tt>과 함께 모든 query expression을 사용할 수 있습니다. [Field lookups](https://docs.djangoproject.com/en/1.11/topics/db/queries/#field-lookups)를 참조하십시오.

<tt style="color: #FF0000">`get()`</tt>을 사용하는 것과 `[0]` 슬라이스로 <tt style="color: #FF0000">`filter()`</tt>를 사용하는 것 사이에는 차이가 있음에 유의하십시오. query와 일치하는 결과가 없으면 <tt style="color: #FF0000">`get()`</tt>은 <tt style="color: #FF0000">`DoesNotExist`</tt> 예외를 발생시킵니다. 이 예외는 query가 수행되는 모델 클래스의 속성입니다. 위 코드에서 primary key가 1 인 <tt style="color: #FF0000">`Entry`</tt> 객체가 없으면 Django는 <tt style="color: #FF0000">`Entry.DoesNotExist`</tt>를 발생시킵니다.

마찬가지로 장고는 하나 이상의 항목이 <tt style="color: #FF0000">`get()`</tt> query와 일치하는 경우 complain을 제기합니다. 이 경우, model class 자체의 속성인 [MultipleObjectsReturned](https://docs.djangoproject.com/en/1.11/ref/exceptions/#django.core.exceptions.MultipleObjectsReturned)가 발생합니다.

### Other QuerySet methods

대부분 데이터베이스에서 객체를 검색해야 할 때 <tt style="color: #FF0000">`all()`</tt>, <tt style="color: #FF0000">`get()`</tt>, <tt style="color: #FF0000">`filter()`</tt> 및 <tt style="color: #FF0000">`exclude()`</tt>를 사용합니다. 그러나 이것이 전부는 아닙니다. 모든 다양한 <tt style="color: #FF0000">`QuerySet`</tt> 메소드의 전체 목록은 [QuerySet API Reference](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#queryset-api)를 참조하십시오.

### Limiting QuerySets

Python의 배열 슬라이스 구문의 서브 세트를 사용하여 QuerySet을 특정 수의 결과로 제한하십시오. 이것은 SQL의 <tt style="color: #FF0000">`LIMIT`</tt> 및 <tt style="color: #FF0000">`OFFSET`</tt> 절과 동일합니다.

예를 들어, 처음 5개 개체 (<tt style="color: #FF0000">`LIMIT 5`</tt>)를 반환합니다.
~~~~
>>> Entry.objects.all()[:5]
~~~~
여섯 번째부터 열 번째까지의 객체를 반환합니다 (<tt style="color: #FF0000">`OFFSET 5 LIMIT 5`</tt>).
~~~~
>>> Entry.objects.all()[5:10]
~~~~
제외 지수 (즉, <tt style="color: #FF0000">`Entry.objects.all()[- 1]`</tt>)는 지원되지 않습니다.

일반적으로 <tt style="color: #FF0000">`QuerySet`</tt>을 슬라이스하면 새 <tt style="color: #FF0000">`QuerySet`</tt>가 반환됩니다. query를 평가하지는 않습니다. 예외는 파이썬 슬라이스 구문의 "step"매개 변수를 사용하는 경우입니다. 예를들어 첫 번째 10 번째 두 번째 객체의 목록을 반환하기 위해 실제로 query를 실행합니다.
~~~~
>>> Entry.objects.all()[:10:2]
~~~~
목록이 아닌 단일 객체를 검색하려면 (예 : <tt style="color: #FF0000">`SELECT foo FROM bar LIMIT 1`</tt>) 슬라이스 대신 간단한 색인을 사용하십시오. 예를들어, 항목을 제목순으로 알파벳 순으로 정렬한 후 데이터베이스의 첫 번째 <tt style="color: #FF0000">`Entry`</tt>을 반환합니다.
~~~~
>>> Entry.objects.order_by('headline')[0]
~~~~
이것은 대략 다음과 같습니다.
~~~~
>>> Entry.objects.order_by('headline')[0:1].get()
~~~~
그러나 이 중 첫 번째 항목은 <tt style="color: #FF0000">`IndexError`</tt>를 발생시키고 두 번째 항목은 지정된 조건과 일치하는 개체가 없는 경우 <tt style="color: #FF0000">`DoesNotExist`</tt>를 발생시킵니다. 자세한 내용은 <tt style="color: #FF0000">`get()`</tt>을 참조하십시오.

### Field lookups

필드 검색은 SQL <tt style="color: #FF0000">`WHERE`</tt> 절의 meat를 지정하는 방법입니다. 그것들은 <tt style="color: #FF0000">`QuerySet`</tt> 메소드 <tt style="color: #FF0000">`filter()`</tt>, <tt style="color: #FF0000">`exclude()`</tt> 및 <tt style="color: #FF0000">`get()`</tt>에 대한 키워드 인수로 지정됩니다.

기본 검색 키워드 인수는 <tt style="color: #FF0000">`field__lookuptype=value`</tt> 형식을 취합니다. (double-underscore). 예 :
~~~~
>>> Entry.objects.filter(pub_date__lte='2006-01-01')
~~~~
대략적인 SQL 문으로 변환하면:
~~~~
SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';
~~~~

<b>How this is possible</b>
<br>파이썬에는 이름과 값이 런타임에 평가되는 임의의 이름 - 값 인수를 허용하는 함수를 정의할 수 있는 기능이 있습니다. 자세한 내용은 공식 Python tutorial의 [Keyword Arguments](https://docs.python.org/3/tutorial/controlflow.html#tut-keywordargs)를 참조하십시오.

조회에서 지정된 필드는 모델 필드의 이름이어야합니다. 하지만 한 가지 예외가 있습니다. <tt style="color: #FF0000">`ForeignKey`</tt>의 경우 <tt style="color: #FF0000">`_id`</tt> 접미사로 필드 이름을 지정할 수 있습니다. 이 경우, value 매개 변수에는 외부 모델의 primary key의 raw value이 포함될 것으로 예상됩니다. 예 :
~~~~
>>> Entry.objects.filter(blog_id=4)
~~~~
잘못된 키워드 인수를 전달하면 조회 함수가 <tt style="color: #FF0000">`TypeError`</tt>를 발생시킵니다.

데이터베이스 API는 약 20개의 조회 유형을 지원합니다. [field lookup reference](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#field-lookups)에서 완전한 참조를 찾을 수 있습니다. 사용할 수 있는 것을 제공하기 위해 다음과 같은 몇 가지 일반적인 조회가 사용됩니다.

[exact](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#std:fieldlookup-exact)
<br>"exact" 일치. 예 :
~~~~
>>> Entry.objects.get(headline__exact="Cat bites dog")
~~~~
다음의 SQL을 생성할 수 있습니다.
~~~~
SELECT ... WHERE headline = 'Cat bites dog';
~~~~
lookup 유형을 제공하지 않으면 (즉, 키워드 인수에 밑줄이 두 개 포함되어 있지 않은 경우) lookup 유형이 <tt style="color: #FF0000">`exact`</tt>하다고 가정합니다.

예를 들어, 다음 두 문은 동일합니다.
~~~~
>>> Blog.objects.get(id__exact=14)  # Explicit form
>>> Blog.objects.get(id=14)         # __exact is implied
~~~~
이는 <tt style="color: #FF0000">`exact`</tt> lookups가 일반적인 경우이므로 편의상 제공됩니다.

[iexact](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#std:fieldlookup-iexact)
<br>대소 문자를 구분하지 않는 일치입니다. query는 :
~~~~
>>> Blog.objects.get(name__iexact="beatles blog")
~~~~
<tt style="color: #FF0000">`'Beatles Blog'`</tt>, <tt style="color: #FF0000">`'beatles blog'`</tt>또는 <tt style="color: #FF0000">`'BeAtlES blog'`</tt>라는 블로그와 일치합니다.

[contains](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#std:fieldlookup-contains)
<br>대소 문자를 구별하는 봉쇄 test. 예 :
~~~~
Entry.objects.get(headline__contains='Lennon')
~~~~
대략적인 SQL 문으로 변환하면:
~~~~
SELECT ... WHERE headline LIKE '%Lennon%';
~~~~
이것은 <tt style="color: #FF0000">`'Today Lennon honored'`</tt>헤드 라인과 일치하지만 <tt style="color: #FF0000">`'today lennon honored'`</tt>은 아닙니다.

대소 문자를 구별하지 않는 버전인 [icontains](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#std:fieldlookup-icontains)도 있습니다.

[startswith](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#std:fieldlookup-startswith), [endswith](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#std:fieldlookup-endswith)
<br>Starts-with, ends-with 검색과 함께 끝납니다. [istartswith]() 및 [iendswith]()라고 하는 대소 문자를 구별하지 않는 버전도 있습니다.
다시 말하지만, 이 정도는 수박 겉핥기 입니다. [field lookup reference](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#field-lookups)에서 완전한 참조를 찾을 수 있습니다.

### Lookups that span relationships

Django는 백그라운드에서 자동으로 SQL <tt style="color: #FF0000">`JOIN`</tt>을 관리하면서 조회시 관계를 "따르도록" 강력하고 직관적인 방법을 제공합니다. 관계를 확장하려면 원하는 필드에 도달할 때까지 모델에서 관련 필드의 필드 이름을 double underscore로 구분하여 사용하십시오.

이 예제는 <tt style="color: #FF0000">`Blog`</tt> 이름이 <tt style="color: #FF0000">`'Beatles Blog'`</tt>인 모든 <tt style="color: #FF0000">`Entry`</tt> 객체를 검색합니다.
~~~~
Entry.objects.filter(blog__name='Beatles Blog')
~~~~
이 스패닝은 원하는만큼 깊을 수 있습니다.

거꾸로도 작동합니다. "역"관계를 나타내기 위해서는 모델의 소문자 이름을 사용하십시오.

이 예제는 헤드 라인에 'Lennon'이 포함 된 항목이 하나 이상있는 모든 Blog 객체를 검색합니다.
~~~~
>>> Blog.objects.filter(entry__headline__contains='Lennon')
~~~~
여러 관계를 거쳐 필터링하고 중간 모델 중 하나가 필터 조건을 만족하는 값을 가지지 않는다면 Django는 빈 상태 (모든 값이 <tt style="color: #FF0000">`NULL`</tt> 임)인 것처럼 처리하지만 유효한 개체는 거기에 있습니다. 이것은 오류가 발생하지 않는다는 것을 의미합니다. 예를 들어이 필터에서 다음을 수행합니다.
~~~~
Blog.objects.filter(entry__authors__name__isnull=True)
~~~~
작성자에 빈 이름이 들어있는 블로그 개체와 항목에 빈 작성자가 있는 블로그 개체를 반환합니다. 후자의 객체를 원하지 않으면 다음과 같이 작성할 수 있습니다.
~~~~
Blog.objects.filter(entry__authors__isnull=False, entry__authors__name__isnull=True)
~~~~

#### Spanning multi-valued relationships

<tt style="color: #FF0000">`ManyToManyField`</tt> 또는 역방향 <tt style="color: #FF0000">`ForeignKey`</tt>를 기반으로 개체를 필터링할 때 관심있는 두 가지 종류의 필터가 있습니다. <tt style="color: #FF0000">`Blog/Entry`</tt> 관계를 고려하십시오 (<b>Blog</b>와 <b>Entry</b>는 일대다 관계입니다). 우리는 헤드 라인에 "Lennon"과 2008년에 출판된 entry가 있는 블로그를 찾는데 관심이 있을 것입니다. 또는 헤드 라인에 "Lennon"이라는 entry가 있는 블로그와 하나의 블로그와 관련된 여러 entry가 있으므로 이 두 가지 query가 가능하며 어떤 경우에는 의미가 있습니다.

<tt style="color: #FF0000">`ManyToManyField`</tt>와 동일한 유형의 상황이 발생합니다. 예를 들어 항목에 <tt style="color: #FF0000">`ManyToManyField`</tt>라는 <tt style="color: #FF0000">`tags`</tt>가 있는 경우 "music"및 "bands"라는 태그에 연결된 항목을 찾거나 "music"이라는 이름의 태그와 상태를 포함하는 항목을 원할 수 있습니다 "public".

이러한 상황을 처리하기 위해 Django는 <tt style="color: #FF0000">`filter()`</tt> 호출을 일관성 있게 처리합니다. 단일 <tt style="color: #FF0000">`filter()`</tt> 호출 내부의 모든 항목이 동시에 적용되어 모든 요구 사항과 일치하는 항목을 필터링합니다. 연속적인 <tt style="color: #FF0000">`filter()`</tt> 호출은 객체 세트를 추가로 제한하지만 다중 값 관계는 이전 <tt style="color: #FF0000">`filter()`</tt> 호출로 선택된 객체가 아닌 기본 모델에 링크된 모든 객체에 적용됩니다.

약간 혼란스러워 보일지도 모릅니다. 그래서 예제가 명확해집니다. 표제에서 "Lennon"과 2008년에 출판된 entry (두 조건을 모두 충족하는 동일한 항목)이 모두 포함된 블로그를 모두 선택하려면 다음과 같이 작성하십시오.
~~~~
Blog.objects.filter(entry__headline__contains='Lennon', entry__pub_date__year=2008)
~~~~
제목에 'Lennon'이 포함된 항목과 2008년에 게시된 항목이 포함된 블로그를 모두 선택하려면 다음과 같이 작성하십시오.
~~~~
Blog.objects.filter(entry__headline__contains='Lennon').filter(entry__pub_date__year=2008)
~~~~
"Lennon"과 2008년 entries을 모두 포함하는 블로그가 하나뿐이며 2008년 entries 중 "Lennon"이 포함되지 않은 블로그가 있다고 가정합니다. 첫 번째 query는 블로그를 반환하지 않지만 두 번째 query는 해당 블로그를 반환합니다.

두 번째 예에서 첫 번째 필터는 query set를 헤드 라인의 'Lennon'이 포함된 항목에 링크된 모든 블로그로 제한합니다. 두 번째 필터는 블로그 집합을 2008년에 게시된 항목과 연결된 블로그 집합으로 더 제한합니다. 두 번째 필터에서 선택한 항목은 첫 번째 필터의 항목과 같을 수도 있고 아닐 수도 있습니다. Entry items가 아닌 각 필터문으로 Blog 항목을 필터링하고 있습니다.

<b>Note</b>
<br>위에서 설명한 것처럼 다중 값 관계에 걸쳐있는 query에 대한 <tt style="color: #FF0000">`filter()`</tt>의 동작은 <tt style="color: #FF0000">`exclude()`</tt>에 대해 동일하게 구현되지 않습니다. 대신, 단일 <tt style="color: #FF0000">`exclude()`</tt> 호출의 조건이 반드시 동일한 항목을 참조하는 것은 아닙니다.

예를 들어, 다음 query는 헤드 라인에 "Lennon"이라는 항목과 2008년에 게시된 항목을 모두 포함하는 블로그를 제외합니다.
~~~~
Blog.objects.exclude(
    entry__headline__contains='Lennon',
    entry__pub_date__year=2008,
)
~~~~
그러나 <tt style="color: #FF0000">`filter()`</tt>를 사용하는 경우와 달리 두 조건을 모두 충족하는 항목을 기반으로 블로그를 제한하지 않습니다. 예를 들어 2008년에 게시된 'Lennon'으로 게시한 항목이 포함되지 않은 블로그를 모두 선택하려면 두 가지 검색어를 입력해야합니다.
~~~~
Blog.objects.exclude(
    entry__in=Entry.objects.filter(
        headline__contains='Lennon',
        pub_date__year=2008,
    ),
)
~~~~

### Filters can reference fields on the model

지금까지는 모델 필드의 값과 상수를 비교하는 필터를 만들었습니다. 그러나 모델 필드의 값을 동일한 모델의 다른 필드와 비교하려면 어떻게해야 할까요?

Django는 이러한 비교를 허용하는 [F expressions](https://docs.djangoproject.com/en/1.11/ref/models/expressions/#django.db.models.F)을 제공합니다. <tt style="color: #FF0000">`F()`</tt>의 인스턴스는 query내의 모델 필드에 대한 참조로 작동합니다. 그런 다음이 참조를 query 필터에 사용하여 동일한 모델 인스턴스의 두 개의 다른 필드 값을 비교할 수 있습니다.

예를 들어 pingback보다 많은 코멘트가 있는 모든 블로그 항목의 목록을 찾으려면 pingbacks 수를 참조하는 <tt style="color: #FF0000">`F()`</tt> 객체를 생성하고 query에서 해당 <tt style="color: #FF0000">`F()`</tt> 객체를 사용합니다.
~~~~
>>> from django.db.models import F
>>> Entry.objects.filter(n_comments__gt=F('n_pingbacks'))
~~~~
Django는 상수와 다른 <tt style="color: #FF0000">`F()`</tt> 객체와 함께 <tt style="color: #FF0000">`F()`</tt> 객체를 사용하여 덧셈, 뺄셈, 곱셈, 나눗셈, modulo 및 지수 연산을 지원합니다. pingback보다 2배 많은 주석이 있는 모든 블로그 entries을 찾으려면 다음과 같이 query를 수정하십시오.
~~~~
>>> Entry.objects.filter(n_comments__gt=F('n_pingbacks') * 2)
~~~~
entry의 등급이 pingback 카운트와 코멘트 카운트의 합보다 작은 모든 entry를 찾으려면, 다음과 같은 query를 발행할 것입니다 :
~~~~
>>> Entry.objects.filter(rating__lt=F('n_comments') + F('n_pingbacks'))
~~~~
double underscore notation을 사용하여 <tt style="color: #FF0000">`F()`</tt> 객체의 관계를 확장할 수도 있습니다. 이중 밑줄이있는 <tt style="color: #FF0000">`F()`</tt> 객체는 관련 객체에 액세스하는데 필요한 조인을 도입합니다. 예를 들어, 작성자 이름이 블로그 이름과 동일한 모든 항목을 검색하려면 다음과 같은 query를 발행할 수 있습니다.
~~~~
>>> Entry.objects.filter(authors__name=F('blog__name'))
~~~~
날짜 및 날짜/시간 필드의 경우 [timedelta](https://docs.python.org/3/library/datetime.html#datetime.timedelta) 객체를 더하거나 뺄 수 있습니다. 다음은 게시된 후 3일 이상 수정된 모든 항목을 반환합니다.
~~~~
>>> from datetime import timedelta
>>> Entry.objects.filter(mod_date__gt=F('pub_date') + timedelta(days=3))
~~~~
<tt style="color: #FF0000">`F()`</tt> 객체는 <tt style="color: #FF0000">`.bitand()`</tt>, <tt style="color: #FF0000">`.bitor()`</tt>, <tt style="color: #FF0000">`.bitrightshift()`</tt> 및 <tt style="color: #FF0000">`.bitleftshift()`</tt>에 의한 비트 연산을 지원합니다. 예 :
~~~~
>>> F('somefield').bitand(16)
~~~~

### The pk lookup shortcut

편의상 Django는 "primary key"를 나타내는 <tt style="color: #FF0000">`pk`</tt> 조회 바로 가기를 제공합니다.

예제 <tt style="color: #FF0000">`Blog`</tt> 모델에서 primary key는 <tt style="color: #FF0000">`id`</tt> 필드이므로 이 세 statements는 동일합니다.
~~~~
>>> Blog.objects.get(id__exact=14) # Explicit form
>>> Blog.objects.get(id=14) # __exact is implied
>>> Blog.objects.get(pk=14) # pk implies id__exact
~~~~
<tt style="color: #FF0000">`pk`</tt>의 사용은 <tt style="color: #FF0000">`__exact`</tt> query에만 국한되지 않습니다. query 용어를 <tt style="color: #FF0000">`pk`</tt>와 결합하여 모델의 primary key에 대한 query를 수행할 수 있습니다.
~~~~
# Get blogs entries with id 1, 4 and 7
>>> Blog.objects.filter(pk__in=[1,4,7])

# Get all blog entries with id > 14
>>> Blog.objects.filter(pk__gt=14)
~~~~
<tt style="color: #FF0000">`pk`</tt> 조회는 join 전체에서 작동합니다. 예를 들어, 다음 세 문장은 동일합니다.
~~~~
>>> Entry.objects.filter(blog__id__exact=3) # Explicit form
>>> Entry.objects.filter(blog__id=3)        # __exact is implied
>>> Entry.objects.filter(blog__pk=3)        # __pk implies __id__exact
~~~~

### Escaping percent signs and underscores in LIKE statements

<tt style="color: #FF0000">`LIKE`</tt> SQL 문 (<tt style="color: #FF0000">`iexact, contains, icontains, startswith, endswith`</tt> 및 <tt style="color: #FF0000">`iendswith`</tt>)과 같은 필드 조회는 <tt style="color: #FF0000">`LIKE`</tt> 문에 사용된 두 개의 특수 문자 (백분율 기호 및 밑줄)를 자동으로 이스케이프합니다. <tt style="color: #FF0000">`LIKE`</tt> 문에서 백분율 기호는 multiple-character 와일드 카드를 나타내며 밑줄 문자는 single-character 와일드 카드를 나타냅니다.

이는 직관적으로 작동해야 함을 의미하므로 추상화가 누출되지 않습니다. 예를 들어 퍼센트 기호가 포함된 모든 항목을 검색하려면 백분율 기호를 다른 문자로 사용하십시오.
~~~~
>>> Entry.objects.filter(headline__contains='%')
~~~~
장고가 인용 처리합니다. 결과 SQL은 다음과 같습니다.
~~~~
SELECT ... WHERE headline LIKE '%\%%';
~~~~
underscore에 대해서도 동일합니다. 백분율 기호와 밑줄은 모두 명백하게 처리됩니다.

### Caching and QuerySets

각 <tt style="color: #FF0000">`QuerySet`</tt>에는 데이터베이스 액세스를 최소화하기 위한 캐시가 포함되어 있습니다. 어떻게 작동하는지 이해하면 가장 효율적인 코드를 작성할 수 있습니다.

새로 생성된 <tt style="color: #FF0000">`QuerySet`</tt>에서 캐시는 비어 있습니다. <tt style="color: #FF0000">`QuerySet`</tt>이 처음 평가 될 때 - 따라서 데이터베이스 query가 발생합니다 - Django는 query 결과를 <tt style="color: #FF0000">`QuerySet`</tt>의 캐시에 저장하고 명시적으로 요청된 결과를 반환합니다 (예 : <tt style="color: #FF0000">`QuerySet`</tt>이 반복되는 경우 다음 요소). <tt style="color: #FF0000">`QuerySet`</tt>의 후속 평가는 캐시된 결과를 다시 사용합니다.

이 캐싱 동작을 염두에 두십시오. <tt style="color: #FF0000">`QuerySet`</tt>를 올바르게 사용하지 않으면 query가 제대로 작동하지 않을 수 있습니다. 예를 들어, 다음은 두 개의 <tt style="color: #FF0000">`QuerySet`</tt>을 만들고 평가한 다음 멀리 버립니다.
~~~~
>>> print([e.headline for e in Entry.objects.all()])
>>> print([e.pub_date for e in Entry.objects.all()])
~~~~
즉, 동일한 데이터베이스 query가 두 번 실행되어 데이터베이스로드가 효과적으로 배가됩니다. 또한 <tt style="color: #FF0000">`Entry`</tt>이 두 요청 사이에 분할된 second에 추가되거나 삭제되었을 수 있기 때문에 두 목록에 동일한 데이터베이스 레코드가 포함되지 않을 수도 있습니다.

이 문제를 피하려면 간단히 <tt style="color: #FF0000">`QuerySet`</tt>을 저장하고 다시 사용하면됩니다.
~~~~
>>> queryset = Entry.objects.all()
>>> print([p.headline for p in queryset]) # Evaluate the query set.
>>> print([p.pub_date for p in queryset]) # Re-use the cache from the evaluation.
~~~~

#### When QuerySets are not cached

Querysets은 항상 결과를 캐시하지 않습니다. query 세트의 일부만 평가할때 캐시가 검사되지만 채워지지 않은 경우 후속 query에서 반환되는 항목은 캐시되지 않습니다. 특히 이는 배열 슬라이스나 인덱스를 사용하여 [limiting the queryset](https://docs.djangoproject.com/en/1.11/topics/db/queries/#limiting-querysets)해도 캐시가 채워지지 않음을 의미합니다.

예를 들어, queryset 오브젝트에서 반복적으로 특정 인덱스를 얻으면 매번 데이터베이스를 조회하게됩니다.
~~~~
>>> queryset = Entry.objects.all()
>>> print(queryset[5]) # Queries the database
>>> print(queryset[5]) # Queries the database again
~~~~
그러나 전체 queryset가 이미 평가된 경우 캐시가 대신 check됩니다.
~~~~
>>> queryset = Entry.objects.all()
>>> [entry for entry in queryset] # Queries the database
>>> print(queryset[5]) # Uses cache
>>> print(queryset[5]) # Uses cache
~~~~
다음은 전체 query 세트를 평가하여 캐시를 채우는 다른 작업의 몇 가지 예입니다.
~~~~
>>> [entry for entry in queryset]
>>> bool(queryset)
>>> entry in queryset
>>> list(queryset)
~~~~
<b>Note</b>
<br>queryset을 인쇄하면 캐시가 채워지지 않습니다. 이는 <tt style="color: #FF0000">`__repr __()`</tt>을 호출하면 전체 query 세트의 조각만 반환되기 때문입니다.


## Complex lookups with Q objects

<tt style="color: #FF0000">`filter()`</tt> 등의 내부 키워드 인수 query는 <tt style="color: #FF0000">`AND`</tt>로 함께 합니다. 더 복잡한 query (예 : <tt style="color: #FF0000">`OR`</tt> 문을 사용하는 query)를 실행해야하는 경우 [Q objects](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.Q)를 사용할 수 있습니다.

<tt style="color: #FF0000">`Q object(django.db.models.Q)`</tt>는 키워드 인수 모음을 캡슐화하는데 사용되는 객체입니다. 이러한 키워드 인수는 위의 "Field lookups"에서처럼 지정됩니다.

예를 들어이 <tt style="color: #FF0000">`Q`</tt> object는 단일 <tt style="color: #FF0000">`LIKE`</tt> query를 캡슐화합니다.
~~~~
from django.db.models import Q
Q(question__startswith='What')
~~~~
Q 오브젝트는 &와 | 연산자를 사용하여 결합될 수 있습니다. 연산자가 두 개의 Q 객체에 사용되면 새로운 Q 객체가 생성됩니다.

예를 들어 다음 statement은 두 개의 "question__startswith" query를 "OR"로 나타내는 단일 Q 개체를 생성합니다.
~~~~
Q(question__startswith='Who') | Q(question__startswith='What')
~~~~
이것은 다음 SQL WHERE 절과 동일합니다.
~~~~
WHERE question LIKE 'Who%' OR question LIKE 'What%'
~~~~
Q 오브젝트를 &와 | 연산자와 괄호를 이용한 그룹화로 임의의 복잡성을 갖는 문장을 작성할 수 있습니다. 또한 Q 객체는 ~ 연산자를 사용하여 무효화될 수 있으므로 일반 query와 부정 (NOT) query를 결합한 결합된 조회가 가능합니다.
~~~~
Q(question__startswith='Who') | ~Q(pub_date__year=2005)
~~~~
키워드 인수 (예 : filter(), exclude(), get())를 사용하는 각 조회 함수는 하나 이상의 Q 객체를 위치 지정되지 않은 인수로 전달할 수도 있습니다. 조회 함수에 여러 개의 Q 오브젝트 인수를 제공하면 인수는 "AND"로 함께 합니다. 예 :
~~~~
Poll.objects.get(
    Q(question__startswith='Who'),
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
)
~~~~
SQL문으로 변환하면 대략 다음과 같습니다.
~~~~
SELECT * from polls WHERE question LIKE 'Who%'
    AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')
~~~~
조회 함수는 Q 오브젝트와 키워드 인수의 사용을 혼합할 수 있습니다. 조회 함수에 제공된 모든 인수(키워드 인수 또는 Q 오브젝트 임)는 AND로 함께 합니다. 그러나 Q 오브젝트가 제공되면 키워드 인수의 정의앞에 와야합니다. 예 :
~~~~
Poll.objects.get(
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),
    question__startswith='Who',
)
~~~~
앞의 예와 동일한 유효한 query가됩니다. 그러나:
~~~~
# INVALID QUERY
Poll.objects.get(
    question__startswith='Who',
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
)
~~~~
유효하지 않습니다.

<b>See also</b>
<br>Django 유닛 테스트의 [OR lookups examples](https://github.com/django/django/blob/master/tests/or_lookups/tests.py)는 Q의 가능한 사용법을 보여줍니다.


## Comparing objects

두 개의 모델 인스턴스를 비교하려면 표준 Python 비교 연산자인 double equals sign (==)을 사용하십시오. 뒤에서 두 모델의 primary key 값을 비교합니다.

위의 <tt style="color: #FF0000">`Entry`</tt> 예제를 사용하면 다음 두 문은 동일합니다.
~~~~
>>> some_entry == other_entry
>>> some_entry.id == other_entry.id
~~~~
모델의 primary key가 <tt style="color: #FF0000">`id`</tt>가 아닌 경우 문제가 없습니다. 비교는 호출된 모든 것이 항상 primary key를 사용합니다. 예를 들어, 모델의 primary key 필드의 이름이 <tt style="color: #FF0000">`name`</tt>인 경우 다음 두 문은 동일합니다.
~~~~
>>> some_obj == other_obj
>>> some_obj.name == other_obj.name
~~~~


## Deleting objects

delete 메소드는 편리하게 [delete()](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model.delete)라는 이름을 갖습니다. 이 메서드는 즉시 개체를 삭제하고 삭제된 개체의 수와 개체 유형당 삭제 수가 있는 dictionary을 반환합니다. 예:
~~~~
>>> e.delete()
(1, {'weblog.Entry': 1})
~~~~
개체를 대량으로 삭제할 수도 있습니다. 모든 <tt style="color: #FF0000">`QuerySet`</tt>에는 해당 <tt style="color: #FF0000">`QuerySet`</tt>의 모든 멤버를 삭제하는 <tt style="color: #FF0000">`delete()`</tt> 메서드가 있습니다.

예를 들어 <tt style="color: #FF0000">`pub_date`</tt> 연도가 2005인 모든 <tt style="color: #FF0000">`Entry`</tt> 객체를 삭제합니다.
~~~~
>>> Entry.objects.filter(pub_date__year=2005).delete()
(5, {'webapp.Entry': 5})
~~~~
가능할 때마다 이것은 SQL로만 실행되므로 개개의 객체 인스턴스의 <tt style="color: #FF0000">`delete()`</tt> 메소드가 반드시 프로세스 중에 호출되지는 않는다는 것을 명심하십시오. 모델 클래스에 커스텀 <tt style="color: #FF0000">`delete()`</tt> 메소드를 제공하고 그것이 호출되도록 하려면, 그 모델의 인스턴스를 "수동으로"삭제해야합니다 (예 : QuerySet을 반복하고 <tt style="color: #FF0000">`delete()`</tt>를 호출하여). 각 개체를 개별적으로) <tt style="color: #FF0000">`QuerySet`</tt>의 대량 <tt style="color: #FF0000">`delete()`</tt> 메서드를 사용하는 대신.

Django는 객체를 삭제할 때 기본적으로 <tt style="color: #FF0000">`ON DELETE CASCADE`</tt> SQL 동작의 동작을 에뮬레이션합니다. 즉, 삭제될 객체를 가리키는 foreign keys를 가진 객체는 함께 삭제됩니다. 예 :
~~~~
b = Blog.objects.get(pk=1)
# This will delete the Blog and all of its Entry objects.
b.delete()
~~~~
이 연쇄 동작은 [ForeignKey](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey)에 대한 [on_delete](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.on_delete) 인수를 통해 customizable 합니다.

<tt style="color: #FF0000">`delete()`</tt>는 <tt style="color: #FF0000">`Manager`</tt> 자체에 표시되지 않는 유일한 <tt style="color: #FF0000">`QuerySet`</tt> 메서드입니다. 이것은 실수로 <tt style="color: #FF0000">`Entry.objects.delete()`</tt>를 요청하지 않고 모든 항목을 삭제하지 못하게 하는 안전 메커니즘입니다. 모든 개체를 삭제하려면 명시적으로 complete query set을 요청해야합니다.
~~~~
Entry.objects.all().delete()
~~~~


## Copying model instances

모델 인스턴스를 복사하는 built-in method는 없지만 모든 필드의 값을 복사하여 새 인스턴스를 쉽게 만들 수 있습니다. 가장 단순한 방법으로 <tt style="color: #FF0000">`pk`</tt>를 <tt style="color: #FF0000">`None`</tt>으로 설정하는 방법이 있습니다. 블로그 예제 사용 :
~~~~
blog = Blog(name='My blog', tagline='Blogging is easy')
blog.save() # blog.pk == 1

blog.pk = None
blog.save() # blog.pk == 2
~~~~
상속을 사용하면 상황이 더욱 복잡해집니다. <tt style="color: #FF0000">`Blog`</tt>의 하위 클래스를 생각해보십시오.
~~~~
class ThemeBlog(Blog):
    theme = models.CharField(max_length=200)

django_blog = ThemeBlog(name='Django', tagline='Django is easy', theme='python')
django_blog.save() # django_blog.pk == 3
~~~~
상속이 작동하는 방법 때문에 <tt style="color: #FF0000">`pk`</tt>와 i<tt style="color: #FF0000">`id`</tt>를 None으로 설정해야합니다.
~~~~
django_blog.pk = None
django_blog.id = None
django_blog.save() # django_blog.pk == 4
~~~~
이 프로세스는 모델의 데이터베이스 테이블에 포함되지 않은 관계를 복사하지 않습니다. 예를 들어, <tt style="color: #FF0000">`Entry`</tt>에는 <tt style="color: #FF0000">`Author`</tt>에 대한 <tt style="color: #FF0000">`ManyToManyField`</tt>가 있습니다. 엔트리를 복제한 후에는 새로운 엔트리에 대해 다대다 관계를 설정해야합니다.
~~~~
entry = Entry.objects.all()[0] # some previous entry
old_authors = entry.authors.all()
entry.pk = None
entry.save()
entry.authors.set(old_authors)
~~~~
<tt style="color: #FF0000">`OneToOneField`</tt>의 경우 일대일 고유 제한을 위반하지 않도록 관련 개체를 복제하고 이를 새 개체의 필드에 할당해야합니다. 예를 들어, <tt style="color: #FF0000">`entry`</tt>가 위와 같이 이미 복제되었다고 가정합니다.
~~~~
detail = EntryDetail.objects.all()[0]
detail.pk = None
detail.entry = entry
detail.save()
~~~~


## Updating multiple objects at once

때로는 <tt style="color: #FF0000">`QuerySet`</tt>의 모든 객체에 대해 특정 값으로 필드를 설정하려고 합니다. [update()](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.query.QuerySet.update) 메서드를 사용하여 이 작업을 수행할 수 있습니다. 예 :
~~~~
# Update all the headlines with pub_date in 2007.
Entry.objects.filter(pub_date__year=2007).update(headline='Everything is the same')
~~~~
이 방법을 사용하여 non-relation 필드와 non-relation <tt style="color: #FF0000">`ForeignKey`</tt> 필드만 설정할 수 있습니다. non-relation 필드를 업데이트하려면 새 값을 상수로 제공하십시오. <tt style="color: #FF0000">`ForeignKey`</tt> 필드를 업데이트하려면 새 값을 가리킬 새 모델 인스턴스로 설정하십시오. 예 :
~~~~
>>> b = Blog.objects.get(pk=1)

# Change every Entry so that it belongs to this Blog.
>>> Entry.objects.all().update(blog=b)
~~~~
<tt style="color: #FF0000">`update()`</tt> 메서드는 즉시 적용되며 query와 일치하는 행 수를 반환합니다. 일부 행에 이미 새 값이 있는 경우 업데이트된 행 수와 다를 수 있습니다. 업데이트되는 <tt style="color: #FF0000">`QuerySet`</tt>에 대한 유일한 제한은 모델의 주 테이블인 하나의 데이터베이스 테이블에만 액세스할 수 있다는 것입니다. 관련 필드를 기준으로 필터링할 수 있지만 모델 주 테이블의 열만 업데이트 할 수 있습니다. 예:
~~~~
>>> b = Blog.objects.get(pk=1)

# Update all the headlines belonging to this Blog.
>>> Entry.objects.select_related().filter(blog=b).update(headline='Everything is the same')
~~~~
<tt style="color: #FF0000">`update()`</tt> 메서드는 SQL 문으로 직접 변환됩니다. 직접 업데이트를 위한 bulk 작업입니다. 모델에서 <tt style="color: #FF0000">`save()`</tt> 메소드를 실행하지 않거나 <tt style="color: #FF0000">`save()`</tt>를 호출한 결과인 <tt style="color: #FF0000">`pre_save`</tt> 또는 <tt style="color: #FF0000">`post_save`</tt> 신호를 내보내거나 <tt style="color: #FF0000">`auto_now`</tt> 필드 옵션을 사용하지 않습니다. <tt style="color: #FF0000">`QuerySet`</tt>의 모든 항목을 저장하고 각 인스턴스에서 <tt style="color: #FF0000">`save()`</tt> 메서드가 호출되도록 하려면 해당 함수를 처리하는데 특별한 함수가 필요하지 않습니다. 그냥 반복해서 호출하고 <tt style="color: #FF0000">`save()`</tt>를 호출하면됩니다.
~~~~
for item in my_queryset:
    item.save()
~~~~
업데이트 호출은 [F expression](https://docs.djangoproject.com/en/1.11/ref/models/expressions/#django.db.models.F)을 사용하여 모델의 다른 필드 값을 기반으로한 필드를 업데이트할 수도 있습니다. 이것은 현재 값을 기반으로 카운터를 증가시킬 때 특히 유용합니다. 예를 들어 블로그의 모든 항목에 대해 pingback 수를 늘리려면 다음과 같이하십시오.
~~~~
>>> Entry.objects.all().update(n_pingbacks=F('n_pingbacks') + 1)
~~~~
그러나 필터 및 제외 절의 <tt style="color: #FF0000">`F()`</tt> 객체와 달리 업데이트에서 <tt style="color: #FF0000">`F()`</tt> 객체를 사용할 때 조인을 도입할 수는 없으며 업데이트되는 모델의 로컬 필드만 참조할 수 있습니다. <tt style="color: #FF0000">`F()`</tt> 객체를 사용하여 조인을 도입하려고 시도하면 <tt style="color: #FF0000">`FieldError`</tt>가 발생합니다.
~~~~
# This will raise a FieldError
>>> Entry.objects.update(headline=F('blog__name'))
~~~~


## Related objects

모델 (<tt style="color: #FF0000">`ForeignKey`</tt>, <tt style="color: #FF0000">`OneToOneField`</tt> 또는 <tt style="color: #FF0000">`ManyToManyField`</tt>)에서 관계를 정의하면 해당 모델의 인스턴스는 관련 객체에 액세스 하기위한 편리한 API를 갖게됩니다.

예를 들어 이 페이지의 상단에 있는 모델을 사용하여 <tt style="color: #FF0000">`Entry`</tt> 객체 <tt style="color: #FF0000">`e`</tt>는 블로그 속성인 <tt style="color: #FF0000">`e.blog`</tt>에 액세스하여 관련 <tt style="color: #FF0000">`Blog`</tt> 객체를 가져올 수 있습니다.

배후에서 이 기능은 Python [descriptors](https://docs.python.org/3/howto/descriptor.html)에 의해 구현됩니다. 이 내용은 사용자에게 중요하지 않지만 호기심 때문에 여기에서 지적합니다.

Django는 관계의 "다른"측면에 대한 API 접근자 - 관련 모델에서 관계를 정의하는 모델에 대한 링크 -를 만듭니다. 예를 들어, <tt style="color: #FF0000">`Blog`</tt> 객체 <tt style="color: #FF0000">`b`</tt>는 <tt style="color: #FF0000">`entry_set`</tt> 속성인 <tt style="color: #FF0000">`b.entry_set.all()`</tt>을 통해 관련된 모든 <tt style="color: #FF0000">`Entry`</tt> 객체의 목록에 액세스할 수 있습니다.

이 섹션의 모든 예제는 이 페이지의 맨 위에 정의된 샘플 <tt style="color: #FF0000">`Blog`</tt>, <tt style="color: #FF0000">`Author`</tt> 및 <tt style="color: #FF0000">`Entry`</tt> 모델을 사용합니다.

### One-to-many relationships

#### Forward

모델에 ForeignKey가 있는 경우 해당 모델의 인스턴스는 모델의 단순 속성을 통해 관련 (foreign) 객체에 액세스할 수 있습니다.

예:
~~~~
>>> e = Entry.objects.get(id=2)
>>> e.blog # Returns the related Blog object.
~~~~
foreign-key 속성을 통해 가져오고 설정할 수 있습니다. 예상대로 foreign-key에 대한 변경 사항은 <tt style="color: #FF0000">`save()`</tt>를 호출할 때까지 데이터베이스에 저장되지 않습니다. 예:
~~~~
>>> e = Entry.objects.get(id=2)
>>> e.blog = some_blog
>>> e.save()
~~~~
<tt style="color: #FF0000">`ForeignKey`</tt> 필드에 <tt style="color: #FF0000">`null=True`</tt>가 설정되어 있으면 (즉, <tt style="color: #FF0000">`NULL`</tt> 값을 허용하는 경우) <tt style="color: #FF0000">`None`</tt>을 할당하여 관계를 제거할 수 있습니다. 예:
~~~~
>>> e = Entry.objects.get(id=2)
>>> e.blog = None
>>> e.save() # "UPDATE blog_entry SET blog
~~~~
관련 객체에 처음 액세스할 때 일대다 관계에 대한 전달 액세스가 캐시됩니다. 동일한 오브젝트 인스턴스에서 foreign key에 대한 후속 액세스가 캐시됩니다. 예:
~~~~
>>> e = Entry.objects.get(id=2)
>>> print(e.blog)  # Hits the database to retrieve the associated Blog.
>>> print(e.blog)  # Doesn't hit the database; uses cached version.
~~~~
[select_related()](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.query.QuerySet.select_related) <tt style="color: #FF0000">`QuerySet`</tt> 메서드는 모든 일대다 관계의 캐시를 미리 재귀적으로 미리 채웁니다. 예:
~~~~
>>> e = Entry.objects.select_related().get(id=2)
>>> print(e.blog)  # Doesn't hit the database; uses cached version.
>>> print(e.blog)  # Doesn't hit the database; uses cached version.
~~~~

#### Following relationships “backward”

모델에 <tt style="color: #FF0000">`ForeignKey`</tt>가 있는 경우 foreign-key 모델의 인스턴스는 첫 번째 모델의 모든 인스턴스를 반환하는 <tt style="color: #FF0000">`Manager`</tt>에 액세스할 수 있습니다. 기본적으로 이 <tt style="color: #FF0000">`Manager`</tt>의 이름은 <tt style="color: #FF0000">`FOO_set`</tt>입니다. 여기서 <tt style="color: #FF0000">`FOO`</tt>는 소문자인 소스 모델 이름입니다. 이 <tt style="color: #FF0000">`Manager`</tt>는 위의 "개체 검색"절에서 설명한대로 필터링하고 조작할 수 있는 <tt style="color: #FF0000">`QuerySet`</tt>을 반환합니다.

예:
~~~~
>>> b = Blog.objects.get(id=1)
>>> b.entry_set.all() # Returns all Entry objects related to Blog.

# b.entry_set is a Manager that returns QuerySets.
>>> b.entry_set.filter(headline__contains='Lennon')
>>> b.entry_set.count()
~~~~
<tt style="color: #FF0000">`ForeignKey`</tt> 정의에서 [related_name](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.related_name) 매개 변수를 설정하여 <tt style="color: #FF0000">`FOO_set`</tt> 이름을 겹쳐 쓸 수 있습니다. 예를 들어 <tt style="color: #FF0000">`Entry`</tt> 모델이 <tt style="color: #FF0000">`blog = ForeignKey(Blog, on_delete=models.CASCADE, related_name='entries')`</tt>로 변경된 경우 위의 예제 코드는 다음과 같습니다.
~~~~
>>> b = Blog.objects.get(id=1)
>>> b.entries.all() # Returns all Entry objects related to Blog.

# b.entries is a Manager that returns QuerySets.
>>> b.entries.filter(headline__contains='Lennon')
>>> b.entries.count()
~~~~

#### Using a custom reverse manager

default로 역관계에 사용되는 [RelatedManager](https://docs.djangoproject.com/en/1.11/ref/models/relations/#django.db.models.fields.related.RelatedManager)는 해당 모델의 [default manager](https://docs.djangoproject.com/en/1.11/topics/db/managers/#manager-names)의 하위 클래스입니다. 특정 query에 대해 다른 관리자를 지정하려면 다음 구문을 사용할 수 있습니다.
~~~~
from django.db import models

class Entry(models.Model):
    #...
    objects = models.Manager()  # Default Manager
    entries = EntryManager()    # Custom Manager

b = Blog.objects.get(id=1)
b.entry_set(manager='entries').all()
~~~~
<tt style="color: #FF0000">`EntryManager`</tt>가 <tt style="color: #FF0000">`get_queryset()`</tt> 메소드에서 기본 필터링을 수행하면 해당 필터링이 <tt style="color: #FF0000">`all()`</tt> 호출에 적용됩니다.

물론 custom 역방향 관리자를 지정하면 custom 메서드를 호출 할 수도 있습니다.
~~~~
b.entry_set(manager='entries').is_published()
~~~~

#### Additional methods to handle related objects

위의 "객체 검색"에 정의된 QuerySet 메소드 외에도 <tt style="color: #FF0000">`ForeignKey Manager`</tt>에는 관련 객체 세트를 처리하는데 사용되는 추가 메소드가 있습니다. 각각의 개요가 아래에 있으며, 자세한 내용은 [related objects reference](https://docs.djangoproject.com/en/1.11/ref/models/relations/)에서 찾을 수 있습니다.

* <tt style="color: #FF0000">`add(obj1, obj2, ...)`</tt>
지정된 모델 객체를 관련 객체 세트에 추가합니다.
* <tt style="color: #FF0000">`create(** kwargs)`</tt>
새 객체를 만들고 저장 한 다음 관련 객체 세트에 배치합니다. 새롭게 생성 된 오브젝트를 돌려줍니다.
* <tt style="color: #FF0000">`remove(obj1, obj2, ...)`</tt>
관련 모델 세트에서 지정된 모델 오브젝트를 제거합니다.
* <tt style="color: #FF0000">`clear()`</tt>
관련 객체 세트에서 모든 객체를 제거합니다.
* <tt style="color: #FF0000">`set(objs)`</tt>
관련 오브젝트 세트를 대체하십시오.
관련 세트의 멤버를 할당하려면 객체 인스턴스의 반복 가능 또는 primary key 값 목록과 함께 <tt style="color: #FF0000">`set()`</tt> 메소드를 사용하십시오. 예 :
~~~~
b = Blog.objects.get(id=1)
b.entry_set.set([e1, e2])
~~~~
이 예에서 <tt style="color: #FF0000">`e1`</tt> 및 <tt style="color: #FF0000">`e2`</tt>는 전체 Entry 인스턴스 또는 정수 primary key 값이 될 수 있습니다.

<tt style="color: #FF0000">`clear()`</tt> 메서드를 사용할 수 있는 경우, iterable(이 경우 목록)의 모든 객체가 세트에 추가되기 전에 기존 객체가 <tt style="color: #FF0000">`entry_set`</tt>에서 제거됩니다. <tt style="color: #FF0000">`clear()`</tt> 메서드를 사용할 수 없는 경우 iterable의 모든 객체는 기존 요소를 제거하지 않고 추가됩니다.

이 섹션에 설명된 각 "역방향"작업은 데이터베이스에 즉각적인 영향을 줍니다. 모든 추가, 생성 및 삭제가 즉시 자동으로 데이터베이스에 저장됩니다.

### Many-to-many relationships

다대다 관계의 양 끝은 다른 쪽 끝으로 자동 API 액세스를 얻습니다. API는 위의 "역방향" 일대다 관계와 동일하게 작동합니다.

유일한 차이점은 속성 이름 지정에 있습니다. [ManyToManyField](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField)를 정의하는 모델은 해당 필드 자체의 속성 이름을 사용하지만 "역방향"모델은 원래 모델의 소문자 모델 이름과 <tt style="color: #FF0000">`'_set'`</tt>의 조합으로 만들어진 이름을 사용합니다. (역방향 일대다 관계와 같습니다.)

다음의 예시가 이해를 도와줄 것입니다.
~~~~
e = Entry.objects.get(id=3)
e.authors.all() # Returns all Author objects for this Entry.
e.authors.count()
e.authors.filter(name__contains='John')

a = Author.objects.get(id=5)
a.entry_set.all() # Returns all Entry objects for this Author.
~~~~
<tt style="color: #FF0000">`ForeignKey`</tt>와 마찬가지로 <tt style="color: #FF0000">`ManyToManyField`</tt>는 <tt style="color: #FF0000">`related_name`</tt>을 지정할 수 있습니다. 위의 예에서 <tt style="color: #FF0000">`Entry`</tt>의 <tt style="color: #FF0000">`ManyToManyField`</tt>가 <tt style="color: #FF0000">`related_name='entries'`</tt>를 지정한 경우 각 작성자 인스턴스는 <tt style="color: #FF0000">`entry_set`</tt> 대신 <tt style="color: #FF0000">`entries`</tt> 속성을 갖습니다.

### One-to-one relationships

일대일 관계는 다대일 관계와 매우 유사합니다. 모델에 [OneToOneField](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField)를 정의하면 해당 모델의 인스턴스는 모델의 단순 속성을 통해 관련 객체에 액세스할 수 있습니다.

예 :
~~~~
class EntryDetail(models.Model):
    entry = models.OneToOneField(Entry, on_delete=models.CASCADE)
    details = models.TextField()

ed = EntryDetail.objects.get(id=2)
ed.entry # Returns the related Entry object.
~~~~
차이점은 "역"query입니다. 일대일 관계의 관련 모델도 <tt style="color: #FF0000">`Manager`</tt> 객체에 액세스할 수 있지만 <tt style="color: #FF0000">`Manager`</tt>는 객체 컬렉션이 아닌 단일 객체를 나타냅니다.
~~~~
e = Entry.objects.get(id=2)
e.entrydetail # returns the related EntryDetail object
~~~~
이 관계에 객체가 할당되어 있지 않으면 Django는 <tt style="color: #FF0000">`DoesNotExist`</tt> 예외를 발생시킵니다.

forward 관계를 지정하는 것과 같은 방법으로 인스턴스를 역 관계에 지정할 수 있습니다.
~~~~
e.entrydetail = ed
~~~~

### How are the backward relationships possible?¶

다른 객체 관계형 맵퍼에서는 양쪽에 관계를 정의해야합니다. Django 개발자는 DRY(Do not Repeat Yourself) 원칙을 위반한다고 생각하므로 장고는 한쪽 끝에서만 관계를 정의해야합니다.

그러나 모델 클래스가 다른 모델 클래스가 로드될 때까지 어떤 다른 모델 클래스가 관련되어 있는지를 모델 클래스가 알지 못한다면 어떻게 할 수 있습니까?

대답은 [app registry](https://docs.djangoproject.com/en/1.11/ref/applications/#django.apps.apps)에 있습니다. Django가 시작되면 [INSTALLED_APPS](https://docs.djangoproject.com/en/1.11/ref/applications/#django.apps.apps)에 나열된 각 응용 프로그램을 가져온 다음 각 응용 프로그램 내부에 <tt style="color: #FF0000">`models`</tt> 모듈을 가져옵니다. 새로운 모델 클래스가 생성 될 때마다 Django는 모든 관련 모델에 역방향 관계를 추가합니다. 관련 모델을 아직 가져 오지 않은 경우 장고는 관련 모델을 가져올 때 관계의 트랙을 유지하고 추가합니다.

이러한 이유로 [INSTALLED_APPS](https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-INSTALLED_APPS)에 나열된 응용 프로그램에서 사용중인 모든 모델을 정의하는 것이 특히 중요합니다. 그렇지 않으면 후방 관계가 제대로 작동하지 않을 수 있습니다.

### Queries over related objects

관련 개체를 포함하는 query는 일반 값 필드가 포함된 query와 동일한 규칙을 따릅니다. 일치시킬 query 값을 지정할 때 개체 인스턴스 자체나 개체의 primary key 값을 사용할 수 있습니다.

예를 들어, <tt style="color: #FF0000">`id=5`</tt> 인 Blog 객체 <tt style="color: #FF0000">`b`</tt>가있는 경우 다음 세 가지 query가 동일합니다.
~~~~
Entry.objects.filter(blog=b) # Query using object instance
Entry.objects.filter(blog=b.id) # Query using id from instance
Entry.objects.filter(blog=5) # Query using id directly
~~~~

### Falling back to raw SQL

Django의 데이터베이스 - 매퍼가 처리하기에는 너무 복잡한 SQL queries를 작성해야할 필요가 있는 경우, 수동으로 SQL을 작성할 수 있습니다. Django는 raw SQL query를 작성하기위한 몇 가지 옵션을 가지고 있습니다. [Performing raw SQL queries](https://docs.djangoproject.com/en/1.11/topics/db/sql/)을 참조하십시오.

마지막으로 Django 데이터베이스 레이어는 데이터베이스에 대한 인터페이스 일뿐입니다. 다른 도구, 프로그래밍 언어 또는 데이터베이스 프레임워크를 통해 데이터베이스에 액세스할 수 있습니다. 당신의 데이터베이스에 대해 Django에 특정한 것은 없습니다.