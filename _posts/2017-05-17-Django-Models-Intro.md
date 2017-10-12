---
layout: post
title:  "Django Model-Intro"
date:   2017-05-17 10:43:59
author: Dean Kim
categories: Rest_Framework
tags:	Django Model
cover:  "/assets/instacode.png"
---

# Model
- 원본 : [공식문서](https://docs.djangoproject.com/en/1.11/topics/db/models/)


모델은 데이터에 대한 단일 정보 소스입니다. 여기에는 저장 중인 데이터의 필수 필드와 동작이 포함됩니다. 일반적으로 각 모델은 단일 데이터베이스 테이블에 매핑됩니다.

basics :
* 각 모델은 [django.db.models.Model](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model)의 subclasses로 Python 클래스입니다.
* 모델의 각 attribute는 데이터베이스 field를 나타냅니다.
* 이 모든 것을 통해 Django는 자동으로 생성된 database-access API를 제공합니다. [Making queries](https://dean-kim.github.io/2017/07/06/Django-Models_and_databases-Making_queries.html)를 참조하십시오.

### Quick example

예로 <tt style="color: #FF0000">`first_name`</tt>, <tt style="color: #FF0000">`last_name`</tt>을 갖는 <tt style="color: #FF0000">`Person`</tt> 모델을 정의합니다.
~~~~
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
~~~~
<tt style="color: #FF0000">`first_name`</tt>, <tt style="color: #FF0000">`last_name`</tt>는 모델의 fields입니다. 각 field는 클래스 attribute으로 지정되며 각 attribute은 데이터베이스 column에 매핑됩니다.
위의 <tt style="color: #FF0000">`Person`</tt> 모델은 다음과 같은 데이터베이스 테이블을 생성합니다.
~~~~
CREATE TABLE myapp_person (
    "id" serial NOT NULL PRIMARY KEY,
    "first_name" varchar(30) NOT NULL,
    "last_name" varchar(30) NOT NULL
);
~~~~
Some technical notes:
* 테이블의 이름인 <tt style="color: #FF0000">`myapp_person`</tt>은 일부 모델 메타 데이터에서 자동으로 파생되지만 재정의 될 수 있습니다. 자세한 내용은 [Table names](https://docs.djangoproject.com/en/1.11/ref/models/options/#table-names)을 참조하십시오.
* id field가 자동으로 추가되지만 이 동작은 overridden 될 수 있습니다. [Automatic primary key fields](https://docs.djangoproject.com/en/1.11/topics/db/models/#automatic-primary-key-fields)를 참조하십시오.
* 이 예제의 CREATE TABLE SQL은 PostgreSQL 구문을 사용하여 포맷되지만 Django는 [settings file](https://docs.djangoproject.com/en/1.11/topics/settings/)에 지정된 데이터베이스 백엔드에 맞게 SQL을 사용합니다.

### Using models

모델을 정의한 후에는 Django에게 해당 모델을 사용할 것이라고 알려야합니다. 설정 파일을 편집하고 [<tt style="color: #FF0000">`INSTALLED_APPS`</tt>](https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-INSTALLED_APPS) 설정을 변경하여 <tt style="color: #FF0000">`models.py`</tt>가 포함 된 모듈의 이름을 추가하면됩니다.
예를 들어, application의 모델이 <tt style="color: #FF0000">`myapp.models`</tt> 모듈 ([<tt style="color: #FF0000">`manage.py startapp`</tt>](https://docs.djangoproject.com/en/1.11/ref/django-admin/#django-admin-startapp) 스크립트로 응용 프로그램 용으로 작성된 패키지 구조)에있는 경우 [<tt style="color: #FF0000">`INSTALLED_APPS`</tt>](https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-INSTALLED_APPS)는 부분적으로 다음과 같이 읽어야합니다.
~~~~
INSTALLED_APPS = [
    #...
    'myapp',
    #...
]
~~~~

### Fields

모델의 가장 중요한 부분과 모델의 유일한 required 부분은 정의하는 데이터베이스 fields의 목록입니다. 필드는 클래스 속성에 의해 지정됩니다. [model API](https://docs.djangoproject.com/en/1.11/ref/models/instances/)와 충돌하는 fields 이름을 <tt style="color: #FF0000">`clean`</tt>, <tt style="color: #FF0000">`save`</tt>, 또는 <tt style="color: #FF0000">`delete`</tt>하지 않도록 주의해야 합니다.
예:
~~~~
from django.db import models

class Musician(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    instrument = models.CharField(max_length=100)

class Album(models.Model):
    artist = models.ForeignKey(Musician, on_delete=models.CASCADE)
    name = models.CharField(max_length=100)
    release_date = models.DateField()
    num_stars = models.IntegerField()
~~~~

### Field types

모델의 각 Field는 해당 [Field](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field) 클래스의 인스턴스여야 합니다. Django는 Field 클래스 유형을 사용하여 몇 가지를 결정합니다.
* 데이터베이스에 저장할 데이터 종류 (예 : <tt style="color: #FF0000">`INTEGER`</tt>, <tt style="color: #FF0000">`VARCHAR`</tt>, <tt style="color: #FF0000">`TEXT`</tt>)를 나타내는 column 유형입니다.
* form field를 렌더링 할 때 사용할 기본 HTML [widget](https://docs.djangoproject.com/en/1.11/ref/forms/widgets/)입니다 (예 : <tt style="color: #FF0000">`<input type = "text">`</tt>, <tt style="color: #FF0000">`<select>`</tt>).
* Django’s 관리자 및 automatically-generated forms에서 사용되는 minimal validation requirements.

Django에는 수십 가지 built-in field types이 있습니다. [model field reference](https://docs.djangoproject.com/en/1.11/ref/models/fields/#model-field-types)에서 전체 목록을 찾을 수 있습니다. Django’s built-in field가 도움이 되지 않으면 own fields를 쉽게 쓸 수 있습니다; [Writing custom model fields](https://docs.djangoproject.com/en/1.11/howto/custom-model-fields/)을 참조하십시오.

### Field options

각 field는 field field-specific arguments 집합을 사용합니다. ([model field reference](https://docs.djangoproject.com/en/1.11/ref/models/fields/#model-field-types)에 문서화되어 있음). 예를 들어, [<tt style="color: #FF0000">`CharField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.CharField)(및 해당 하위 클래스)에는 데이터를 저장하는 데 사용되는 <tt style="color: #FF0000">`VARCHAR`</tt> 데이터베이스 필드의 크기를 지정하는 [<tt style="color: #FF0000">`max_length`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.CharField.max_length) argument가 필요합니다.
또한 모든 field 유형에 사용할 수 있는 일반적인 인수 집합이 있습니다. 모두 선택 사항입니다. 그것들은 [reference](https://docs.djangoproject.com/en/1.11/ref/models/fields/#common-model-field-options)에서 완전히 설명되어 있지만, 가장 자주 사용되는 것들에 대한 간단한 요약이 있습니다 :
[<tt style="color: #FF0000">`null`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.null)
<tt style="color: #FF0000">`True`</tt>이면 Django는 빈 값을 <tt style="color: #FF0000">`NULL`</tt>로 데이터베이스에 저장합니다. 기본값은 <tt style="color: #FF0000">`False`</tt>입니다.
[<tt style="color: #FF0000">`blank`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.blank)
<tt style="color: #FF0000">`True`</tt>이면 필드를 비워 둘 수 있습니다. 기본값은 <tt style="color: #FF0000">`False`</tt>입니다.
이것은 <tt style="color: #FF0000">`null`</tt>과 다릅니다. null은 순전히 데이터베이스와 관련된 반면 <tt style="color: #FF0000">`blank`</tt>는 validation와 관련이 있습니다. 필드에 <tt style="color: #FF0000">`blank=True`</tt>가 있으면 양식 유효성 검사에서 빈 값을 입력 할 수 있습니다. 필드에 <tt style="color: #FF0000">`blank=False`</tt>가 있으면 필드가 필요합니다.
[<tt style="color: #FF0000">`choices`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.choices)
이 field의 선택 항목으로 사용할 2-tuples의 반복 가능한 (예 : list 또는 tuples) 이것이 주어지면 기본 양식 위젯은 표준 텍스트 필드 대신 선택 상자가 되어 주어진 선택 사항으로 선택을 제한합니다.
선택 목록은 다음과 같습니다.
~~~~
YEAR_IN_SCHOOL_CHOICES = (
    ('FR', 'Freshman'),
    ('SO', 'Sophomore'),
    ('JR', 'Junior'),
    ('SR', 'Senior'),
    ('GR', 'Graduate'),
)
~~~~
각 tuple의 첫 번째 요소는 데이터베이스에 저장될 값입니다. 두 번째 요소는 기본 양식 widget 또는 [<tt style="color: #FF0000">`ModelChoiceField`</tt>](https://docs.djangoproject.com/en/1.11/ref/forms/fields/#django.forms.ModelChoiceField)에 표시됩니다. 모델 인스턴스가 주어지면 선택 필드의 표시 값은 <tt style="color: #FF0000">`get_FOO_display()`</tt> 메소드를 사용하여 액세스할 수 있습니다. 예 :
~~~~
from django.db import models

class Person(models.Model):
    SHIRT_SIZES = (
        ('S', 'Small'),
        ('M', 'Medium'),
        ('L', 'Large'),
    )
    name = models.CharField(max_length=60)
    shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)
~~~~

~~~~
>>> p = Person(name="Fred Flintstone", shirt_size="L")
>>> p.save()
>>> p.shirt_size
'L'
>>> p.get_shirt_size_display()
'Large'
~~~~
[<tt style="color: #FF0000">`default`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.default)
필드의 기본값입니다. 값 또는 호출 가능 객체 일 수 있습니다. 호출 가능하면 새로운 객체가 생성 될 때마다 호출됩니다.

[<tt style="color: #FF0000">`help_text`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.help_text)
form widget과 함께 표시되는 추가 "도움말"텍스트. form에서 필드를 사용하지 않아도 문서화에 유용합니다.

[<tt style="color: #FF0000">`primary_key`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.primary_key)
<tt style="color: #FF0000">`True`</tt>이면 이 필드는 모델의 primary key가 됩니다.
모델의 모든 필드에 대해 [<tt style="color: #FF0000">`primary_key=True`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.primary_key)를 지정하지 않으면 Django가 기본 키를 보유 할 <tt style="color: #FF0000">`IntegerField`</tt>를 자동으로 추가하므로 해당 필드를 재정의하려는 경우가 아니면 모든 필드에서 [<tt style="color: #FF0000">`primary_key=True`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.primary_key)를 설정할 필요가 없습니다. default primary-key 동작. 자세한 내용은 [Automatic primary key fields](https://docs.djangoproject.com/en/1.11/topics/db/models/#automatic-primary-key-fields)를 참조하십시오.
primary key 필드는 읽기 전용입니다. 기존 개체의 primary key 값을 변경한 다음 저장하면 이전 개체와 함께 새 개체가 만들어집니다. 예 :
~~~~
from django.db import models

class Fruit(models.Model):
    name = models.CharField(max_length=100, primary_key=True)
~~~~

~~~~
>>> fruit = Fruit.objects.create(name='Apple')
>>> fruit.name = 'Pear'
>>> fruit.save()
>>> Fruit.objects.values_list('name', flat=True)
['Apple', 'Pear']
~~~~
[<tt style="color: #FF0000">`unique`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.unique)
<tt style="color: #FF0000">`True`</tt>이면 이 필드는 테이블 전체에서 고유해야합니다.
다시 말하지만, 가장 일반적인 필드 옵션에 대한 간단한 설명입니다. 전체 세부 사항은 [ common model field option reference](https://docs.djangoproject.com/en/1.11/ref/models/fields/#common-model-field-options)에서 찾을 수 있습니다.

### Automatic primary key fields

기본적으로 Django는 각 모델에 다음 필드를 제공합니다.
~~~~
id = models.AutoField(primary_key=True)
~~~~
자동적으로 증가하는 primary key 입니다.
custom primary key를 지정하려면 필드 중 하나에서 [<tt style="color: #FF0000">`primary_key=True`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.primary_key)를 지정하면됩니다. Django가 [<tt style="color: #FF0000">`Field.primary_key`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.primary_key)를 명시적으로 설정했다고 판단하면 자동 ID 열을 추가하지 않습니다.
각 모델은 [<tt style="color: #FF0000">`primary_key=True`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.primary_key) (명시적으로 선언되거나 자동으로 추가됨)를 갖기 위해 정확히 하나의 필드가 필요합니다.

### Verbose field names

[<tt style="color: #FF0000">`ForeignKey`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey), [<tt style="color: #FF0000">`ManyToManyField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField) 및 [<tt style="color: #FF0000">`OneToOneField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField)를 제외한 각 필드 유형은 선택적 첫 번째 위치 argument - 자세한 이름을 사용합니다. verbose 이름이 주어지지 않으면 Django는 밑줄을 공백으로 변환하여 필드의 attribute 이름을 사용하여 verbose 이름을 자동으로 만듭니다.
verbose 이름 <tt style="color: #FF0000">`"person's first name"`</tt>작성 예제입니다.
~~~~
first_name = models.CharField("person's first name", max_length=30)
~~~~
verbose 이름 <tt style="color: #FF0000">`"first name"`</tt>작성 예제입니다.
~~~~
first_name = models.CharField(max_length=30)
~~~~
[<tt style="color: #FF0000">`ForeignKey`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey), [<tt style="color: #FF0000">`ManyToManyField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField) 및 [<tt style="color: #FF0000">`OneToOneField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField)는 첫 번째 argument가 모델 클래스여야하므로 verbose_name 키워드 argument를 사용하십시오.
~~~~
poll = models.ForeignKey(
    Poll,
    on_delete=models.CASCADE,
    verbose_name="the related poll",
)
sites = models.ManyToManyField(Site, verbose_name="list of sites")
place = models.OneToOneField(
    Place,
    on_delete=models.CASCADE,
    verbose_name="related place",
)
~~~~
(관습적으로)verbose_name의 첫 문자를 대문자로 사용하지 않습니다. Django는 필요한 첫 문자를 자동으로 대문자로 만듭니다.

### Relationships

분명히, 관계형 데이터베이스의 힘은 테이블을 서로 연관시키는 데 있습니다. Django는 many-to-one, many-to-many 및 one-to-one 데이터베이스 관계의 세 가지 가장 일반적인 유형을 정의하는 방법을 제공합니다.

#### Many-to-one relationships

many-to-one 관계를 정의하려면 [<tt style="color: #FF0000">`django.db.models.ForeignKey`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey)를 사용하십시오. 다른 [<tt style="color: #FF0000">`Field`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field) 유형과 마찬가지로 모델의 클래스 attribute로 포함하여 사용합니다.
[<tt style="color: #FF0000">`ForeignKey`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey)에는 위치 argument가 필요합니다. 모델이 관련된 클래스입니다.
예를 들어 <tt style="color: #FF0000">`Car`</tt> 모델에 <tt style="color: #FF0000">`Manufacturer`</tt>가있는 경우 - 즉, <tt style="color: #FF0000">`Manufacturer`</tt>가 여러 <tt style="color: #FF0000">`Car`</tt>를 생산하지만 각 <tt style="color: #FF0000">`Car`</tt>에만 하나의 <tt style="color: #FF0000">`Manufacturer`</tt>가있는 경우 - 다음 정의를 사용하십시오.
~~~~
from django.db import models

class Manufacturer(models.Model):
    # ...
    pass

class Car(models.Model):
    manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
    # ...
~~~~
[recursive relationships](https://docs.djangoproject.com/en/1.11/ref/models/fields/#recursive-relationships) (자체와 many-to-one 관계가있는 객체)와 [아직 정의되지 않은 모델과의 관계](https://docs.djangoproject.com/en/1.11/ref/models/fields/#lazy-relationships)를 생성할 수도 있습니다. 자세한 내용은 [the model field reference](https://docs.djangoproject.com/en/1.11/ref/models/fields/#ref-foreignkey)를 참조하십시오.
[<tt style="color: #FF0000">`ForeignKey`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey) 필드의 이름 (위의 예제에서 <tt style="color: #FF0000">`Manufacturer`</tt>)은 소문자인 모델의 이름이 될 것을 권장하지만 필수는 아닙니다. 물론 원하는대로 필드를 호출 할 수 있습니다. 예 :
~~~~
class Car(models.Model):
    company_that_makes_it = models.ForeignKey(
        Manufacturer,
        on_delete=models.CASCADE,
    )
    # ...
~~~~
<b>See also</b>

[<tt style="color: #FF0000">`ForeignKey`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey) 필드는 [the model field reference](https://docs.djangoproject.com/en/1.11/ref/models/fields/#foreign-key-arguments)에서 설명하는 추가 arguments를 허용합니다. 이 옵션은 관계가 작동하는 방법을 정의하는 데 도움이됩니다. 모두 선택 사항입니다.

역방향 관련 객체 액세스에 대한 자세한 내용은 [Following relationships backward example](https://docs.djangoproject.com/en/1.11/topics/db/queries/#backwards-related-objects)를 참조하십시오.

샘플 코드는 [Many-to-one relationship model example](https://docs.djangoproject.com/en/1.11/topics/db/examples/many_to_one/)를 참조하십시오.

#### Many-to-many relationships

many-to-many 관계를 정의하려면 [<tt style="color: #FF0000">`ManyToManyField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField)를 사용하십시오. 다른 [<tt style="color: #FF0000">`Field`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field) 유형과 마찬가지로 모델의 클래스 attribute으로 포함하여 사용합니다.
[<tt style="color: #FF0000">`ManyToManyField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField)에는 위치 argument가 필요합니다. 모델이 관련된 클래스입니다.
예를 들어, <tt style="color: #FF0000">`Pizza`</tt>에 여러 <tt style="color: #FF0000">`Topping`</tt> 개체가있는 경우 - <tt style="color: #FF0000">`Topping`</tt>이 여러 <tt style="color: #FF0000">`Pizza`</tt>에 있을 수 있으며 각 <tt style="color: #FF0000">`Pizza`</tt>에 여러 <tt style="color: #FF0000">`Topping`</tt>이있는 경우 - 여기에 그 <tt style="color: #FF0000">`Pizza`</tt>를 나타내는 방법이 있습니다.
~~~~
from django.db import models

class Topping(models.Model):
    # ...
    pass

class Pizza(models.Model):
    # ...
    toppings = models.ManyToManyField(Topping)
~~~~
[<tt style="color: #FF0000">`ForeignKey`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey)와 마찬가지로 [recursive relationships](https://docs.djangoproject.com/en/1.11/ref/models/fields/#recursive-relationships) (자체와 many-to-many 관계가 있는 객체)와 [아직 정의되지 않은 모델과의 관계](https://docs.djangoproject.com/en/1.11/ref/models/fields/#lazy-relationships)를 만들 수도 있습니다.
[<tt style="color: #FF0000">`ManyToManyField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField) (위의 예제에서 <tt style="color: #FF0000">`Topping`</tt>)의 이름은 관련 모델 객체 세트를 설명하는 복수형으로 제안되지만 필수는 아닙니다.
[<tt style="color: #FF0000">`ManyToManyField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField)가있는 모델은 어디에 삽입하는지가 중요하지 않지만 두 모델 중 하나에만 삽입해야합니다. - 모두에 넣는 것은 아닙니다.
일반적으로 [<tt style="color: #FF0000">`ManyToManyField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField) 인스턴스는 폼에서 편집할 객체에 있어야합니다. 위의 예에서 <tt style="color: #FF0000">`Topping`</tt>은 여러 개의 <tt style="color: #FF0000">`Pizza`</tt>에 <tt style="color: #FF0000">`Topping`</tt>을하는 것보다 <tt style="color: #FF0000">`Topping`</tt>을 하는 <tt style="color: #FF0000">`Pizza`</tt>에 대해 생각하는 것이 자연스럽기 때문에 <tt style="color: #FF0000">`Pizza`</tt> (<tt style="color: #FF0000">`Pizza`</tt> ManyToManyField가있는 <tt style="color: #FF0000">`Topping`</tt>이 아닌)에 있습니다. 위에 설정된 방식대로 <tt style="color: #FF0000">`Pizza`</tt> 양식을 사용하면 사용자가 <tt style="color: #FF0000">`Topping`</tt>을 선택할 수 있습니다.

<b>See also</b>

전체 예제는 [Many-to-many relationship model example](https://docs.djangoproject.com/en/1.11/topics/db/examples/many_to_many/)를 참조하십시오.

[<tt style="color: #FF0000">`ManyToManyField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField) 필드는 [the model field reference](https://docs.djangoproject.com/en/1.11/ref/models/fields/#manytomany-arguments)에서 설명하는 많은 추가 인수를 허용합니다. 이 옵션은 관계가 작동하는 방법을 정의하는 데 도움이 됩니다. 모두 선택 사항입니다.

#### Extra fields on many-to-many relationships

pizza와 toppings을 섞거나 일치시키는 것과 같은 단순한 many-to-many 관계만 처리할 때는 표준 [<tt style="color: #FF0000">`ManyToManyField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField)만 있으면됩니다. 그러나 때로는 두 모델 간의 관계에 데이터를 연결해야 할 수도 있습니다.
예를 들어 음악가가 속한 음악 그룹을 추적하는 application의 경우를 생각해보십시오. 개인과 회원이 속해있는 그룹간에는 many-to-many 관계가 있으므로 이 관계를 나타내기 위해 [<tt style="color: #FF0000">`ManyToManyField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField)를 사용할 수 있습니다. 그러나 그룹에 가입 한 날짜와 같이 수집하고자하는 멤버쉽에 대한 세부 정보가 많이 있습니다.
이러한 상황에서 Django는 many-to-many 관계를 관리하는 데 사용될 모델을 지정할 수 있습니다. 그러면 중개 모델에 필드를 추가 할 수 있습니다. 중개 모델은 [<tt style="color: #FF0000">`through`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField.through) argument를 사용하여 중개자 역할을 하는 모델을 가리키는 [<tt style="color: #FF0000">`ManyToManyField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField)와 연결됩니다. 뮤지션의 경우, 코드는 다음과 같습니다.
~~~~
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=128)

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership')

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Membership(models.Model):
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)
~~~~
중개 모델을 설정할 때, many-to-many 관계와 관련된 모델에 대한 foreign keys를 명시적으로 지정합니다. 이 명시적 선언은 두 모델이 관련되는 방식을 정의합니다.
중개 모델에는 몇 가지 제한 사항이 있습니다.
* 중개 모델에는 소스 모델에 대한 foreign key가 하나만 포함되어야합니다 (이 예에서는 <tt style="color: #FF0000">`Group`</tt>입니다). 또는 Django가 [<tt style="color: #FF0000">`ManyToManyField.through_fields`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField.through_fields)를 사용하여 관계에 사용해야하는 foreign key를 명시적으로 지정해야합니다. 둘 이상의 foreign key가 있고 <tt style="color: #FF0000">`through_fields`</tt>가 지정되지 않은 경우 validation error가 발생합니다. 비슷한 제한이 대상 모델에 대한 foreign key에도 적용됩니다 (이 예에서는 <tt style="color: #FF0000">`Person`</tt>입니다).
* 중개 모델을 통해 many-to-many 관계를 갖는 모델의 경우 동일한 모델에 대한 두 개의 foreign key가 허용되지만 many-to-many 관계의 두 (다른) 측면으로 처리됩니다. 두 개 이상의 foreign key가있는 경우 위와 같이 through_fields도 지정해야합니다. 그렇지 않으면 validation error가 발생합니다.
* 중개 모델을 사용하여 모델에서 many-to-many 관계를 정의 할 때는 [<tt style="color: #FF0000">`symmetrical=False`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField.symmetrical)를 사용해야합니다 (모델 필드 참조 참조).

이제 중개 모델 (이 경우 <tt style="color: #FF0000">`Membership`</tt>)을 사용하도록 [<tt style="color: #FF0000">`ManyToManyField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField)를 설정했으므로 이제 many-to-many 관계를 만들 준비가 된 것입니다. 중개 모델의 인스턴스를 생성하여 이를 수행합니다.
~~~~
>>> ringo = Person.objects.create(name="Ringo Starr")
>>> paul = Person.objects.create(name="Paul McCartney")
>>> beatles = Group.objects.create(name="The Beatles")
>>> m1 = Membership(person=ringo, group=beatles,
...     date_joined=date(1962, 8, 16),
...     invite_reason="Needed a new drummer.")
>>> m1.save()
>>> beatles.members.all()
<QuerySet [<Person: Ringo Starr>]>
>>> ringo.group_set.all()
<QuerySet [<Group: The Beatles>]>
>>> m2 = Membership.objects.create(person=paul, group=beatles,
...     date_joined=date(1960, 8, 1),
...     invite_reason="Wanted to form a band.")
>>> beatles.members.all()
<QuerySet [<Person: Ringo Starr>, <Person: Paul McCartney>]>
~~~~
일반적인 many-to-many 필드와 달리 <tt style="color: #FF0000">`add()`</tt>, <tt style="color: #FF0000">`create()`</tt> 또는 <tt style="color: #FF0000">`set()`</tt>을 사용하여 관계를 만들 수 없습니다.
~~~~
>>> # The following statements will not work
>>> beatles.members.add(john)
>>> beatles.members.create(name="George Harrison")
>>> beatles.members.set([john, paul, ringo, george])
~~~~
왜 그럴까요? <tt style="color: #FF0000">`Person`</tt>과 <tt style="color: #FF0000">`Group`</tt> 간의 관계만 만들 수는 없습니다. <tt style="color: #FF0000">`Membership`</tt> 모델에 필요한 관계에 대한 모든 세부 사항을 지정해야합니다. 간단한 <tt style="color: #FF0000">`add()`</tt>, <tt style="color: #FF0000">`create()`</tt> 및 할당 호출은 이러한 추가 세부 사항을 지정하는 방법을 제공하지 않습니다. 결과적으로, 중개 모델을 사용하는 many-to-many 관계에 대해서는 사용 불가능합니다. 이러한 유형의 관계를 작성하는 유일한 방법은 중개 모델의 인스턴스를 작성하는 것입니다.
유사한 이유로 [<tt style="color: #FF0000">`remove()`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/relations/#django.db.models.fields.related.RelatedManager.remove) 메서드가 비활성화됩니다. 예를 들어, 중개 모델에 의해 정의된 사용자 정의 테이블을 통해 <tt style="color: #FF0000">`(model1, model2)`</tt> 쌍의 고유성을 적용하지 않으면 <tt style="color: #FF0000">`remove()`</tt> 호출은 어떤 중개 모델 인스턴스를 삭제해야하는지에 대한 충분한 정보를 제공하지 않습니다.
~~~~
>>> Membership.objects.create(person=ringo, group=beatles,
...     date_joined=date(1968, 9, 4),
...     invite_reason="You've been gone for a month and we miss you.")
>>> beatles.members.all()
<QuerySet [<Person: Ringo Starr>, <Person: Paul McCartney>, <Person: Ringo Starr>]>
>>> # This will not work because it cannot tell which membership to remove
>>> beatles.members.remove(ringo)
~~~~
그러나 [<tt style="color: #FF0000">`clear()`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/relations/#django.db.models.fields.related.RelatedManager.clear) 메서드는 인스턴스의 모든 many-to-many 관계를 제거하는 데 사용할 수 있습니다.
~~~~
>>> # Beatles have broken up
>>> beatles.members.clear()
>>> # Note that this deletes the intermediate model instances
>>> Membership.objects.all()
<QuerySet []>
~~~~
중개 모델의 인스턴스를 작성하여 many-to-many 관계를 설정하면 query를 실행할 수 있습니다. 일반적인 many-to-many 관계와 마찬가지로 many-to-many 관련 모델의 특성을 사용하여 query 할 수 ​​있습니다.
~~~~
# Find all the groups with a member whose name starts with 'Paul'
>>> Group.objects.filter(members__name__startswith='Paul')
<QuerySet [<Group: The Beatles>]>
~~~~
중개 모델을 사용할 때 해당 attributes에 대해서도 query 할 수 ​​있습니다.
~~~~
# Find all the members of the Beatles that joined after 1 Jan 1961
>>> Person.objects.filter(
...     group__name='The Beatles',
...     membership__date_joined__gt=date(1961,1,1))
<QuerySet [<Person: Ringo Starr]>
~~~~
회원 정보에 액세스해야하는 경우 <tt style="color: #FF0000">`Membership`</tt> 모델에 직접 query하여 할 수 있습니다.
~~~~
>>> ringos_membership = Membership.objects.get(group=beatles, person=ringo)
>>> ringos_membership.date_joined
datetime.date(1962, 8, 16)
>>> ringos_membership.invite_reason
'Needed a new drummer.'
~~~~
동일한 정보에 접근하는 또 다른 방법은 <tt style="color: #FF0000">`Person`</tt> 객체에서 [many-to-many reverse relationship](https://docs.djangoproject.com/en/1.11/topics/db/queries/#m2m-reverse-relationships)를 query하는 것입니다.
~~~~
>>> ringos_membership = ringo.membership_set.get(group=beatles)
>>> ringos_membership.date_joined
datetime.date(1962, 8, 16)
>>> ringos_membership.invite_reason
'Needed a new drummer.'
~~~~

#### One-to-one relationships

one-to-one 관계를 정의하려면 [<tt style="color: #FF0000">`OneToOneField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField)를 사용하십시오. 다른 Field 유형과 마찬가지로 모델의 클래스 attribute으로 포함하여 사용합니다.
이것은 객체의 primary key에서 객체가 어떤 방식으로 다른 객체로 "확장"할 때 가장 유용합니다.
[<tt style="color: #FF0000">`OneToOneField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField)에는 위치 argument인 모델과 관련된 클래스가 필요합니다.
예를 들어, "장소"의 데이터베이스를 구축했다면 데이터베이스에 주소, 전화번호 등과 같은 표준적인 stuff를 만들 수 있습니다. 그런 다음, <tt style="color: #FF0000">`Restaurant`</tt>의 데이터베이스를 구축하고 <tt style="color: #FF0000">`Restaurant`</tt> 모델에서 해당 필드를 복제하는 대신 <tt style="color: #FF0000">`Restaurant`</tt>을 [<tt style="color: #FF0000">`OneToOneField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField) to <tt style="color: #FF0000">`Place`</tt>로 만들 수 있습니다 (식당은 "장소"이기 때문에). 사실, 이것을 처리하기 위해 암시적 one-to-one 관계가 포함된 상속을 일반적으로 사용합니다.
[<tt style="color: #FF0000">`ForeignKey`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey)와 마찬가지로 [recursive relationship](https://docs.djangoproject.com/en/1.11/ref/models/fields/#recursive-relationships)를 정의 할 수 있으며 [아직 정의되지 않은 모델에 대한 참조](https://docs.djangoproject.com/en/1.11/ref/models/fields/#lazy-relationships)를 만들 수 있습니다.

<b>See also</b>

전체 예제는 [One-to-one relationship model example](https://docs.djangoproject.com/en/1.11/topics/db/examples/one_to_one/)를 참조하십시오.

[<tt style="color: #FF0000">`OneToOneField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField) 필드는 선택적 [<tt style="color: #FF0000">`parent_link`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField.parent_link) argument도 허용합니다.
[<tt style="color: #FF0000">`OneToOneField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField) 클래스는 모델에서 자동으로 primary key가됩니다. 원하는 경우 [<tt style="color: #FF0000">`primary_key`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.primary_key) argument를 수동으로 전달할 수는 있지만 더 이상 사실이 아닙니다. 따라서 이제 단일 모델에 [<tt style="color: #FF0000">`OneToOneField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField) 유형의 여러 필드를 포함할 수 있습니다.

### Models across files

모델을 다른 앱의 모델과 연결하는 것은 perfectly OK합니다. 이렇게 하려면 모델이 정의된 파일의 맨 위에 관련 모델을 가져옵니다. 그런 다음 필요할 때마다 다른 모델 클래스를 참조하십시오. 예 :
~~~~
from django.db import models
from geography.models import ZipCode

class Restaurant(models.Model):
    # ...
    zip_code = models.ForeignKey(
        ZipCode,
        on_delete=models.SET_NULL,
        blank=True,
        null=True,
    )
~~~~

#### Field name restrictions

Django는 모델 필드 이름에 두 가지 제한이 있습니다.
1. 파이썬 예약어는 파이썬 구문 오류가 발생하기 때문에 필드 이름이 될 수 없습니다. 예 :
~~~~
class Example(models.Model):
    pass = models.IntegerField() # 'pass' is a reserved word!
~~~~
2. 필드 이름은 Django의 query 조회 구문이 작동하는 방식으로 두 개 이상의 underscore를 포함 할 수 없습니다. 예 :
~~~~
class Example(models.Model):
    foo__bar = models.IntegerField() # 'foo__bar' has two underscores!
~~~~
그러나 이러한 제한 사항은 필드 이름이 데이터베이스 column 이름과 일치할 필요가 없기 때문에 해결될 수 있습니다. [<tt style="color: #FF0000">`db_column`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.db_column) 옵션을 참조하십시오.
Django는 모든 기본 SQL 쿼리의 모든 데이터베이스 테이블 이름과 column 이름을 이스케이프 처리하므로 SQL 예약어 (예 : <tt style="color: #FF0000">`join`</tt>, <tt style="color: #FF0000">`where`</tt> 또는 <tt style="color: #FF0000">`select`</tt>)는 모델 필드 이름으로 사용할 수 있습니다. 특정 데이터베이스 엔진의 인용 구문을 사용합니다.

#### Custom field types

기존 모델 필드 중 하나를 사용하여 용도에 맞지 않거나 덜 일반적으로 사용되는 데이터베이스 column 유형을 활용하려는 경우 자체 필드 클래스를 만들 수 있습니다. 자체 필드 작성에 대한 전체 내용은 [Writing custom model fields](https://docs.djangoproject.com/en/1.11/howto/custom-model-fields/)에 나와 있습니다.

### Meta options

다음과 같이 내부 클래스 Meta를 사용하여 모델 메타 데이터를 설정할 수 있습니다.
~~~~
from django.db import models

class Ox(models.Model):
    horn_length = models.IntegerField()

    class Meta:
        ordering = ["horn_length"]
        verbose_name_plural = "oxen"
~~~~
모델 메타 데이터는 ordering 옵션 ([<tt style="color: #FF0000">`ordering`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/options/#django.db.models.Options.ordering)), 데이터베이스 테이블 이름 ([<tt style="color: #FF0000">`db_table`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/options/#django.db.models.Options.db_table)) 또는 인간이 읽을 수 있는 단수 및 복수 이름 ([<tt style="color: #FF0000">`verbose_name`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/options/#django.db.models.Options.verbose_name) 및 [<tt style="color: #FF0000">`verbose_name_plural`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/options/#django.db.models.Options.verbose_name_plural))과 같이 "필드가 아닌 모든 것"입니다. 아무 것도 필요하지 않으며 <tt style="color: #FF0000">`class Meta`</tt>를 모델에 추가하는 것은 완전히 선택 사항입니다.
가능한 모든 메타 옵션의 전체 목록은 [model option reference](https://docs.djangoproject.com/en/1.11/ref/models/options/)에서 찾을 수 있습니다.

### Model attributes

<b>object</b>

모델의 가장 중요한 attribute은 [<tt style="color: #FF0000">`Manager`</tt>](https://docs.djangoproject.com/en/1.11/topics/db/managers/#django.db.models.Manager)입니다. Django 모델에 데이터베이스 query 연산이 제공되고 데이터베이스에서 [retrieve the instances](https://docs.djangoproject.com/en/1.11/topics/db/queries/#retrieving-objects)하는데 사용되는 인터페이스입니다. custom <tt style="color: #FF0000">`Manager`</tt>가 정의되지 않은 경우 default 이름은 [<tt style="color: #FF0000">`objects`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/class/#django.db.models.Model.objects)입니다. Manager는 모델 클래스가 아닌 모델 인스턴스를 통해서만 액세스할 수 있습니다.

### Model methods

모델에 custom methods를 정의하여 custom "row-level" functionality를 객체에 추가합니다. [<tt style="color: #FF0000">`Manager`</tt>](https://docs.djangoproject.com/en/1.11/topics/db/managers/#django.db.models.Manager) 메소드는 "table-wide"작업을 수행하는 것이지만, 모델 메소드는 특정 모델 인스턴스에서 작동해야합니다.
This is a valuable technique for keeping business logic in one place – the model.
예를 들어,이 모델에는 몇 가지 custom 메소드가 있습니다.
~~~~
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    birth_date = models.DateField()

    def baby_boomer_status(self):
        "Returns the person's baby-boomer status."
        import datetime
        if self.birth_date < datetime.date(1945, 8, 1):
            return "Pre-boomer"
        elif self.birth_date < datetime.date(1965, 1, 1):
            return "Baby boomer"
        else:
            return "Post-boomer"

    @property
    def full_name(self):
        "Returns the person's full name."
        return '%s %s' % (self.first_name, self.last_name)
~~~~
이 예제의 마지막 메서드는 [property](https://docs.djangoproject.com/en/1.11/glossary/#term-property)입니다.
[model instance reference](https://docs.djangoproject.com/en/1.11/ref/models/instances/)에는 [methods automatically given to each model](https://docs.djangoproject.com/en/1.11/ref/models/instances/#model-instance-methods)의 전체 목록이 있습니다. 대부분의 모델을 override할 수 있습니다. 아래에서 [overriding predefined model methods](https://docs.djangoproject.com/en/1.11/topics/db/models/#overriding-predefined-model-methods) 할 수 있습니다.하지만 거의 항상 정의하고 싶은 몇 가지 방법이 있습니다.

<tt style="color: #FF0000">`__str__()`</tt>(Python 3)

모든 객체의 유니코드 "representation"을 반환하는 Python "magic method". 이것은 모델 인스턴스를 강제로 문자열로 표시해야할 때마다 Python과 Django가 사용하게 될 것입니다. 특히 이것은 interactive 콘솔이나 admin에 개체를 표시할 때 발생합니다.
항상이 메소드를 정의하고 싶을 것이다. default는 전혀 도움이 되지 않습니다.

[<tt style="color: #FF0000">`get_absolute_url()`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model.get_absolute_url)

Django는 객체의 URL을 calculate하는 방법을 알려줍니다. Django는 admin interface에서 이것을 사용하며 언제든지 객체의 URL을 찾아야합니다.
객체를 고유하게 식별하는 URL을 가진 객체는 이 메소드를 정의해야합니다.

#### Overriding predefined model methods

customize 할 데이터베이스 behavior를 캡슐화하는 또 다른 [model methods](https://docs.djangoproject.com/en/1.11/ref/models/instances/#model-instance-methods) 집합이 있습니다. 특히 [<tt style="color: #FF0000">`save()`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model.save) 및 [<tt style="color: #FF0000">`delete()`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model.delete) 작업 방식을 변경하려는 경우가 많습니다.
alter behavior 위해 이 방법들 (그리고 다른 어떤 모델 방법)을 자유롭게 override할 수 있습니다.
built-in 메서드를 재정의하기 위한 classic 사용 사례는 개체를 저장할 때마다 어떤 일이 일어나길 원할 때입니다. 예를 들어 (받아들이는 parameters의 문서는 [<tt style="color: #FF0000">`save()`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model.save)를 참조하십시오) :
~~~~
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def save(self, *args, **kwargs):
        do_something()
        super(Blog, self).save(*args, **kwargs) # Call the "real" save() method.
        do_something_else()
~~~~
저장을 방지 할 수도 있습니다.
~~~~
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def save(self, *args, **kwargs):
        if self.name == "Yoko Ono's blog":
            return # Yoko shall never have her own blog!
        else:
            super(Blog, self).save(*args, **kwargs) # Call the "real" save() method.
~~~~
superclass 메소드를 호출하는 것을 기억하는 것이 중요합니다. <tt style="color: #FF0000">`super(Blog, self).save(*args, **kwargs)`</tt> business입니다. 객체가 여전히 데이터베이스에 저장되도록합니다. superclass 메서드를 호출하는 것을 잊어버리면 기본 behavior가 발생하지 않고 데이터베이스에 손을 대지 않습니다.
또한 모델 메서드에 전달할 수 있는 arguments를 전달하는 것이 중요합니다. 즉, <tt style="color: #FF0000">`*args`</tt>, <tt style="color: #FF0000">`**kwargs`</tt> bit가 하는 것입니다. Django는 때때로 빌트인 모델 메서드의 기능을 확장하여 새로운 arguments를 추가합니다. 메소드 정의에서 <tt style="color: #FF0000">`*args`</tt>, <tt style="color: #FF0000">`**kwargs`</tt>를 사용하면 코드가 추가될 때 해당 arguments를 자동으로 지원한다는 보장이 있습니다.

<b>Note</b>

[deleting objects in bulk using a QuerySet](https://docs.djangoproject.com/en/1.11/topics/db/queries/#topics-db-queries-delete) 또는 [<tt style="color: #FF0000">`cascading delete`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.on_delete)의 결과로 객체의 [<tt style="color: #FF0000">`delete()`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model.delete) 메소드가 반드시 호출되지는 않습니다. customized delete logic를 실행하려면 [<tt style="color: #FF0000">`pre_delete`</tt>](https://docs.djangoproject.com/en/1.11/ref/signals/#django.db.models.signals.pre_delete) and/or [<tt style="color: #FF0000">`post_delete`</tt>](https://docs.djangoproject.com/en/1.11/ref/signals/#django.db.models.signals.post_delete) 신호를 사용할 수 있습니다.
불행하게도 [<tt style="color: #FF0000">`save()`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model.save), [<tt style="color: #FF0000">`pre_save`</tt>](https://docs.djangoproject.com/en/1.11/ref/signals/#django.db.models.signals.pre_save) 및 [<tt style="color: #FF0000">`post_save`</tt>](https://docs.djangoproject.com/en/1.11/ref/signals/#django.db.models.signals.post_save)가 호출되지 않으므로 객체를 대량으로 [<tt style="color: #FF0000">`creating`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.query.QuerySet.bulk_create) or [<tt style="color: #FF0000">`updating`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.query.QuerySet.update) 할 때 해결 방법이 없습니다.

### Executing custom SQL

또 다른 common pattern은 모델 메서드 및 모듈 수준 메서드에 custom SQL 문을 작성하는 것입니다. raw SQL 사용에 대한 자세한 내용은 [using raw SQL](https://docs.djangoproject.com/en/1.11/topics/db/sql/)에 대한 설명서를 참조하십시오.


## Model inheritance

Django의 모델 상속은 일반적인 클래스 상속이 Python에서 작동하는 방식과 거의 동일하게 작동하지만 페이지 시작 부분의 기본 사항은 계속 따라야합니다. 이는 기본 클래스가 [<tt style="color: #FF0000">`django.db.models.Model`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model)을 서브 클래스화해야 함을 의미합니다.
부모 모델이 자신만의 모델 (자체 데이터베이스 테이블 포함)이 될지 또는 부모가 자식 모델을 통해서만 볼 수 있는 공통 정보를 보유하고 있는지 여부만 결정하면됩니다.
장고에서 가능한 세 가지 스타일의 상속이 있습니다.
1. 흔히 부모 클래스를 사용하여 각 하위 모델에 대해 입력하지 않으려는 정보를 보유하기를 원할 것입니다. 이 클래스는 따로 분리하여 사용하지 않으므로 [Abstract base classes](https://docs.djangoproject.com/en/1.11/topics/db/models/#abstract-base-classes)가 여러분이 추구하는 것입니다.
2. 기존 모델을 하위 클래스화 (다른 애플리케이션의 일부분 일 수도 있음)하고 각 모델에 자체 데이터베이스 테이블을 갖기를 원한다면 [Multi-table inheritance](https://docs.djangoproject.com/en/1.11/topics/db/models/#multi-table-inheritance)이 필요합니다.
3. 마지막으로, 모델 필드를 변경하지 않고 모델의 파이썬 수준 동작 만 수정하려는 경우 [Proxy models](https://docs.djangoproject.com/en/1.11/topics/db/models/#proxy-models)을 사용할 수 있습니다.


### Abstract base classes

추상 기본 클래스는 몇 가지 공통된 정보를 여러 다른 모델에 넣으려 할 때 유용합니다. 당신은 당신의 기본 클래스를 작성하고 [Meta](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-options) 클래스에 <tt style="color: #FF0000">`abstract=True`</tt>를 넣는다. 이 모델은 데이터베이스 테이블을 만드는 데 사용되지 않습니다. 대신 다른 모델의 기본 클래스로 사용될 때 해당 필드는 자식 클래스의 필드에 추가됩니다. 자식의 이름과 같은 이름을 가진 추상 기본 클래스의 필드를 갖는 것은 오류입니다 (장고는 예외를 발생시킵니다).
예 :
~~~~
from django.db import models

class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()

    class Meta:
        abstract = True

class Student(CommonInfo):
    home_group = models.CharField(max_length=5)
~~~~

<tt style="color: #FF0000">`Student`</tt> 모델에는 <tt style="color: #FF0000">`age`</tt>, <tt style="color: #FF0000">`name`</tt> 및 <tt style="color: #FF0000">`home_group`</tt>의 세 가지 필드가 있습니다. <tt style="color: #FF0000">`CommonInfo`</tt> 모델은 abstract적인 기본 클래스이기 때문에 일반 Django 모델로 사용할 수 없습니다. 데이터베이스 테이블을 생성하거나 관리자가 없으므로 직접 인스턴스화 하거나 저장할 수 없습니다.
많은 용도로 이 유형의 모델 상속이 원하는 것입니다. 이것은 데이터베이스 레벨에서 자식 모델당 하나의 데이터베이스 테이블만 생성하면서 Python 레벨에서 공통 정보를 제외시키는 방법을 제공합니다.

#### Meta inheritance

abstract 기본 클래스가 생성되면 Django는 기본 클래스에서 선언한 [Meta](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-options) 내부 클래스를 속성으로 사용할 수 있게합니다. 자식 클래스가 자신의 [Meta](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-options) 클래스를 선언하지 않으면 부모 클래스의 [Meta](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-options)를 상속받습니다. 자식이 부모의 [Meta](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-options) 클래스를 확장하려고하면 하위 클래스를 하위 클래스로 만들 수 있습니다. 
예 :
~~~~
from django.db import models

class CommonInfo(models.Model):
    # ...
    class Meta:
        abstract = True
        ordering = ['name']

class Student(CommonInfo):
    # ...
    class Meta(CommonInfo.Meta):
        db_table = 'student_info'
~~~~
Django는 abstract 기본 클래스의 [Meta](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-options) 클래스를 조정합니다. [Meta](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-options) attribute을 설치하기 전에 <tt style="color: #FF0000">`abstract=False`</tt>로 설정합니다. 즉, abstract 기본 클래스의 자식은 자동으로 abstract 클래스 자체가 되지 않습니다. 물론 다른 abstract 기본 클래스에서 상속받은 abstract 기본 클래스를 만들 수 있습니다. 매번 <tt style="color: #FF0000">`abstract=True`</tt> 를 명시적으로 설정하는 것을 기억하면됩니다.
일부 속성은 abstract 기본 클래스의 [Meta](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-options) 클래스에 포함하는 것이 타당하지 않습니다. 예를 들어, <tt style="color: #FF0000">`db_table`</tt>을 포함하는 것은 모든 자식 클래스 (자신의 [Meta](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-options)를 지정하지 않은 클래스)가 동일한 데이터베이스 테이블을 사용한다는 것을 의미합니다. 이는 거의 확실하지 않습니다.

#### Be careful with related_name and related_query_name

<tt style="color: #FF0000">`ForeignKey`</tt> 또는 <tt style="color: #FF0000">`ManyToManyField`</tt>에서 [<tt style="color: #FF0000">`related_name`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.related_name) 또는 [<tt style="color: #FF0000">`related_query_name`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.related_query_name)을 사용하는 경우 필드의 unique reverse 이름 및 쿼리 이름을 항상 지정해야합니다. 이 클래스의 필드는 매번 attributes ([<tt style="color: #FF0000">`related_name`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.related_name) 및 [<tt style="color: #FF0000">`related_query_name`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.related_query_name) 포함)에 대해 동일한 값을 가질 때마다 각 하위 클래스에 포함되므로 일반적으로 abstract 기본 클래스에서 문제가 발생합니다.
이 문제를 해결하려면 abstract 기본 클래스에서 [<tt style="color: #FF0000">`related_name`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.related_name) 또는 [<tt style="color: #FF0000">`related_query_name`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.related_query_name)을 사용할 때 값의 일부에 <tt style="color: #FF0000">`'%(app_label)s'`</tt>및 <tt style="color: #FF0000">`'%(class)s'`</tt>가 있어야합니다.
* <tt style="color: #FF0000">`'%(class)s'`</tt>는 필드가 사용되는 하위 클래스의 lower-cased 이름으로 대체됩니다.
* <tt style="color: #FF0000">`'%(app_label)s'`</tt>는 하위 클래스가 포함 된 lower-cased 이름으로 바뀝니다. 설치된 각 application 이름은 고유해야하며 각 application 내의 모델 클래스 이름도 고유해야 하므로 결과 이름이 달라집니다.

예를 들어 app <tt style="color: #FF0000">`common/models.py`</tt>
~~~~
from django.db import models

class Base(models.Model):
    m2m = models.ManyToManyField(
        OtherModel,
        related_name="%(app_label)s_%(class)s_related",
        related_query_name="%(app_label)s_%(class)ss",
    )

    class Meta:
        abstract = True

class ChildA(Base):
    pass

class ChildB(Base):
    pass
~~~~
다른 app <tt style="color: #FF0000">`rare/models.py`</tt>
~~~~
from common.models import Base

class ChildB(Base):
    pass
~~~~
<tt style="color: #FF0000">`common.ChildA.m2m`</tt> 필드의 reverse 이름은 <tt style="color: #FF0000">`common_childa_related`</tt>이고 reverse 쿼리 이름은 <tt style="color: #FF0000">`common_childas`</tt>입니다. <tt style="color: #FF0000">`common.ChildB.m2m`</tt> 필드의 reverse 이름은 <tt style="color: #FF0000">`common_childb_related`</tt>이고 reverse 쿼리 이름은 <tt style="color: #FF0000">`common_childbs`</tt>입니다. 마지막으로, <tt style="color: #FF0000">`rare.ChildB.m2m`</tt> 필드의 reverse 이름은 <tt style="color: #FF0000">`rare_childb_related`</tt>이고 reverse 쿼리 이름은 <tt style="color: #FF0000">`rare_childbs`</tt>입니다. <tt style="color: #FF0000">`'%(class)s'`</tt>및 <tt style="color: #FF0000">`'%(app_label)s'`</tt>부분을 사용하여 관련 이름 또는 관련 검색어 이름을 작성하는 방법은 사용자에게 달렸지만 사용을 잊어버리면 장고는 시스템 검사를 수행할 때 오류가 발생합니다 (또는 마이그레이션 실행할 때).
abstract 기본 클래스의 필드에 [<tt style="color: #FF0000">`related_name`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.related_name) attribute를 지정하지 않으면 기본 reverse 이름은 필드를 직접 선언한 경우와 마찬가지로 <tt style="color: #FF0000">'_set'</tt>이 오는 자식 클래스의 이름이됩니다. 자식 클래스에. 예를 들어, 위 코드에서 [<tt style="color: #FF0000">`related_name`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.related_name) attribute를 생략하면 <tt style="color: #FF0000">`m2m`</tt> 필드의 reverse 이름은 <tt style="color: #FF0000">`ChildA`</tt>의 경우 <tt style="color: #FF0000">`childa_set`</tt>이되고 <tt style="color: #FF0000">`ChildB`</tt> 필드의 경우 <tt style="color: #FF0000">`childb_set`</tt>이됩니다.

Changed in Django 1.10:
Interpolation of '%(app_label)s' and '%(class)s' for related_query_name was added.

### Multi-table inheritance

Django가 지원하는 모델 상속의 두 번째 유형은 계층 구조의 각 모델이 모두 하나의 모델일 때입니다. 각 모델은 자체 데이터베이스 테이블에 해당하며 개별적으로 쿼리하고 생성할 수 있습니다. 상속 관계는 자동으로 생성 된 [<tt style="color: #FF0000">`OneToOneField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField)를 통해 하위 모델과 각 부모 사이의 링크를 도입합니다. 예 :
~~~~
from django.db import models

class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

class Restaurant(Place):
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)
~~~~
<tt style="color: #FF0000">`Place`</tt>의 모든 필드는 <tt style="color: #FF0000">`Restaurant`</tt>에서 사용할 수 있습니다. 데이터는 다른 데이터베이스 테이블에 저장됩니다. 그래서 둘 다 가능합니다 :
~~~~
>>> Place.objects.filter(name="Bob's Cafe")
>>> Restaurant.objects.filter(name="Bob's Cafe")
~~~~
Restaurant이 있는 Place가 있는 경우 모델 이름의 소문자 버전을 사용하여 Place 개체에서 Restaurant 개체로 이동할 수 있습니다.
~~~~
>>> p = Place.objects.get(id=12)
# If p is a Restaurant object, this will give the child class:
>>> p.restaurant
<Restaurant: ...>
~~~~
그러나 위 예제의 <tt style="color: #FF0000">`p`</tt>가 <tt style="color: #FF0000">`Restaurant`</tt>이 아닌 경우 (즉, <tt style="color: #FF0000">`Place`</tt> 객체로 직접 작성되었거나 다른 클래스의 부모인 경우) <tt style="color: #FF0000">`p.restaurant`</tt>를 참조하면 <tt style="color: #FF0000">`Restaurant.DoesNotExist`</tt> exception이 발생합니다.
<tt style="color: #FF0000">`Place`</tt>에 연결된 <tt style="color: #FF0000">`Restaurant`</tt>에 자동으로 생성된 [<tt style="color: #FF0000">`OneToOneField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField)가 다음과 같이 표시됩니다.
~~~~
place_ptr = models.OneToOneField(
    Place, on_delete=models.CASCADE,
    parent_link=True,
)
~~~~
<tt style="color: #FF0000">`Restaurant`</tt>에서 [<tt style="color: #FF0000">`parent_link=True`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField.parent_link)로 자신의 [<tt style="color: #FF0000">`OneToOneField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField)를 선언하여 해당 필드를 재정의할 수 있습니다.

#### Meta and multi-table inheritance

다중 테이블 상속 상황에서 자식 클래스가 부모의 [Meta](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-options) 클래스에서 상속받는 것은 의미가 없습니다. 모든 [Meta](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-options) 옵션은 이미 상위 클래스에 적용되었고 다시 적용하면 모순된 행동만 발생합니다 (이는 기본 클래스가 자체적으로 존재하지 않는 abstract 기본 클래스의 경우와 대조적입니다).
따라서 자식 모델은 부모의 [Meta](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-options) 클래스에 액세스할 수 없습니다. 그러나 자식이 부모로부터 동작을 상속하는 몇 가지 제한된 경우가 있습니다. 자식이 [<tt style="color: #FF0000">`ordering`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/options/#django.db.models.Options.ordering) attribute나 [<tt style="color: #FF0000">`get_latest_by`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/options/#django.db.models.Options.get_latest_by) attribute를 지정하지 않으면 해당 attribute를 부모로부터 상속합니다.
부모가 ordering을 하고 있고 자식이 natural ordering를 갖지 않게 하기 위해 명시적으로 disable 할 수 있습니다.
~~~~
class ChildModel(ParentModel):
    # ...
    class Meta:
        # Remove parent's ordering effect
        ordering = []
~~~~

#### Inheritance and reverse relations

multi-table 상속은 암시적 [<tt style="color: #FF0000">`OneToOneField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField)를 사용하여 자식과 부모를 연결하기 때문에 위의 예에서처럼 부모에서 자식으로 이동할 수 있습니다. 그러나 이는 [<tt style="color: #FF0000">`ForeignKey`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey) 및 [<tt style="color: #FF0000">`ForeignKey`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey), [<tt style="color: #FF0000">`ManyToManyField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField) 관계에 대한 기본 [<tt style="color: #FF0000">`related_name`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.related_name) 값인 이름을 사용합니다. 이러한 관계 유형을 상위 모델의 하위 클래스에 배치하는 경우 해당 필드 각각에 [<tt style="color: #FF0000">`related_name`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.related_name) attribute을 지정해야 합니다. 지정하지 않으면 Django는 validation error를 발생시킵니다.
예를 들어 위의 <tt style="color: #FF0000">`Place`</tt> 클래스를 다시 사용하여 [<tt style="color: #FF0000">`ManyToManyField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField)를 사용하여 다른 하위 클래스를 만듭니다.
~~~~
class Supplier(Place):
    customers = models.ManyToManyField(Place)
~~~~
This results in the error:
~~~~
Reverse query name for 'Supplier.customers' clashes with reverse query
name for 'Supplier.place_ptr'.

HINT: Add or change a related_name argument to the definition for
'Supplier.customers' or 'Supplier.place_ptr'.
~~~~
<tt style="color: #FF0000">`customers_name`</tt> 필드에 <tt style="color: #FF0000">`related_name`</tt>을 다음과 같이 추가하면 <tt style="color: #FF0000">`models.ManyToManyField(Place, related_name = 'provider')`</tt> 오류가 해결됩니다.

#### Specifying the parent link field

앞서 언급했듯이 Django는 자동으로 [<tt style="color: #FF0000">`OneToOneField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField)를 생성하여 하위 클래스를 non-abstract 부모 모델로 다시 연결합니다. 부모에게 다시 연결되는 attribute의 이름을 제어하려면 고유한 [<tt style="color: #FF0000">`OneToOneField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField)를 만들고 [<tt style="color: #FF0000">`parent_link=True`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.OneToOneField.parent_link)로 설정하여 필드가 부모 클래스의 링크임을 나타낼 수 있습니다.


### Proxy models

[multi-table inheritance](https://docs.djangoproject.com/en/1.11/topics/db/models/#multi-table-inheritance)를 사용할 때 모델의 각 하위 클래스에 대해 새 데이터베이스 테이블이 만들어 집니다. 서브 클래스는 기본 클래스에없는 추가 데이터 필드를 저장할 장소가 필요하기 때문에 일반적으로 원하는 동작입니다. 그러나 때로는 모델의 Python 동작을 변경하기를 원할 수도 있습니다. 아마도 default manager를 변경하거나 새로운 메소드를 추가하기만 하면됩니다.
이것은 프록시 모델 상속을위한 것입니다 : 원래 모델에 대한 프록시를 만듭니다. 프록시 모델의 인스턴스를 생성, 삭제 및 업데이트 할 수 있으며 원본 (non-proxied) 모델을 사용하는 것처럼 모든 데이터가 저장됩니다. 차이점은 원본을 변경하지 않고 프록시에서 기본 모델 순서 또는 default manager와 같은 것을 변경할 수 있다는 것입니다.
프록시 모델은 일반 모델처럼 선언됩니다. Django에게 <tt style="color: #FF0000">`Meta`</tt> 클래스의 [<tt style="color: #FF0000">`proxy`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/options/#django.db.models.Options.proxy) attribute을 <tt style="color: #FF0000">`True`</tt>로 설정하여 프록시 모델임을 알린다.
예를 들어, <tt style="color: #FF0000">`Person`</tt> 모델에 메소드를 추가하려고 한다고 가정하십시오. 다음과 같이 할 수 있습니다.
~~~~
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)

class MyPerson(Person):
    class Meta:
        proxy = True

    def do_something(self):
        # ...
        pass
~~~~
<tt style="color: #FF0000">`MyPerson`</tt> 클래스는 상위 <tt style="color: #FF0000">`Person`</tt> 클래스와 동일한 데이터베이스 테이블에서 작동합니다. 특히 <tt style="color: #FF0000">`Person`</tt>의 새 인스턴스는 <tt style="color: #FF0000">`MyPerson`</tt>을 통해 액세스할 수 있으며 그 반대의 경우도 가능합니다.
~~~~
>>> p = Person.objects.create(first_name="foobar")
>>> MyPerson.objects.get(first_name="foobar")
<MyPerson: foobar>
~~~~
이제 일반 <tt style="color: #FF0000">`Person`</tt> 쿼리는 unordered하고 <tt style="color: #FF0000">`OrderedPerson`</tt> 쿼리는 <tt style="color: #FF0000">`last_name`</tt>에 의해 정렬됩니다.
프록시 모델은 [일반 모델과 동일한 방식](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-and-multi-table-inheritance)으로 Meta attributes을 상속받습니다.

#### QuerySets still return the model that was requested

Django가 Person 객체를 쿼리할 때마다 <tt style="color: #FF0000">`MyPerson`</tt> 객체를 반환할 방법이 없습니다. <tt style="color: #FF0000">`Person`</tt> 객체에 대한 queryset은 이러한 유형의 객체를 반환합니다. 프록시 객체의 요점은 원래 <tt style="color: #FF0000">`Person`</tt>을 사용하는 코드가 이를 사용하고 사용자 코드가 포함된 확장을 사용할 수 있다는 것입니다 (다른 코드는 그다지 의존하지 않음). 그것은 <tt style="color: #FF0000">`Person`</tt> (또는 다른) 모델을 어디서나 own creation로 대체하는 방법이 아닙니다.

#### Base class restrictions

프록시 모델은 정확히 하나의 non-abstract 모델 클래스를 상속해야합니다. 프록시 모델은 다른 데이터베이스 테이블의 rows 사이에 연결을 제공하지 않으므로 여러 non-abstract 모델을 상속받을 수 없습니다. 프록시 모델은 모델 필드를 정의하지 않으면 abstract 모델 클래스를 상속 할 수 있습니다. 프록시 모델은, 공통의 non-abstract 부모 클래스를 공유하는 임의의 수의 프록시 모델로부터 상속할 수 있습니다.

<b>Changed in Django 1.10:</b>
이전 버전에서는 프록시 모델이 동일한 상위 클래스를 공유하는 둘 이상의 프록시 모델을 상속하지 못했습니다.

#### Proxy model managers

프록시 모델에 모델 managers를 지정하지 않으면 모델 부모로부터 managers를 상속받습니다. 프록시 모델에서 managers를 정의하면 상위 클래스에 정의된 managers는 계속 사용할 수 있지만 default가 됩니다.
위의 예를 계속하면 <tt style="color: #FF0000">`Person`</tt> 모델을 쿼리할 때 사용되는 default managers를 다음과 같이 변경할 수 있습니다.
~~~~
from django.db import models

class NewManager(models.Manager):
    # ...
    pass

class MyPerson(Person):
    objects = NewManager()

    class Meta:
        proxy = True
~~~~
기존 default를 바꾸지 않고 프록시에 new manager를 추가하려면 [custom manager](https://docs.djangoproject.com/en/1.11/topics/db/managers/#custom-managers-and-inheritance) 설명서에 설명된 기술을 사용할 수 있습니다. new manager를 포함하는 base 클래스를 작성하고 primary base 클래스 다음에 상속합니다.
~~~~
# Create an abstract class for the new manager.
class ExtraManagers(models.Model):
    secondary = NewManager()

    class Meta:
        abstract = True

class MyPerson(Person, ExtraManagers):
    class Meta:
        proxy = True
~~~~
아마도이 작업을 자주 수행 할 필요는 없지만 하고자 할 경우 가능합니다.

#### Differences between proxy inheritance and unmanaged models

프록시 모델 상속은 모델의 <tt style="color: #FF0000">`Meta`</tt> 클래스에서 [<tt style="color: #FF0000">`managed`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/options/#django.db.models.Options.managed) attribute을 사용하여 관리되지 않는 모델을 만드는 것과 매우 비슷하게 보일 수 있습니다.
신중하게 [<tt style="color: #FF0000">`Meta.db_table`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/options/#django.db.models.Options.db_table)을 설정하면 관리되지 않는 모델을 만들어 기존 모델을 shadows 처리하고 Python 메서드를 추가 할 수 있습니다. 그러나 변경 작업을 수행 할 경우 두 복사본을 동기화된 상태로 유지해야 하므로 repetitive and fragile합니다.
반면에 프록시 모델은 프록싱 중인 모델과 정확히 동일하게 동작합니다. 부모 모델은 필드와 managers를 직접 상속하므로 항상 부모 모델과 동기화됩니다.
The general rules are:
1. 기존 모델이나 데이터베이스 테이블을 미러링하고 원래 데이터베이스 테이블 columns을 모두 원하지 않으면 <tt style="color: #FF0000">`Meta.managed=False`</tt>를 사용하십시오. 이 옵션은 일반적으로 Django가 제어하지 않는 데이터베이스 뷰와 테이블을 모델링 할 때 유용합니다.
2. 모델의 Python-only 동작을 변경하려고하지만 원본과 동일한 필드를 모두 유지하려면 <tt style="color: #FF0000">`Meta.proxy=True`</tt>를 사용하십시오. 이렇게하면 데이터를 저장할 때 프록시 모델이 원본 모델의 저장소 구조와 정확히 일치하도록 설정됩니다.

### Multiple inheritance

Python의 서브 클래싱과 마찬가지로, Django 모델은 여러 부모 모델로부터 상속받을 수 있습니다. 일반적인 Python name resolution rules이 적용된다는 것을 명심하십시오. 특정 이름 (예 : [Meta](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-options))이 나타나는 첫 번째 base 클래스가 사용됩니다. 예를 들어 여러 부모가 [Meta](https://docs.djangoproject.com/en/1.11/topics/db/models/#meta-options) 클래스를 포함하면 첫 번째 부모만 사용되며 다른 모든 부모는 무시됩니다.
일반적으로 여러 부모로부터 상속하지 않아도됩니다. 이것이 유용할 때의 주요 use-case는 "mix-in"클래스입니다 : mix-in을 상속받은 모든 클래스에 특정 추가 필드나 메서드를 추가하는 것입니다. 가능한 한 간단하고 직관적인 상속 계층 구조를 유지하여 특정 정보가 어디서 왔는지 알아내려고 노력해야 할 필요가 없습니다.
common ID primary key 필드가있는 여러 모델을 상속하면 오류가 발생합니다. 다중 상속을 올바르게 사용하려면 base 모델에서 명시적 [<tt style="color: #FF0000">`AutoField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.AutoField)를 사용할 수 있습니다.
~~~~
class Article(models.Model):
    article_id = models.AutoField(primary_key=True)
    ...

class Book(models.Model):
    book_id = models.AutoField(primary_key=True)
    ...

class BookReview(Book, Article):
    pass
~~~~
Or use a common ancestor to hold the [<tt style="color: #FF0000">`AutoField`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.AutoField):
~~~~
class Piece(models.Model):
    pass

class Article(Piece):
    ...

class Book(Piece):
    ...

class BookReview(Book, Article):
    pass
~~~~

### Field name “hiding” is not permitted

일반적인 Python 클래스 상속에서는 자식 클래스가 부모 클래스의 모든 attribute을 재정의 할 수 있습니다. Django에서는 모델 필드에 일반적으로 허용되지 않습니다. non-abstract 모델 base 클래스에 <tt style="color: #FF0000">`author`</tt>라는 필드가있는 경우 해당 기본 클래스에서 상속하는 클래스에서 다른 모델 필드를 만들거나 <tt style="color: #FF0000">`author`</tt>라는 속성을 정의 할 수 없습니다.
이 제한은 abstract 모델에서 상속된 모델 필드에는 적용되지 않습니다. 이러한 필드는 다른 필드나 값으로 무시되거나 <tt style="color: #FF0000">`field_name=None`</tt>을 설정하여 제거할 수 있습니다.

<b>Changed in Django 1.10:</b>
The ability to override abstract fields was added.

<b>Warning</b>
모델 managers는 abstract base 클래스에서 상속됩니다. 상속된 [<tt style="color: #FF0000">`Manager`</tt>](https://docs.djangoproject.com/en/1.11/topics/db/managers/#django.db.models.Manager)가 참조하는 상속된 필드를 재정의하면 미묘한 버그가 발생할 수 있습니다. [custom managers and model inheritance](https://docs.djangoproject.com/en/1.11/topics/db/managers/#custom-managers-and-inheritance)을 참조하십시오.

<b>Note</b>
일부 필드는 모델의 extra attributes을 정의합니다. [<tt style="color: #FF0000">`ForeignKey`</tt>](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey)는 필드 이름에 <tt style="color: #FF0000">`_id`</tt>가 추가된 extra attributes과 외부 모델의 <tt style="color: #FF0000">`related_name`</tt> 및 <tt style="color: #FF0000">`related_query_name`</tt>을 정의합니다.
이러한 extra attributes은 extra attributes을 더 이상 정의하지 않도록 해당 필드를 변경하거나 제거하지 않으면 무시할 수 없습니다.

상위 모델의 필드를 재정의하면 새 인스턴스를 초기화 (<tt style="color: #FF0000">`Model.__ init__`</tt>에서 초기화 할 필드 지정) 및 serialization와 같은 영역에서 어려움이 발생합니다. 이것들은 일반적인 Python 클래스 상속이 똑같은 방식으로 처리할 필요가 없는 기능이므로 Django 모델 상속과 Python 클래스 상속의 차이점은 임의적(arbitrary)이지 않습니다.
이 제한은 [Field](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field) 인스턴스인 attributes에만 적용됩니다. 원하는 경우 일반 Python attributes을 재정의할 수 있습니다. 또한 Python이 인식하는 attributes의 이름에만 적용됩니다. 데이터베이스 column 이름을 수동으로 지정하는 경우 다중 테이블 상속을 위해 자식 및 조상 모델에 같은 column 이름을 사용할 수 있습니다 (두 개의 다른 데이터베이스 테이블의 column).
Django는 조상 모델에서 모델 필드를 override하면 [FieldError](https://docs.djangoproject.com/en/1.11/ref/exceptions/#django.core.exceptions.FieldError)를 발생시킵니다.

### Organizing models in a package

[<tt style="color: #FF0000">`manage.py startapp`</tt>](https://docs.djangoproject.com/en/1.11/ref/django-admin/#django-admin-startapp) 명령은 <tt style="color: #FF0000">`models.py`</tt> 파일을 포함하는 application 구조를 만듭니다. 모델이 여러 개인 경우 별도의 파일로 구성하면 유용합니다.
그렇게 하려면 모델 패키지를 만드십시오. <tt style="color: #FF0000">`models.py`</tt>를 제거하고 <tt style="color: #FF0000">`myapp/models/`</tt> 디렉토리를 만들고 <tt style="color: #FF0000">`__init__.py`</tt> 파일과 모델을 저장할 파일을 만듭니다. <tt style="color: #FF0000">`__init__.py`</tt> 파일에서 모델을 가져와야합니다.
예를 들어, <tt style="color: #FF0000">`models`</tt> 디렉토리에 <tt style="color: #FF0000">`organic.py`</tt>와 <tt style="color: #FF0000">`synthetic.py`</tt>가 있다면 :
~~~~
myapp/models/__init__.py
from .organic import Person
from .synthetic import Robot
~~~~
<tt style="color: #FF0000">`from .models import *`</tt>에서 사용하는 대신 명시적으로 각 모델을 가져 오면 네임 스페이스가 복잡해지지 않고 코드가 읽기 쉽고 코드 분석 도구가 유용하게 유지된다는 이점이 있습니다.

<b>See also</b>
[The Models Reference](https://docs.djangoproject.com/en/1.11/ref/models/)
모델 필드, 관련 객체 및 <tt style="color: #FF0000">`QuerySet`</tt>을 포함한 모든 모델 관련 API를 포함합니다.