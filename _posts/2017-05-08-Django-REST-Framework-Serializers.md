---
layout: post
title:  "Django REST Framework -Serializers"
date:   2017-05-08 08:43:59
author: Dean Kim
categories: Rest_framework
tags:	Django Rest_framework
cover:  "/assets/instacode.png"
---

# Serializers
- 원본 : [공식문서](http://www.django-rest-framework.org/api-guide/serializers/)



Serializers는 querysets 및 모델 인스턴스와 같은 복잡한 데이터를 JSON, XML 또는 기타 컨텐트 유형으로 쉽게 렌더링 할 수 있는 네이티브 Python 데이터 유형으로 변환해줍니다.
또한 serializer는 deserialization을 제공하여 들어오는 데이터의 유효성을 먼저 확인한 후에 구문 분석 된 데이터를 복합 형식으로 다시 변환 할 수 있습니다.

REST Framework의 serializers는 Django의 Form 및 ModelForm 클래스와 매우 유사하게 작동합니다.
Serializer 클래스는 모델 인스턴스와 querysets를 다루는 serializer를 생성하는 빠른 방법인 ModelSerializer 클래스만이 아니라 response의 출력을 제어하기 위한 강력하고 일반적인 방법을 제공하는 Serializer 클래스를 제공합니다.


## Serializers 선언하기

다음의 예시 코드를 참고.

~~~~
from datetime import datetime

class Comment(object):
    def __init__(self, email, content, created=None):
        self.email = email
        self.content = content
        self.created = created or datetime.now()

comment = Comment(email='leila@example.com', content='foo bar')
~~~~
Comment 객체에 해당하는 데이터를 serialize / deserialize 하는데 사용할 serializer를 선언했습니다.

serializer를 선언하는 것은 form을 선언하는 것과 비슷합니다.

~~~~
from rest_framework import serializers

class CommentSerializer(serializers.Serializer):
    email = serializers.EmailField()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
~~~~

## Serializing objects

CommentSerializer를 사용하여 주석 또는 주석 목록을 serialize 할 수 있습니다. 다시 말하면, Serializer 클래스를 사용하는 것은 Form 클래스를 사용하는 것과 비슷합니다.
~~~~
serializer = CommentSerializer(comment)
serializer.data
# {'email': 'leila@example.com', 'content': 'foo bar', 'created': '2016-01-27T15:17:10.375877'}
~~~~
여기에서 모델 인스턴스를 Python 기본 데이터 유형으로 변환했습니다. serialize 과정을 마무리하기 위해 데이터를 json으로 렌더링합니다.
~~~~
from rest_framework.renderers import JSONRenderer

json = JSONRenderer().render(serializer.data)
json
# b'{"email":"leila@example.com","content":"foo bar","created":"2016-01-27T15:17:10.375877"}'
~~~~

## Deserializing objects

deserialization도 비슷합니다. 먼저 Python 네이티브 데이터 타입으로 스트림을 파싱합니다.
~~~~
from django.utils.six import BytesIO
from rest_framework.parsers import JSONParser

stream = BytesIO(json)
data = JSONParser().parse(stream)
~~~~
그 다음 네이티브 데이터 유형을 dictionary of validated data 로 복원합니다.
~~~~
serializer = CommentSerializer(data=data)
serializer.is_valid()
# True
serializer.validated_data
# {'content': 'foo bar', 'email': 'leila@example.com', 'created': datetime.datetime(2012, 08, 22, 16, 20, 09, 822243)}
~~~~

## Saving instances

만약 validated data를 기반으로 완전한 객체 인스턴스를 반환하려면 <tt style="color: #FF0000">`.create()`</tt> 와 <tt style="color: #FF0000">`update()`</tt> 메소드 중 하나 혹은 둘 모두를 구현해야 합니다. 예 :
~~~~
class CommentSerializer(serializers.Serializer):
    email = serializers.EmailField()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()

    def create(self, validated_data):
        return Comment(**validated_data)

    def update(self, instance, validated_data):
        instance.email = validated_data.get('email', instance.email)
        instance.content = validated_data.get('content', instance.content)
        instance.created = validated_data.get('created', instance.created)
        return instance
~~~~
만약 객체 인스턴스가 Django 모델과 일치하는 경우 이 메소드들이 객체를 데이터베이스에 저장하도록 해야합니다 예를 들어, Comment가 Django 모델인 경우 메소드들은 다음과 같습니다.
~~~~
def create(self, validated_data):
        return Comment.objects.create(**validated_data)

    def update(self, instance, validated_data):
        instance.email = validated_data.get('email', instance.email)
        instance.content = validated_data.get('content', instance.content)
        instance.created = validated_data.get('created', instance.created)
        instance.save()
        return instance
~~~~
이제 데이터를 deserializing 할 때, <tt style="color: #FF0000">`save()`</tt>를 호출해서 validated data 기반으로 객체 인스턴스를 반환할 수 있습니다.
~~~~
comment = serializer.save()
~~~~
<tt style="color: #FF0000">`save()`</tt>를 호출하면 serializer 클래스를 인스턴스화 할 때 기존 인스턴스가 전달되었는지 여부에 따라 새 인스턴스를 만들거나 기존 인스턴스를 업데이트합니다.
~~~~
# .save() will create a new instance.
serializer = CommentSerializer(data=data)

# .save() will update the existing `comment` instance.
serializer = CommentSerializer(comment, data=data)
~~~~
<tt style="color: #FF0000">`.create()`</tt>, <tt style="color: #FF0000">`.update()`</tt> 메서드는 선택사항입니다. serializer 클래스의 use-case에 따라 둘 중 하나 혹은 모두를 구현할 수 있습니다.

### Passing additional attributes to <tt style="color: #FF0000">`.save()`</tt>

때로는 인스턴스를 저장하는 시점에 뷰 코드가 추가 데이터를 추가 할 수 있어야 합니다. 이 추가 데이터에는 현재 사용자, 현재 시간 또는 요청 데이터의 일부가 아닌 다른 정보가 포함될 수 있습니다.
<tt style="color: #FF0000">`save()`</tt>를 호출할 때 추가적인 키워드 arguments를 포함시킬 수 있습니다. 예 :
~~~~
serializer.save(owner=request.user)
~~~~
추가적인 키워드 arguments는 <tt style="color: #FF0000">`.create()`</tt>, <tt style="color: #FF0000">`.update()`</tt>가 호출될 때 validated_data argument에 포함됩니다.

### Overriding <tt style="color: #FF0000">`.save()`</tt> directly.

어떤 경우에는 <tt style="color: #FF0000">`.create()`</tt>와 <tt style="color: #FF0000">`.update()`</tt> 메소드 이름이 의미가 없을 수 있습니다. 예를 들어 contact 양식에서 새로운 인스턴스를 만들지 않고 이메일 혹은 기타 메세지로 대신 보낼 수 있습니다.
이 경우들에서 <tt style="color: #FF0000">`.save()`</tt>를 직접 읽고 무시할 수 있습니다. 예 :
~~~~
class ContactForm(serializers.Serializer):
    email = serializers.EmailField()
    message = serializers.CharField()

    def save(self):
        email = self.validated_data['email']
        message = self.validated_data['message']
        send_email(from=email, message=message)
~~~~
이 경우 해당 serializer의 <tt style="color: #FF0000">`.validated_data`</tt> 속성에 직접 access 해야 합니다.


## Validation

데이터를 deserializing할 때 validated data에 접근하기 전 항상 <tt style="color: #FF0000">`.is_valid()`</tt>를 호출하거나 객체 인스턴스를 저장해야 합니다. 만약 validation errors가 발생할 경우 <tt style="color: #FF0000">`.errors`</tt> 속성에 결과 오류 메세지를 나타내는 dictionary가 포함됩니다.
예 :
~~~~
serializer = CommentSerializer(data={'email': 'foobar', 'content': 'baz'})
serializer.is_valid()
# False
serializer.errors
# {'email': [u'Enter a valid e-mail address.'], 'created': [u'This field is required.']}
~~~~
dictionary 안의 각각의 key는 field name 이며 values는 해당 field에 해당하는 오류 메세지의 문자열 목록입니다.
<tt style="color: #FF0000">`non_field_errors`</tt> key가 있을 수도 있으며 일반적인 validation errors의 나열일 수 있습니다.
<tt style="color: #FF0000">`non_field_errors`</tt> key의 name은 <tt style="color: #FF0000">`NON_FIELD_ERRORS_KEY`</tt> REST framework 설정을 사용해서 사용자 정의할 수 있습니다.

item 목록을 deserializing할 때 오류들은 deserialized items을 나타내는 dictionaries 목록으로 반환됩니다.

### Raising an exception on invalid data

<tt style="color: #FF0000">`.is_valid()`</tt> 메소드는 validation errors가 있는 경우 <tt style="color: #FF0000">`serializers.ValidationError`</tt> 예외를 발생시키는 <tt style="color: #FF0000">`raise_exception`</tt> flag를 선택적으로 사용합니다.
이러한 예외는 REST framework에서 제공하는 기본 예외 처리 handler가 자동으로 처리하며 default로 <b style="color: #FF0000">`HTTP 400 Bad Request`</b>를 반환합니다.
~~~~
# Return a 400 response if the data was invalid.
serializer.is_valid(raise_exception=True)
~~~~

### Field-level validation

<tt style="color: #FF0000">`Serializer`</tt> 서브 클래스에 <tt style="color: #FF0000">`.validate_<field_name>`</tt> 메소드를 추가하여 custom field-level validation을 지정할 수 있습니다. 이것들은 Django forms의 <tt style="color: #FF0000">`.clean_<field_name>`</tt> 메소드와 비슷합니다.
이 메소드들은 하나의 argument를 받으며 이 argument는 validation에 필요한 field value 입니다.
<tt style="color: #FF0000">`validate_<field_name>`</tt> 메서드는 validated value을 반환하거나 <tt style="color: #FF0000">`serializers.ValidationError`</tt>를 발생시켜야합니다. 예 :
~~~~
from rest_framework import serializers

class BlogPostSerializer(serializers.Serializer):
    title = serializers.CharField(max_length=100)
    content = serializers.CharField()

    def validate_title(self, value):
        """
        Check that the blog post is about Django.
        """
        if 'django' not in value.lower():
            raise serializers.ValidationError("Blog post is not about Django")
        return value
~~~~
참고 : <tt style="color: #FF0000">`<field_name>`</tt>이 <tt style="color: #FF0000">`required=False`</tt> 매개 변수를 사용하여 serializer에 선언 된 경우 필드가 포함되어 있지 않으면 validation 단계가 수행되지 않습니다.

### Object-level validation

multiple fields에 액세스해야하는 validation를 수행하려면 <tt style="color: #FF0000">`Serializer`</tt> 하위 클래스에 <tt style="color: #FF0000">`.validate()`</tt>라는 메서드를 추가합니다. 이 메소드는 field values의 dictionary 인 single argument를 취합니다. 필요한 경우 <tt style="color: #FF0000">`ValidationError`</tt>를 발생시키거나 validated values를 반환해야합니다. 예 :
~~~~
from rest_framework import serializers

class EventSerializer(serializers.Serializer):
    description = serializers.CharField(max_length=100)
    start = serializers.DateTimeField()
    finish = serializers.DateTimeField()

    def validate(self, data):
        """
        Check that the start is before the stop.
        """
        if data['start'] > data['finish']:
            raise serializers.ValidationError("finish must occur after start")
        return data
~~~~

### Validators

serializer의 개별 필드는 validators를 field 인스턴스에 선언하여 다음과 같이 정의 할 수 있습니다.
~~~~
def multiple_of_ten(value):
    if value % 10 != 0:
        raise serializers.ValidationError('Not a multiple of ten')

class GameRecord(serializers.Serializer):
    score = IntegerField(validators=[multiple_of_ten])
    ...
~~~~

또한 Serializer 클래스에는 전체 field 데이터 set에 적용되는 재사용 가능한 validators가 포함될 수 있습니다. 이 validators는 다음과 같이 내부 <tt style="color: #FF0000">`Meta`</tt> 클래스에 선언함으로써 포함됩니다.
~~~~
class EventSerializer(serializers.Serializer):
    name = serializers.CharField()
    room_number = serializers.IntegerField(choices=[101, 102, 103, 201])
    date = serializers.DateField()

    class Meta:
        # Each room only has one event per day.
        validators = UniqueTogetherValidator(
            queryset=Event.objects.all(),
            fields=['room_number', 'date']
        )
~~~~
더 많은 정보는 [validators documentation참고](http://www.django-rest-framework.org/api-guide/validators/)


## Accessing the initial data and instance

serializer 인스턴스에 초기 객체 또는 queryset를 전달할 때 객체는 <tt style="color: #FF0000">`.instance`</tt>로 사용 가능합니다. 초기 객체가 전달되지 않으면 <tt style="color: #FF0000">`.instance`</tt> 속성은 <tt style="color: #FF0000">`None`</tt>이 됩니다.
데이터를 serializer 인스턴스에 전달할 때 수정되지 않은 데이터는 <tt style="color: #FF0000">`.initial_data`</tt>로 사용 가능합니다. data 키워드 argument가 전달되지 않으면 <tt style="color: #FF0000">`.initial_data`</tt> 속성이 존재하지 않습니다.


## Partial updates

기본적으로 serializer는 모든 required fields에 values을 전달해야하며 그렇지 않으면 validation errors가 발생합니다. partial 업데이트를 허용하기 위해 <tt style="color: #FF0000">`partial`</tt> argument를 사용할 수 있습니다.
~~~~
# Update `comment` with partial data
serializer = CommentSerializer(comment, data={'content': u'foo bar'}, partial=True)
~~~~


## Dealing with nested objects

앞의 예제처럼 단순한 데이터 유형만을 가진 객체를 다루는 경우에는 문제가 없지만 객체의 일부 속성이 문자열, 날짜 또는 정수와 같은 단순한 데이터 유형이 아닌 복잡한 객체를 표현해야 하는 경우가 있습니다.
<tt style="color: #FF0000">`Serializer`</tt> 클래스 자체는 <tt style="color: #FF0000">`Field`</tt> 유형이며, 한 객체 유형이 다른 객체 유형 내에 nested되어있는 관계를 나타내는 데 사용할 수 있습니다.
~~~~
class UserSerializer(serializers.Serializer):
    email = serializers.EmailField()
    username = serializers.CharField(max_length=100)

class CommentSerializer(serializers.Serializer):
    user = UserSerializer()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
~~~~
nested 된 representation이 <tt style="color: #FF0000">`None`</tt> value를 선택적으로 받아 들일 수 있으면 <tt style="color: #FF0000">`required=False`</tt> flag를 nested 된 serializer에 전달해야합니다.
~~~~
class CommentSerializer(serializers.Serializer):
    user = UserSerializer(required=False)  # May be an anonymous user.
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
~~~~
마찬가지로 nested representation이 item 목록일 경우 nested 된 serialized에 <tt style="color: #FF0000">`many=True`</tt> flag를 전달해야합니다.
~~~~
class CommentSerializer(serializers.Serializer):
    user = UserSerializer(required=False)
    edits = EditItemSerializer(many=True)  # A nested list of 'edit' items.
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
~~~~


## Writable nested representations

데이터의 deserializing를 지원하는 nested representations을 처리 할 때 nested 된 객체의 오류는 nested 된 객체의 field name 아래에 nested 됩니다.
~~~~
serializer = CommentSerializer(data={'user': {'email': 'foobar', 'username': 'doe'}, 'content': 'baz'})
serializer.is_valid()
# False
serializer.errors
# {'user': {'email': [u'Enter a valid e-mail address.']}, 'created': [u'This field is required.']}
~~~~
비슷하게, `.validated_data` 속성은 nested 된 데이터 구조를 포함합니다.

### Writing <tt style="color: #FF0000">`.create()`</tt> methods for nested representations

쓰기 가능한 nested representations을을 지원하려면 여러 객체를 저장하는 <tt style="color: #FF0000">`.create()`</tt> 또는 <tt style="color: #FF0000">`.update()`</tt> 메소드를 작성해야합니다.
다음 예제에서는 nested 된 프로필 개체가 있는 사용자 만들기를 처리하는 방법을 보여줍니다.
~~~~
class UserSerializer(serializers.ModelSerializer):
    profile = ProfileSerializer()

    class Meta:
        model = User
        fields = ('username', 'email', 'profile')

    def create(self, validated_data):
        profile_data = validated_data.pop('profile')
        user = User.objects.create(**validated_data)
        Profile.objects.create(user=user, **profile_data)
        return user
~~~~

### Writing <tt style="color: #FF0000">`.update()`</tt> methods for nested representations

업데이트의 경우 관계 업데이트를 처리하는 방법에 대해 신중하게 생각하고 싶을 것입니다. 예를 들어 관계에 대한 데이터가 <tt style="color: #FF0000">`None`</tt> 또는 제공되지 않은 경우 다음 중 어떤 것이 발생해야합니까?
* 관계를 데이터베이스에서 <tt style="color: #FF0000">`NULL`</tt>로 설정
* 연관된 인스턴스 삭제
* 데이터를 무시하고 인스턴스를 그대로 유지
* validation error를 발생시킴

다음은 이전 <tt style="color: #FF0000">`UserSerializer`</tt> 클래스의 <tt style="color: #FF0000">`update()`</tt> 메소드 예제입니다.
~~~~
def update(self, instance, validated_data):
        profile_data = validated_data.pop('profile')
        # Unless the application properly enforces that this field is
        # always set, the follow could raise a `DoesNotExist`, which
        # would need to be handled.
        profile = instance.profile

        instance.username = validated_data.get('username', instance.username)
        instance.email = validated_data.get('email', instance.email)
        instance.save()

        profile.is_premium_member = profile_data.get(
            'is_premium_member',
            profile.is_premium_member
        )
        profile.has_support_contract = profile_data.get(
            'has_support_contract',
            profile.has_support_contract
         )
        profile.save()

        return instance
~~~~
nested 된 생성 및 업데이트의 동작이 모호할 수 있고 관련 모델간에 복잡한 종속성이 필요할 수 있으므로 REST framework 3에서는 이러한 메서드를 항상 명시적으로 작성해야합니다. 기본 <tt style="color: #FF0000">`ModelSerializer`</tt> <tt style="color: #FF0000">`.create()`</tt> 및 <tt style="color: #FF0000">`.update()`</tt> 메소드는 쓰기 가능한 nested representations에 대한 지원을 포함하지 않습니다.
자동적인 기입 가능한 nested representations의 일부를 자동적으로 서포트하는 third party package가, 3.1 릴리스와 함께 릴리스 될 가능성도 있습니다.

### Handling saving related instances in model manager classes

serializer에 여러 관련 인스턴스를 저장하는 대신 올바른 인스턴스를 생성하는 사용자 정의 모델 관리자 클래스를 작성할 수 있습니다.
예를 들어 <tt style="color: #FF0000">`User`</tt> 인스턴스와 <tt style="color: #FF0000">`Profile`</tt> 인스턴스가 항상 pair로 함께 생성되도록하고 싶다고 가정합하면, 다음과 같은 커스텀 매니저 클래스를 작성할 수 있습니다.
~~~~
class UserManager(models.Manager):
    ...

    def create(self, username, email, is_premium_member=False, has_support_contract=False):
        user = User(username=username, email=email)
        user.save()
        profile = Profile(
            user=user,
            is_premium_member=is_premium_member,
            has_support_contract=has_support_contract
        )
        profile.save()
        return user
~~~~
이 관리자 클래스는 user 인스턴스와 profile 인스턴스가 항상 동시에 생성되는 것 보다 잘 캡슐화합니다. serializer 클래스의 <tt style="color: #FF0000">`.create()`</tt> 메서드를 새 관리자 메서드를 사용하도록 다시 작성할 수 있습니다.
~~~~
def create(self, validated_data):
    return User.objects.create(
        username=validated_data['username'],
        email=validated_data['email']
        is_premium_member=validated_data['profile']['is_premium_member']
        has_support_contract=validated_data['profile']['has_support_contract']
    )
~~~~
* [model managers에 대한 더 많은 정보](https://docs.djangoproject.com/en/1.11/topics/db/managers/)
* [모델과 관리자 클래스 사용에 관한 블로그 포스트를 참조](https://www.dabapps.com/blog/django-models-and-encapsulation/)


## Dealing with multiple objects

<tt style="color: #FF0000">`Serializer`</tt> 클래스는 객체 목록의 serializing 또는 deserializing를 처리 할 수도 있습니다.

### Serializing multiple objects

단일 객체 인스턴스 대신 queryset 또는 객체 목록을 serialize 하려면 serializer를 인스턴스화 할 때 <tt style="color: #FF0000">`many=True`</tt> flag를 전달해야합니다. 그런 다음 serializing 할 queryset 또는 객체 목록을 전달할 수 있습니다.
~~~~
queryset = Book.objects.all()
serializer = BookSerializer(queryset, many=True)
serializer.data
# [
#     {'id': 0, 'title': 'The electric kool-aid acid test', 'author': 'Tom Wolfe'},
#     {'id': 1, 'title': 'If this is a man', 'author': 'Primo Levi'},
#     {'id': 2, 'title': 'The wind-up bird chronicle', 'author': 'Haruki Murakami'}
# ]
~~~~

### Deserializing multiple objects

여러 객체를 비 직렬화하는 기본 비헤이비어는 여러 객체 생성을 지원하지만 여러 객체 업데이트를 지원하지 않는 것입니다. 이러한 경우 중 하나를 지원하거나 사용자 지정하는 방법에 대한 자세한 내용은 아래의 [ListSerializer](http://www.django-rest-framework.org/api-guide/serializers/#listserializer) 설명서를 참조하십시오.


## Including extra context

serialize되는 객체 외에도 추가 된 컨텍스트를 serialize에 제공해야하는 경우가 있습니다. 한 가지 일반적인 경우는 하이퍼링크 된 관계를 포함하는 serializer를 사용하는 경우이며, serializer가 현재 요청에 액세스하여 정규화 된 URL을 제대로 생성 할 수 있어야합니다.
serializer를 인스턴스화 할 때 <tt style="color: #FF0000">`context`</tt> argument를 전달하여 임의의 추가 컨텍스트를 제공할 수 있습니다. 예 :
~~~~
serializer = AccountSerializer(account, context={'request': request})
serializer.data
# {'id': 6, 'owner': u'denvercoder9', 'created': datetime.datetime(2013, 2, 12, 09, 44, 56, 678870), 'details': 'http://example.com/accounts/6/details'}
~~~~
컨텍스트 dictionary는 custom <tt style="color: #FF0000">`.to_representation()`</tt> 메소드와 같은 serializer field 로직 내에서 <tt style="color: #FF0000">`self.context`</tt> 속성에 액세스하여 사용할 수 있습니다.


## ModelSerializer

Django model definitions와 밀접하게 매핑되는 serializer가 필요할 때가 있습니다.

<tt style="color: #FF0000">`ModelSerializer`</tt> 클래스는 Model fields에 해당하는 fields가 있는 <tt style="color: #FF0000">`Serializer`</tt> 클래스를 자동으로 만들 수있는 바로 가기를 제공합니다.

<tt style="color: #FF0000">`ModelSerializer`</tt> 클래스는 다음을 제외하고는 일반 <tt style="color: #FF0000">`Serializer`</tt> 클래스와 동일합니다.
* 모델을 기반으로 set of fields가 자동으로 생성됩니다.
* unique_together validator와 같은 serializer에 대한 validator를 자동으로 생성합니다.
* <tt style="color: #FF0000">`.create()`</tt> 및 <tt style="color: #FF0000">`.update()`</tt>의 간단한 default implementations을 포함합니다.

ModelSerializer 선언은 다음과 같습니다.
~~~~
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ('id', 'account_name', 'users', 'created')
~~~~
기본적으로 클래스의 모든 모델 fields는 해당 serializer fields에 매핑됩니다.
모델의 foreign keys와 같은 관계는 <tt style="color: #FF0000">`PrimaryKeyRelatedField`</tt>에 매핑됩니다. [serializer 관계 문서](http://www.django-rest-framework.org/api-guide/relations/)에 명시된대로 명시적으로 포함되지 않으면 역방향 관계가 기본적으로 포함되지 않습니다.

### Inspecting a <tt style="color: #FF0000">`ModelSerializer`</tt>

Serializer 클래스는 유용한 fields representation 문자열을 생성하므로 필드의 상태를 완전히 검사 할 수 있습니다. 이는 자동으로 생성되는 fields 및 validators set를 결정하려는 <tt style="color: #FF0000">`ModelSerializer`</tt>로 작업 할 때 특히 유용합니다.
이렇게하려면, Django shell을 열고, <tt style="color: #FF0000">`python manage.py shell`</tt>을 사용하고, serializer 클래스를 가져 와서 인스턴스화하고, 객체 표현을 출력하십시오.
~~~~
from myapp.serializers import AccountSerializer
serializer = AccountSerializer()
print(repr(serializer))
AccountSerializer():
    id = IntegerField(label='ID', read_only=True)
    name = CharField(allow_blank=True, max_length=100, required=False)
    owner = PrimaryKeyRelatedField(queryset=User.objects.all())
~~~~


## Specifying which fields to include

default fields의 subset을 모델 serializer에서만 사용하려는 경우 <tt style="color: #FF0000">`ModelForm`</tt>에서와 마찬가지로 <tt style="color: #FF0000">`fields`</tt>를 사용하거나 <tt style="color: #FF0000">`exclude`</tt>옵션을 사용할 수 있습니다. <tt style="color: #FF0000">`fields`</tt> attribute을 사용하여 serialized 해야 하는 모든 fields를 명시적으로 설정하는 것이 좋습니다. 이렇게하면 모델이 변경 될 때 실수로 데이터가 노출 될 가능성이 줄어 듭니다.
예 :
~~~~
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ('id', 'account_name', 'users', 'created')
~~~~
또한 <tt style="color: #FF0000">`fields`</tt> attribute을 특수 값 <tt style="color: #FF0000">`'__all__'`</tt>으로 설정하여 모델의 모든 fields를 사용해야 함을 나타낼 수 있습니다.
예 :
~~~~
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = '__all__'
~~~~
serializer에서 제외 할 fields 목록에 <tt style="color: #FF0000">`exclude`</tt> attribute을 설정할 수 있습니다.
예 :
~~~~
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        exclude = ('users',)
~~~~
위의 예에서 <tt style="color: #FF0000">`Account`</tt> 모델에 <tt style="color: #FF0000">`account_name`</tt>, <tt style="color: #FF0000">`users`</tt> 및 <tt style="color: #FF0000">`created`</tt> 필드가 세 개있는 경우 <tt style="color: #FF0000">`account_name`</tt>, <tt style="color: #FF0000">`created`</tt> fields가 serialize되도록 생성됩니다.
<tt style="color: #FF0000">`fields`</tt> 및 <tt style="color: #FF0000">`exclude`</tt> attributes의 이름은 일반적으로 모델 클래스의 모델 fields에 매핑됩니다.
또는 <tt style="color: #FF0000">`fields`</tt> 옵션의 이름은 모델 클래스에 존재하는 arguments를 취하지 않는 properties이나 메소드에 매핑 할 수 있습니다.


## Specifying nested serialization

default <tt style="color: #FF0000">`ModelSerializer`</tt>는 관계에 primary keys를 사용하지만 <tt style="color: #FF0000">`depth`</tt> 옵션을 사용하여 nested representations을 쉽게 생성 할 수도 있습니다.
~~~~
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ('id', 'account_name', 'users', 'created')
        depth = 1
~~~~
<tt style="color: #FF0000">`depth`</tt> 옵션은 flat representation으로 되돌리기 전에 탐색해야하는 관계의 depth를 나타내는 정수 값으로 설정해야합니다.
serialization가 수행되는 방식을 customize하려면 field를 직접 정의해야합니다.


## Specifying fields explicitly

<tt style="color: #FF0000">`ModelSerializer`</tt>에 extra fields를 추가하거나 <tt style="color: #FF0000">`Serializer`</tt> 클래스에서와 마찬가지로 클래스의 fields를 선언하여 기본 fields를 재정의 할 수 있습니다.
~~~~
class AccountSerializer(serializers.ModelSerializer):
    url = serializers.CharField(source='get_absolute_url', read_only=True)
    groups = serializers.PrimaryKeyRelatedField(many=True)

    class Meta:
        model = Account
~~~~
extra fields는 모델의 모든 property 또는 호출 가능 항목에 해당 할 수 있습니다.


## Specifying read only fields

여러 fields를 read-only로 지정할 수 있습니다. 각 필드를 <tt style="color: #FF0000">`read_only=True`</tt> attribute으로 명시적으로 추가하는 대신, 바로 가기 메타 옵션 인 <tt style="color: #FF0000">`read_only_fields`</tt>를 사용할 수 있습니다.
이 옵션은 field names의 list 또는 tuple 이어야하며 다음과 같이 선언됩니다.
~~~~
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ('id', 'account_name', 'users', 'created')
        read_only_fields = ('account_name',)
~~~~
<tt style="color: #FF0000">`editable=False`</tt> set 및 <tt style="color: #FF0000">`AutoField`</tt> fields가있는 모델 fields는 default로 read-only으로 설정되며 <tt style="color: #FF0000">`read_only_fields`</tt> 옵션에 추가 할 필요가 없습니다.


참고 :
read-only field가 모델 수준의 <tt style="color: #FF0000">`unique_together`</tt> 제약 조건의 일부인 특별한 경우가 있습니다. 이 경우 field는 제약 조건의 validate을 검사하기 위해 serializer 클래스에서 필요하지만 사용자가 편집 할 수 없도록 해야합니다.
이를 처리하는 올바른 방법은 <tt style="color: #FF0000">`read_only=True`</tt> 및 <tt style="color: #FF0000">`default=...`</tt> 키워드 arguments를 제공하여 serializer에서 필드를 명시적으로 지정하는 것입니다.
예로 <tt style="color: #FF0000">`unique_together`</tt> 다른 식별자와 구분된 currently authenticated 된 <tt style="color: #FF0000">`User`</tt>에 대한 read-only 관계입니다. 이 경우 사용자 field를 다음과 같이 선언합니다.
~~~~
user = serializers.PrimaryKeyRelatedField(read_only=True, default=serializers.CurrentUserDefault())
~~~~
* [Validators Documentation](http://www.django-rest-framework.org/api-guide/validators/)
* [UniqueTogetherValidator](http://www.django-rest-framework.org/api-guide/validators/#uniquetogethervalidator)
* [CurrentUserDefault](http://www.django-rest-framework.org/api-guide/validators/#currentuserdefault)


## Additional keyword arguments

또한 <tt style="color: #FF0000">`extra_kwargs`</tt> 옵션을 사용하여 fields에 임의의 추가 키워드 arguments를 지정할 수 있는 shortcut이 있습니다. <tt style="color: #FF0000">`read_only_fields`</tt>의 경우와 마찬가지로, 이것은 serializer에서 필드를 명시적으로 선언 할 필요가 없음을 의미합니다.
이 옵션은 field 이름을 키워드 arguments dictionary에 매핑하는 dictionary입니다. 예 :
~~~~
class CreateUserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ('email', 'username', 'password')
        extra_kwargs = {'password': {'write_only': True}}

    def create(self, validated_data):
        user = User(
            email=validated_data['email'],
            username=validated_data['username']
        )
        user.set_password(validated_data['password'])
        user.save()
        return user
~~~~


## Relational fields

모델 인스턴스를 serializing 할 때 관계를 나타내기 위해 선택할 수있는 여러 가지 방법이 있습니다. `ModelSerializer`의 default representation은 관련 인스턴스의 primary keys를 사용하는 것입니다.
다른 representations은 하이퍼링크를 사용하여 serializing, complete nested representations을 serializing하거나 custom representation을 사용하여 serializing하는 것을 포함합니다.
[serializer relations documentation](http://www.django-rest-framework.org/api-guide/relations/)


## Customizing field mappings

ModelSerializer 클래스는 serializer를 인스턴스화 할 때 serializer fields가 자동으로 결정되는 방식을 변경하기 위해 재정의 할 수 있는 API도 제공합니다.
일반적으로 <tt style="color: #FF0000">`ModelSerializer`</tt>가 기본적으로 필요한 fields를 생성하지 않으면 클래스에 명시적으로 추가하거나 단순히 대신에 일반 <tt style="color: #FF0000">`Serializer`</tt> 클래스를 사용해야합니다. 그러나 경우에 따라 특정 모델에 대해 serializer fields가 생성되는 방식을 정의하는 새 기본 클래스를 만들 수도 있습니다.
~~~~
.serializer_field_mapping
~~~~
Django 모델 클래스와 REST framework serializer 클래스의 매핑. 이 맵핑을 겹쳐 쓰면 각 모델 클래스에 사용해야하는 default serializer 클래스를 변경할 수 있습니다.
~~~~
.serializer_related_field
~~~~
이 property은 default로 관계형 fields에 사용되는 serializer fields 클래스 여야합니다.
<tt style="color: #FF0000">`ModelSerializer`</tt>의 경우 기본값은 <tt style="color: #FF0000">`PrimaryKeyRelatedField`</tt>입니다.
<tt style="color: #FF0000">`HyperlinkedModelSerializer`</tt>의 경우 기본값은 <tt style="color: #FF0000">`serializers.HyperlinkedRelatedField`</tt>입니다.
~~~~
serializer_url_field
~~~~
serializer의 <tt style="color: #FF0000">`url`</tt> field에 사용해야하는 serializer field 클래스입니다.
<tt style="color: #FF0000">`serializers.HyperlinkedIdentityField`</tt>가 기본값입니다.
~~~~
serializer_choice_field
~~~~
serializer의 선택 field에 사용해야하는 serializer field 클래스입니다.
<tt style="color: #FF0000">`serializers.ChoiceField`</tt>가 기본값입니다.

### The field_class and field_kwargs API

다음 메서드는 serializer에 자동으로 포함되어야하는 각 field의 클래스 및 키워드 arguments를 결정하기 위해 호출됩니다. 이 메소드들 각각은 <tt style="color: #FF0000">`(field_class, field_kwargs)`</tt>의 두 tuple을 반환해야합니다.
~~~~
.build_standard_field(self, field_name, model_field)
~~~~
표준 모델 field에 매핑되는 serializer field를 생성하기 위해 호출됩니다.
default implementation는 <tt style="color: #FF0000">`serializer_field_mapping`</tt> attribute에 근거한 serializer 클래스를 반환합니다.
~~~~
.build_relational_field(self, field_name, relation_info)
~~~~
관계형 모델 field에 매핑되는 serializer field를 생성하기 위해 호출됩니다.
default implementation는 <tt style="color: #FF0000">`serializer_relational_field`</tt> attribute에 근거한 serializer 클래스를 반환합니다.
<tt style="color: #FF0000">`relation_info`</tt> argument는 명명 된 tuple이며 <tt style="color: #FF0000">`model_field`</tt>, <tt style="color: #FF0000">`related_model`</tt>, <tt style="color: #FF0000">`to_many`</tt> 및 <tt style="color: #FF0000">`has_through_model`</tt> properties를 포함합니다.
~~~~
.build_nested_field(self, field_name, relation_info, nested_depth)
~~~~
<tt style="color: #FF0000">`depth`</tt> 옵션이 설정된 경우 관계형 모델 field에 매핑되는 serializer field를 생성하기 위해 호출됩니다.
default implementation는 <tt style="color: #FF0000">`ModelSerializer`</tt> 또는 <tt style="color: #FF0000">`HyperlinkedModelSerializer`</tt>를 기반으로 nested된 serializer 클래스를 dynamically로 만듭니다.
<tt style="color: #FF0000">`nested_depth`</tt>는 <tt style="color: #FF0000">`depth`</tt> 옵션의 value에서 1을 뺀 값입니다.
<tt style="color: #FF0000">`relation_info`</tt> argument는 명명 된 tuple이며 <tt style="color: #FF0000">`model_field`</tt>, <tt style="color: #FF0000">`related_model`</tt>, <tt style="color: #FF0000">`to_many`</tt> 및 <tt style="color: #FF0000">`has_through_model`</tt> properties을 포함합니다.
~~~~
.build_property_field(self, field_name, model_class)
~~~~
모델 클래스의 속성 또는 zero-argument 메서드에 매핑되는 serializer field를 생성하기 위해 호출됩니다.
default implementation는 <tt style="color: #FF0000">`ReadOnlyField`</tt> 클래스를 반환합니다.
~~~~
.build_url_field(self, field_name, model_class)
~~~~
serializer 자신의 <tt style="color: #FF0000">`url`</tt> field에 대한 serializer field를 생성하기 위해 호출됩니다. default implementation은 <tt style="color: #FF0000">`HyperlinkedIdentityField`</tt> 클래스를 반환합니다.
~~~~
.build_unknown_field(self, field_name, model_class)
~~~~
field 이름이 모델 field 또는 모델 property에 매핑되지 않았을 때 호출됩니다. subclasses에 의해 이 동작을 customize해도, default implementation에서는 error가 발생합니다.


## HyperlinkedModelSerializer

<tt style="color: #FF0000">`HyperlinkedModelSerializer`</tt> 클래스는 primary keys가 아닌 관계를 나타 내기 위해 하이퍼링크를 사용한다는 점을 제외하고는 <tt style="color: #FF0000">`ModelSerializer`</tt> 클래스와 유사합니다.
기본적으로 serializer에는 primary key field 대신 <tt style="color: #FF0000">`url`</tt> field가 포함됩니다.
url field는 <tt style="color: #FF0000">`HyperlinkedIdentityField`</tt> serializer field를 사용하여 표시되며 모델의 모든 관계는 <tt style="color: #FF0000">`HyperlinkedRelatedField`</tt> serializer field를 사용하여 표시됩니다.
primary key를 <tt style="color: #FF0000">`fields`</tt> 옵션에 추가하여 명시적으로 포함시킬 수 있습니다. 예 :
~~~~
class AccountSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Account
        fields = ('url', 'id', 'account_name', 'users', 'created')
~~~~


## Absolute and relative URLs

<tt style="color: #FF0000">`HyperlinkedModelSerializer`</tt>를 인스턴스화 할 때는 현재 <tt style="color: #FF0000">`request`</tt>를 serializer 컨텍스트에 포함해야합니다. 예 :
~~~~
serializer = AccountSerializer(queryset, context={'request': request})
~~~~
이렇게하면 하이퍼링크에 적절한 호스트 이름이 포함될 수 있으므로 결과 representation은 다음과 같은 정규화 된 URL을 사용합니다.
~~~~
http://api.example.com/accounts/1/
~~~~
다음과 같은 상대 URL이 아닙니다.
~~~~
/accounts/1/
~~~~
상대 URL을 사용하려면 serializer 컨텍스트에서 <tt style="color: #FF0000">`{'request': None}`</tt>을 명시적으로 전달해야합니다.


## How hyperlinked views are determined

모델 인스턴스에 하이퍼링크하기 위해 어떤 뷰를 사용해야 하는지 결정할 수 있는 방법이 필요합니다.
기본적으로 하이퍼링크는 <tt style="color: #FF0000">`'{model_name}-detail'`</tt>스타일과 일치하는 보기 이름과 일치해야하며 <tt style="color: #FF0000">`pk`</tt> 키워드 argument로 인스턴스를 찾습니다.
다음과 같이 <tt style="color: #FF0000">`extra_kwargs`</tt> 설정에서 <tt style="color: #FF0000">`view_name`</tt> 및 <tt style="color: #FF0000">`lookup_field`</tt> 옵션 중 하나 또는 둘 모두를 사용하여 URL field view 이름 및 lookup field를 무시할 수 있습니다.
~~~~
class AccountSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Account
        fields = ('account_url', 'account_name', 'users', 'created')
        extra_kwargs = {
            'url': {'view_name': 'accounts', 'lookup_field': 'account_name'},
            'users': {'lookup_field': 'username'}
        }
~~~~
또는 serializer에서 fields를 명시적으로 설정할 수 있습니다. 예 :
~~~~
class AccountSerializer(serializers.HyperlinkedModelSerializer):
    url = serializers.HyperlinkedIdentityField(
        view_name='accounts',
        lookup_field='slug'
    )
    users = serializers.HyperlinkedRelatedField(
        view_name='user-detail',
        lookup_field='username',
        many=True,
        read_only=True
    )

    class Meta:
        model = Account
        fields = ('url', 'account_name', 'users', 'created')
~~~~
Tip :
하이퍼링크로 표시된 representations과 URL conf를 적절하게 일치시키는 것은 때로는 약간의 실수 일 수 있습니다. <tt style="color: #FF0000">`HyperlinkedModelSerializer`</tt> 인스턴스의 <tt style="color: #FF0000">`repr`</tt>을 출력하는 것은 관계가 매핑 할 것으로 예상되는 뷰 이름과 lookup fields 를 정확하게 검사하는 데 특히 유용합니다.


## Changing the URL field name

URL 입력 field의 이름은 'url'로 defaults 설정됩니다. <tt style="color: #FF0000">`URL_FIELD_NAME`</tt> 설정을 사용하여 이 값을 전역적으로 재정의 할 수 있습니다.


## ListSerializer

<tt style="color: #FF0000">`ListSerializer`</tt> 클래스는 여러 개체를 한 번에 serialize하고 validating하는 동작을 제공합니다. 일반적으로 <tt style="color: #FF0000">`ListSerializer`</tt>를 직접 사용할 필요는 없지만 대신 serializer를 인스턴스화 할 때 <tt style="color: #FF0000">`many=True`</tt>를 전달해야합니다.
serializer가 인스턴스화되고 <tt style="color: #FF0000">`many=True`</tt>가 전달되면 <tt style="color: #FF0000">`ListSerializer`</tt> 인스턴스가 만들어집니다. 그런 다음 serializer 클래스는 부모 <tt style="color: #FF0000">`ListSerializer`</tt>의 자식이됩니다.
다음 인수는 <tt style="color: #FF0000">`ListSerializer`</tt> field나 <tt style="color: #FF0000">`many=True`</tt>로 전달되는 serializer에도 전달할 수 있습니다.
~~~~
allow_empty
~~~~
이것은 default로 <tt style="color: #FF0000">`True`</tt>이지만 빈 입력을 유효한 입력으로 허용하지 않으려면 <tt style="color: #FF0000">'False'</tt>로 설정할 수 있습니다.


## Customizing <tt style="color: #FF0000">`ListSerializer`</tt> behavior

<tt style="color: #FF0000">`ListSerializer`</tt> 동작을 사용자 정의하는 경우가 몇 가지 있습니다. 예 :
* 특정 요소가 목록의 다른 요소와 충돌하지 않는지 확인하는 등 목록의 particular validation를 제공합니다.
* 여러 객체의 작성 또는 업데이트 동작을 사용자 정의합니다.
이 경우 serializer <tt style="color: #FF0000">`Meta`</tt> 클래스에서 <tt style="color: #FF0000">`list_serializer_class`</tt> 옵션을 사용하여 <tt style="color: #FF0000">`many=True`</tt>가 전달될 때 사용되는 클래스를 수정할 수 있습니다.
예 :

~~~~
class CustomListSerializer(serializers.ListSerializer):
    ...

class CustomSerializer(serializers.Serializer):
    ...
    class Meta:
        list_serializer_class = CustomListSerializer
~~~~

### Customizing multiple create

여러 객체 생성을 위한 default implementation는 목록의 각 item에 대해 <tt style="color: #FF0000">`.create()`</tt>를 호출하는 것입니다. 이 동작을 customize하려면 <tt style="color: #FF0000">`many=True`</tt>가 전달 될 때 사용되는 <tt style="color: #FF0000">`ListSerializer`</tt> 클래스에서 <tt style="color: #FF0000">`.create()`</tt> 메서드를 customize해야합니다.
예 :

~~~~
class BookListSerializer(serializers.ListSerializer):
    def create(self, validated_data):
        books = [Book(**item) for item in validated_data]
        return Book.objects.bulk_create(books)

class BookSerializer(serializers.Serializer):
    ...
    class Meta:
        list_serializer_class = BookListSerializer
~~~~

### Customizing multiple update

기본적으로 <tt style="color: #FF0000">`ListSerializer`</tt> 클래스는 여러 업데이트를 지원하지 않습니다. 이는 삽입 및 삭제에 대해 예상되는 동작이 모호하기 때문입니다.
여러 업데이트를 지원하려면 명시적으로 업데이트해야 합니다. 여러 개의 업데이트 코드를 작성하는 경우 다음 사항에 유의하십시오.
* 데이터 목록의 각 item에 대해 어떤 인스턴스를 업데이트해야 하는지 어떻게 결정합니까?
* 삽입을 어떻게 처리해야합니까? 그것들은 유효하지 않습니까? 아니면 새로운 객체를 만드나요?
* removals은 어떻게 처리해야합니까? 객체 삭제 또는 관계 제거를 의미합니까? 그들은 무시해야합니까, 아니면 무효합니까?
* ordering은 어떻게 처리해야합니까? 두 항목의 position 변경은 상태 변경을 의미합니까 아니면 무시됩니까?
인스턴스 serializer에 명시적 <tt style="color: #FF0000">`id`</tt> field를 추가해야합니다. default implicitly-generate된 <tt style="color: #FF0000">`id`</tt> field는 <tt style="color: #FF0000">`read_only`</tt>로 표시됩니다. 이로 인해 업데이트시 제거됩니다. 명시적으로 선언하면 serializer의 <tt style="color: #FF0000">`update`</tt> 메소드 목록에서 사용할 수 있습니다.
다음은 여러 업데이트를 구현하는 방법에 대한 예입니다.

~~~~
class BookListSerializer(serializers.ListSerializer):
    def update(self, instance, validated_data):
        # Maps for id->instance and id->data item.
        book_mapping = {book.id: book for book in instance}
        data_mapping = {item['id']: item for item in validated_data}

        # Perform creations and updates.
        ret = []
        for book_id, data in data_mapping.items():
            book = book_mapping.get(book_id, None)
            if book is None:
                ret.append(self.child.create(data))
            else:
                ret.append(self.child.update(book, data))

        # Perform deletions.
        for book_id, book in book_mapping.items():
            if book_id not in data_mapping:
                book.delete()

        return ret

class BookSerializer(serializers.Serializer):
    # We need to identify elements in the list using their primary key,
    # so use a writable field here, rather than the default which would be read-only.
    id = serializers.IntegerField()

    ...
    id = serializers.IntegerField(required=False)

    class Meta:
        list_serializer_class = BookListSerializer
~~~~
third party package가 REST framework 2에있는 <tt style="color: #FF0000">`allow_add_remove`</tt> 동작과 유사하게 여러 업데이트 작업에 대한 자동 지원을 제공하는 3.1 릴리스에 함께 포함될 수 있습니다.

### Customizing ListSerializer initialization

<tt style="color: #FF0000">`many=True`</tt>가 있는 serializer가 인스턴스화되면 자식 <tt style="color: #FF0000">`Serializer`</tt> 클래스와 상위 <tt style="color: #FF0000">`ListSerializer`</tt> 클래스 모두에 대해 <tt style="color: #FF0000">`.__ init __()`</tt> 메서드에 전달할 arguments 및 키워드 arguments를 결정해야 합니다.
default implementation은, <tt style="color: #FF0000">`validator`</tt>를 제외하고, 양쪽 모두의 클래스에 모든 arguments, custom 키워드 arguments를 건네줍니다. 양쪽 모두 자식 serializer 클래스를 대상으로 하고 있습니다.
때로 <tt style="color: #FF0000">`many=True`</tt>가 전달 될 때 자식 및 부모 클래스의 인스턴스화 방법을 명시적으로 지정해야 할 수도 있습니다. <tt style="color: #FF0000">`many_init`</tt> 클래스 메소드를 사용하면 그렇게 할 수 있습니다.
~~~~
@classmethod
    def many_init(cls, *args, **kwargs):
        # Instantiate the child serializer.
        kwargs['child'] = cls()
        # Instantiate the parent list serializer.
        return CustomListSerializer(*args, **kwargs)
~~~~


## BaseSerializer

대체 serialization 및 deserialization 스타일을 쉽게 지원하는 데 사용할 수 있는 <tt style="color: #FF0000">`BaseSerializer`</tt> 클래스입니다.
이 클래스는 <tt style="color: #FF0000">`Serializer`</tt> 클래스와 같은 기본 API를 구현합니다.
* <tt style="color: #FF0000">`.data`</tt> - primitive representation 결과를 반환합니다.
* <tt style="color: #FF0000">`.is_valid()`</tt> - incoming data를 Deserializes, validates 합니다.
* <tt style="color: #FF0000">`.validated_data`</tt> - validated incoming data를 반환합니다.
* <tt style="color: #FF0000">`.errors`</tt> - validation 동안 발생한 error를 반환합니다.
* <tt style="color: #FF0000">`.save()`</tt> - 객체 인스턴스에 validated data를 유지합니다.

serializer 클래스에서 지원할 기능에 따라 무시(기각)할 수 있는 네 가지 메서드가 있습니다.
* <tt style="color: #FF0000">`.to_representation()`</tt> - read operations를 하기 위해 이 support serialization를 무시(기각)할 수 있습니다.
* <tt style="color: #FF0000">`.to_internal_value()`</tt> - write operations를 하기 위해 이 support serialization를 무시(기각)할 수 있습니다.
* <tt style="color: #FF0000">`.create()`</tt>, <tt style="color: #FF0000">`.update()`</tt> - support saving instances를 하기 위해 2 메소드 모두 무시(기각)할 수 있습니다.

이 클래스는 <tt style="color: #FF0000">`Serializer`</tt> 클래스와 동일한 인터페이스를 제공하기 때문에 일반 <tt style="color: #FF0000">`Serializer`</tt> 또는 <tt style="color: #FF0000">`ModelSerializer`</tt>에서 사용하던 것과 똑같이 generic class-based 뷰와 함께 사용할 수 있습니다.
<tt style="color: #FF0000">`BaseSerializer`</tt> 클래스는 browsable  API에서 HTML 양식을 생성하지 않는다는 점이 유일한 차이입니다. 이는 반환하는 데이터에 각 field를 적절한 HTML 입력으로 렌더링 할 수있는 모든 필드 정보를 포함하지 않기 때문입니다.

#### Read-only <b>`BaseSerializer`</b> classes

<tt style="color: #FF0000">`BaseSerializer`</tt> 클래스를 사용하여 read-only <tt style="color: #FF0000">`serializer`</tt>를 implement하려면 <tt style="color: #FF0000">`.to_representation()`</tt> 메서드를 재정의해야합니다. 간단한 Django 모델을 사용하는 예제를 살펴 보겠습니다.
~~~~
class HighScore(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    player_name = models.CharField(max_length=10)
    score = models.IntegerField()
~~~~
<tt style="color: #FF0000">`HighScore`</tt> 인스턴스를 primitive 데이터 유형으로 변환하기위한 읽기 read-only serializer 변환기를 만드는 것은 간단합니다.
~~~~
class HighScoreSerializer(serializers.BaseSerializer):
    def to_representation(self, obj):
        return {
            'score': obj.score,
            'player_name': obj.player_name
        }
~~~~
이제 이 클래스를 사용하여 단일 <tt style="color: #FF0000">`HighScore`</tt> 인스턴스를 serialize 할 수 있습니다.
~~~~
@api_view(['GET'])
def high_score(request, pk):
    instance = HighScore.objects.get(pk=pk)
    serializer = HighScoreSerializer(instance)
    return Response(serializer.data)
~~~~
또는 이를 사용하여 여러 인스턴스를 serialize 할 수 있습니다.
~~~~
@api_view(['GET'])
def all_high_scores(request):
    queryset = HighScore.objects.order_by('-score')
    serializer = HighScoreSerializer(queryset, many=True)
    return Response(serializer.data)
~~~~

#### Read-write <b>`BaseSerializer`</b> classes

read-write serializer를 만들려면 먼저 <tt style="color: #FF0000">`.to_internal_value()`</tt> 메서드를 implement해야 합니다. 이 메서드는 객체 인스턴스를 구성하는 데 사용될 validated values을 반환하고 제공된 데이터가 잘못된 형식인 경우 <tt style="color: #FF0000">`ValidationError`</tt>를 발생시킬 수 있습니다.
<tt style="color: #FF0000">`.to_internal_value()`</tt>를 implement하면 serializer에서 basic validation API를 사용할 수 있으며 <tt style="color: #FF0000">`.is_valid()`</tt>, <tt style="color: #FF0000">`.validated_data`</tt> 및 <tt style="color: #FF0000">`.errors`</tt>를 사용할 수 있습니다.
<tt style="color: #FF0000">`.save()`</tt>도 지원하려면 <tt style="color: #FF0000">`.create()`</tt> 및 <tt style="color: #FF0000">`.update()`</tt> 메소드 중 하나 또는 모두를 구현해야합니다.
다음은 읽기 및 쓰기 operations을 모두 지원하도록 업데이트 된 이전의 <tt style="color: #FF0000">`HighScoreSerializer`</tt>의 완성된 예제입니다.
~~~~
class HighScoreSerializer(serializers.BaseSerializer):
    def to_internal_value(self, data):
        score = data.get('score')
        player_name = data.get('player_name')

        # Perform the data validation.
        if not score:
            raise ValidationError({
                'score': 'This field is required.'
            })
        if not player_name:
            raise ValidationError({
                'player_name': 'This field is required.'
            })
        if len(player_name) > 10:
            raise ValidationError({
                'player_name': 'May not be more than 10 characters.'
            })

        # Return the validated values. This will be available as
        # the `.validated_data` property.
        return {
            'score': int(score),
            'player_name': player_name
        }

    def to_representation(self, obj):
        return {
            'score': obj.score,
            'player_name': obj.player_name
        }

    def create(self, validated_data):
        return HighScore.objects.create(**validated_data)
~~~~

### Creating new base classes

<tt style="color: #FF0000">`BaseSerializer`</tt> 클래스는 특정 serialization 스타일을 처리하거나 alternative storage backends와 통합하기 위해 새 generic serializer 클래스를 implement하려는 경우에도 유용합니다.
다음 클래스는 임의의 객체를 primitive representations으로 강제 변환 할 수 있는 generic serializer의 예입니다.
~~~~
class ObjectSerializer(serializers.BaseSerializer):
    """
    A read-only serializer that coerces arbitrary complex objects
    into primitive representations.
    """
    def to_representation(self, obj):
        for attribute_name in dir(obj):
            attribute = getattr(obj, attribute_name)
            if attribute_name('_'):
                # Ignore private attributes.
                pass
            elif hasattr(attribute, '__call__'):
                # Ignore methods and other callables.
                pass
            elif isinstance(attribute, (str, int, bool, float, type(None))):
                # Primitive types can be passed through unmodified.
                output[attribute_name] = attribute
            elif isinstance(attribute, list):
                # Recursively deal with items in lists.
                output[attribute_name] = [
                    self.to_representation(item) for item in attribute
                ]
            elif isinstance(attribute, dict):
                # Recursively deal with items in dictionaries.
                output[attribute_name] = {
                    str(key): self.to_representation(value)
                    for key, value in attribute.items()
                }
            else:
                # Force anything else to its string representation.
                output[attribute_name] = str(attribute)
~~~~


# Advanced serializer usage

## Overriding serialization and deserialization behavior

serializer 클래스의 serialization, deserialization 또는 validation를 변경해야하는 경우 <tt style="color: #FF0000">`.to_representation()`</tt> 또는 <tt style="color: #FF0000">`.to_internal_value()`</tt> 메서드를 재정의하여 그렇게 할 수 있습니다.
이것이 유용한 이유는 다음과 같습니다.
* 새로운 serializer base classes에 대한 새로운 behavior 추가.
* 기존 클래스의 behavior를 약간 수정합니다.
* 많은 양의 데이터를 반환하는 자주 액세스되는 API endpoint의 serialization 성능 향상.

이 메소드의 signatures은 다음과 같습니다.
~~~~
.to_representation(self, obj)
~~~~
serialization가 필요한 객체 인스턴스를 가져 와서 primitive representation을 반환해야합니다. 일반적으로 이것은 built-in Python datatypes의 구조를 반환하는 것을 의미합니다. 처리 할 수 있는 정확한 유형은 API에 대해 구성한 렌더링 클래스에 따라 다릅니다.
~~~~
.to_internal_value(self, data)
~~~~
unvalidated incoming data를 입력 받아 <tt style="color: #FF0000">`erializer.validated_data`</tt>로 사용할 수 있는 validated data를 반환해야합니다. serializer 클래스에서 <tt style="color: #FF0000">`.save()`</tt>가 호출되면 반환 value도 <tt style="color: #FF0000">`.create()`</tt> 또는 <tt style="color: #FF0000">`.update()`</tt> 메서드에 전달됩니다.
유효성 검사가 실패하면 메서드는 <tt style="color: #FF0000">`serializers.ValidationError(errors)`</tt>를 발생시켜야합니다. 일반적으로 여기에있는 <tt style="color: #FF0000">`errors`</tt> argument는 field 이름을 오류 메시지에 매핑하는 dictionary입니다.
이 메소드에 전달 된 <tt style="color: #FF0000">`data`</tt> argument는 일반적으로 <tt style="color: #FF0000">`request.data`</tt>의 값이므로 제공하는 datatype은 API에 대해 구성한 parser 클래스에 따라 다릅니다.


## Serializer Inheritance

Django forms과 마찬가지로 상속을 통해 serializer를 확장하고 다시 사용할 수 있습니다. 이를 통해 많은 수의 serializer에서 사용할 수있는 부모 클래스의 공통 fields 또는 메서드 집합을 선언 할 수 있습니다. 예를 들어,
~~~~
class MyBaseSerializer(Serializer):
    my_field = serializers.CharField()

    def validate_my_field(self):
        ...

class MySerializer(MyBaseSerializer):
    ...
~~~~
Django의 <tt style="color: #FF0000">`Model`</tt>과 <tt style="color: #FF0000">`ModelForm`</tt> 클래스와 마찬가지로, serializer의 내부 <tt style="color: #FF0000">`Meta`</tt> 클래스는 부모의 내부 메타 클래스를 암시 적으로 상속받지 않습니다. <tt style="color: #FF0000">`Meta`</tt> 클래스가 부모 클래스에서 상속 받기를 원하면 명시적으로 그렇게해야합니다. 예 :
~~~~
class AccountSerializer(MyBaseSerializer):
    class Meta(MyBaseSerializer.Meta):
        model = Account
~~~~
일반적으로 내부 메타 클래스에서는 상속을 사용하지 않고 모든 옵션을 명시적으로 선언하는 것이 좋습니다.
또한 다음과 같은 주의 사항이 serializer 상속에 적용됩니다.
* 일반적인 Python name resolution rules가 적용됩니다. <tt style="color: #FF0000">`Meta`</tt> 내부 클래스를 선언하는 여러 기본 클래스가 있는 경우 첫 번째 클래스만 사용됩니다. 이것은 자녀의 <tt style="color: #FF0000">`Meta`</tt>가 존재한다면 메타를 의미하고, 그렇지 않으면 첫 번째 부모의 <tt style="color: #FF0000">`Meta`</tt> 등을 의미합니다.
* 하위 클래스에서 이름을 <tt style="color: #FF0000">`None`</tt>으로 설정하여 부모 클래스에서 상속 된 <tt style="color: #FF0000">`Field`</tt>를 선언적으로 제거 할 수 있습니다.

~~~~
class MyBaseSerializer(ModelSerializer):
    my_field = serializers.CharField()

class MySerializer(MyBaseSerializer):
    my_field = None
~~~~

그러나 이 방법을 사용하는 경우에만 상위 클래스에 의해 선언적으로 정의된 field에서 opt-out 할 수 있습니다. <tt style="color: #FF0000">`ModelSerializer`</tt>가 디폴트 필드를 생성하는 것을 막지는 않습니다. 기본 입력란을 opt-out하려면 [포함 할 입력란 지정을 참조하십시오.](http://www.django-rest-framework.org/api-guide/serializers/#specifying-which-fields-to-include)


## Dynamically modifying fields


serializer가 초기화되면 serializer에서 설정된 fields dictionary에 <tt style="color: #FF0000">`.fields`</tt> attribute을 사용하여 액세스 할 수 있습니다. 이 attribute에 액세스하고 수정하면 serializer를 동적으로 수정할 수 있습니다.
<tt style="color: #FF0000">`fields`</tt> argument를 직접 수정하면 serializer 선언 시점이 아닌 runtime시 serializer fields의 argument 변경과 같은 흥미로운 작업을 수행 할 수 있습니다.

## 예시

예를 들어, serializer에서 초기화 할 때 사용할 fields를 설정하려면 다음과 같이 serializer 클래스를 만들 수 있습니다.
~~~~
class DynamicFieldsModelSerializer(serializers.ModelSerializer):
    """
    A ModelSerializer that takes an additional `fields` argument that
    controls which fields should be displayed.
    """

    def __init__(self, *args, **kwargs):
        # Don't pass the 'fields' arg up to the superclass
        fields = kwargs.pop('fields', None)

        # Instantiate the superclass normally
        super(DynamicFieldsModelSerializer, self).__init__(*args, **kwargs)

        if fields is not None:
            # Drop any fields that are not specified in the `fields` argument.
            allowed = set(fields)
            existing = set(self.fields.keys())
            for field_name in existing - allowed:
                self.fields.pop(field_name)
~~~~
이렇게하면 다음을 수행 할 수 있습니다.
~~~~
class UserSerializer(DynamicFieldsModelSerializer):
     class Meta:
         model = User
         fields = ('id', 'username', 'email')

print UserSerializer(user)
{'id': 2, 'username': 'jonwatts', 'email': 'jon@example.com'}

print UserSerializer(user, fields=('id', 'email'))
{'id': 2, 'email': 'jon@example.com'}
~~~~


## Customizing the default fields

REST framework 2에서는 개발자가 <tt style="color: #FF0000">`ModelSerializer`</tt> 클래스가 default fields 집합을 자동으로 생성하는 방법을 재정의 할 수 있는 API를 제공했습니다.
이 API는 <tt style="color: #FF0000">`.get_field()`</tt>, <tt style="color: #FF0000">`.get_pk_field()`</tt> 와 다른 메소드들을 포함합니다.
serializer가 근본적으로 3.0으로 다시 디자인 되었기 때문에 이 API는 더 이상 존재하지 않습니다. 생성된 fields는 여전히 수정할 수 있지만 소스 코드를 참조해야하며, 변경 사항이 API의 비공개 비트에 해당하면 변경 될 수 있음을 알고 있어야합니다.


## Third party packages

다음의 third party packages가 이용 가능합니다.

### Django REST marshmallow

[django-rest-marshmallow](http://www.tomchristie.com/django-rest-marshmallow/) 패키지는 python [marshmallow](https://marshmallow.readthedocs.io/en/latest/) 라이브러리를 사용하여 serializers의 alternative implementation을 제공합니다. REST framework serializers와 동일한 API를 제공하며 일부 use-cases에서는 drop-in 대체로 사용할 수 있습니다.

### Serpy

[serpy](https://github.com/clarkduvall/serpy) 패키지는 속도 향상을 위해 만들어진 serializer의 alternative implementation입니다. Serpy은 복잡한 datatypes을 simple native types으로 serializes합니다. native types은 JSON 또는 필요한 다른 형식으로 쉽게 변환 할 수 있습니다.

### MongoengineModelSerializer

[django-rest-framework-mongoengine](https://github.com/umutbozkurt/django-rest-framework-mongoengine) 패키지는 <tt style="color: #FF0000">`MongoEngineModelSerializer`</tt> serializer 클래스를 제공하여 MongoDB를 Django REST framework의 저장소 계층으로 사용할 수 있도록 지원합니다.

### GeoFeatureModelSerializer

[django-rest-framework-gis](https://github.com/djangonauts/django-rest-framework-gis) 패키지는 GeoJSON을 읽기 및 쓰기 작업 모두에 지원하는 <tt style="color: #FF0000">`GeoFeatureModelSerializer`</tt> serializer 클래스를 제공합니다.

### HStoreSerializer

[django-rest-framework-hstore](https://github.com/djangonauts/django-rest-framework-hstore) 패키지는 [django-hstore](https://github.com/djangonauts/django-hstore) <tt style="color: #FF0000">`DictionaryField`</tt> 모델 필드와 <tt style="color: #FF0000">`schema-mode`</tt> feature을 지원하는 <tt style="color: #FF0000">`HStoreSerializer`</tt>를 제공합니다.

### Dynamic REST

[Dynamic-Rest](https://github.com/AltSchool/dynamic-rest) 패키지는 ModelSerializer 및 ModelViewSet 인터페이스를 확장하고 필터링, 정렬, serializer에서 정의한 모든 fields 및 관계를 포함 / 제외하는 API 쿼리 parameters를 추가합니다.

### Dynamic Fields Mixin

[drf-dynamic-fields](https://github.com/dbrgn/drf-dynamic-fields) 패키지는 serializer마다 fields를 URL parameter로 지정된 서브 세트로 동적으로 제한하기 위해 mixin을 제공합니다.

### DRF FlexFields

[drf-flex-fields](https://github.com/rsinger86/drf-flex-fields) 패키지는 ModelSerializer 및 ModelViewSet을 확장하여 fields를 동적으로 설정하고 기본 필드를 nested 모델로 확장하는 데 일반적으로 사용되는 기능을 URL parameters 및 serializer 클래스 정의에서 모두 제공합니다.

### Serializer Extensions

[django-rest-framework-serializer-extensions](https://github.com/evenicoulddoit/django-rest-framework-serializer-extensions) 패키지는 보기 / 요청 단위로 fields를 정의 할 수 있게 하여 serializers를 DRY up 할 수 있는 도구 모음을 제공합니다. fields를 whitelisted나 blacklisted에 올릴 수 있으며 자식 serializer를 선택적으로 확장 할 수 있습니다.

### HTML JSON Forms

[html-json-forms](https://github.com/wq/html-json-forms) 패키지는 [HTML JSON Form specification](https://www.w3.org/TR/html-json-forms/)에 따라 <tt style="color: #FF0000">`<form>`</tt> submissions을 처리하는 algorithm 및 serializer를 제공합니다. serializer는 HTML 내에서 임의로 nested된 JSON 구조를 쉽게 처리합니다. 예를 들어 <tt style="color: #FF0000">`<input name = "items [0] [id]"value = "5">`</tt>는 <tt style="color: #FF0000">`{ "items": {{ "id": "5"}}}`</tt>로 해석됩니다.

### DRF-Base64

[DRF-Base64](https://bitbucket.org/levit_scs/drf_base64)는 base64 인코딩 파일의 업로드를 처리하는 일련의 field 및 모델 serializers를 제공합니다.

### QueryFields

[djangorestframework-queryfields](http://djangorestframework-queryfields.readthedocs.io/en/latest/)를 사용하면 API 클라이언트가 포함 / 제외 query parameters를 통해 response에서 어떤 fields를 보낼지 지정할 수 있습니다.

### DRF Writable Nested

[drf-writable-nested](https://github.com/Brogency/drf-writable-nested) 패키지는 nested된 관련 데이터로 모델을 작성 / 업데이트 할 수 있는 쓰기 가능한 nested 모델 serializer를 제공합니다.