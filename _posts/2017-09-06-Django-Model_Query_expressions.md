---
layout: post
title:  "Django Models_Query_expressions"
date:   2017-09-06 10:43:59
author: Dean Kim
categories: Django
tags:	Django Models Query_expressions
cover:  "/assets/instacode.png"
---

# Query Expressions
- 원본 : [공식문서](https://docs.djangoproject.com/en/1.11/ref/models/expressions/)

쿼리 표현식은 update, create, filter, order by, annotation 또는 aggregate의 일부로 사용할 수 있는 값 또는 계산을 설명합니다. 쿼리를 작성하는 데 사용할 수 있는 여러 가지 기본 제공 식(아래에 설명 됨)이 있습니다. 표현식은 결합되거나, 경우에 따라 중첩되어 더 복잡한 계산을 형성할 수 있습니다.


## Supported arithmetic

Django는 파이썬 상수, 변수 및 다른 표현식을 사용하여 쿼리 표현식에 더하기, 빼기, 곱하기, 나누기, 모듈로 산술 및 전원 연산자를 지원합니다.


## Some examples

~~~~
from django.db.models import F, Count, Value
from django.db.models.functions import Length, Upper

# Find companies that have more employees than chairs.
Company.objects.filter(num_employees__gt=F('num_chairs'))

# Find companies that have at least twice as many employees
# as chairs. Both the querysets below are equivalent.
Company.objects.filter(num_employees__gt=F('num_chairs') * 2)
Company.objects.filter(
    num_employees__gt=F('num_chairs') + F('num_chairs'))

# How many chairs are needed for each company to seat all employees?
>>> company = Company.objects.filter(
...    num_employees__gt=F('num_chairs')).annotate(
...    chairs_needed=F('num_employees') - F('num_chairs')).first()
>>> company.num_employees
120
>>> company.num_chairs
50
>>> company.chairs_needed
70

# Create a new company using expressions.
>>> company = Company.objects.create(name='Google', ticker=Upper(Value('goog')))
# Be sure to refresh it if you need to access the field.
>>> company.refresh_from_db()
>>> company.ticker
'GOOG'

# Annotate models with an aggregated value. Both forms
# below are equivalent.
Company.objects.annotate(num_products=Count('products'))
Company.objects.annotate(num_products=Count(F('products')))

# Aggregates can contain complex computations also
Company.objects.annotate(num_offerings=Count(F('products') + F('services')))

# Expressions can also be used in order_by()
Company.objects.order_by(Length('name').asc())
Company.objects.order_by(Length('name').desc())
~~~~


## Built-in Expressions

<b>Note</b>

이 표현식은 <tt style="color: #FF0000">`django.db.models.expressions`</tt>와 <tt style="color: #FF0000">`django.db.models.aggregates`</tt>에 정의되어 있습니다. 편의를 위해 사용 가능하며 일반적으로 [django.db.models](https://docs.djangoproject.com/en/1.11/topics/db/models/#module-django.db.models)에서 가져옵니다.
<b>END</b>

### F() expressions

<b>class F[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/expressions/#F)

<tt style="color: #FF0000">`F()`</tt>객체는 모델 필드 또는 annotated column의 값을 나타냅니다. 실제로 데이터베이스에서 파이썬 메모리로 가져오지 않고 모델 필드 값을 참조하고 이를 사용하여 데이터베이스 작업을 수행할 수 있습니다.

대신 Django는 <tt style="color: #FF0000">`F()`</tt> 객체를 사용하여 데이터베이스 수준에서 필요한 작업을 설명하는 SQL 표현식을 생성합니다.

이것은 예를 통해 이해하기가 가장 쉽습니다. 일반적으로, 다음과 같이 할 수 있습니다.
~~~~
# Tintin filed a news story!
reporter = Reporters.objects.get(name='Tintin')
reporter.stories_filed += 1
reporter.save()
~~~~

여기서 우리는 <tt style="color: #FF0000">`reporter.stories_filed`</tt>의 값을 데이터베이스에서 메모리로 가져와 친숙한 파이썬 연산자를 사용하여 조작한 다음 데이터베이스에 다시 저장했습니다. 하지만 대신 우리는 다음과 같이 할 수 있습니다.
~~~~
from django.db.models import F

reporter = Reporters.objects.get(name='Tintin')
reporter.stories_filed = F('stories_filed') + 1
reporter.save()
~~~~

<tt style="color: #FF0000">`reporter.stories_filed = F('stories_filed') + 1`</tt>은 일반적인 Python에서 값을 인스턴스 속성에 할당한 것처럼 보이지만 사실 데이터베이스에 대한 작업을 설명하는 SQL 구문입니다.

장고는 <tt style="color: #FF0000">`F()`</tt>의 인스턴스를 만나면 표준 파이썬 연산자를 오버라이드하여 캡슐화된 SQL 표현식을 생성합니다. 이 경우 <tt style="color: #FF0000">`reporter.stories_filed`</tt>가 나타내는 데이터베이스 필드를 증가시키도록 데이터베이스에 지시합니다.

<tt style="color: #FF0000">`reporter.stories_filed`</tt>에 어떤 value가 있건 간에, 파이썬은 그것에 대해 결코 알지 못합니다 - 그것은 전적으로 데이터베이스에 의해 처리됩니다. 모든 Python은 Django의 <tt style="color: #FF0000">`F()`</tt> 클래스를 통해 SQL 구문을 만들어 필드를 참조하고 작업을 설명합니다.

이 방법으로 저장된 새 값에 액세스하려면 객체를 다시 로드해야합니다.
~~~~
reporter = Reporters.objects.get(pk=reporter.pk)
# Or, more succinctly:
reporter.refresh_from_db()
~~~~

위와 같이 단일 인스턴스에 대한 작업에서 뿐만 아니라 <tt style="color: #FF0000">`F()`</tt>는 <tt style="color: #FF0000">`update()`</tt>와 함께 객체 인스턴스의 <tt style="color: #FF0000">`QuerySets`</tt>에서 사용할 수 있습니다. 이것은 위의 <tt style="color: #FF0000">`get()`</tt>과 [save()](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model.save)를 사용했던 두 개의 쿼리를 하나의 쿼리로 줄입니다.
~~~~
reporter = Reporters.objects.filter(name='Tintin')
reporter.update(stories_filed=F('stories_filed') + 1)
~~~~

또한 <tt style="color: #FF0000">`update()`</tt>를 사용하여 여러 객체의 필드 값을 증가시킬 수 있습니다. 데이터베이스에서 파이썬으로 모든 객체를 가져와서 루핑하고, 각 객체의 필드 값을 증가시키고, 각각을 다시 저장하는 것보다 훨씬 빠릅니다. 데이터베이스 :
~~~~
Reporter.objects.all().update(stories_filed=F('stories_filed') + 1)
~~~~

따라서 <tt style="color: #FF0000">`F()`</tt>는 다음과 같은 방법으로 성능 이점을 제공할 수 있습니다.

* 파이썬이 아닌 데이터베이스를 사용하여 작업 수행
* 일부 작업에 필요한 쿼리 수 감소

#### Avoiding race conditions using F()

<tt style="color: #FF0000">`F()`</tt>의 또 다른 유용한 이점은 파이썬이 아닌 데이터베이스가 필드 값을 업데이트하면 경쟁 조건을 피할 수 있다는 것입니다.

위의 첫 번째 예제에서 두 파이썬 스레드가 코드를 실행하면 한 스레드가 다른 스레드가 데이터베이스에서 값을 검색 한 후에 필드 값을 검색, 증가 및 저장할 수 있습니다. 두 번째 스레드가 저장하는 값은 원래 값을 기반으로 합니다. 첫 번째 스레드의 작업이 손실됩니다.

데이터베이스가 필드를 업데이트할 책임이 있는 경우 프로세스가 보다 강력합니다. <tt style="color: #FF0000">`save()`</tt> 또는 <tt style="color: #FF0000">`update()`</tt>가 실행될 때 인스턴스가 검색되었을 때의 값보다 데이터베이스의 필드 값을 기준으로 필드를 업데이트하기만 합니다.

#### F() assignments persist after Model.save()

모델 필드에 할당된 <tt style="color: #FF0000">`F()`</tt> 객체는 모델 인스턴스를 저장한 후에도 유지되며 각 <tt style="color: #FF0000">`save()`</tt>에 적용됩니다. 예 :
~~~~
reporter = Reporters.objects.get(name='Tintin')
reporter.stories_filed = F('stories_filed') + 1
reporter.save()

reporter.name = 'Tintin Jr.'
reporter.save()
~~~~

<tt style="color: #FF0000">`stories_filed`</tt>는 이 경우 두 번 업데이트됩니다. 처음에 <tt style="color: #FF0000">`1`</tt>이면 최종 값은 <tt style="color: #FF0000">`3`</tt>이됩니다.

#### Using F() in filters

<tt style="color: #FF0000">`F()`</tt>는 <tt style="color: #FF0000">`QuerySet`</tt> 필터에서도 매우 유용합니다. QuerySet 필터를 사용하면 Python 값이 아닌 필드 값을 기준으로 객체 집합을 필터링할 수 있습니다.

이것은 [쿼리에서 F() 표현식을 사용할 때](https://docs.djangoproject.com/en/1.11/topics/db/queries/#using-f-expressions-in-filters) 문서화됩니다.

#### Using F() with annotations

<tt style="color: #FF0000">`F()`</tt>는 다른 필드를 산술로 결합하여 모델에 동적 필드를 만드는 데 사용할 수 있습니다.
~~~~
company = Company.objects.annotate(
    chairs_needed=F('num_employees') - F('num_chairs'))
~~~~

만약 당신이 결합하고 있는 필드가 다른 타입이라면 장고에게 어떤 종류의 필드가 반환될지를 알려줘야 합니다. <tt style="color: #FF0000">`F()`</tt>는 <tt style="color: #FF0000">`output_field`</tt>를 직접 지원하지 않으므로 [ExpressionWrapper](https://docs.djangoproject.com/en/1.11/ref/models/expressions/#django.db.models.ExpressionWrapper)로 표현식을 래핑해야합니다.

~~~~
from django.db.models import DateTimeField, ExpressionWrapper, F

Ticket.objects.annotate(
    expires=ExpressionWrapper(
        F('active_at') + F('duration'), output_field=DateTimeField()))
~~~~


<tt style="color: #FF0000">`ForeignKey`</tt>와 같은 관계형 필드를 참조 할 때 <tt style="color: #FF0000">`F()`</tt>는 모델 인스턴스가 아닌 기본 키 값을 반환합니다.
~~~~
>> car = Company.objects.annotate(built_by=F('manufacturer'))[0]
>> car.manufacturer
<Manufacturer: Toyota>
>> car.built_by
3
~~~~





<tt style="color: #FF0000">`QuerySet`</tt>
































