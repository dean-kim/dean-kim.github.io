---
layout: post
title:  "Django REST Framework -Validators"
date:   2017-05-11 10:43:59
author: Dean Kim
categories: Rest_Framework
tags:	Django Rest_framework
cover:  "/assets/instacode.png"
---

# Validators
- 원본 : [공식문서](http://www.django-rest-framework.org/api-guide/validators/)

Validators는 다른 유형의 fields간에 validation logic를 다시 사용하는 데 유용할 수 있습니다.
- Django 공식문서

대부분 REST framework에서 validation를 처리하는 경우 default field validation에 의존하거나 serializer 또는 field 클래스에 대한 명시적인 validation 메소드를 작성만 하면됩니다.
그러나 때로는 validation logic를 재사용 가능한 components에 배치하여 codebase 전체에서 쉽게 재사용 할 수 있습니다. 이 작업은 validator functions와 validator 클래스를 사용하여 수행 할 수 있습니다.


## Validation in REST framework

Django의 REST framework serializers의 Validation는 Django의 <tt style="color: #FF0000">`ModelForm`</tt> 클래스에서 Validation이 작동하는 방식과 조금 다르게 처리됩니다.
<tt style="color: #FF0000">`ModelForm`</tt>을 사용하면 validation가 부분적으로 form에서 수행되고 부분적으로 모델 인스턴스에서 수행됩니다. REST framework를 사용하면 validation는 전체적으로 serializer 클래스에서 수행됩니다. 이는 다음과 같은 이점이 있습니다.
* 적절한 구분을 제공하여 코드 동작을보다 명확하게 만듭니다.
* 바로 가기 <tt style="color: #FF0000">`ModelSerializer`</tt> 클래스를 사용하거나 명시적 <tt style="color: #FF0000">`Serializer`</tt> 클래스를 사용하는 것은 쉽게 전환할 수 있습니다. <tt style="color: #FF0000">`ModelSerializer`</tt>에 사용되는 모든 validation 동작은 복제가 간단합니다.
* serializer 인스턴스의 <tt style="color: #FF0000">`repr`</tt>을 출력하면 적용되는 validation rules이 정확하게 표시됩니다. 모델 인스턴스에서 추가 숨겨진 validation 동작이 호출되지 않습니다.

<tt style="color: #FF0000">`ModelSerializer`</tt>를 사용하면 이 모든 것이 자동으로 처리됩니다. 대신 <tt style="color: #FF0000">`Serializer`</tt> 클래스를 사용하여 drop down하려면 validation rules을 명시적으로 정의해야합니다.
예 : 
REST framework가 명시적 validation을 사용하는 방법의 예로, 고유성 제약 조건이 있는 field가 있는 간단한 모델 클래스를 사용합니다.
~~~~
class CustomerReportRecord(models.Model):
    time_raised = models.DateTimeField(default=timezone.now, editable=False)
    reference = models.CharField(unique=True, max_length=20)
    description = models.TextField()
~~~~
다음은 <tt style="color: #FF0000">`CustomerReportRecord`</tt> 인스턴스를 생성하거나 업데이트 할 때 사용할 수 있는 기본 <tt style="color: #FF0000">`ModelSerializer`</tt>입니다.
~~~~
class CustomerReportSerializer(serializers.ModelSerializer):
    class Meta:
        model = CustomerReportRecord
~~~~
<tt style="color: #FF0000">`manage.py shell`</tt>을 사용하여 Django shell을 열면됩니다.
~~~~
from project.example.serializers import CustomerReportSerializer
serializer = CustomerReportSerializer()
print(repr(serializer))
CustomerReportSerializer():
    id = IntegerField(label='ID', read_only=True)
    time_raised = DateTimeField(read_only=True)
    reference = CharField(max_length=20, validators=[<UniqueValidator(queryset=CustomerReportRecord.objects.all())>])
    description = CharField(style={'type': 'textarea'})
~~~~
여기서 흥미로운 부분은 <tt style="color: #FF0000">`reference`</tt> field입니다. 고유성 제약이 serializer field의 validator에 의해 명시적으로 적용되고 있음을 알 수 있습니다.
이 더 명시적인 스타일 때문에 REST framework에는 core Django에서는 사용할 수 없는 몇 가지 validator 클래스가 포함되어 있습니다. 이 클래스들은 아래에 자세히 설명되어 있습니다.


## UniqueValidator

이 validator를 사용하여 모델 fields에 <tt style="color: #FF0000">`unique=True`</tt> 제약 조건을 적용 할 수 있습니다. 하나의 필수 인수와 선택적 <tt style="color: #FF0000">`messages`</tt> argument를 취합니다.
* <tt style="color: #FF0000">`queryset`</tt>(필수) - 이것은 고유성을 강요해야하는 queryset입니다.
* <tt style="color: #FF0000">`message`</tt> - validation가 실패 시 사용해야하는 오류 메시지
* <tt style="color: #FF0000">`lookup`</tt> - being validated value로 기존 인스턴스를 찾는 데 사용 된 조회입니다. 기본값은 <tt style="color: #FF0000">`'exact'`</tt>입니다.

이 validator는 다음과 같이 serializer fields에 적용되어야합니다.
~~~~
from rest_framework.validators import UniqueValidator

slug = SlugField(
    max_length=100,
    validators=[UniqueValidator(queryset=BlogPost.objects.all())]
)
~~~~


## UniqueTogetherValidator

이 validator를 사용하여 모델 인스턴스에 <tt style="color: #FF0000">`unique_together`</tt> 제약 조건을 적용 할 수 있습니다. 여기에는 두 개의 필수 arguments와 단일 선택적 <tt style="color: #FF0000">`messages`</tt> argument가 있습니다.
* <tt style="color: #FF0000">`queryset`</tt>(필수) - 이것은 고유성을 강요해야하는 queryset입니다.
* <tt style="color: #FF0000">`fields`</tt>(필수) - 고유한 집합을 만들어야 하는 field 이름의 목록 또는 tuple입니다. 이들은 serializer 클래스의 field로 존재해야합니다.
* <tt style="color: #FF0000">`message`</tt> - validation가 실패 시 사용해야하는 오류 메시지

이 validator는 다음과 같이 serializer fields에 적용되어야합니다.
~~~~
from rest_framework.validators import UniqueTogetherValidator

class ExampleSerializer(serializers.Serializer):
    # ...
    class Meta:
        # ToDo items belong to a parent list, and have an ordering defined
        # by the 'position' field. No two items in a given list may share
        # the same position.
        validators = [
            UniqueTogetherValidator(
                queryset=ToDoItem.objects.all(),
                fields=('list', 'position')
            )
        ]
~~~~
참조 : 
<tt style="color: #FF0000">`UniqueTogetherValidation`</tt> 클래스는 항상 적용되는 모든 fields가 항상 필요한 것으로 처리된다는 암시적 제약 조건을 부과합니다. <tt style="color: #FF0000">`default`</tt> values가 있는 fields는 사용자 입력에서 생략된 경우에도 항상 value을 제공하므로 예외입니다.


## UniqueForDateValidator
## UniqueForMonthValidator
## UniqueForYearValidator

이 validator들은 model 인스턴스에 대해 <tt style="color: #FF0000">`unique_for_date`</tt>, <tt style="color: #FF0000">`unique_for_month`</tt> 및 <tt style="color: #FF0000">`unique_for_year`</tt> 제약 조건을 적용하는데 사용할 수 있습니다. 다음과 같은 arguments를 갖습니다.
* <tt style="color: #FF0000">`queryset`</tt>(필수) - 이것은 고유성을 강요해야하는 queryset입니다.
* <tt style="color: #FF0000">`field`</tt>(필수) - 지정된 날짜 범위의 고유성에 대한 validate하는 field 이름입니다. 이것은 serializer 클래스의 field로 존재해야합니다.
* <tt style="color: #FF0000">`date_field`</tt>(필수) - 고유성 제한 조건의 날짜 범위를 결정하는 데 사용할 field 이름입니다. 이것은 serializer 클래스의 field로 존재해야합니다.
* <tt style="color: #FF0000">`message`</tt> - validation가 실패 시 사용해야하는 오류 메시지

이 validator는 다음과 같이 serializer fields에 적용되어야합니다.
~~~~
from rest_framework.validators import UniqueForYearValidator

class ExampleSerializer(serializers.Serializer):
    # ...
    class Meta:
        # Blog posts should have a slug that is unique for the current year.
        validators = [
            UniqueForYearValidator(
                queryset=BlogPostItem.objects.all(),
                field='slug',
                date_field='published'
            )
        ]
~~~~
validation에 사용되는 날짜 field는 항상 serializer 클래스에 있어야합니다. validation이 실행될 때까지 default value에 사용되는 value가 생성되지 않기 때문에 모델 클래스 <tt style="color: #FF0000">`default=...`</tt>에 간단히 의존 할 수 없습니다.
API를 어떻게 동작시키는 지에 따라 이 스타일을 사용하려는 두 가지 스타일이 있습니다. <tt style="color: #FF0000">`ModelSerializer`</tt>를 사용하는 경우 REST framework에서 생성하는 defaults를 사용하는 것이 좋지만 <tt style="color: #FF0000">`Serializer`</tt>를 사용하거나 보다 명시적인 제어를 원한다면 아래에 설명된 스타일을 사용하십시오.

### Using with a writable date field.

사용자가 날짜 field를 볼 수는 있지만 편집 할 수 없도록하려면 <tt style="color: #FF0000">`read_only=True`</tt>로 설정하고 추가로 <tt style="color: #FF0000">`default=...`</tt> argument를 설정하십시오.
~~~~
published = serializers.DateTimeField(read_only=True, default=timezone.now)
~~~~
이 필드는 사용자에게 쓸 수 없지만 default value은 여전히 <tt style="color: #FF0000">`​​validated_data`</tt>로 전달됩니다.

### Using with a hidden date field.

날짜 field를 사용자가 완전히 숨기려면 <tt style="color: #FF0000">`HiddenField`</tt>를 사용하십시오. 이 field 유형은 사용자 입력을 허용하지 않고 대신 항상 기본값을 serializer의 <tt style="color: #FF0000">`validated_data`</tt>로 반환합니다.
~~~~
published = serializers.HiddenField(default=timezone.now)
~~~~
참조 :
<tt style="color: #FF0000">`UniqueFor<Range>Validation`</tt> 클래스는 적용되는 fields가 항상 필요한 것으로 처리된다는 암시적 제약 조건을 적용합니다. <tt style="color: #FF0000">`default`</tt> values가 있는 fields는 사용자 입력에서 생략된 경우에도 항상 value를 제공하므로 예외입니다.


## Advanced field defaults

serializer의 여러 field에 적용되는 Validators는 API 클라이언트가 제공해서는 안되지만 Validators의 입력으로 사용할 수 있는 field 입력이 필요할 수 있습니다.
이러한 유형의 validation에 사용할 수있는 두 가지 패턴은 다음과 같습니다.
* <tt style="color: #FF0000">`HiddenField`</tt> 사용. 이 field는 <tt style="color: #FF0000">`validated_data`</tt>에 있지만 serializer output representation에서는 사용되지 않습니다.
* <tt style="color: #FF0000">`read_only=True`</tt>와 함께 표준 field를 사용하지만 <tt style="color: #FF0000">`default=...`</tt> argument도 포함합니다. 이 field는 serializer output representation에 사용되지만 사용자가 직접 설정할 수는 없습니다.

REST framework는 이 컨텍스트에서 유용 할 수 있는 몇 가지 defaults를 포함합니다.

### CurrentUserDefault

현재 사용자를 나타내는 데 사용할 수 있는 default 클래스입니다. 이것을 사용하기 위해서, serializer를 인스턴스화 할 때 'request'가 컨텍스트 dictionary의 일부로 제공되어야합니다.
~~~~
owner = serializers.HiddenField(
    default=serializers.CurrentUserDefault()
)
~~~~

### CreateOnlyDefault

create operations 중 default argument만을 설정하는 데 사용할 수있는 default 클래스. 업데이트 중 field는 생략됩니다.
하나의 argument를 취하는데,이 argument는 create operations 중 사용해야하는 default valu이거나 default valu인 하나의 argument입니다.
~~~~
created_at = serializers.DateTimeField(
    read_only=True,
    default=serializers.CreateOnlyDefault(timezone.now)
)
~~~~


## Limitations of validators

<tt style="color: #FF0000">`ModelSerializer`</tt>가 생성하는 기본 serializer 클래스를 사용하는 대신 validation를 명시적으로 처리해야하는 모호한 경우가 있습니다.
이러한 경우 serializer <tt style="color: #FF0000">`Meta.validators`</tt> attribute에 대한 빈 목록을 지정하여 자동 생성 된 validators를 사용하지 않도록 설정할 수 있습니다.

### Optional fields

기본적으로 "unique together" validation은 모든 fields가 <tt style="color: #FF0000">`required=True`</tt>인지 확인합니다. 경우에 따라 fields 중 하나에 명시적으로 <tt style="color: #FF0000">`required=False`</tt>를 적용하면 원하는 validation 동작이 모호할 수 있습니다.
이 경우 일반적으로 serializer 클래스에서 validator를 제외하고 <tt style="color: #FF0000">`.validate()`</tt> 메서드 또는 뷰에서 validation logic 를 명시적으로 작성해야합니다.
예 :
~~~~
class BillingRecordSerializer(serializers.ModelSerializer):
    def validate(self, data):
        # Apply custom validation either here, or in the view.

    class Meta:
        fields = ('client', 'date', 'amount')
        extra_kwargs = {'client': {'required': 'False'}}
        validators = []  # Remove a default "unique together" constraint.
~~~~

### Updating nested serializers

기존 인스턴스에 업데이트를 적용 할 때 uniqueness validators는 현재 인스턴스를 고uniqueness check에서 제외합니다. 현재 인스턴스는 uniqueness check의 컨텍스트에서 사용할 수 있습니다. 이 attribute는 serializer의 attribute으로 존재하기 때문에 처음에는 serializer를 인스턴스화할 때 <tt style="color: #FF0000">`instance=...`</tt>를 사용하여 전달되었습니다.
nested 된 serializer에 대한 업데이트 operations의 경우 인스턴스를 사용할 수 없으므로 이 제외를 적용할 방법이 없습니다.
다시 말하면, serializer 클래스에서 validator를 명시적으로 제거하고 validator 제약 조건에 대한 코드를 명시적으로 <tt style="color: #FF0000">`.validate()`</tt> 메서드 또는 뷰에 작성하려고합니다.

### Debugging complex cases

<tt style="color: #FF0000">`ModelSerializer`</tt> 클래스가 어떤 동작을 하는지 확실히 모를 경우 <tt style="color: #FF0000">`manage.py shell`</tt>을 실행하고 serializer의 인스턴스를 출력하여 자동으로 생성되는 fields와 validators를 inspect할 수 있는 것이 좋습니다.
~~~~
serializer = MyComplexModelSerializer()
print(serializer)
class MyComplexModelSerializer:
    my_fields = ...
~~~~
또한 복잡한 경우에는 default <tt style="color: #FF0000">`ModelSerializer`</tt> behavior를 사용하는 대신 serializer 클래스를 명시적으로 정의하는 것이 더 나을 수 있습니다. 여기에는 조금 더 많은 코드가 포함되지만 결과로 발생하는 동작이 더 명쾌하게 전달됩니다.


## Writing custom validators

Django의 기존 validators를 사용하거나 custom validators를 작성할 수 있습니다.

### Function based

validator는 실패 시 <tt style="color: #FF0000">`serializers.ValidationError`</tt>를 발생시키는 호출 가능 함수일 수 있습니다.
~~~~
def even_number(value):
    if value % 2 != 0:
        raise serializers.ValidationError('This field must be an even number.')
~~~~

#### Field-level validation

<tt style="color: #FF0000">`Serializer`</tt> subclass에 <tt style="color: #FF0000">`.validate_<field_name>`</tt> 메소드를 추가하여 custom field-leve validation을 지정할 수 있습니다. 이 내용은 [Serializer 문서](http://www.django-rest-framework.org/api-guide/serializers/#field-level-validation)에 설명되어 있습니다.

### Class-based

class-based validator를 작성하려면 <tt style="color: #FF0000">`__call__`</tt> 메소드를 사용하십시오. class-based validator는 동작을 parameterize하고 다시 사용할 수 있으므로 유용합니다.
~~~~
class MultipleOf(object):
    def __init__(self, base):
        self.base = base

    def __call__(self, value):
        if value % self.base != 0:
            message = 'This field must be a multiple of %d.' % self.base
            raise serializers.ValidationError(message)
~~~~

#### Using <tt style="color: #FF0000">`set_context()`</tt>

일부 advanced cases에서는 validator를 추가 컨텍스트로 사용되는 serializer field로 전달할 수 있습니다. class-based validator에서 <tt style="color: #FF0000">`set_context`</tt> 메서드를 선언하면됩니다.
~~~~
def set_context(self, serializer_field):
    # Determine if this is an update or a create operation.
    # In `__call__` we can then use that information to modify the validation behavior.
    self.is_update = serializer_field.parent.instance is not None
~~~~