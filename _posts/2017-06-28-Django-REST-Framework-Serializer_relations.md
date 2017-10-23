---
layout: post
title:  "Django REST Framework -Serializer_relations"
date:   2017-06-28 10:43:59
author: Dean Kim
categories: Rest_Framework
tags:	Django Rest_framework
cover:  "/assets/instacode.png"
---

# Serializer relations
- 원본 : [공식문서](http://www.django-rest-framework.org/api-guide/relations/)



나쁜 프로그래머는 코드에 대해 걱정합니다. 좋은 프로그래머는 데이터 구조와 그 관계에 대해 걱정합니다. - [Linus Torvalds](https://lwn.net/Articles/193245/)

관계 필드는 모델 관계를 나타내는데 사용됩니다. <tt style="color: #FF0000">`ForeignKey`</tt>, <tt style="color: #FF0000">`ManyToManyField`</tt> 및 <tt style="color: #FF0000">`OneToOneField`</tt> 관계는 물론 reverse 관계 및 <tt style="color: #FF0000">`GenericForeignKey`</tt>와 같은 custom 관계를 적용할 수 있습니다.

<b>Note:</b> 관계형 필드는 <tt style="color: #FF0000">`relations.py`</tt>에 선언되어 있지만, 관습에 따라 <tt style="color: #FF0000">`from rest_framework import serializer`</tt>을 이용해 <tt style="color: #FF0000">`serializer`</tt> 모듈에서 가져와야하며, <tt style="color: #FF0000">`serializers.<FieldName>`</tt>로 필드를 참조해야합니다.

#### Inspecting relationships.

<tt style="color: #FF0000">`ModelSerializer`</tt> 클래스를 사용하면 serializer 필드와 관계가 자동으로 생성됩니다. 이러한 자동 생성 필드를 Inspecting하는 것은 관계 스타일을 customize하는 방법을 결정하는데 유용한 도구가 될 수 있습니다.

이렇게하려면 Django shell을 열고 <tt style="color: #FF0000">`python manage.py shell`</tt>을 사용하고 serializer 클래스를 가져와서 인스턴스화하고 object representation을 print하십시오.
~~~~
>>> from myapp.serializers import AccountSerializer
>>> serializer = AccountSerializer()
>>> print repr(serializer)  # Or `print(repr(serializer))` in Python 3.x.
AccountSerializer():
    id = IntegerField(label='ID', read_only=True)
    name = CharField(allow_blank=True, max_length=100, required=False)
    owner = PrimaryKeyRelatedField(queryset=User.objects.all())
~~~~


## API Reference

다양한 유형의 관계 필드를 설명하기 위해 예제에 몇 가지 간단한 모델을 사용합니다. 우리 모델은 음악 앨범과 각 앨범에 수록된 트랙을 대상으로 합니다.
~~~~
class Album(models.Model):
    album_name = models.CharField(max_length=100)
    artist = models.CharField(max_length=100)

class Track(models.Model):
    album = models.ForeignKey(Album, related_name='tracks', on_delete=models.CASCADE)
    order = models.IntegerField()
    title = models.CharField(max_length=100)
    duration = models.IntegerField()

    class Meta:
        unique_together = ('album', 'order')
        ordering = ['order']

    def __unicode__(self):
        return '%d: %s' % (self.order, self.title)
~~~~

### StringRelatedField

<tt style="color: #FF0000">`StringRelatedField`</tt>는 <tt style="color: #FF0000">`__unicode__`</tt> 메소드를 사용하여 관계의 대상을 나타내는데 사용할 수 있습니다.
<br>예를 들어 다음 serializer와 같습니다.
~~~~
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.StringRelatedField(many=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
~~~~
다음과 같은 representation으로 serialize합니다.
~~~~
{
    'album_name': 'Things We Lost In The Fire',
    'artist': 'Low',
    'tracks': [
        '1: Sunflower',
        '2: Whitetail',
        '3: Dinosaur Act',
        ...
    ]
}
~~~~
이 필드는 읽기 전용입니다.
<br><b>Arguments:</b>
* <tt style="color: #FF0000">`many`</tt>: 일대 다 관계에 적용되려면 이 인수를 <tt style="color: #FF0000">`True`</tt>로 설정해야합니다.

### PrimaryKeyRelatedField

<tt style="color: #FF0000">`PrimaryKeyRelatedField`</tt>는 primary key를 사용하여 관계의 대상을 나타내는데 사용할 수 있습니다.
<br>예를 들어 다음 serializer와 같습니다.
~~~~
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.PrimaryKeyRelatedField(many=True, read_only=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
~~~~
다음과 같은 representation으로 serialize합니다.
~~~~
{
    'album_name': 'Undun',
    'artist': 'The Roots',
    'tracks': [
        89,
        90,
        91,
        ...
    ]
}
~~~~
default로 이 필드는 read-write이지만 <tt style="color: #FF0000">`read_only`</tt> 플래그를 사용하여이 동작을 변경할 수 있습니다.
<br><b>Arguments:</b>
* <tt style="color: #FF0000">`queryset`</tt>: field input의 validating할 때 모델 인스턴스 조회에 사용되는 쿼리 세트입니다. 관계는 명시적으로 쿼리 세트를 설정하거나 <tt style="color: #FF0000">`read_only=True`</tt>로 설정해야합니다.
* <tt style="color: #FF0000">`many`</tt>: 일대 다 관계에 적용되면 이 인수를 <tt style="color: #FF0000">`True`</tt>로 설정해야합니다.
* <tt style="color: #FF0000">`allow_null`</tt>: <tt style="color: #FF0000">`True`</tt>로 설정하면 필드에 <tt style="color: #FF0000">`None`</tt> 값 또는 null 허용 관계에 대한 빈 문자열을 허용합니다. Defaults는 <tt style="color: #FF0000">`False`</tt>입니다.
* <tt style="color: #FF0000">`pk_field`</tt>: 기본 키 값의 serialization/deserialization를 제어하는 ​​필드로 설정합니다. 예를 들어 <tt style="color: #FF0000">`pk_field=UUIDField(format='hex')`</tt>는 UUID primary key를 컴팩트 16 진수 representation으로 serialize합니다.

### HyperlinkedRelatedField

<tt style="color: #FF0000">`HyperlinkedRelatedField`</tt>는 하이퍼링크를 사용하여 관계의 대상을 나타내는데 사용할 수 있습니다.
<br>예를 들어 다음 serializer와 같습니다.
~~~~
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.HyperlinkedRelatedField(
        many=True,
        read_only=True,
        view_name='track-detail'
    )

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
~~~~
다음과 같은 representation으로 serialize합니다.
~~~~
{
    'album_name': 'Graceland',
    'artist': 'Paul Simon',
    'tracks': [
        'http://www.example.com/api/tracks/45/',
        'http://www.example.com/api/tracks/46/',
        'http://www.example.com/api/tracks/47/',
        ...
    ]
}
~~~~
default로 이 필드는 read-write이지만 <tt style="color: #FF0000">`read_only`</tt> 플래그를 사용하여이 동작을 변경할 수 있습니다.

<b>Note:</b> 이 필드는 <tt style="color: #FF0000">`lookup_field`</tt> 및 <tt style="color: #FF0000">`lookup_url_kwarg`</tt> 인수를 사용하여 설정된대로 단일 URL 키워드 인수를 허용하는 URL에 매핑되는 객체용으로 설계되었습니다.
<br>URL의 일부로 single primary key 또는 슬러그 인수가 포함된 URL에 적합합니다.
<br>좀 더 복잡한 하이퍼 링크 representation이 필요하다면 아래의 [custom hyperlinked fields](http://www.django-rest-framework.org/api-guide/relations/#custom-hyperlinked-fields) 섹션에 설명대로 필드를 customize해야합니다.
<br><b>Arguments:</b>
* <tt style="color: #FF0000">`view_name`</tt>: 관계의 대상으로 사용해야하는 뷰 이름. [the standard router classes](http://www.django-rest-framework.org/api-guide/routers/#defaultrouter)를 사용하는 경우 <tt style="color: #FF0000">`<modelname>-detail`</tt> 형식의 문자열이됩니다. <b>필수.</b>
* <tt style="color: #FF0000">`queryset`</tt>: field input을 validating할 때 모델 인스턴스 조회에 사용되는 쿼리 세트입니다. 관계는 명시적으로 쿼리 세트를 설정하거나 <tt style="color: #FF0000">`read_only=True`</tt>로 설정해야합니다.
* <tt style="color: #FF0000">`many`</tt>: 대다 관계에 적용되면 이 인수를 <tt style="color: #FF0000">`True`</tt>로 설정해야합니다.
* <tt style="color: #FF0000">`allow_null`</tt>: <tt style="color: #FF0000">`True`</tt>로 설정하면 필드에 <tt style="color: #FF0000">`None`</tt> 값 또는 null 허용 관계에 대한 빈 문자열을 허용합니다. Defaults는 <tt style="color: #FF0000">`False`</tt>입니다.
* <tt style="color: #FF0000">`lookup_field`</tt>: 조회에 사용해야하는 대상의 필드입니다. 참조된 뷰의 URL 키워드 인수에 correspond해야합니다. Default는 <tt style="color: #FF0000">`'pk'`</tt>입니다.
* <tt style="color: #FF0000">`lookup_url_kwarg`</tt>: 조회 필드에 해당하는 URL 구성 파일에 정의된 키워드 인수의 이름입니다. Defaults로 <tt style="color: #FF0000">`lookup_field`</tt>와 동일한 값을 사용합니다.
* <tt style="color: #FF0000">`format`</tt>: format suffixes를 사용하는 경우 하이퍼링크된 필드는 <tt style="color: #FF0000">`format`</tt> 인수를 사용하여 재정의하지 않는한 대상에 대해 동일한 format suffix를 사용합니다.

### SlugRelatedField

<tt style="color: #FF0000">`SlugRelatedField`</tt>는 대상의 필드를 사용하여 관계의 대상을 나타내는데 사용할 수 있습니다.
<br>예를 들어 다음 serializer와 같습니다.
~~~~
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.SlugRelatedField(
        many=True,
        read_only=True,
        slug_field='title'
     )

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
~~~~
다음과 같은 representation으로 serialize합니다.
~~~~
{
    'album_name': 'Dear John',
    'artist': 'Loney Dear',
    'tracks': [
        'Airport Surroundings',
        'Everything Turns to You',
        'I Was Only Going Out',
        ...
    ]
}
~~~~
기본적으로 이 필드는 read-write이지만 <tt style="color: #FF0000">`read_only`</tt> 플래그를 사용하여 이 동작을 변경할 수 있습니다.
<br><tt style="color: #FF0000">`SlugRelatedField`</tt>를 read-write 필드로 사용할 때는 일반적으로 슬러그 필드가 <tt style="color: #FF0000">`unique=True`</tt>인 모델 필드에 해당하는지 확인해야합니다.
<br><b>Arguments:</b>
* <tt style="color: #FF0000">`slug_field`</tt>: 타겟을 나타내는데 사용되어야 하는 필드. 주어진 인스턴스를 고유하게 식별하는 필드여야합니다. 예: <tt style="color: #FF0000">`username`</tt>. 필수
* <tt style="color: #FF0000">`queryset`</tt>: 필드 입력의 validating할 때 모델 인스턴스 조회에 사용되는 쿼리 세트입니다. 관계는 명시적으로 쿼리 세트를 설정하거나 <tt style="color: #FF0000">`read_only=True`</tt>로 설정해야합니다.
* <tt style="color: #FF0000">`many`</tt>: 대다 관계에 적용되려면 이 인수를 <tt style="color: #FF0000">`True`</tt>로 설정해야합니다.
* <tt style="color: #FF0000">`allow_null`</tt>: <tt style="color: #FF0000">`True`</tt>로 설정된 경우 필드는 <tt style="color: #FF0000">`None`</tt> 값 또는 null 허용 관계에 대한 빈 문자열을 허용합니다. Defaults는 <tt style="color: #FF0000">`False`</tt>입니다.

### HyperlinkedIdentityField

이 필드는 HyperlinkedModelSerializer의 <tt style="color: #FF0000">`'url'`</tt>필드와 같은 ID관계로 적용할 수 있습니다. 객체의 속성에도 사용할 수 있습니다. 다음 serializer를 예시로 참고하세요.
~~~~
class AlbumSerializer(serializers.HyperlinkedModelSerializer):
    track_listing = serializers.HyperlinkedIdentityField(view_name='track-list')

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'track_listing')
~~~~
다음과 같은 representation으로 serialize합니다.
~~~~
{
    'album_name': 'The Eraser',
    'artist': 'Thom Yorke',
    'track_listing': 'http://www.example.com/api/track_list/12/',
}
~~~~
이 필드는 항상 read-only입니다.
<br><b>Arguments:</b>
* <tt style="color: #FF0000">`view_name`</tt>: 관계의 대상으로 사용해야하는 뷰 이름. [the standard router classes](http://www.django-rest-framework.org/api-guide/routers/#defaultrouter)를 사용하는 경우 <tt style="color: #FF0000">`<model_name>-detail`</tt> 형식의 문자열이됩니다. 필수.
* <tt style="color: #FF0000">`lookup_field`</tt>: 조회에 사용해야하는 대상의 필드입니다. 참조된 뷰의 URL 키워드 인수에 해당해야합니다. Default는 <tt style="color: #FF0000">`'pk'`</tt>입니다.
* <tt style="color: #FF0000">`lookup_url_kwarg`</tt>: 조회 필드에 해당하는 URL 구성 파일에 정의된 키워드 인수의 이름입니다. Default는 <tt style="color: #FF0000">`lookup_field`</tt>와 동일한 값을 사용합니다.
* <tt style="color: #FF0000">`format`</tt>: 형식 접미어를 사용하는 경우 하이퍼 링크 된 필드는 <tt style="color: #FF0000">`format`</tt> 인수를 사용하여 재정의하지 않는 한 대상에 대해 동일한 format suffix를 사용합니다.


## Nested relationships
 
Nested 관계는 serializer를 필드로 사용하여 표현할 수 있습니다.
<br>필드가 다 대 관계를 나타내는데 사용되는 경우 serializer 필드에 <tt style="color: #FF0000">`many=True`</tt> 플래그를 추가해야합니다.

#### Example

다음 serializer를 예시로 참고하세요.
~~~~
class TrackSerializer(serializers.ModelSerializer):
    class Meta:
        model = Track
        fields = ('order', 'title', 'duration')

class AlbumSerializer(serializers.ModelSerializer):
    tracks = TrackSerializer(many=True, read_only=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
~~~~
다음과 같은 representation으로 serialize합니다.
~~~~
>>> album = Album.objects.create(album_name="The Grey Album", artist='Danger Mouse')
>>> Track.objects.create(album=album, order=1, title='Public Service Announcement', duration=245)
<Track: Track object>
>>> Track.objects.create(album=album, order=2, title='What More Can I Say', duration=264)
<Track: Track object>
>>> Track.objects.create(album=album, order=3, title='Encore', duration=159)
<Track: Track object>
>>> serializer = AlbumSerializer(instance=album)
>>> serializer.data
{
    'album_name': 'The Grey Album',
    'artist': 'Danger Mouse',
    'tracks': [
        {'order': 1, 'title': 'Public Service Announcement', 'duration': 245},
        {'order': 2, 'title': 'What More Can I Say', 'duration': 264},
        {'order': 3, 'title': 'Encore', 'duration': 159},
        ...
    ],
}
~~~~

### Writable nested serializers

기본적으로 nested serializer는 read-only입니다. nested serializer 필드에 대한 쓰기 작업을 지원하려면 <tt style="color: #FF0000">`create()`</tt> and/or <tt style="color: #FF0000">`update()`</tt> 메서드를 만들어 자식 관계를 저장하는 방법을 명시적으로 지정해야합니다.
~~~~
class TrackSerializer(serializers.ModelSerializer):
    class Meta:
        model = Track
        fields = ('order', 'title', 'duration')

class AlbumSerializer(serializers.ModelSerializer):
    tracks = TrackSerializer(many=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')

    def create(self, validated_data):
        tracks_data = validated_data.pop('tracks')
        album = Album.objects.create(**validated_data)
        for track_data in tracks_data:
            Track.objects.create(album=album, **track_data)
        return album

>>> data = {
    'album_name': 'The Grey Album',
    'artist': 'Danger Mouse',
    'tracks': [
        {'order': 1, 'title': 'Public Service Announcement', 'duration': 245},
        {'order': 2, 'title': 'What More Can I Say', 'duration': 264},
        {'order': 3, 'title': 'Encore', 'duration': 159},
    ],
}
>>> serializer = AlbumSerializer(data=data)
>>> serializer.is_valid()
True
>>> serializer.save()
<Album: Album object>
~~~~

### Custom relational fields

기존의 관계형 스타일이 필요하지 않은 경우가 드물지만 모델 인스턴스에서 출력 representation을 생성하는 방법을 정확하게 설명하는 완전히 custom 관계형 필드를 구현할 수 있습니다.
<br>custom 관계 필드를 구현하려면 <tt style="color: #FF0000">`RelatedField`</tt>를 재정의하고 <tt style="color: #FF0000">`.to_representation(self, value)`</tt> 메서드를 구현해야합니다. 이 메서드는 필드의 대상을 <tt style="color: #FF0000">`value`</tt> 인수로 사용하고 대상을 serialize하는 데 사용해야하는 representation을 반환해야합니다. <tt style="color: #FF0000">`value`</tt> 인수는 일반적으로 모델 인스턴스입니다.
<br>read-write 관계형 필드를 구현하려면 <tt style="color: #FF0000">`.to_internal_value(self, data)`</tt> 메소드도 구현해야합니다.
<br><tt style="color: #FF0000">`context`</tt>를 기반으로 동적 쿼리 세트를 제공하려면 클래스에서 <tt style="color: #FF0000">`.queryset`</tt>을 지정하는 대신 또는 필드를 초기화 할 때 <tt style="color: #FF0000">`.get_queryset(self)`</tt>를 재정의할 수 있습니다.

#### Example

예를 들어, 순서, 제목 및 기간을 사용하여 트랙을 custom 문자열 representation으로 serialize하는 관계형 필드를 정의할 수 있습니다.
~~~~
import time

class TrackListingField(serializers.RelatedField):
    def to_representation(self, value):
        duration = time.strftime('%M:%S', time.gmtime(value.duration))
        return 'Track %d: %s (%s)' % (value.order, value.name, duration)

class AlbumSerializer(serializers.ModelSerializer):
    tracks = TrackListingField(many=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
~~~~
이 custom 필드는 다음 representation으로 serialize됩니다.
~~~~
{
    'album_name': 'Sometimes I Wish We Were an Eagle',
    'artist': 'Bill Callahan',
    'tracks': [
        'Track 1: Jim Cain (04:39)',
        'Track 2: Eid Ma Clack Shaw (04:19)',
        'Track 3: The Wind and the Dove (04:34)',
        ...
    ]
}
~~~~

### Custom hyperlinked fields

어떤 경우에는 하나 이상의 조회 필드가 필요한 URL을 나타내기 위해 하이퍼 링크된 필드의 동작을 customize해야 할 수 있습니다.
<br><tt style="color: #FF0000">`HyperlinkedRelatedField`</tt>를 재정의하여 이를 수행할 수 있습니다. 덮어쓸 수 있는 두 가지 방법이 있습니다.
<br><b>get_url(self, obj, view_name, request, format)</b>
<br>get_url 메소드는 객체 인스턴스를 URL representation에 매핑하는 데 사용됩니다.
<br><tt style="color: #FF0000">`view_name`</tt> 및 <tt style="color: #FF0000">`lookup_field`</tt> 속성이 URL conf와 정확하게 일치하도록 구성되지 않은 경우 <tt style="color: #FF0000">`NoReverseMatch`</tt>를 발생시킬 수 있습니다.
<br><b>get_object(self, queryset, view_name, view_args, view_kwargs)</b>
<br>쓰기 가능한 하이퍼 링크 필드를 지원하려면 들어오는 URL을 그들이 나타내는 객체로 다시 매핑하기 위해 <tt style="color: #FF0000">`get_object`</tt>를 오버라이드해야합니다. read-only 하이퍼 링크 필드의 경우 이 메서드를 재정의할 필요가 없습니다.
<br>이 메소드의 반환 값은 일치하는 URL conf 인수에 해당하는 객체 여야합니다.
<br><tt style="color: #FF0000">`ObjectDoesNotExist`</tt> 예외가 발생할 수 있습니다.

#### Example

다음과 같이 두 개의 키워드 인수를 취하는 고객 객체의 URL이 있다고 가정해보십시오.
~~~~
/api/<organization_slug>/customers/<customer_pk>/
~~~~

단일의 룩 업 필드만을 받아들이는 디폴트의 구현에서는 이것을 표현할 수 없습니다.
<br>이 경우 우리가 원하는 동작을 얻으려면 <tt style="color: #FF0000">`HyperlinkedRelatedField`</tt>를 재정의해야합니다.
~~~~
from rest_framework import serializers
from rest_framework.reverse import reverse

class CustomerHyperlink(serializers.HyperlinkedRelatedField):
    # We define these as class attributes, so we don't need to pass them as arguments.
    view_name = 'customer-detail'
    queryset = Customer.objects.all()

    def get_url(self, obj, view_name, request, format):
        url_kwargs = {
            'organization_slug': obj.organization.slug,
            'customer_pk': obj.pk
        }
        return reverse(view_name, kwargs=url_kwargs, request=request, format=format)

    def get_object(self, view_name, view_args, view_kwargs):
        lookup_kwargs = {
           'organization__slug': view_kwargs['organization_slug'],
           'pk': view_kwargs['customer_pk']
        }
        return self.get_queryset().get(**lookup_kwargs)
~~~~
이 스타일을 일반 뷰와 함께 사용하려는 경우 올바른 조회 동작을 얻으려면 뷰에서 <tt style="color: #FF0000">`.get_object`</tt>를 재정의해야합니다.
<br>일반적으로 가능한 경우 API representations에 플랫 스타일을 사용하는 것이 좋지만 nested URL 스타일은 적당히 사용하면 합리적일 수 있습니다.


## Further notes

### The `queryset` argument

<tt style="color: #FF0000">`queryset`</tt> 인수는 쓰기 가능한 관계 필드에만 필요하며 이 경우 원시 기본 입력에서 모델 인스턴스로 매핑되는 모델 인스턴스 조회를 수행하는 데 사용됩니다.
<br>버전 2.x에서 serializer 클래스는 <tt style="color: #FF0000">`ModelSerializer`</tt> 클래스가 사용되는 경우 <tt style="color: #FF0000">`queryset`</tt> 인수를 자동으로 결정할 수 있습니다.
<br>이 동작은 이제 항상 쓰기 가능한 관계형 필드에 대해 명시적 <tt style="color: #FF0000">`queryset`</tt> 인수를 사용하여 대체됩니다.
<br>이렇게하면 <tt style="color: #FF0000">`ModelSerializer`</tt>가 제공하는 숨겨진 '마법'양이 줄어들고 필드의 동작이 더 명확해지며 <tt style="color: #FF0000">`ModelSerializer`</tt> 바로 가기를 사용하거나 완전하게 명시적인 <tt style="color: #FF0000">`Serializer`</tt> 클래스를 사용하는 것이 쉽다는 것을 확실하게 보장할 수 있습니다.

### Customizing the HTML display

모델의 내장 __str__ 메소드는 <tt style="color: #FF0000">`choices`</tt> 속성을 채우는데 사용된 객체의 문자열 representations을 생성하는데 사용됩니다. 이러한 선택 사항은 browsable API에서 선택된 HTML 입력을 채우는 데 사용됩니다.
<br>그러한 입력에 대해 사용자 정의된 표현을 제공하려면 <tt style="color: #FF0000">`RelatedField`</tt> 서브 클래스의 <tt style="color: #FF0000">`display_value()`</tt>를 대체하십시오. 이 메서드는 모델 객체를 수신하고 모델 객체를 나타내는데 적합한 문자열을 반환해야합니다. 예 :
~~~~
class TrackPrimaryKeyRelatedField(serializers.PrimaryKeyRelatedField):
    def display_value(self, instance):
        return 'Track: %s' % (instance.title)
~~~~

### Select field cutoffs

browsable API에서 렌더링될 때 관계형 필드는 기본적으로 최대 1000개의 선택 가능한 항목만 표시합니다. 더 많은 항목이 있으면 "1000 개 이상의 항목 ..."과 함께 비활성화된 옵션이 표시됩니다.
<br>이 동작은 매우 많은 수의 관계가 표시되어 허용되는 시간 범위 내에서 템플릿을 렌더링할 수 없도록 하기 위한 것입니다.
<br>이 동작을 제어하는 ​​데 사용할 수 있는 두 개의 키워드 인수가 있습니다.
* <tt style="color: #FF0000">`html_cutoff`</tt> - 설정된 경우 HTML 선택 드롭 다운에 의해 표시 될 최대 선택 항목 수입니다. 제한을 해제하려면 <tt style="color: #FF0000">`None`</tt>으로 설정하십시오. Defaults는 <tt style="color: #FF0000">`1000`</tt>입니다.
* <tt style="color: #FF0000">`html_cutoff_text`</tt> - 설정된 경우 HTML 선택 드롭 다운에서 최대 항목 수가 잘린 경우 텍스트 표시기가 표시됩니다. Defaults는 <tt style="color: #FF0000">`'More than {count} items…'`</tt>입니다.

<tt style="color: #FF0000">`HTML_SELECT_CUTOFF`</tt> 및 <tt style="color: #FF0000">`HTML_SELECT_CUTOFF_TEXT`</tt> 설정을 사용하여 전역적으로 제어할 수도 있습니다.
<br>잘라내기가 시행되는 경우 HTML 양식에 일반 입력 필드를 대신 사용할 수 있습니다. <tt style="color: #FF0000">`style`</tt> 키워드 인수를 사용하면됩니다. 예 :
~~~~
assigned_to = serializers.SlugRelatedField(
   queryset=User.objects.all(),
   slug_field='username',
   style={'base_template': 'input.html'}
)
~~~~

### Reverse relations

역방향 관계는 <tt style="color: #FF0000">`ModelSerializer`</tt> 및 <tt style="color: #FF0000">`HyperlinkedModelSerializer`</tt> 클래스에 자동으로 포함되지 않습니다. 역관계를 포함시키려면 필드 목록에 명시적으로 추가해야합니다. 예 :
~~~~
class AlbumSerializer(serializers.ModelSerializer):
    class Meta:
        fields = ('tracks', ...)
~~~~
일반적으로 필드 이름으로 사용할 수 있는 적절한 <tt style="color: #FF0000">`related_name`</tt> 인수를 관계에 설정했는지 확인하고자합니다. 예 :
~~~~
class Track(models.Model):
    album = models.ForeignKey(Album, related_name='tracks', on_delete=models.CASCADE)
    ...
~~~~
역방향 관계에 대한 관련 이름을 설정하지 않은 경우 <tt style="color: #FF0000">`fields`</tt> 인수에 자동으로 생성된 관련 이름을 사용해야합니다. 예 :
~~~~
class AlbumSerializer(serializers.ModelSerializer):
    class Meta:
        fields = ('track_set', ...)
~~~~
[reverse relationships](https://docs.djangoproject.com/en/1.11/topics/db/queries/#following-relationships-backward)에 대한 자세한 내용은 장고 문서를 참고하세요.

### Generic relationships

generic foreign key를 serialize하려면 custom 필드를 정의하여 관계 대상을 serialize하는 방법을 명시적으로 결정해야합니다.
<br>예를 들어 다른 임의의 모델과 generic relationship가 있는 태그에 대해 다음 모델이 제공됩니다.
~~~~
class TaggedItem(models.Model):
    """
    Tags arbitrary model instances using a generic relation.

    See: https://docs.djangoproject.com/en/stable/ref/contrib/contenttypes/
    """
    tag_name = models.SlugField()
    content_type = models.ForeignKey(ContentType, on_delete=models.CASCADE)
    object_id = models.PositiveIntegerField()
    tagged_object = GenericForeignKey('content_type', 'object_id')

    def __unicode__(self):
        return self.tag_name
~~~~
그리고 다음 두 모델은 태그와 관련이 있을 수 있습니다.
~~~~
class Bookmark(models.Model):
    """
    A bookmark consists of a URL, and 0 or more descriptive tags.
    """
    url = models.URLField()
    tags = GenericRelation(TaggedItem)


class Note(models.Model):
    """
    A note consists of some text, and 0 or more descriptive tags.
    """
    text = models.CharField(max_length=1000)
    tags = GenericRelation(TaggedItem)
~~~~
태그가 지정된 인스턴스를 serialize하는데 사용할 수 있는 custom 필드를 정의하여 각 인스턴스의 유형을 사용하여 serialize해야하는 방식을 결정할 수 있습니다.
~~~~
class TaggedObjectRelatedField(serializers.RelatedField):
    """
    A custom field to use for the `tagged_object` generic relationship.
    """

    def to_representation(self, value):
        """
        Serialize tagged objects to a simple textual representation.
        """
        if isinstance(value, Bookmark):
            return 'Bookmark: ' + value.url
        elif isinstance(value, Note):
            return 'Note: ' + value.text
        raise Exception('Unexpected type of tagged object')
~~~~
관계의 타겟이 nested representation을 필요로 하는 경우 <tt style="color: #FF0000">`.to_representation()`</tt> 메소드 내부에서 필요한 serializer를 사용할 수 있습니다.
~~~~
 def to_representation(self, value):
        """
        Serialize bookmark instances using a bookmark serializer,
        and note instances using a note serializer.
        """
        if isinstance(value, Bookmark):
            serializer = BookmarkSerializer(value)
        elif isinstance(value, Note):
            serializer = NoteSerializer(value)
        else:
            raise Exception('Unexpected type of tagged object')

        return serializer.data
~~~~
GenericRelation 필드를 사용하여 표현된 reverse generic keys는 관계의 대상 유형이 항상 알려져 있으므로 일반 관계형 필드 유형을 사용하여 serialize할 수 있습니다.
<br>자세한 내용은 [the Django documentation on generic relations.](https://docs.djangoproject.com/en/1.11/ref/contrib/contenttypes/#id1)를 참조하십시오.

### ManyToManyFields with a Through Model

기본적으로 지정된 <tt style="color: #FF0000">`through`</tt> 모델을 사용하여 <tt style="color: #FF0000">`ManyToManyField`</tt>를 대상으로 하는 관계형 필드는 읽기 전용으로 설정됩니다.
<br>관통 모델을 사용하여 <tt style="color: #FF0000">`ManyToManyField`</tt>를 가리키는 관계 필드를 명시적으로 지정하면 <tt style="color: #FF0000">`read_only`</tt>를 True로 설정해야합니다.


## Third Party Packages

다음 타사 패키지도 제공됩니다.

### DRF Nested Routers

[drf-nested-routers package](https://github.com/alanjds/drf-nested-routers)는 nested resources로 작업하기 위한 라우터 및 관계 필드를 제공합니다.

### Rest Framework Generic Relations

[rest-framework-generic-relations](https://github.com/Ian-Foote/rest-framework-generic-relations) 라이브러리는 generic foreign keys에 대한 read/write serialization를 제공합니다.