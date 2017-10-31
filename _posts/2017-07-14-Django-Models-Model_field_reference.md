---
layout: post
title:  "Django Models_and_databases-Making_queries"
date:   2017-07-14 10:43:59
author: Dean Kim
categories: Django
tags:	Django Models Model_field_reference
cover:  "/assets/instacode.png"
---

# Model field reference
- 원본 : [공식문서](https://docs.djangoproject.com/en/1.11/ref/models/fields/)


이 문서는 Django가 제공하는 [field options](https://docs.djangoproject.com/en/1.11/ref/models/fields/#field-options)과 [field types](https://docs.djangoproject.com/en/1.11/ref/models/fields/#field-types)을 포함하여 [Field](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field)의 모든 API 참조를 포함합니다.

<b>See also</b>
<br>내장 필드가 트릭을 하지 않는다면 [django-localflavor](https://github.com/django/django-localflavor)([문서](https://django-localflavor.readthedocs.io/en/latest/))를 시도해 볼 수 있습니다. 여기에는 특정 국가 및 문화에 유용한 다양한 코드가 들어 있습니다.

또한 [사용자 정의 모델 필드](https://docs.djangoproject.com/en/1.11/howto/custom-model-fields/)를 쉽게 작성할 수 있습니다.

<b>Note</b>
<br>기술적으로, 이들 모델은 [django.db.models.fields](https://docs.djangoproject.com/en/1.11/ref/models/fields/#module-django.db.models.fields)에 정의되어 있습니다만 편의상 [django.db.models](https://docs.djangoproject.com/en/1.11/topics/db/models/#module-django.db.models)로 가져옵니다. 표준 규칙은 <tt style="color: #FF0000">`from django.db import models`</tt>에서 사용하고 models.<Foo>Field처럼 필드를 모델로 참조하는 것입니다.

## Field option

다음 인수는 모든 필드 유형에서 사용할 수 있습니다. 모두 선택 사항입니다.

### null

<b>Field.null</b>

<tt style="color: #FF0000">`True`</tt>이면 Django는 빈 값을 <tt style="color: #FF0000">`NULL`</tt>로 데이터베이스에 저장합니다. Default는 <tt style="color: #FF0000">`False`</tt>입니다.

<tt style="color: #FF0000">`CharField`</tt> 및 <tt style="color: #FF0000">`TextField`</tt>와 같은 문자열 기반 필드에서는 <tt style="color: #FF0000">`null`</tt>을 사용하지 마십시오. 문자열 기반 필드에 <tt style="color: #FF0000">`null=True`</tt>가 있으면 "no data"에 대해 가능한 두 개의 값 (<tt style="color: #FF0000">`NULL`</tt> 및 빈 문자열)이 있음을 의미합니다. 대부분의 경우, "데이터 없음"에 대해 가능한 두 가지 값을 갖는 것은 불필요합니다. Django 규칙은 <tt style="color: #FF0000">`NULL`</tt>이 아닌 빈 문자열을 사용하는 것입니다. 한 가지 예외는 <tt style="color: #FF0000">`CharField`</tt>에 <tt style="color: #FF0000">`unique=True`</tt>와 <tt style="color: #FF0000">`blank=True`</tt>가 설정된 경우입니다. 이 경우 빈 값을 갖는 여러 오브젝트를 저장할 때 고유 제한 조건 위반을 피하려면 <tt style="color: #FF0000">`null=True`</tt>가 필요합니다.

문자열 기반 및 비 문자열 기반 필드의 경우, <tt style="color: #FF0000">`null`</tt> 매개 변수가 데이터베이스 저장소에만 영향을 주기 때문에 양식에서 빈 값을 허용하려면 <tt style="color: #FF0000">`blank=True`</tt>로 설정해야합니다. (아래의 blank를 참조하세요)

<b>Note</b>
<br>Oracle 데이터베이스 백엔드를 사용할 때 이 값과 상관없이 <tt style="color: #FF0000">`NULL`</tt> 값이 저장되어 빈 문자열을 나타냅니다.

<tt style="color: #FF0000">`BooleanField`</tt>로 <tt style="color: #FF0000">`null`</tt> 값을 허용하려면 대신 <tt style="color: #FF0000">`NullBooleanField`</tt>를 사용하십시오.

### blank

<b>Field.blank</b>

<tt style="color: #FF0000">`True`</tt>이면 필드는 비워 둘 수 있습니다. Default는 <tt style="color: #FF0000">`False`</tt>입니다.

이 값은 <tt style="color: #FF0000">`null`</tt>과 다릅니다. <tt style="color: #FF0000">`null`</tt>은 순전히 데이터베이스와 관련된 반면, <tt style="color: #FF0000">`blank`</tt>는 유효성 검사와 관련이 있습니다. 필드에 <tt style="color: #FF0000">`blank=True`</tt>가 있으면 양식 유효성 검사에서 빈 값을 허용합니다. 필드에 <tt style="color: #FF0000">`blank=False`</tt>가 있으면 필드가 필요합니다.

### choices

<b>Field.choices</b>

이 필드의 선택 항목으로 사용할 정확한 두 항목 (예 : [(A, B), (A, B) ...])의 반복 가능 항목으로 구성된 반복 가능한 항목 (예 : list 또는 튜플)입니다. 이것이 주어지면, 기본 양식 위젯은 표준 텍스트 필드 대신 이 선택 항목을 가진 선택 상자가됩니다.

각 튜플의 첫 번째 요소는 모델에 설정할 실제 값이고 두 번째 요소는 사람이 읽을 수 있는 이름입니다. 예 :
~~~~
YEAR_IN_SCHOOL_CHOICES = (
    ('FR', 'Freshman'),
    ('SO', 'Sophomore'),
    ('JR', 'Junior'),
    ('SR', 'Senior'),
)
~~~~
일반적으로 모델 클래스 내에서 선택 사항을 정의하고 각 값에 대해 적절한 이름의 상수를 정의하는 것이 가장 좋습니다.
~~~~
from django.db import models

class Student(models.Model):
    FRESHMAN = 'FR'
    SOPHOMORE = 'SO'
    JUNIOR = 'JR'
    SENIOR = 'SR'
    YEAR_IN_SCHOOL_CHOICES = (
        (FRESHMAN, 'Freshman'),
        (SOPHOMORE, 'Sophomore'),
        (JUNIOR, 'Junior'),
        (SENIOR, 'Senior'),
    )
    year_in_school = models.CharField(
        max_length=2,
        choices=YEAR_IN_SCHOOL_CHOICES,
        default=FRESHMAN,
    )

    def is_upperclass(self):
        return self.year_in_school in (self.JUNIOR, self.SENIOR)
~~~~
모델 클래스 외부에서 선택 목록을 정의한 다음 참조 할 수는 있지만 모델 클래스 내의 각 선택 항목에 대한 선택 사항과 이름을 정의하면 해당 정보를 사용하는 클래스와 모든 정보가 유지되고 선택 사항을 쉽게 참조할 수 있습니다 (예 : <tt style="color: #FF0000">`Student.SOPHOMORE`</tt>는 <tt style="color: #FF0000">`Student`</tt> 모델을 가져온 곳이면 어디에서나 사용할 수 있습니다.)

조직의 목적으로 사용할 수 있는 명명된 그룹으로 사용 가능한 선택 사항을 수집할 수도 있습니다.
~~~~
MEDIA_CHOICES = (
    ('Audio', (
            ('vinyl', 'Vinyl'),
            ('cd', 'CD'),
        )
    ),
    ('Video', (
            ('vhs', 'VHS Tape'),
            ('dvd', 'DVD'),
        )
    ),
    ('unknown', 'Unknown'),
)
~~~~
각 튜플의 첫 번째 요소는 그룹에 적용할 이름입니다. 두 번째 요소는 반복 가능한 2-튜플이며 각 2-튜플에는 옵션에 대한 값과 사람이 읽을 수 있는 이름이 들어 있습니다. 그룹화된 옵션은 단일 목록 내의 그룹화되지 않은 옵션과 결합될 수 있습니다 (예 : 이 예시에서는 알 수 없는 옵션).

<tt style="color: #FF0000">`choices`</tt> 항목이 있는 각 모델 필드에 대해 Django는 필드의 현재 값에 대한 사람이 읽을 수 있는 이름을 검색하는 메소드를 추가합니다. 데이터베이스 API 문서에서 [get_FOO_display()](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model.get_FOO_display)를 참조하십시오.

선택 사항은 모든 반복 가능한 객체일 수 있으며 반드시 list이나 튜플일 필요는 없습니다. 이를 통해 선택 사항을 동적으로 구성할 수 있습니다. 그러나 자신이 역동적인 <tt style="color: #FF0000">`choices`</tt>을 해킹하는 경우 <tt style="color: #FF0000">`ForeignKey`</tt>가 있는 적절한 데이터베이스 테이블을 사용하는 것이 좋습니다. <tt style="color: #FF0000">`choices`</tt> 사항은 많은 경우 변경되지 않는 정적 데이터를 의미합니다.

<tt style="color: #FF0000">`blank=False`</tt>가 <tt style="color: #FF0000">`default`</tt>와 함께 필드에 설정되어 있지 않으면 <tt style="color: #FF0000">`"---------"`</tt>가 포함된 레이블이 선택 상자와 함께 렌더링됩니다. 이 동작을 무시하려면 <tt style="color: #FF0000">`None`</tt>을 포함하는 <tt style="color: #FF0000">`choices`</tt> 항목에 튜플을 추가합니다. 예 : <tt style="color: #FF0000">`(None, 'Your String for Display')`</tt>. 또는 <tt style="color: #FF0000">`CharField`</tt>에서와 같이 <tt style="color: #FF0000">`None`</tt> 대신 빈 문자열을 사용할 수 있습니다.

### db_column

<b>Field.bd_column</b>

이 필드에 사용할 데이터베이스 열의 이름. 이것이 주어지지 않으면 장고는 필드의 이름을 사용합니다.

데이터베이스 열 이름이 SQL 예약어이거나 Python 변수 이름에서 허용되지 않는 문자 (특히 하이픈)를 포함해도 OK입니다. Django는 배후에서 열과 테이블 이름을 인용합니다.

### db_index

<b>Field.db_index</b>

<tt style="color: #FF0000">`True`</tt>이면 이 필드에 데이터베이스 index가 작성됩니다.

### db_tablespace¶

<b>Field.db_tablespace</b>

이 필드의 index가 작성된 경우, 이 필드의 index에 사용할 [database tablespace](https://docs.djangoproject.com/en/1.11/topics/db/tablespaces/)의 이름. default는 프로젝트의 <tt style="color: #FF0000">`DEFAULT_INDEX_TABLESPACE`</tt> 설정 (설정된 경우) 또는 모델의 <tt style="color: #FF0000">`db_tablespace`</tt> (있는 경우)입니다. 백엔드가 인덱스의 테이블 공간을 지원하지 않으면 이 옵션은 무시됩니다.

### default

<b>Field.default</b>

필드의 기본값입니다. 값 또는 호출할 수 있는 객체가 될 수 있습니다. 호출 가능하면 새로운 객체가 생성될 때마다 호출됩니다.

default는 변경 가능한 오브젝트 (모델 인스턴스, <tt style="color: #FF0000">`list`</tt>, <tt style="color: #FF0000">`set`</tt> 등)일 수 없습니다. 이는 해당 오브젝트의 동일한 인스턴스에 대한 참조가 모든 새 모델 인스턴스의 default로 사용되기 때문입니다. 대신 원하는 default를 호출 가능 코드로 래핑하십시오. 예를 들어, <tt style="color: #FF0000">`JSONField`</tt>의 기본 <tt style="color: #FF0000">`dict`</tt>를 지정하려면 다음 함수를 사용하십시오.
~~~~
def contact_default():
    return {"email": "to1@example.com"}

contact_info = JSONField("ContactInfo", default=contact_default)
~~~~
<tt style="color: #FF0000">`lambda`</tt>는 [serialized by migrations](https://docs.djangoproject.com/en/1.11/topics/migrations/#migration-serializing) 할 수 없으므로 <tt style="color: #FF0000">`default`</tt>와 같은 필드 옵션에는 사용할 수 없습니다. 다른 주의 사항에 대해서는 해당 설명서를 참조하십시오.

모델 인스턴스에 매핑되는 <tt style="color: #FF0000">`ForeignKey`</tt>와 같은 필드의 경우 defaults는 모델 인스턴스 대신 참조하는 필드 값 (<tt style="color: #FF0000">`to_field`</tt>가 설정되지 않은 경우 <tt style="color: #FF0000">`pk`</tt>)이어야 합니다.

defaults는 새 모델 인스턴스가 만들어지고 값이 필드에 제공되지 않을 때 사용됩니다. 필드가 primary key인 경우 필드가 <tt style="color: #FF0000">`None`</tt>으로 설정된 경우 default가 사용됩니다.

### editable

<b>Field.editable</b>

<tt style="color: #FF0000">`False`</tt>인 경우 필드는 admin 또는 다른 <tt style="color: #FF0000">`ModelForm`</tt>에 표시되지 않습니다. 또한 [model validation](https://docs.djangoproject.com/en/1.11/ref/models/instances/#validating-objects) 도중 건너 뜁니다. Defaults는 <tt style="color: #FF0000">`True`</tt>입니다.

### error_messages

<b>Field.error_messages</b>

<tt style="color: #FF0000">`error_messages`</tt> 인수를 사용하면 필드에서 발생시키는 default 메시지를 무시할 수 있습니다. 무시하려는 오류 메시지와 일치하는 키가 있는 dictionary를 전달하십시오.

오류 메시지 키에는 <tt style="color: #FF0000">`null, blank, invalid, invalid_choice, unique`</tt> 및 <tt style="color: #FF0000">`unique_for_date`</tt>가 포함됩니다. 추가 오류 메시지 키는 아래 [Field types](https://docs.djangoproject.com/en/1.11/ref/models/fields/#field-types) 섹션의 각 필드에 지정됩니다.

이러한 오류 메시지는 양식에 전파되지 않는 경우가 많습니다. [Considerations regarding model’s error_messages](https://docs.djangoproject.com/en/1.11/topics/forms/modelforms/#considerations-regarding-model-errormessages)을 참조하십시오.

### help_text

<b>Field.help_text</b>

form widget과 함께 표시되는 추가 "help"텍스트. 필드가 form에서 사용되지 않아도 문서화에 유용합니다.

이 값은 automatically-generated forms에서 HTML-escaped 처리되지 않습니다. 원하는 경우 <tt style="color: #FF0000">`help_text`</tt>에 HTML을 포함시킬 수 있습니다. 예 :
~~~~
help_text="Please use the following format: <em>YYYY-MM-DD</em>."
~~~~
또는 일반 텍스트와 <tt style="color: #FF0000">`django.utils.html.escape()`</tt>를 사용하여 HTML 특수 문자를 이스케이프 처리할 수 ​​있습니다. cross-site 스크립팅 공격을 피하기 위해 신뢰할 수 없는 사용자가 올 수 있는 도움말 텍스트를 이스케이프 처리해야 합니다.

### primary_key

<b>Field.primary_key</b>

<tt style="color: #FF0000">`True`</tt>이면 이 필드는 모델의 primary key입니다.

모델의 모든 필드에 대해 <tt style="color: #FF0000">`primary_key=True`</tt>를 지정하지 않으면 Django는 primary key를 보유할 <tt style="color: #FF0000">`AutoField`</tt>를 자동으로 추가하므로 해당 필드를 재정의하지 않는 한 모든 필드에서 <tt style="color: #FF0000">`primary_key=True`</tt>를 설정할 필요가 없습니다. default primary key 동작. 자세한 내용은 [Automatic primary key fields](https://docs.djangoproject.com/en/1.11/topics/db/models/#automatic-primary-key-fields)를 참조하십시오.

<tt style="color: #FF0000">`primary_key=True`</tt>는 <tt style="color: #FF0000">`null=False`</tt> 및 <tt style="color: #FF0000">`unique=True`</tt>를 의미합니다. 하나의 primary key만 객체에 허용됩니다.

primary key 필드는 읽기 전용입니다. 기존 개체의 primary key 값을 변경한 다음 저장하면 이전 개체와 함께 새 개체가 만들어집니다.

### unique

<b>Field.unique</b>

<tt style="color: #FF0000">`True`</tt>이면 이 필드는 테이블 전체에서 고유해야합니다.

이는 데이터베이스 레벨 및 모델 검증에 의해 시행됩니다. <tt style="color: #FF0000">`unique`</tt> 필드에 중복 값이 있는 모델을 저장하려고 하면 모델의 <tt style="color: #FF0000">`save()`</tt> 메소드에 의해 [django.db.IntegrityError](https://docs.djangoproject.com/en/1.11/ref/exceptions/#django.db.IntegrityError)가 발생합니다.

이 옵션은 <tt style="color: #FF0000">`ManyToManyField`</tt> 및 <tt style="color: #FF0000">`OneToOneField`</tt>를 제외한 모든 필드 유형에 유효합니다.

<tt style="color: #FF0000">`unique`</tt>이 <tt style="color: #FF0000">`True`</tt>이면 <tt style="color: #FF0000">`db_index`</tt>를 지정할 필요가 없습니다. unique은 인덱스 생성을 의미하기 때문입니다.

### unique_for_date

<b>Field.unique_for_date</b>

이것을 <tt style="color: #FF0000">`DateField`</tt> 또는 <tt style="color: #FF0000">`DateTimeField`</tt>의 이름으로 설정하여 이 필드가 날짜 필드의 값에 대해 고유해야 합니다.

예를 들어, <tt style="color: #FF0000">`unique_for_date="pub_date"`</tt>인 필드 <tt style="color: #FF0000">`title`</tt>이 있으면 Django는 같은 <tt style="color: #FF0000">`title`</tt>과 <tt style="color: #FF0000">`pub_date`</tt>를 가진 두 개의 레코드를 입력할 수 없습니다.

<tt style="color: #FF0000">`DateTimeField`</tt>를 가리키도록 설정하면 필드의 날짜 부분만 고려됩니다. 게다가 <tt style="color: #FF0000">`USE_TZ`</tt>가 <tt style="color: #FF0000">`True`</tt>일 때, 객체가 저장될 때 [current time zone](https://docs.djangoproject.com/en/1.11/topics/i18n/timezones/#default-current-time-zone)에서 검사가 수행됩니다.

모델 validation 중에는 [Model.validate_unique()](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model.validate_unique)에 의해 적용되지만 데이터베이스 수준에서는 적용되지 않습니다. [model_validate_unique()](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model.validate_unique)는 <tt style="color: #FF0000">`unique_for_date`</tt> 제약 조건이 [ModelForm](https://docs.djangoproject.com/en/1.11/topics/forms/modelforms/#django.forms.ModelForm)의 일부가 아닌 필드를 포함하면 (예 : 필드 중 하나가 <tt style="color: #FF0000">`exclude`</tt>에 나열되거나 <tt style="color: #FF0000">`editable=False`</tt>인 경우) 해당 특정 제약 조건에 대한 validation을 건너 뜁니다.

### unique_for_month

<b>Field.unique_for_month</b>

<tt style="color: #FF0000">`unique_for_date`</tt>와 같지만 월을 기준으로 필드가 고유해야합니다.

### unique_for_year

<b>Field.unique_for_year</b>

<tt style="color: #FF0000">`unique_for_date, unique_for_month`</tt>와 같습니다.

### verbose_name

<b>Filed.verbose_name</b>

사람이 읽을 수 있는 이름입니다. 자세한 이름이 주어지지 않으면 Django는 밑줄을 공백으로 변환하여 필드의 속성 이름을 사용하여 verbose name을 자동으로 만듭니다. [Verbose field names](https://docs.djangoproject.com/en/1.11/topics/db/models/#verbose-field-names)을 참조하십시오.

### validators

<b>Field.validators</b>

이 필드에 실행할 validators 목록입니다. 자세한 내용은 [validators documentation](https://docs.djangoproject.com/en/1.11/ref/validators/)를 참조하십시오.

#### Registering and fetching lookups

Field는 [lookup registration API](https://docs.djangoproject.com/en/1.11/ref/models/lookups/#lookup-registration-api)를 구현합니다. API를 사용하여 필드 클래스에 사용할 수 있는 조회와 필드에서 조회를 가져오는 방법을 customize할 수 있습니다.


## Field types

### AutoField

<tt style="color: #FF0000">`class AutoField(**options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#AutoField)

사용 가능한 ID에 따라 자동으로 증가하는 <tt style="color: #FF0000">`IntegerField`</tt>입니다. 보통 이것을 직접 사용할 필요는 없습니다. 별도로 지정하지 않으면 primary key 필드가 자동으로 모델에 추가됩니다. [Automatic primary key fields](https://docs.djangoproject.com/en/1.11/topics/db/models/#automatic-primary-key-fields)를 참조하십시오.

### BigAutoField

<tt style="color: #FF0000">`class BigAutoField(**options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#BigAutoField)
~~~~
New in Django 1.10.
~~~~
<tt style="color: #FF0000">`1`</tt>에서 <tt style="color: #FF0000">`9223372036854775807`</tt>까지의 숫자에 맞도록 보장된다는 점을 제외하고는 <tt style="color: #FF0000">`AutoField`</tt>와 매우 유사한 64비트 정수입니다.

### BigIntegerField

<tt style="color: #FF0000">`class BigIntegerField(**options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#BigIntegerField)

<tt style="color: #FF0000">`-9223372036854775808`</tt>에서 <tt style="color: #FF0000">`9223372036854775807`</tt>까지의 숫자를 맞출 수 있다는 것을 제외하고 <tt style="color: #FF0000">`IntegerField`</tt>와 매우 흡사한 64비트 정수입니다.이 필드의 기본 양식 위젯은 [TextInput](https://docs.djangoproject.com/en/1.11/ref/forms/widgets/#django.forms.TextInput)입니다.

### BinaryField

<tt style="color: #FF0000">`class BinaryField(**options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#BinaryField)

원시 이진 데이터를 저장하는 필드입니다. <tt style="color: #FF0000">`bytes`</tt> 할당만 지원합니다. 이 입력란에는 기능이 제한되어 있습니다. 예를 들어, <tt style="color: #FF0000">`BinaryField`</tt> 값에 대한 쿼리 집합을 필터링할 수 없습니다. 또한 <tt style="color: #FF0000">`BinaryField`</tt>를 [ModelForm](https://docs.djangoproject.com/en/1.11/topics/forms/modelforms/#django.forms.ModelForm)에 포함시키는 것도 불가능합니다.

<b>Abusing BinaryField
<br>데이터베이스에 파일을 저장하는 것에 대해 생각할 수도 있지만, 99%의 경우에는 잘못된 디자인이라고 생각하십시오. 이 필드는 적절한 [static files](https://docs.djangoproject.com/en/1.11/howto/static-files/) 처리를 대체하지 않습니다.

### BooleanField

<tt style="color: #FF0000">`class BooleanField(**options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#BooleanField)

true/false 필드.

이 필드의 default form widget은 [CheckboxInput](https://docs.djangoproject.com/en/1.11/ref/forms/widgets/#django.forms.CheckboxInput)입니다.

<tt style="color: #FF0000">`null`</tt> 값을 받아들일 필요가 있다면 대신 <tt style="color: #FF0000">`NullBooleanField`</tt>를 사용하십시오.

<tt style="color: #FF0000">`Field.default`</tt>가 정의되어 있지않으면 <tt style="color: #FF0000">`BooleanField`</tt>의 defaults는 <tt style="color: #FF0000">`None`</tt>입니다.

### CharField

<tt style="color: #FF0000">`class CharField(max_length=None, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#CharField)

작은 문자열(대/소문자 구분)을 위한 문자열 필드입니다.

많은 양의 텍스트의 경우 <tt style="color: #FF0000">`TextField`</tt>를 사용하십시오.

이 필드의 기본 양식 위젯은 <tt style="color: #FF0000">`TextInput`입니다.

<tt style="color: #FF0000">`CharField`</tt>에는 필수 인수가 하나 더 있습니다.

<b>CharField.max_length</b>
<br>필드의 최대 길이 (문자 수)입니다. max_length는 데이터베이스 레벨과 Django의 validation에서 적용됩니다.

<b>Note</b>
<br>여러 데이터베이스 백엔드로 이식할 수 있어야 하는 응용 프로그램을 작성하는 경우 일부 백엔드의 경우 <tt style="color: #FF0000">`max_length`</tt>에 대한 제한 사항이 있음을 알아야합니다. 자세한 내용은 [database backend notes](https://docs.djangoproject.com/en/1.11/ref/databases/)를 참조하십시오.

### CommaSeparatedIntegerField

<tt style="color: #FF0000">`class CommaSeparatedIntegerField(max_length=None, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#CommaSeparatedIntegerField)
~~~~
버전 1.9부터는 사용되지 않음 :
이 필드는 validator=[validate_comma_separated_integer_list]가 있는 CharField를 사용하여 더 이상 사용되지 않습니다.
~~~~
정수 필드는 쉼표로 구분됩니다. <tt style="color: #FF0000">`CharField`</tt>와 마찬가지로 <tt style="color: #FF0000">`max_length`</tt> 인수가 필요하며 여기에 언급된 데이터베이스 이식성에 대한 참고 사항을 주의해야 합니다.

### DateField

<tt style="color: #FF0000">`class DateField(auto_now=False, auto_now_add=False, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#DateField)

Python에서 <tt style="color: #FF0000">`datetime.date`</tt> 인스턴스로 표현되는 날짜입니다. 추가로 몇 가지 선택적 인수가 있습니다.

#### DateField.auto_now

개체가 저장될 때마다 지금 필드를 자동으로 설정합니다. "마지막으로 수정된"타임 스탬프에 유용합니다. 현재 날짜는 항상 사용됩니다. 재정의할 수 있는 default가 아닙니다.

이 필드는 <tt style="color: #FF0000">`Model.save()`</tt>를 호출할 때 자동으로 업데이트됩니다. 이 필드는 <tt style="color: #FF0000">`QuerySet.update()`</tt>와 같은 다른 방법으로 다른 필드를 업데이트할 때 업데이트되지 않지만 이와 같은 업데이트에서 필드의 custom 값을 지정할 수는 있습니다.

#### DateField.auto_now_add

객체가 처음 생성될 때 필드를 자동으로 설정합니다. 타임 스탬프 생성에 유용합니다. 현재 날짜는 항상 사용됩니다. 재정의할 수 있는 default가 아닙니다. 따라서 객체를 만들 때 이 필드의 값을 설정하더라도 무시됩니다. 이 필드를 수정하려면 <tt style="color: #FF0000">`auto_now_add=True`</tt> 대신 다음을 설정하십시오.

* <tt style="color: #FF0000">`DateField`</tt>의 경우 : <tt style="color: #FF0000">`default=date.today`</tt> - from <tt style="color: #FF0000">`datetime.date.today()`</tt>
* <tt style="color: #FF0000">`DateTimeField`</tt>의 경우 : <tt style="color: #FF0000">`default=timezone.now`</tt> - <tt style="color: #FF0000">`django.utils.timezone.now()`</tt>의 경우

이 필드의 default form widget은 <tt style="color: #FF0000">`TextInput`</tt>입니다. admin은 JavaScript 캘린더와 "오늘"에 대한 단축키를 추가합니다. <tt style="color: #FF0000">`invalid_date`</tt> 오류 메시지 키가 추가로 포함됩니다.

<tt style="color: #FF0000">`auto_now_add`</tt>, <tt style="color: #FF0000">`auto_now`</tt> 및 <tt style="color: #FF0000">`default`</tt> 옵션은 상호 배타적입니다. 이러한 옵션을 조합하면 오류가 발생합니다.

<b>Note</b>
<br>현재 구현된 것처럼 <tt style="color: #FF0000">`auto_now`</tt> 또는 <tt style="color: #FF0000">`auto_now_add`</tt>를 <tt style="color: #FF0000">`True`</tt>로 설정하면 해당 필드는 <tt style="color: #FF0000">`editable=False`</tt> 및 <tt style="color: #FF0000">`blank=True`</tt>로 설정됩니다.

<b>Note</b>
<br><tt style="color: #FF0000">`auto_now`</tt> 및 <tt style="color: #FF0000">`auto_now_add`</tt> 옵션은 생성 또는 업데이트할 때 항상 [default timezone](https://docs.djangoproject.com/en/1.11/topics/i18n/timezones/#default-current-time-zone)의 날짜를 사용합니다. 다른 것이 필요한 경우 <tt style="color: #FF0000">`auto_now`</tt> 또는 <tt style="color: #FF0000">`auto_now_add`</tt>를 사용하는 대신 자신의 호출 가능 default를 사용하거나 <tt style="color: #FF0000">`save()`</tt>를 무시할 수 있습니다. 또는 <tt style="color: #FF0000">`DateField`</tt> 대신 <tt style="color: #FF0000">`DateTimeField`</tt>를 사용하고 표시 시간에 datetime에서 date로의 변환을 처리하는 방법을 결정하십시오.

### DateTimeField

<tt style="color: #FF0000">`class DateTimeField(auto_now=False, auto_now_add=False, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#DateTimeField)

Python에서 <tt style="color: #FF0000">`datetime.datetime`</tt> 인스턴스로 표현되는 날짜와 시간. <tt style="color: #FF0000">`DateField`</tt>와 동일한 추가 인수를 사용합니다.

이 필드의 기본 양식 위젯은 단일 <tt style="color: #FF0000">`TextInput`</tt>입니다. 관리자는 JavaScript 바로 가기가 있는 두 개의 별도 <tt style="color: #FF0000">`TextInput`</tt> 위젯을 사용합니다.

### DecimalField

<tt style="color: #FF0000">`class DecimalField(max_digits=None, decimal_places=None, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#DecimalField)

고정 소수점 이하의 십진수로 파이썬에서 <tt style="color: #FF0000">`Decimal`</tt> 인스턴스로 표현됩니다. 두 가지 <tt style="color: #FF0000">`필수`</tt> 인수가 있습니다.

#### DecimalField.max_digits

숫자에 허용되는 최대 자릿수입니다. 이 수는 <tt style="color: #FF0000">`decimal_places`</tt>보다 크거나 같아야합니다.

#### DecimalField.decimal_places

숫자와 함께 저장할 소수 자릿수입니다.

예를 들어, 소수점 이하 2자리의 resolution로 <tt style="color: #FF0000">`999`</tt>까지의 숫자를 저장하려면 다음을 사용합니다.
~~~~
models.DecimalField(..., max_digits=5, decimal_places=2)
~~~~
소수점 이하 10자리의 resolution로 약 10억 개의 숫자 저장 :
~~~~
models.DecimalField(..., max_digits=19, decimal_places=10)
~~~~
이 필드의 default form widget은 [localize](https://docs.djangoproject.com/en/1.11/ref/forms/fields/#django.forms.Field.localize)가 <tt style="color: #FF0000">`False`</tt>일 때 <tt style="color: #FF0000">`NumberInput`</tt>이고 그렇지 않으면 <tt style="color: #FF0000">`TextInput`</tt>입니다.

<b>Note</b>
<br><tt style="color: #FF0000">`FloatField`</tt> 및 <tt style="color: #FF0000">`DecimalField`</tt> 클래스 간의 차이점에 대한 자세한 내용은 [FloatField vs. DecimalField](https://docs.djangoproject.com/en/1.11/ref/models/fields/#floatfield-vs-decimalfield)를 참조하십시오.

### DurationField

<tt style="color: #FF0000">`class DurationField(**options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#DurationField)

[timedelta](https://docs.python.org/3/library/datetime.html#datetime.timedelta)가 Python으로 모델링한 시간주기 저장 필드. PostgreSQL에서 사용되는 데이터 유형은 <tt style="color: #FF0000">`interval`</tt>이며 Oracle에서는 데이터 유형이 <tt style="color: #FF0000">`INTERVAL DAY(9) TO SECOND(6)`</tt>입니다. 그렇지 않으면 microseconds의 <tt style="color: #FF0000">`bigint`</tt>가 사용됩니다.

<b>Note</b>
<br><tt style="color: #FF0000">`DurationField`</tt>를 사용한 산술은 대부분의 경우에 작동합니다. 그러나 PostgreSQL 이외의 모든 데이터베이스에서 <tt style="color: #FF0000">`DurationField`</tt>의 값을 <tt style="color: #FF0000">`DateTimeField`</tt>의 산술 인스턴스와 비교하는 것은 예상대로 작동하지 않습니다.

### EmailField

<tt style="color: #FF0000">`class EmailField(max_length=254, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#EmailField)

값이 유효한 전자 메일 주소인지 확인하는 <tt style="color: #FF0000">`CharField`</tt>입니다. <tt style="color: #FF0000">`EmailValidator`</tt>를 사용하여 입력을 validate 합니다.

### FileField

<tt style="color: #FF0000">`class FileField(upload_to=None, max_length=100, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/files/#FileField)

파일 업로그 필드입니다.

<b>Note</b>
<br><tt style="color: #FF0000">`primary_key`</tt> 인수는 지원되지 않으며 사용되는 경우 오류를 발생시킵니다.

두 개의 추가적인 인수를 갖습니다.

#### FileField.upload_to

이 속성은 업로드 디렉토리와 파일 이름을 설정하는 방법을 제공하며 두 가지 방법으로 설정할 수 있습니다. 두 경우 모두 값은 <tt style="color: #FF0000">`Storage.save()`</tt> 메서드에 전달됩니다.

문자열 값을 지정하면 <tt style="color: #FF0000">`strftime()`</tt> 형식이 포함될 수 있습니다. 이 형식은 업로드된 파일이 주어진 디렉토리를 채우지 않도록 파일 업로드 date/time으로 대체됩니다. 예 :
~~~~
class MyModel(models.Model):
    # file will be uploaded to MEDIA_ROOT/uploads
    upload = models.FileField(upload_to='uploads/')
    # or...
    # file will be saved to MEDIA_ROOT/uploads/2015/01/30
    upload = models.FileField(upload_to='uploads/%Y/%m/%d/')
~~~~
기본 [FileSystemStorage](https://docs.djangoproject.com/en/1.11/ref/files/storage/#django.core.files.storage.FileSystemStorage)를 사용하는 경우 문자열 값은 [MEDIA_ROOT](https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-MEDIA_ROOT) 경로에 추가되어 업로드된 파일이 저장될 로컬 파일 시스템의 위치를 ​​형성합니다. 다른 저장소를 사용하는 경우 해당 저장소의 설명서를 확인하여 저장소의 <tt style="color: #FF0000">`upload_to`</tt> 처리 방법을 확인하십시오.

<tt style="color: #FF0000">`upload_to`</tt>는 함수와 같은 호출 가능 함수일 수도 있습니다. 이것은 파일 이름을 포함하여 업로드 경로를 얻기 위해 호출됩니다. 이 호출 가능 객체는 두 개의 인수를 받아들여 스토리지 시스템으로 전달되는 UNIX 스타일 경로 (슬래시 포함)를 반환해야합니다. 두 가지 인수는 다음과 같습니다.
<table style="border:1px solid black">
<tr>
<th style="width:120px;border:1px solid black;">Argument</th><th style="border:1px solid black">Description</th>
</tr>
<tr>
<td style="width:120px;text-align:center;border:1px solid black">instance</td><td style="border:1px solid black">FileField가 정의된 모델의 인스턴스입니다. 보다 구체적으로, 이것은 현재 파일이 첨부되는 특정 인스턴스입니다.

대부분의 경우 이 개체는 아직 데이터베이스에 저장되지 않았으므로 default AutoField를 사용하는 경우 primary key 필드의 값이 아직 없을 수 있습니다.</td>
</tr>
<tr>
<td style="width:120px;text-align:center;border:1px solid black">filname</td><td style="border:1px solid black">원래 파일에 주어진 파일 이름. 이것은 최종 목적지 경로를 결정할 때 고려될 수도 있고 고려되지 않을 수도 있다.</td>
</tr>
</table>

예시:
~~~~
def user_directory_path(instance, filename):
    # file will be uploaded to MEDIA_ROOT/user_<id>/<filename>
    return 'user_{0}/{1}'.format(instance.user.id, filename)

class MyModel(models.Model):
    upload = models.FileField(upload_to=user_directory_path)
~~~~

#### FileField.storage

파일의 저장 및 검색을 처리하는 저장 개체입니다. 이 객체를 제공하는 방법에 대한 자세한 내용은 [Managing files](https://docs.djangoproject.com/en/1.11/topics/files/)를 참조하십시오.

이 필드의 default form widget은 [ClearableFileInput](https://docs.djangoproject.com/en/1.11/ref/forms/widgets/#django.forms.ClearableFileInput)입니다.

모델에서 <tt style="color: #FF0000">`FileField`</tt> 또는 <tt style="color: #FF0000">`ImageField`</tt> (아래 참조)를 사용하면 몇 단계가 수행됩니다.

1. 설정 파일에서 Django가 업로드된 파일을 저장할 디렉토리의 전체 경로로 <tt style="color: #FF0000">`MEDIA_ROOT`</tt>을 정의해야합니다. 성능을 위해 이러한 파일은 데이터베이스에 저장되지 않습니다. [MEDIA_URL](https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-MEDIA_URL)을 해당 디렉토리의 기본 공개 URL로 정의하십시오. 이 디렉터리에 웹 서버의 사용자 계정이 쓸 수 있는지 확인하십시오.
2. 모델에 <tt style="color: #FF0000">`FileField`</tt> 또는 <tt style="color: #FF0000">`ImageField`</tt>를 추가하고 <tt style="color: #FF0000">`upload_to`</tt> 옵션을 정의하여 업로드된 파일에 사용할 <tt style="color: #FF0000">`MEDIA_ROOT`</tt>의 하위 디렉토리를 지정합니다.
3. 데이터베이스에 저장되는 것은 모두 파일에 대한 경로입니다 (<tt style="color: #FF0000">`MEDIA_ROOT`</tt>에 상대적임). Django가 제공하는 편의 [url](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.fields.files.FieldFile.url) 속성을 사용하기를 원할 것입니다. 예를 들어 <tt style="color: #FF0000">`ImageField`</tt>의 이름이 <tt style="color: #FF0000">`mug_shot`</tt>인 경우 <tt style="color: #FF0000">`{{ object.mug_shot.url }}`</tt> 템플릿을 사용하여 이미지의 절대 경로를 가져올 수 있습니다.

예를 들어 <tt style="color: #FF0000">`MEDIA_ROOT`</tt>이 <tt style="color: #FF0000">`'/home/media'`</tt>로 설정되고 <tt style="color: #FF0000">`upload_to`</tt>가 <tt style="color: #FF0000">`'photos/%Y/%m/%d'`</tt>로 설정되어 있다고 가정해 보겠습니다. <tt style="color: #FF0000">`upload_to`</tt>의 <tt style="color: #FF0000">`'%Y/%m/%d'`</tt>부분은 <tt style="color: #FF0000">`strftime()`</tt> 형식입니다. <tt style="color: #FF0000">`'%Y'`</tt>는 네 자리 숫자 연도이고 <tt style="color: #FF0000">`'%m'`</tt>은 두 자리 숫자 월이고 <tt style="color: #FF0000">`'%d'`</tt>은 두 자리 날짜입니다. 2007년 1월 15일에 파일을 업로드하면 <tt style="color: #FF0000">`/home/media/photos/2007/01/15`</tt> 디렉토리에 저장됩니다.

업로드된 파일의 디스크상의 파일 이름이나 파일의 크기를 검색하려면 [name](https://docs.djangoproject.com/en/1.11/ref/files/file/#django.core.files.File.name) 및 [size](https://docs.djangoproject.com/en/1.11/ref/files/file/#django.core.files.File.size) 속성을 각각 사용할 수 있습니다. 사용 가능한 속성 및 메서드에 대한 자세한 내용은 [File](https://docs.djangoproject.com/en/1.11/ref/files/file/#django.core.files.File) 클래스 참조 및 [Managing files](https://docs.djangoproject.com/en/1.11/topics/files/) 항목 가이드를 참조하십시오.

<b>Note</b>
<br>이 파일은 모델을 데이터베이스에 저장하는 과정의 일부로 저장되므로 디스크에 사용된 실제 파일 이름은 모델을 저장해야만 신뢰할 수 있습니다.

업로드된 파일의 상대 URL은 <tt style="color: #FF0000">`url`</tt> 속성을 사용하여 얻을 수 있습니다. 내부적으로 이것은 기본 [Storage](https://docs.djangoproject.com/en/1.11/ref/files/storage/#django.core.files.storage.Storage) 클래스의 [url()](https://docs.djangoproject.com/en/1.11/ref/files/storage/#django.core.files.storage.Storage.url) 메서드를 호출합니다.

업로드된 파일을 다룰 때마다 업로드할 파일과 파일의 유형에 세심한 주의를 기울여 보안 허점을 피하십시오. 업로드된 모든 파일을 Validate하여 파일이 사용자가 생각하는 것임을 확신합니다. 예를 들어, 누군가 맹목적으로 검증없이 파일을 웹 서버의 문서 루트에 있는 디렉토리에 업로드하게 한다면 누군가가 CGI 또는 PHP 스크립트를 업로드하고 사이트의 URL을 방문하여 해당 스크립트를 실행할 수 있습니다. 그것을 허용하지 마십시오.

또한 업로드된 HTML 파일은 브라우저가 (서버가 아닌) 실행할 수 있기 때문에 XSS 또는 CSRF 공격과 동일한 보안 위협이 될 수 있습니다.

<tt style="color: #FF0000">`FileField`</tt> 인스턴스는 기본 최대 길이가 100자인 <tt style="color: #FF0000">`varchar`</tt> 열로 데이터베이스에 만들어집니다. 다른 필드와 마찬가지로 <tt style="color: #FF0000">`max_length`</tt> 인수를 사용하여 최대 길이를 변경할 수 있습니다.

### FileField and FieldFile

<tt style="color: #FF0000">`class FieldFile`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/files/#FieldFile)

모델에서 <tt style="color: #FF0000">`FileField`</tt>에 액세스하면 <tt style="color: #FF0000">`FieldFile`</tt> 인스턴스가 기본 파일에 액세스하기 위한 프록시로 제공됩니다.

<tt style="color: #FF0000">`FieldFile`</tt>의 API는 <tt style="color: #FF0000">`File`</tt>의 API와 매우 다른 점이 하나 있습니다. 클래스에 의해 래핑된 객체는 반드시 파이썬의 내장 파일 객체에 대한 래퍼일 필요는 없습니다. 대신 [Storage.open()](https://docs.djangoproject.com/en/1.11/ref/files/storage/#django.core.files.storage.Storage.open) 메서드의 결과를 둘러싼 래퍼입니다. 이 메서드는 <tt style="color: #FF0000">`File`</tt> 객체일 수도 있고 <tt style="color: #FF0000">`File`</tt> API의 custom 저장소 구현일 수도 있습니다.

<tt style="color: #FF0000">`read()`</tt> 및 <tt style="color: #FF0000">`write()`</tt>와 같이 <tt style="color: #FF0000">`File`</tt>에서 상속된 API 외에도 <tt style="color: #FF0000">`FieldFile`</tt>에는 기본 파일과 상호 작용하는데 사용할 수 있는 몇 가지 메서드가 포함되어 있습니다.

<b>Warning</b>
<br>이 클래스의 두 가지 메소드 <tt style="color: #FF0000">`save()`</tt> 및 <tt style="color: #FF0000">`delete()`</tt>는 기본적으로 연관된 <tt style="color: #FF0000">`FieldFile`</tt>의 모델 객체를 데이터베이스에 저장합니다.

#### FieldFile.name

연결된 <tt style="color: #FF0000">`FileField`</tt>의 <tt style="color: #FF0000">`Storage`</tt> 루트에서 상대 경로를 포함한 파일의 이름입니다.

#### FieldFile.size

기본 <tt style="color: #FF0000">`Storage.size()`</tt> 메서드의 결과입니다.

#### FieldFile.url

기본 <tt style="color: #FF0000">`Storage`</tt> 클래스의 <tt style="color: #FF0000">`url()`</tt> 메서드를 호출하여 파일의 상대 URL에 액세스하는 읽기 전용 속성입니다.

#### FieldFile.open(mode='rb')[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/files/#FieldFile.open)

지정된 <tt style="color: #FF0000">`mode`</tt>에서 이 인스턴스와 관련된 파일을 열거나 다시 엽니다. 표준 파이썬 <tt style="color: #FF0000">`open()`</tt> 메소드와는 달리, 파일 디스크립터를 리턴하지 않는다.

기본 파일은 액세스할 때 암시적으로 열리므로 기본 파일에 포인터를 재설정하거나 <tt style="color: #FF0000">`mode`</tt>를 변경하는 경우를 제외하고는 이 메서드를 호출할 필요가 없습니다.

#### FieldFile.close()[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/files/#FieldFile.close)

표준 파이썬 <tt style="color: #FF0000">`file.close()`</tt> 메소드와 유사하게 동작하고 이 인스턴스와 관련된 파일을 닫습니다.

#### FieldFile.save(name, content, save=True)[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/files/#FieldFile.save)

이 메서드는 파일 이름과 파일 내용을 가져와서 필드의 저장소 클래스에 전달한 다음 저장된 파일을 모델 필드와 연결합니다. 모델의 <tt style="color: #FF0000">`FileField`</tt> 인스턴스에 파일 데이터를 수동으로 연결하려면 <tt style="color: #FF0000">`save()`</tt> 메서드를 사용하여 해당 파일 데이터를 유지합니다.

두 개의 필수 인수를 취합니다 : <tt style="color: #FF0000">`name`</tt>은 파일의 이름이고 <tt style="color: #FF0000">`content`</tt>는 파일 내용을 포함하는 객체입니다. 선택적 <tt style="color: #FF0000">`save`</tt> 인수는 이 필드와 연관된 파일이 변경된 후에 모델 인스턴스가 저장되는지 여부를 제어합니다. <tt style="color: #FF0000">`True`</tt>가 default.

<tt style="color: #FF0000">`content`</tt> 인수는 Python의 내장 파일 객체가 아닌 [django.core.files.File](https://docs.djangoproject.com/en/1.11/ref/files/file/#django.core.files.File)의 인스턴스 여야합니다. 다음과 같이 기존의 Python 파일 객체에서 <tt style="color: #FF0000">`File`</tt>을 생성할 수 있습니다 :
~~~~
from django.core.files import File
# Open an existing file using Python's built-in open()
f = open('/path/to/hello.world')
myfile = File(f)
~~~~
또는 다음과 같이 파이썬 문자열로 만들 수 있습니다 :
~~~~
from django.core.files.base import ContentFile
myfile = ContentFile("hello world")
~~~~
더 많은 내용은 [Managing files](https://docs.djangoproject.com/en/1.11/topics/files/)를 참조하세요.

#### FieldFile.delete(save=True)[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/files/#FieldFile.delete)

이 인스턴스와 관련된 파일을 삭제하고 필드의 모든 특성을 지웁니다. note: 이 메소드는 <tt style="color: #FF0000">`delete()`</tt>가 호출될 때 열려있을 경우 파일을 닫습니다.

선택적 <tt style="color: #FF0000">`save`</tt> 인수는 이 필드와 연관된 파일이 삭제된 후에 모델 인스턴스가 저장되는지 여부를 제어합니다. <tt style="color: #FF0000">`True`</tt>가 default.

모델을 삭제하면 관련 파일이 삭제되지 않습니다. 분리된 파일을 정리해야 하는 경우 직접 처리해야합니다 (예 : 수동으로 실행하거나 cron 같은 것을 통해 주기적으로 실행하도록 예약된 맞춤 관리 명령 사용).

### FilePathField¶

<tt style="color: #FF0000">`class FilePathField(path=None, match=None, recursive=False, max_length=100, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#FilePathField)

파일 시스템의 특정 디렉토리에 있는 파일 이름으로 제한된 <tt style="color: #FF0000">`CharField`</tt>. 세 가지 특별한 인수들이 있으며 그 중 첫 번째 인수가 <tt style="color: #FF0000">`필수`</tt>입니다.

#### FilePathField.path

필수 사항. 이 <tt style="color: #FF0000">`FilePathField`</tt>가 선택해야하는 디렉토리에 대한 절대 파일 시스템 경로. 예 : <tt style="color: #FF0000">`"/home/images"`</tt>.

#### FilePathField.match

선택 사항. <tt style="color: #FF0000">`FilePathField`</tt>가 파일 이름을 필터링하는데 사용할 정규 표현식입니다 (문자열). 정규 표현식은 전체 경로가 아닌 기본 파일 이름에 적용됩니다. 예 : <tt style="color: #FF0000">`"foo.*\.txt$"`</tt>. <tt style="color: #FF0000">`foo23.txt`</tt>이지만 <tt style="color: #FF0000">`bar.txt`</tt> 또는 <tt style="color: #FF0000">`foo23.png`</tt>가 아닌 파일과 일치합니다.

#### FilePathField.recursive

선택 사항. <tt style="color: #FF0000">`True`</tt> or <tt style="color: #FF0000">`False`</tt>. default는 <tt style="color: #FF0000">`False`</tt>입니다. <tt style="color: #FF0000">`path`</tt>의 모든 하위 디렉토리가 포함되어야 하는지 여부를 지정합니다.

#### FilePathField.allow_files

선택 사항. <tt style="color: #FF0000">`True`</tt> or <tt style="color: #FF0000">`False`</tt>. default는 <tt style="color: #FF0000">`True`</tt>입니다. 지정된 위치의 파일을 포함할지 여부를 지정합니다. 이 중 하나 또는 둘 모두의 <tt style="color: #FF0000">`allow_folders`</tt>가 <tt style="color: #FF0000">`True`</tt> 여야합니다.

#### FilePathField.allow_folders

선택 사항. <tt style="color: #FF0000">`True`</tt> or <tt style="color: #FF0000">`False`</tt>. default는 <tt style="color: #FF0000">`False`</tt>입니다. 지정된 위치의 폴더를 포함할지 여부를 지정합니다. 이 중 하나 또는 둘 모두의 <tt style="color: #FF0000">`allow_files`</tt>는 <tt style="color: #FF0000">`True`</tt> 여야합니다.


물론 이러한 인수를 함께 사용할 수 있습니다.

하나의 잠재적인 하위 경로는 전체 경로가 아닌 기본 파일 이름에 적용됩니다. 이 예시는 다음과 같습니다. :
~~~~
FilePathField(path="/home/images", match="foo.*", recursive=True)
~~~~
...는 기본 파일 이름(foo.png 및 bar.png)에 적용되므로 /home/images/foo.png와 일치하지만 /home/images/foo/bar.png는 일치하지 않습니다.

<tt style="color: #FF0000">`FilePathField`</tt> 인스턴스는 기본 최대 길이가 100자인 <tt style="color: #FF0000">`varchar`</tt> 열로 데이터베이스에 만들어집니다. 다른 필드와 마찬가지로 <tt style="color: #FF0000">`max_length`</tt> 인수를 사용하여 최대 길이를 변경할 수 있습니다.

### FloatField

<tt style="color: #FF0000">`class FloatField(**options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#FloatField)

<tt style="color: #FF0000">`float`</tt> 인스턴스로 파이썬에서 표현된 부동 소수점 숫자.

이 필드의 default form widget은 [localize](https://docs.djangoproject.com/en/1.11/ref/forms/fields/#django.forms.Field.localize)가 <tt style="color: #FF0000">`False`</tt> 일 때 [NumberInput](https://docs.djangoproject.com/en/1.11/ref/forms/widgets/#django.forms.NumberInput)이고 그렇지 않으면 [TextInput](https://docs.djangoproject.com/en/1.11/ref/forms/widgets/#django.forms.TextInput)입니다.

<b>FloatField vs. DecimalField</b>
<br><tt style="color: #FF0000">`FloatField`</tt> 클래스는 때때로 <tt style="color: #FF0000">`DecimalField`</tt> 클래스와 혼합됩니다. 둘 다 실수를 나타내지만 그 수를 다르게 나타냅니다. <tt style="color: #FF0000">`FloatField`</tt>는 내부적으로 Python의 float 유형을 사용하고 <tt style="color: #FF0000">`DecimalField`</tt>는 Python의 <tt style="color: #FF0000">`Decimal`</tt> 유형을 사용합니다. 이 둘의 차이점에 대한 정보는 [decimal](https://docs.python.org/3/library/decimal.html#module-decimal) 모듈에 대한 Python 문서를 참조하십시오.

### ImageField

<tt style="color: #FF0000">`class ImageField(upload_to=None, height_field=None, width_field=None, max_length=100, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/files/#ImageField)

<tt style="color: #FF0000">`FileField`</tt>의 모든 특성과 메서드를 상속하지만 업로드된 개체가 유효한 이미지인지 확인합니다.

<tt style="color: #FF0000">`FileField`</tt>에 사용할 수 있는 특수 특성 외에도 <tt style="color: #FF0000">`ImageField`</tt>에는 <tt style="color: #FF0000">`height`</tt> 및 <tt style="color: #FF0000">`width`</tt> 특성이 있습니다.

이러한 속성에 대한 쿼리를 용이하게하기 위해 <tt style="color: #FF0000">`ImageField`</tt>에는 두 개의 추가 선택적 인수가 있습니다.

#### ImageField.height_field

모델 인스턴스가 저장될 때마다 이미지 높이로 자동 채워지는 모델 필드의 이름입니다.

#### ImageField.width_field

모델 인스턴스가 저장될 때마다 이미지 너비가 자동으로 채워지는 모델 필드의 이름입니다.

[Pillow](https://pillow.readthedocs.io/en/latest/) 라이브러리가 필요합니다.

<tt style="color: #FF0000">`ImageField`</tt> 인스턴스는 기본 최대 길이가 100자인 <tt style="color: #FF0000">`varchar`</tt> 열로 데이터베이스에 만들어집니다. 다른 필드와 마찬가지로 <tt style="color: #FF0000">`max_length`</tt> 인수를 사용하여 최대 길이를 변경할 수 있습니다.

이 필드의 default form widget은 <tt style="color: #FF0000">`ClearableFileInput`</tt>입니다.

### IntegerField¶

<tt style="color: #FF0000">`class IntegerField(**options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#IntegerField)

정수. <tt style="color: #FF0000">`-2147483648`</tt>에서 <tt style="color: #FF0000">`2147483647`</tt>까지의 값은 장고가 지원하는 모든 데이터베이스에서 안전합니다. 이 필드의 기본 양식 위젯은 <tt style="color: #FF0000">`localize`</tt>가 <tt style="color: #FF0000">`False`</tt> 일 때 <tt style="color: #FF0000">`NumberInput`</tt>이고 그렇지 않으면 <tt style="color: #FF0000">`TextInput`</tt>입니다.

### GenericIPAddressField

<tt style="color: #FF0000">`class GenericIPAddressField(protocol=’both’, unpack_ipv4=False, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#GenericIPAddressField)

문자열 형식의 IPv4 또는 IPv6 주소 (예 : <tt style="color: #FF0000">`192.0.2.30`</tt> 또는 <tt style="color: #FF0000">`2a02:42fe::4`</tt>) 이 필드의 default form widget은 <tt style="color: #FF0000">`TextInput`</tt>입니다.

IPv6 주소 정규화는 [RFC 4291#section-2.2]() 섹션 2.2를 따르며, 그 섹션의 3 단락에서 제안된 IPv4 포맷 (예: <tt style="color: #FF0000">`::ffff:192.0.2.0`</tt>)을 사용합니다. 예를 들어 <tt style="color: #FF0000">`2001:0::0:01`</tt>은 <tt style="color: #FF0000">`2001::1`</tt>로 표준화되고 <tt style="color: #FF0000">`::ffff:0a0a:0a0a`</tt>는 <tt style="color: #FF0000">`::ffff:10.10.10.10`</tt>으로 정규화됩니다. 모든 문자는 소문자로 변환됩니다.

#### GenericIPAddressField.protocol

지정된 프로토콜에 대한 유효한 입력을 제한합니다. 허용되는 값은 <tt style="color: #FF0000">`'both'`</tt>(default), <tt style="color: #FF0000">`'IPv4'`</tt>또는 <tt style="color: #FF0000">`'IPv6'`</tt>입니다. 일치는 대소 문자를 구분하지 않습니다.

#### GenericIPAddressField.unpack_ipv4

<tt style="color: #FF0000">`::ffff:192.0.2.1`</tt>과 같은 IPv4 매핑된 주소의 압축을 풉니다. 이 옵션을 사용하면 주소가 <tt style="color: #FF0000">`192.0.2.1`</tt>로 압축 해제됩니다. Default은 사용하지 않습니다. <tt style="color: #FF0000">`protocol`</tt>이 <tt style="color: #FF0000">`'both'`</tt>로 설정된 경우에만 사용할 수 있습니다.

공백값을 허용하는 경우 공백 값은 null로 저장되므로 null 값을 허용해야합니다.

### NullBooleanField

<tt style="color: #FF0000">`class NullBooleanField(**options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#NullBooleanField)

<tt style="color: #FF0000">`BooleanField`</tt>와 같으나 옵션 중 하나로 <tt style="color: #FF0000">`NULL`</tt>을 허용합니다. <tt style="color: #FF0000">`BooleanField`</tt> 대신 <tt style="color: #FF0000">`null=True`</tt>를 사용하십시오. 이 필드의 default form widget은 <tt style="color: #FF0000">`NullBooleanSelect`</tt>입니다.

### PositiveIntegerField

<tt style="color: #FF0000">`class PositiveIntegerField(**options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#PositiveIntegerField)

<tt style="color: #FF0000">`IntegerField`</tt>와 같지만 양수 또는 0이어야 합니다. <tt style="color: #FF0000">`0`</tt>에서 <tt style="color: #FF0000">`2147483647`</tt> 사이의 값은 장고가 지원하는 모든 데이터베이스에서 안전합니다. 이전 버전과의 호환성을 위해 <tt style="color: #FF0000">`0`</tt> 값이 허용됩니다.

### PositiveSmallIntegerField

<tt style="color: #FF0000">`class PositiveSmallIntegerField(**options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#PositiveSmallIntegerField)

<tt style="color: #FF0000">`PositiveIntegerField`</tt>와 같지만 특정 (database-dependent) 지점에서만 값을 허용합니다. <tt style="color: #FF0000">`0`</tt>에서 <tt style="color: #FF0000">`32767`</tt> 사이의 값은 장고가 지원하는 모든 데이터베이스에서 안전합니다.

### SlugField

<tt style="color: #FF0000">`class SlugField(max_length=50, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#SlugField)

[Slug](https://docs.djangoproject.com/en/1.11/glossary/#term-slug)는 신문 용어입니다. 슬러그는 글자, 숫자, 밑줄 또는 하이픈만 포함하는 짧은 레이블입니다. 일반적으로 URL에 사용됩니다.

CharField와 마찬가지로 <tt style="color: #FF0000">`max_length`</tt>를 지정할 수 있습니다 (이 절에서 데이터베이스 이식성 및 <tt style="color: #FF0000">`max_length`</tt>에 대한 참고 사항도 참조하십시오). <tt style="color: #FF0000">`max_length`</tt>를 지정하지 않으면 Django는 default 길이인 50을 사용합니다.

<tt style="color: #FF0000">`Field.db_index`</tt>를 <tt style="color: #FF0000">`True`</tt>로 설정합니다.

다른 값의 값을 기반으로 SlugField를 자동으로 미리 채우는 것이 유용합니다. [prepopulated_fields](https://docs.djangoproject.com/en/1.11/ref/contrib/admin/#django.contrib.admin.ModelAdmin.prepopulated_fields)를 사용하여 관리자가 자동으로 이 작업을 수행할 수 있습니다.

#### SlugField.allow_unicode

<tt style="color: #FF0000">`True`</tt>이면 필드에 ASCII 문자외에도 유니 코드 문자가 허용됩니다. Defaults는 <tt style="color: #FF0000">`False`</tt>입니다.

### SmallIntegerField

<tt style="color: #FF0000">`class SmallIntegerField(**options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#SmallIntegerField)

<tt style="color: #FF0000">`IntegerField`</tt>와 같지만 특정 (database-dependent) 지점에서만 값을 허용합니다. <tt style="color: #FF0000">`-32768`</tt>에서 <tt style="color: #FF0000">`32767`</tt> 사이의 값은 장고가 지원하는 모든 데이터베이스에서 안전합니다.

### TextField

<tt style="color: #FF0000">`class TextField(**options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#TextField)

큰 텍스트 필드. 이 필드의 default form widget은 <tt style="color: #FF0000">`Textarea`</tt>입니다.

<tt style="color: #FF0000">`max_length`</tt> 속성을 지정하면 auto-generated form field의 <tt style="color: #FF0000">`Textarea`</tt> 위젯에 반영됩니다. 그러나 모델이나 데이터베이스 수준에서는 적용되지 않습니다. <tt style="color: #FF0000">`CharField`</tt>를 사용하십시오.

### URLField

<tt style="color: #FF0000">`class URLField(max_length=200, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#URLField)

URL의 <tt style="color: #FF0000">`CharField`</tt>입니다.

이 필드의 기본 양식 위젯은 <tt style="color: #FF0000">`TextInput`</tt>입니다.

모든 <tt style="color: #FF0000">`CharField`</tt> 서브 클래스와 같이, <tt style="color: #FF0000">`URLField`</tt>는 선택적 <tt style="color: #FF0000">`max_length`</tt> 인수를 취합니다. <tt style="color: #FF0000">`max_length`</tt>를 지정하지 않으면 default 200이 사용됩니다.

### UUIDField

<tt style="color: #FF0000">`class UUIDField(**options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#UUIDField)

보편적으로 유일한 식별자를 저장하기 위한 필드. 파이썬의 [UUID](https://docs.python.org/3/library/uuid.html#uuid.UUID) 클래스를 사용합니다. PostgreSQL에서 사용되면 <tt style="color: #FF0000">`uuid`</tt> 데이터 유형에 저장되고, 그렇지 않으면 <tt style="color: #FF0000">`char(32)`</tt>에 저장됩니다.

범용 고유 식별자는 <tt style="color: #FF0000">`primary_key`</tt>에 대한 <tt style="color: #FF0000">`AutoField`</tt> 대신 사용할 수 있는 좋은 방법입니다. 데이터베이스가 UUID를 생성하지 않으므로 <tt style="color: #FF0000">`default`</tt>을 사용하는 것이 좋습니다.
~~~~
import uuid
from django.db import models

class MyUUIDModel(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    # other fields
~~~~
<tt style="color: #FF0000">`UUID`</tt>의 인스턴스가 아니라 호출 가능 객체 (괄호 생략)가 <tt style="color: #FF0000">`default`</tt>로 전달된다는 점에 유의하십시오.


## Relationship fields

Django는 관계를 나타내는 일련의 필드를 정의합니다.

### ForeignKey

<tt style="color: #FF0000">`class ForeignKey(othermodel, on_delete, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/related/#ForeignKey)

다 대일 관계. 위치 인수가 필요합니다. 모델이 관련된 클래스입니다.

재귀 관계 (자체와 다대일 관계가있는 객체)를 만들려면 <tt style="color: #FF0000">`models.ForeignKey('self', on_delete = models.CASCADE)`</tt>를 사용합니다.

아직 정의되지 않은 모델에 관계를 작성해야 하는 경우 모델 오브젝트 자체가 아닌 모델 이름을 사용할 수 있습니다.
~~~~
from django.db import models

class Car(models.Model):
    manufacturer = models.ForeignKey(
        'Manufacturer',
        on_delete=models.CASCADE,
    )
    # ...

class Manufacturer(models.Model):
    # ...
    pass
~~~~
이 방식으로 [abstract models](https://docs.djangoproject.com/en/1.11/topics/db/models/#abstract-base-classes)에 정의된 관계는 모델이 구체적인 모델로 서브 클래싱되고 추상 모델의 <tt style="color: #FF0000">`app_label`</tt>과 관련이 없는 경우 해결됩니다.
~~~~
products/models.py
----------
from django.db import models

class AbstractCar(models.Model):
    manufacturer = models.ForeignKey('Manufacturer', on_delete=models.CASCADE)

    class Meta:
        abstract = True
~~~~

~~~~
production/models.py
----------
from django.db import models
from products.models import AbstractCar

class Manufacturer(models.Model):
    pass

class Car(AbstractCar):
    pass

# Car.manufacturer will point to `production.Manufacturer` here.
~~~~

다른 응용 프로그램에 정의된 모델을 참조하려면 전체 응용 프로그램 레이블로 모델을 명시적으로 지정할 수 있습니다. 예를 들어 위의 <tt style="color: #FF0000">`Manufacturer`</tt> 모델이 <tt style="color: #FF0000">`production`</tt>이라는 다른 애플리케이션에 정의되어 있는 경우 다음을 사용해야합니다.
~~~~
class Car(models.Model):
    manufacturer = models.ForeignKey(
        'production.Manufacturer',
        on_delete=models.CASCADE,
    )
~~~~
이러한 종류의 참조는 두 응용 프로그램간의 resolving circular 종속성을 해결할 때 유용할 수 있습니다.

데이터베이스 인덱스는 <tt style="color: #FF0000">`ForeignKey`</tt>에 자동으로 작성됩니다. <tt style="color: #FF0000">`db_index`</tt>를 <tt style="color: #FF0000">`False`</tt>로 설정하여 이 기능을 비활성화할 수 있습니다. Join이 아닌 일관성을 위해 ForeignKey를 작성하거나 부분 또는 다중 컬럼 색인과 같은 대체 색인을 작성할 경우 색인의 오버 헤드를 피할 수 있습니다.

#### Database Representation

배후에서 Django는 <tt style="color: #FF0000">`"_id"`</tt>를 필드 이름에 추가하여 데이터베이스 column 이름을 만듭니다. 위의 예에서 <tt style="color: #FF0000">`Car`</tt> 모델의 데이터베이스 테이블에는 <tt style="color: #FF0000">`manufacturer_id`</tt> column이 있습니다. <tt style="color: #FF0000">`db_column`</tt>을 지정하여 이를 명시적으로 변경할 수 있습니다. 그러나 사용자 정의 SQL을 작성하지 않으면 코드에서 데이터베이스 column 이름을 처리할 필요가 없습니다. 모델 개체의 필드 이름은 항상 처리됩니다.

#### Arguments

<tt style="color: #FF0000">`ForeignKey`</tt>는 관계가 작동하는 방식에 대한 세부 정보를 정의하는 다른 인수를 허용합니다.

<tt style="color: #FF0000">`ForeignKey.on_delete`</tt>

<tt style="color: #FF0000">`ForeignKey`</tt>가 참조하는 객체가 삭제되면 Django는 <tt style="color: #FF0000">`on_delete`</tt> 인수에 의해 지정된 SQL 제약 조건의 동작을 에뮬레이션합니다. 예를 들어 nullable <tt style="color: #FF0000">`ForeignKey`</tt>가 있고 참조된 객체가 삭제될 때 null로 설정되기를 원할 경우 :
~~~~
user = models.ForeignKey(
    User,
    models.SET_NULL,
    blank=True,
    null=True,
)
~~~~
<tt style="color: #FF0000">`on_delete`</tt>에 가능한 값은 <tt style="color: #FF0000">`django.db.models`</tt>에 있습니다.

* <tt style="color: #FF0000">`CASCADE`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/deletion/#CASCADE)
<br>Cascade deletes. Django는 ON DELETE CASCADE SQL 제약 조건의 동작을 에뮬레이션하고 ForeignKey가 포함된 객체도 삭제합니다.

* <tt style="color: #FF0000">`PROTECT`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/deletion/#PROTECT)
<br>[django.db.IntegrityError](https://docs.djangoproject.com/en/1.11/ref/exceptions/#django.db.IntegrityError)의 하위 클래스인 [ProtectedError](https://docs.djangoproject.com/en/1.11/ref/exceptions/#django.db.models.ProtectedError)를 발생시켜 참조된 객체의 삭제를 방지합니다.

* <tt style="color: #FF0000">`SET_NULL`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/deletion/#SET_NULL)
<br><tt style="color: #FF0000">`ForeignKey null`</tt>을 설정하십시오. <tt style="color: #FF0000">`null`</tt>가 <tt style="color: #FF0000">`True`</tt>인 경우에만 가능합니다.

* <tt style="color: #FF0000">`SET_DEFAULT`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/deletion/#SET_DEFAULT)
<br><tt style="color: #FF0000">`ForeignKey`</tt>를 기본값으로 설정하십시오. <tt style="color: #FF0000">`ForeignKey`</tt>의 기본값을 설정해야합니다.

* <tt style="color: #FF0000">`SET()`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/deletion/#SET)
<br><tt style="color: #FF0000">`ForeignKey`</tt>를 [SET()](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.SET)에 전달된 값으로 설정하거나 호출 가능 객체가 전달된 경우 호출한 결과. 대부분의 경우 models.py를 가져올 때 쿼리를 실행하지 않으려면 호출 가능을 전달해야합니다.
~~~~
from django.conf import settings
from django.contrib.auth import get_user_model
from django.db import models

def get_sentinel_user():
    return get_user_model().objects.get_or_create(username='deleted')[0]

class MyModel(models.Model):
    user = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.SET(get_sentinel_user),
    )
~~~~

* <tt style="color: #FF0000">`DO_NOTHING`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/deletion/#DO_NOTHING)
<br>아무런 조치도 하지 마십시오. 데이터베이스 백엔드가 [referential integrity](https://ko.wikipedia.org/wiki/%EC%B0%B8%EC%A1%B0_%EB%AC%B4%EA%B2%B0%EC%84%B1)을 적용하면 데이터베이스 필드에 SQL <tt style="color: #FF0000">`ON DELETE`</tt> 제약 조건을 수동으로 추가하지 않으면 [IntegrityError](https://docs.djangoproject.com/en/1.11/ref/exceptions/#django.db.IntegrityError)가 발생합니다.

<tt style="color: #FF0000">`ForeignKey.limit_choices_to`</tt>

<tt style="color: #FF0000">`ModelForm`</tt> 또는 admin을 사용하여 이 필드를 렌더링할 때 이 필드에 사용할 수 있는 선택 항목에 대한 제한을 설정합니다 (기본적으로 쿼리 세트의 모든 객체를 선택할 수 있음). dictionary, [Q](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#django.db.models.Q) 오브젝트 또는 호출 가능한 리턴 dictionary 또는 Q 오브젝트가 사용될 수 있습니다.
<br>예:
~~~~
staff_member = models.ForeignKey(
    User,
    on_delete=models.CASCADE,
    limit_choices_to={'is_staff': True},
)
~~~~
<tt style="color: #FF0000">`ModelForm`</tt>의 해당 필드가 <tt style="color: #FF0000">`is_staff=True`</tt>인 <tt style="color: #FF0000">`Users`</tt>만 나열하도록 합니다. 이것은 장고 admin에게 도움이 될 수 있습니다.

예를 들어 Python <tt style="color: #FF0000">`datetime`</tt> 모듈과 함께 사용하여 날짜 범위에 따라 선택을 제한하는 경우 호출 가능 형식이 유용할 수 있습니다. 예 :
~~~~
def limit_pub_date_choices():
    return {'pub_date__lte': datetime.date.utcnow()}

limit_choices_to = limit_pub_date_choices
~~~~
<tt style="color: #FF0000">`limit_choices_to`</tt>가 [complex queries](https://docs.djangoproject.com/en/1.11/topics/db/queries/#complex-lookups-with-q)에 유용한 <tt style="color: #FF0000">`Q object`</tt>를 반환하면 모델의 <tt style="color: #FF0000">`ModelAdmin`</tt>에서 필드가 <tt style="color: #FF0000">`raw_id_fields`</tt>에 나열되어 있지 않을 때 관리자가 사용할 수 있는 선택 항목에만 영향을 미칩니다.

<b>Note</b>
<br><tt style="color: #FF0000">`limit_choices_to`</tt>에 호출된 호출이 사용되면 새 양식이 인스턴스화 될 때마다 호출됩니다. 또한 관리 명령이나 관리자와 같이 모델의 유효성을 검사할 때 호출될 수도 있습니다. 관리자는 다양한 엣지 경우에 폼 입력을 여러 번 검증하기 위해 쿼리 세트를 구성하므로 호출 가능 함수가 여러 번 호출될 가능성이 있습니다.

<tt style="color: #FF0000">`ForeignKey.related_name`</tt>

관련 객체에서 이 객체에 대한 관계에 사용할 이름입니다. 또한 <tt style="color: #FF0000">`related_query_name`</tt>(대상 모델의 역 필터 이름에 사용할 이름)의 기본값입니다. 전체 설명과 예제는 [related objects documentation](https://docs.djangoproject.com/en/1.11/topics/db/queries/#backwards-related-objects)를 참조하십시오. [abstract models](https://docs.djangoproject.com/en/1.11/topics/db/models/#abstract-base-classes)에 관계를 정의할 때 이 값을 설정해야합니다. 그렇게하면 [some special syntax](https://docs.djangoproject.com/en/1.11/topics/db/models/#abstract-related-name)을 사용할 수 있습니다.

Django가 backwards 관계를 생성하지 않기를 원한다면, <tt style="color: #FF0000">`related_name`</tt>을 '+'로 설정하거나 '+'로 끝내십시오. 예를 들어, 이렇게하면 <tt style="color: #FF0000">`User`</tt> 모델이 이 모델에 대한 backwards 관계를 갖지 않게됩니다.
~~~~
user = models.ForeignKey(
    User,
    on_delete=models.CASCADE,
    related_name='+',
)
~~~~

<tt style="color: #FF0000">`ForeignKey.related_query_name`</tt>

대상 모델에서 역방향 필터 이름에 사용할 이름입니다. 설정되어 있는 경우 <tt style="color: #FF0000">`related_name`</tt> 또는 <tt style="color: #FF0000">`default_related_name`</tt>의 값이 기본값이며, 그렇지 않은 경우 기본값은 모델의 이름입니다.
~~~~
# Declare the ForeignKey with related_query_name
class Tag(models.Model):
    article = models.ForeignKey(
        Article,
        on_delete=models.CASCADE,
        related_name="tags",
        related_query_name="tag",
    )
    name = models.CharField(max_length=255)

# That's now the name of the reverse filter
Article.objects.filter(tag__name="important")
~~~~
<tt style="color: #FF0000">`related_name`</tt>과 마찬가지로 <tt style="color: #FF0000">`related_query_name`</tt>은 [some special syntax](https://docs.djangoproject.com/en/1.11/topics/db/models/#abstract-related-name)을 통해 앱 레이블 및 클래스 보간을 지원합니다.

<tt style="color: #FF0000">`ForeignKey.to_field`</tt>

관계가 있는 관련 객체의 필드입니다. 기본적으로 Django는 관련 객체의 기본 키를 사용합니다. 다른 필드를 참조하는 경우 해당 필드는 <tt style="color: #FF0000">`unique=True`</tt>이어야합니다.

<tt style="color: #FF0000">`ForeignKey.db_constraint`</tt>

이 ForeignKey에 대해 데이터베이스에 제약 조건을 만들지 여부를 제어합니다. default는 <tt style="color: #FF0000">`True`</tt>이며, 거의 확실합니다. 이것을 <tt style="color: #FF0000">`False`</tt>로 설정하면 데이터 무결성에 매우 나쁠 수 있습니다. 즉, 다음과 같은 시나리오가 있습니다.

* 유효하지 않은 기존 데이터가 있습니다.
* 당신은 당신의 데이터베이스를 파괴하고 있습니다.

이 값을 <tt style="color: #FF0000">`False`</tt>로 설정하면 존재하지 않는 관련 객체에 액세스할 경우 해당 <tt style="color: #FF0000">`DoesNotExist`</tt> 예외가 발생합니다.

<tt style="color: #FF0000">`ForeignKey.swappable`</tt>

이 <tt style="color: #FF0000">`ForeignKey`</tt>가 스왑 가능 모델을 가리키는 경우 마이그레이션 프레임 워크의 반응을 제어합니다. 기본값이 <tt style="color: #FF0000">`True`</tt>이면 <tt style="color: #FF0000">`ForeignKey`</tt>가 <tt style="color: #FF0000">`settings.AUTH_USER_MODEL`</tt> (또는 다른 스왑 가능 모델 설정)의 현재 값과 일치하는 모델을 가리키면 모델을 직접적으로가 아닌 관계의 설정에 대한 참조를 사용하여 마이그레이션에 저장됩니다.

모델이 항상 스왑된 모델 (예 : 사용자 정의 사용자 모델 전용으로 설계된 프로파일 모델인 경우)을 가리켜야 한다고 확신하는 경우에만 <tt style="color: #FF0000">`False`</tt>로 바꾸십시오.

<tt style="color: #FF0000">`False`</tt>로 설정한다고 해서 스왑 가능 모델을 스왑 아웃해도 참조할 수 있다는 의미는 아닙니다. <tt style="color: #FF0000">`False`</tt>는 이 <tt style="color: #FF0000">`ForeignKey`</tt>로 수행된 마이그레이션이 항상 사용자가 지정한 정확한 모델을 참조한다는 것을 의미합니다. (예를 들어 사용자가 지원하지 않는 사용자 모델로 사용자가 실행하려고 하면 실패합니다.)

의심스러운 경우 기본값을 <tt style="color: #FF0000">`True`</tt>로 두십시오.

### ManyToManyField

<tt style="color: #FF0000">`class ManyToManyField(othermodel, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/related/#ManyToManyField)

many-to-many 관계. 위치 지정 인수가 필요합니다. 모형이 관련된 클래스이며, [recursive](https://docs.djangoproject.com/en/1.11/ref/models/fields/#recursive-relationships) 및 [lazy](https://docs.djangoproject.com/en/1.11/ref/models/fields/#lazy-relationships) 관계를 포함하여 <tt style="color: #FF0000">`ForeignKey`</tt>에서와 똑같이 작동합니다.

관련 객체는 필드의 [RelatedManager](https://docs.djangoproject.com/en/1.11/ref/models/relations/#django.db.models.fields.related.RelatedManager)를 사용하여 추가, 제거 또는 생성할 수 있습니다.

#### Database Representation

배후에서 Django는 many-to-many 관계를 표현하기 위해 중간 join 테이블을 생성합니다. 기본적으로 이 테이블 이름은 many-to-many 필드의 이름과 그 테이블을 포함하는 모델의 테이블 이름을 사용하여 생성됩니다. 일부 데이터베이스는 특정 길이 이상의 테이블 이름을 지원하지 않으므로 이러한 테이블 이름은 자동으로 64자로 잘리고 uniqueness hash가 사용됩니다. 즉, <tt style="color: #FF0000">`author_books_9cdf4`</tt>와 같은 테이블 이름을 볼 수 있습니다. 이것은 정상입니다. [db_table](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField.db_table) 옵션을 사용하여 join 테이블의 이름을 수동으로 제공할 수 있습니다.

#### Arguments

<tt style="color: #FF0000">`ManyToManyField`</tt>는 관계 함수가 제어되는 방식을 제어하는 ​​추가 인수 세트 (모두 선택 가능)를 허용합니다.

<tt style="color: #FF0000">`ManyToManyField.related_name`</tt>

same as [ForeignKey.related_name](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.related_name)

<tt style="color: #FF0000">`ManyToManyField.related_query_name`</tt>

same as [ForeignKey.related_query_name](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.related_query_name)

<tt style="color: #FF0000">`ManyToManyField.limit_choices_to`</tt>

same as [ForeignKey.limit_choices_to](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.limit_choices_to)

<tt style="color: #FF0000">`limit_choices_to`</tt>는 [through](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField.through) 매개 변수를 사용하여 지정된 사용자 지정 중간 테이블이 있는 <tt style="color: #FF0000">`ManyToManyField`</tt>에서 사용될 때 아무런 영향을 주지 않습니다.

<tt style="color: #FF0000">`ManyToManyField.symmetrical`</tt>

자체에 대한 ManyToManyFields의 정의에만 사용됩니다. 다음 모델을 고려하십시오.
~~~~
from django.db import models

class Person(models.Model):
    friends = models.ManyToManyField("self")
~~~~
Django는 이 모델을 처리할 때 <tt style="color: #FF0000">`ManyToManyField`</tt>가 있음을 식별하므로 <tt style="color: #FF0000">`Person`</tt> 클래스에 <tt style="color: #FF0000">`person_set`</tt> 속성을 추가하지 않습니다. 대신, <tt style="color: #FF0000">`ManyToManyField`</tt>는 대칭이라고 가정합니다. 즉, 제가 당신의 친구라면, 당신은 내 친구입니다.

<tt style="color: #FF0000">`self`</tt>와의 many-to-many 관계에서 대칭을 원하지 않으면 [symmetrical](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField.symmetrical)을 <tt style="color: #FF0000">`False`</tt>으로 설정하십시오. 이렇게하면 Django가 역방향 관계에 대한 설명자를 추가하여 <tt style="color: #FF0000">`ManyToManyField`</tt> 관계가 비대칭이되도록합니다.

<tt style="color: #FF0000">`ManyToManyField.through`</tt>

Django는 many-to-many 관계를 관리하기 위한 테이블을 자동으로 생성합니다. 그러나 중개 테이블을 수동으로 지정하려면 <tt style="color: #FF0000">`through`</tt> 옵션을 사용하여 사용할 중간 테이블을 나타내는 Django 모델을 지정할 수 있습니다.

이 옵션의 가장 일반적인 용도는 [추가 데이터를 many-to-many 관계와 연관](https://docs.djangoproject.com/en/1.11/topics/db/models/#intermediary-manytomany)시키려는 경우입니다.

명시적 <tt style="color: #FF0000">`through`</tt> 모델을 지정하지 않은 경우, 연관을 보유하기 위해 작성된 테이블에 직접 액세스하는데 사용할 수 있는 암시적 <tt style="color: #FF0000">`through`</tt> 모델 클래스가 여전히 존재합니다. 모델을 연결하는 세 개의 필드가 있습니다.

원본 및 대상 모델이 다른 경우 다음 필드가 생성됩니다.

*<tt style="color: #FF0000">`id`</tt>: the primary key of the relation.
*<tt style="color: #FF0000">`<containing_model>_id`</tt>: <tt style="color: #FF0000">`ManyToManyField`</tt>를 선언한 모델의 <tt style="color: #FF0000">`id`</tt>입니다.
*<tt style="color: #FF0000">`<other_model>_id`</tt>: <tt style="color: #FF0000">`ManyToManyField`</tt>가 가리키는 모델의 <tt style="color: #FF0000">`id`</tt>입니다.

<tt style="color: #FF0000">`ManyToManyField`</tt>가 동일한 모델에서 시작하여 동일한 모델을 가리키는 경우 다음 필드가 생성됩니다.

*<tt style="color: #FF0000">`id`</tt>: the primary key of the relation.
*<tt style="color: #FF0000">`from_<model>_id`</tt>: 모델 (즉, 소스 인스턴스)를 가리키는 인스턴스의 <tt style="color: #FF0000">`id`</tt>.
*<tt style="color: #FF0000">`to_<model>_id`</tt>: 관계가 가리키는 인스턴스의 <tt style="color: #FF0000">`id`</tt> (즉, 타겟 모델 인스턴스).

이 클래스는 일반 모델과 같이 주어진 모델 인스턴스에 대한 관련 레코드를 쿼리하는데 사용할 수 있습니다.

<tt style="color: #FF0000">`ManyToManyField.through_fields`</tt>

사용자 지정 매개 모델이 지정된 경우에만 사용됩니다. Django는 일반적으로 many-to-many 관계를 자동으로 설정하기 위해 사용할 중개 모델의 필드를 결정합니다. 그러나 다음 모델을 고려하십시오.
~~~~
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=50)

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(
        Person,
        through='Membership',
        through_fields=('group', 'person'),
    )

class Membership(models.Model):
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    inviter = models.ForeignKey(
        Person,
        on_delete=models.CASCADE,
        related_name="membership_invites",
    )
    invite_reason = models.CharField(max_length=64)
~~~~
<tt style="color: #FF0000">`Membership`</tt>에는 <tt style="color: #FF0000">`Person`</tt> (person과 inviter)에 대한 두 개의 ForeignKey가 있어 관계가 모호해지고 장고는 사용할 관계를 알 수 없습니다. 이 경우 위의 예제에서와 같이 장고가 <tt style="color: #FF0000">`through_fields`</tt>를 사용하여 사용해야하는 ForeignKey를 명시적으로 지정해야합니다.

<tt style="color: #FF0000">`through_fields`</tt>는 Two-tuple<tt style="color: #FF0000">`('field1', 'field2')`</tt>을 허용합니다. 여기서 <tt style="color: #FF0000">`field1`</tt>은 <tt style="color: #FF0000">`ManyToManyField`</tt>가 정의된 모델의 ForeignKey 이름이고 (이 경우 <tt style="color: #FF0000">`group`</tt>) <tt style="color: #FF0000">`field2`</tt>는 ForeignKey의 이름입니다. 목표 모델 (이 경우 <tt style="color: #FF0000">`person`</tt>).

many-to-many 관계에 참여하는 모델 중 하나 (또는 ​​두 모델 모두)의 중개 모델에 둘 이상의 ForeignKey가 있는 경우에는 <tt style="color: #FF0000">`through_fields`</tt>를 지정해야합니다. 이것은 중개 모델이 사용되고 모델에 ForeignKey가 두 개 이상이거나 Django가 사용할 두 개의 명시적으로 지정하려는 경우에도 [recursive relationships](https://docs.djangoproject.com/en/1.11/ref/models/fields/#recursive-relationships)에 적용됩니다.

중간 모델을 사용하는 재귀 관계는 항상 비대칭, 즉 [symmetrical=False](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ManyToManyField.symmetrical)로 정의되므로 "source"과 "target"이라는 개념이 있습니다. 이 경우 <tt style="color: #FF0000">`'field1'`</tt>은 관계의 'source'로 취급되고 'target2'는 <tt style="color: #FF0000">`'field2'`</tt>로 취급됩니다.

<tt style="color: #FF0000">`ManyToManyField.db_table`</tt>

many-to-many 데이터를 저장하기 위해 작성할 테이블의 이름. 이것이 제공되지 않는다면, Django는 관계를 정의하는 모델의 테이블과 필드 자체의 이름을 기반으로 기본 이름을 사용합니다.

<tt style="color: #FF0000">`ManyToManyField.db_constraint`</tt>

중개 테이블의 ForeignKey에 대해 데이터베이스에 제약 조건을 만들어야하는지 여부를 제어합니다. default는 <tt style="color: #FF0000">`True`</tt>이며, 거의 확실하게 원하는 것입니다. 이것을 <tt style="color: #FF0000">`False`</tt>로 설정하면 데이터 무결성이 매우 나쁠 수 있습니다. 즉, 다음과 같은 시나리오가 있습니다.

* 유효하지 않은 기존 데이터가 있습니다.
* 당신은 당신의 데이터베이스를 파괴하고 있습니다.

<tt style="color: #FF0000">`db_constraint`</tt>와 <tt style="color: #FF0000">`through`</tt>를 모두 전달하는 것은 오류입니다.

<tt style="color: #FF0000">`ManyToManyField.swappable¶`</tt>

이 <tt style="color: #FF0000">`ManyToManyField`</tt>가 스왑 가능 모델을 가리키는 경우 마이그레이션 프레임워크의 반응을 제어합니다. 이것이 <tt style="color: #FF0000">`True`</tt>이면 - default - <tt style="color: #FF0000">`ManyToManyField`</tt>가 <tt style="color: #FF0000">`settings.AUTH_USER_MODEL`</tt>(또는 다른 스왑 가능 모델 설정)의 현재 값과 일치하는 모델을 가리키는 경우 관계는 모델을 직접적으로 하지 않고 설정에 대한 참조를 사용하여 마이그레이션에 저장됩니다.

모델이 항상 스왑된 모델 (예 : 사용자 정의 사용자 모델 전용으로 설계된 프로파일 모델인 경우)을 가리켜야 한다고 확신하는 경우에만 <tt style="color: #FF0000">`False`</tt>로 바꾸십시오.

의심스러운 경우 default를 <tt style="color: #FF0000">`True`</tt>로 두십시오.

<tt style="color: #FF0000">`ManyToManyField`</tt>는 [validators](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.validators)를 지원하지 않습니다.

데이터베이스 레벨에서 관계를 요구할 방법이 없기 때문에 <tt style="color: #FF0000">`null`</tt>은 아무 효과가 없습니다.

### OneToOneField

<tt style="color: #FF0000">`class OneToOneField(othermodel, on_delete, parent_link=False, **options)`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/related/#OneToOneField)

one-to-one 관계. 개념적으로 이는 <tt style="color: #FF0000">`unique=True`</tt> 인 <tt style="color: #FF0000">`ForeignKey`</tt>와 유사하지만 관계의 "reverse"측은 하나의 객체를 직접 반환합니다.

이는 어떤 방법으로 다른 모델을 "extends"하는 모델의 primary key로 가장 유용합니다. [다중 테이블 상속](https://docs.djangoproject.com/en/1.11/topics/db/models/#multi-table-inheritance)은 하위 모델에서 상위 모델로 암시적 one-to-one 관계를 추가하여 구현됩니다.

모델이 관련될 클래스인 하나의 위치 인수가 필요합니다. 이는 [recursive](https://docs.djangoproject.com/en/1.11/ref/models/fields/#recursive-relationships) 및 [lazy](https://docs.djangoproject.com/en/1.11/ref/models/fields/#lazy-relationships) 관계와 관련된 모든 옵션을 포함하여 <tt style="color: #FF0000">`ForeignKey`</tt>에서와 똑같이 작동합니다.

<tt style="color: #FF0000">`OneToOneField`</tt>에 <tt style="color: #FF0000">`related_name`</tt> 인수를 지정하지 않으면 Django는 현재 모델의 소문자 이름을 기본값으로 사용합니다.

다음 예제를 참조하십시오.
~~~~
from django.conf import settings
from django.db import models

class MySpecialUser(models.Model):
    user = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
    )
    supervisor = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name='supervisor_of',
    )
~~~~
<tt style="color: #FF0000">`User`</tt> 모델에는 다음과 같은 속성이 있습니다.
~~~~
>>> user = User.objects.get(pk=1)
>>> hasattr(user, 'myspecialuser')
True
>>> hasattr(user, 'supervisor_of')
True
~~~~
관련 테이블의 항목이 없는 경우 역방향 관계에 액세스할 때 <tt style="color: #FF0000">`DoesNotExist`</tt> 예외가 발생합니다. 예를 들어 사용자에게 <tt style="color: #FF0000">`MySpecialUser`</tt>가 지정한 관리자가 없는 경우 :
~~~~
>>> user.supervisor_of
Traceback (most recent call last):
    ...
DoesNotExist: User matching query does not exist.
~~~~
또한 <tt style="color: #FF0000">`OneToOneField`</tt>는 <tt style="color: #FF0000">`ForeignKey`</tt>에서 허용하는 모든 추가 인수와 하나의 추가 인수를 허용합니다.

<tt style="color: #FF0000">`OneToOneField.parent_link`</tt>

<tt style="color: #FF0000">`True`</tt>이면 다른 [concrete model](https://docs.djangoproject.com/en/1.11/glossary/#term-concrete-model)을 상속한 모델에서 이 필드를 서브 클래싱하여 일반적으로 암시적으로 생성되는 여분의 <tt style="color: #FF0000">`OneToOneField`</tt>가 아니라 부모 클래스에 대한 링크로 사용해야 함을 나타냅니다.

<tt style="color: #FF0000">`OneToOneField`</tt>의 사용 예제는 [one-to-one 관계](https://docs.djangoproject.com/en/1.11/topics/db/examples/one_to_one/)를 참조하십시오.


### Field API reference

<tt style="color: #FF0000">`class Field`</tt>[source](https://docs.djangoproject.com/en/1.11/_modules/django/db/models/fields/#Field)

필드는 데이터베이스 테이블 열을 나타내는 추상 클래스입니다. Django는 데이터베이스 테이블 (db_type ())을 생성하기 위해 필드를 사용하여 Python 타입을 데이터베이스 (get_prep_value ())에 매핑하고 역순으로 (from_db_value ()) 데이터베이스에 매핑합니다.

따라서 필드는 다른 Django API, 특히 모델과 쿼리 세트에서 기본적인 부분입니다.

모델에서 필드는 클래스 속성으로 인스턴스화되고 특정 테이블 열을 나타냅니다 (모델 참조). 그것은 null과 unique와 같은 속성과 Django가 필드 값을 데이터베이스 특정 값으로 매핑하는 데 사용하는 메소드를 가지고 있습니다.

Field는 RegisterLookupMixin의 하위 클래스이므로 Transform 및 Lookup을 QuerySets에 사용할 수 있도록 등록 할 수 있습니다 (예 : field_name__exact = "foo"). 모든 기본 제공 조회는 기본적으로 등록됩니다.

CharField와 같은 Django의 모든 내장 필드는 Field의 특정 구현입니다. 사용자 정의 필드가 필요한 경우 기본 제공 필드를 서브 클래스로 만들거나 필드를 처음부터 작성할 수 있습니다. 두 경우 모두 사용자 정의 모델 필드 작성을 참조하십시오.



## Field attribute reference

모든 <tt style="color: #FF0000">`Field`</tt> 인스턴스에는 동작을 내성할 수 있는 여러 속성이 있습니다. 필드의 기능에 따라 코드를 작성해야 할 때 <tt style="color: #FF0000">`isinstance`</tt> 검사 대신 이러한 속성을 사용하십시오. 이러한 속성은 [Model._meta API](https://docs.djangoproject.com/en/1.11/ref/models/meta/#model-meta-field-api)와 함께 사용하여 특정 필드 유형에 대한 검색 범위를 좁힐 수 있습니다. 사용자 정의 모델 필드는 이러한 플래그를 구현해야합니다.

### Attributes for fields

<tt style="color: #FF0000">`Field.auto_created`</tt>

모델 상속에 사용되는 <tt style="color: #FF0000">`OneToOneField`</tt>와 같이 필드가 자동으로 만들어졌는지 여부를 나타내는 Boolean flag입니다.

<tt style="color: #FF0000">`Field.concrete`</tt>

필드에 연결된 데이터베이스 열이 있는지 여부를 나타내는 Boolean flag입니다.

<tt style="color: #FF0000">`Field.hidden`</tt>

필드가 숨겨진 필드가 아닌 다른 기능 (예 : <tt style="color: #FF0000">`GenericForeignKey`</tt>를 구성하는 <tt style="color: #FF0000">`content_type`</tt> 및 <tt style="color: #FF0000">`object_id`</tt> 필드)을 백업하는데 사용되는지 여부를 나타내는 Boolean flag입니다. <tt style="color: #FF0000">`hidden`</tt> flag는 모델의 공용 필드 서브 세트를 구성하는 항목과 모델의 모든 필드를 구별하는 데 사용됩니다.

<b>Note</b>
<br>[Options.get_fields()](https://docs.djangoproject.com/en/1.11/ref/models/meta/#django.db.models.options.Options.get_fields)는 기본적으로 숨겨진 필드를 제외합니다. 결과에서 숨겨진 필드를 반환하려면 <tt style="color: #FF0000">`include_hidden=True`</tt>를 전달하십시오.

<tt style="color: #FF0000">`Field.is_relation`</tt>

필드에 기능에 대한 하나 이상의 다른 모델 (예 : <tt style="color: #FF0000">`ForeignKey`</tt>, <tt style="color: #FF0000">`ManyToManyField`</tt>, <tt style="color: #FF0000">`OneToOneField`</tt> 등)에 대한 참조가 포함되어 있는지 나타내는 Boolean flag입니다.

<tt style="color: #FF0000">`Field.model`</tt>

필드가 정의되고 있는 모델을 반환합니다. 필드가 모델의 수퍼 클래스에 정의된 경우 <tt style="color: #FF0000">`모델`</tt>은 인스턴스의 클래스가 아닌 수퍼 클래스를 참조합니다.


### Attributes for fields with relations

이러한 속성은 카디널리티(농도, 집합의 원소 개수) 및 관계의 기타 세부 사항을 쿼리하는데 사용됩니다. 이 속성은 모든 필드에 있습니다. 그러나 필드가 관계 유형 (<tt style="color: #FF0000">`Field.is_relation=True`</tt>)인 경우 boolean values(<tt style="color: #FF0000">`None`</tt>이 아닌)만 있습니다.

<tt style="color: #FF0000">`Field.many_to_many`</tt>

필드가 many-to-many 관계를 갖는 경우 <tt style="color: #FF0000">`True`</tt>인 Boolean flag. 그렇지 않으면 <tt style="color: #FF0000">`False`</tt>. 이것이 <tt style="color: #FF0000">`True`</tt>인 Django에 포함된 유일한 필드는 <tt style="color: #FF0000">`ManyToManyField`</tt>입니다.

<tt style="color: #FF0000">`Field.many_to_one`</tt>

필드에 many-to-one 관계가 있는 경우 Boolean flag가 <tt style="color: #FF0000">`True`</tt> (예 : <tt style="color: #FF0000">`ForeignKey`</tt>). 그렇지 않으면 <tt style="color: #FF0000">`False`</tt>.

<tt style="color: #FF0000">`Field.one_to_many`</tt>

필드에 one-to-many 관계가 있는 경우 (예 : <tt style="color: #FF0000">`GenericRelation`</tt> 또는 <tt style="color: #FF0000">`ForeignKey`</tt>의 역관계) Boolean flag가 <tt style="color: #FF0000">`True`</tt>. 그렇지 않으면 <tt style="color: #FF0000">`False`</tt>.

<tt style="color: #FF0000">`Field.one_to_one`</tt>

Field가 <tt style="color: #FF0000">`OneToOneField`</tt>와 같이 one-to-one 관계를 갖는 경우 Boolean flag가 <tt style="color: #FF0000">`True`</tt>. 그렇지 않으면 <tt style="color: #FF0000">`False`</tt>.

<tt style="color: #FF0000">`Field.related_model`</tt>

필드가 관련된 모델을 가리킵니다. 예를 들어, <tt style="color: #FF0000">`Author`</tt>의 <tt style="color: #FF0000">`ForeignKey(Author, on_delete=models.CASCADE)`</tt>. <tt style="color: #FF0000">`GenericForeignKey`</tt>의 <tt style="color: #FF0000">`related_model`</tt>은 항상 <tt style="color: #FF0000">`None`</tt>입니다.