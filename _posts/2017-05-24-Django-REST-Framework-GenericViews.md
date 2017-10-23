---
layout: post
title:  "Django REST Framework -Generic views"
date:   2017-05-24 10:43:59
author: Dean Kim
categories: Rest_Framework
tags:	Django Rest_framework
cover:  "/assets/instacode.png"
---

# Generic views
- 원본 : [공식문서](http://www.django-rest-framework.org/api-guide/generic-views/)


Django의 generic views는 일반적인 사용 패턴의 shortcut로 개발되었습니다 ... 뷰 개발에서 발견되는 특정 공통 idioms 및 패턴을 가져와서 추상화하므로 반복하지 않아도 일반적인 데이터 뷰를 빠르게 작성할 수 있습니다.
"[Django Documentation](https://docs.djangoproject.com/en/1.11/ref/class-based-views/#base-vs-generic-views)"


class-based 뷰의 주요 이점 중 하나는 재사용 가능한 동작을 구성하는 것입니다. REST framework는 일반적으로 사용되는 패턴을 제공하는 다수의 pre-built 된 뷰를 제공함으로써 이를 활용합니다.
REST framework에서 제공하는 generic views를 사용하면 데이터베이스 모델과 밀접하게 매핑되는 API 뷰를 빠르게 빌드할 수 있습니다.
generic views가 API의 요구 사항에 맞지 않으면 regular <tt style="color: #FF0000">`APIView`</tt> 클래스를 사용하여 드롭 다운하거나 generic views에서 사용하는 mixins 및 base 클래스를 재사용하여 고유한 재사용 가능한 generic views 세트를 작성할 수 있습니다.


## Examples

일반적으로 generic views를 사용하는 경우 view를 override할 수 있고 several 클래스 attributes를 설정합니다.
~~~~
from django.contrib.auth.models import User
from myapp.serializers import UserSerializer
from rest_framework import generics
from rest_framework.permissions import IsAdminUser

class UserList(generics.ListCreateAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = (IsAdminUser,)
~~~~
보다 복잡한 경우에는 뷰 클래스의 다양한 메서드를 재정의할 수도 있습니다. 예를 들면.
~~~~
class UserList(generics.ListCreateAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = (IsAdminUser,)

    def list(self, request):
        # Note the use of `get_queryset()` instead of `self.queryset`
        queryset = self.get_queryset()
        serializer = UserSerializer(queryset, many=True)
        return Response(serializer.data)
~~~~
매우 간단한 경우에는 <tt style="color: #FF0000">`.as_view()`</tt> 메소드를 사용하여 클래스 attributes를 전달할 수 있습니다. 예를 들어 URLconf에 다음 항목이 포함될 수 있습니다.
~~~~
url(r'^/users/', ListCreateAPIView.as_view(queryset=User.objects.all(), serializer_class=UserSerializer), name='user-list')
~~~~


## API Reference

### GenericAPIView

이 클래스는 REST framework의 <tt style="color: #FF0000">`APIView`</tt> 클래스를 확장하여 표준 목록 및 detail 뷰에 일반적으로 필요한 동작을 추가합니다.
제공된 각각의 구체적인 generic views는 <tt style="color: #FF0000">`GenericAPIView`</tt>를 하나 이상의 mixin 클래스와 결합하여 만듭니다.

#### Attributes

<b>Basic settings:</b>
다음 속성은 기본 뷰 동작을 제어합니다.
* <tt style="color: #FF0000">`queryset`</tt> - 이 뷰에서 객체를 반환하는 데 사용해야하는 쿼리 세트입니다. 일반적으로 이 attribute를 설정하거나 <tt style="color: #FF0000">`get_queryset()`</tt> 메소드를 대체해야 합니다. 뷰 메소드를 override하는 경우, 이 property에 직접 액세스하는 대신 <tt style="color: #FF0000">`get_queryset()`</tt>을 호출하는 것이 중요합니다. <tt style="color: #FF0000">`queryset`</tt>은 한 번 evaluated되고 그 결과는 모든 subsequent requests에 대해 캐시됩니다.
* <tt style="color: #FF0000">`serializer_class`</tt> - input의 validating과 deserializing 및 출력 serializing에 사용하는 serializer 클래스입니다. 일반적으로 이 attribute을 설정하거나 <tt style="color: #FF0000">`get_serializer_class()`</tt> 메소드를 대체해야합니다.
* <tt style="color: #FF0000">`lookup_field`</tt> - 개별 모델 인스턴스의 개체 조회를 수행하는데 사용해야 하는 모델 필드입니다. 기본값은 <tt style="color: #FF0000">`'pk'`</tt>입니다. 하이퍼링크 된 API를 사용할 때 맞춤 값을 사용해야하는 경우 API view 및 serializer 클래스가 조회 필드를 설정해야합니다.
* <tt style="color: #FF0000">`lookup_url_kwarg`</tt> - 객체 검색에 사용해야하는 URL 키워드 argument입니다. URL conf에는 이 값에 해당하는 키워드 argument가 포함되어야합니다. 설정을 해제하면 <tt style="color: #FF0000">`lookup_field`</tt>와 동일한 값을 사용합니다.

<b>Pagination:</b>
다음 attributes은 목록보기와 함께 사용될 때 pagination을 제어하는 ​​데 사용됩니다.
* <tt style="color: #FF0000">`pagination_class`</tt> - list pagination을 지정할 때 사용해야 하는 pagination 클래스. Defaults는 <tt style="color: #FF0000">`'rest_framework.pagination.PageNumberPagination'`</tt>인 <tt style="color: #FF0000">`DEFAULT_PAGINATION_CLASS`</tt> 설정과 동일한 값입니다. <tt style="color: #FF0000">`pagination_class=None`</tt>으로 설정하면 이 보기에서 pagination을 사용할 수 없습니다.

<b>Filtering:</b>
* <tt style="color: #FF0000">`filter_backends`</tt> - 쿼리 세트를 필터링하는 데 사용해야 하는 필터 백엔드 클래스의 목록입니다. Defaults는 <tt style="color: #FF0000">`DEFAULT_FILTER_BACKENDS`</tt> 설정과 동일한 값입니다.

### Methods

<b>Base methods:</b>

[`get_queryset(self)`](http://www.django-rest-framework.org/api-guide/generic-views/#get_querysetself)

리스트 뷰에 사용되는 쿼리 세트를 돌려줍니다. 상세 뷰 내의 lookups의 베이스로서 사용됩니다. <tt style="color: #FF0000">`queryset`</tt> attribute에 의해 지정된 queryset을 리턴하는 것이 Defaults입니다.
이 메소드는 <tt style="color: #FF0000">`self.queryset`</tt>에 직접 액세스하는 대신 항상 사용되어야 하며 <tt style="color: #FF0000">`self.queryset`</tt>은 한 번만 평가되고 그 결과는 모든 subsequent requests에 대해 캐시됩니다.
request을 수행하는 사용자에게 특정한 queryset 반환과 같은 동적 동작을 제공하기 위해 재정의 될 수 있습니다.
<br>예:
~~~~
def get_queryset(self):
    user = self.request.user
    return user.accounts.all()
~~~~

[`get_object(self)`](http://www.django-rest-framework.org/api-guide/generic-views/#get_objectself)

상세 뷰에 사용되어야 하는 객체 인스턴스를 반환합니다. Defaults로 <tt style="color: #FF0000">`lookup_field`</tt> parameter를 사용하여 기본 쿼리 세트를 필터링합니다.
둘 이상의 URL kwarg를 기반으로 하는 객체 조회 보다 복잡한 동작을 제공하기 위해 overridden될 수 있습니다.
<br>예:
~~~~
def get_object(self):
    queryset = self.get_queryset()
    filter = {}
    for field in self.multiple_lookup_fields:
        filter[field] = self.kwargs[field]

    obj = get_object_or_404(queryset, **filter)
    self.check_object_permissions(self.request, obj)
    return obj
~~~~
API에 object level permissions이 없으면 선택적으로 <tt style="color: #FF0000">`self.check_object_permissions`</tt>를 제외하고 단순히 <tt style="color: #FF0000">`get_object_or_404`</tt> 조회에서 객체를 반환할 수 있습니다.

[`filter_queryset(self, queryset)`](http://www.django-rest-framework.org/api-guide/generic-views/#filter_querysetself-queryset)

주어진 쿼리 세트를 사용중인 필터 백엔드를 사용하여 새로운 쿼리 세트를 반환합니다.
<br>예:
~~~~
def filter_queryset(self, queryset):
    filter_backends = (CategoryFilter,)

    if 'geo_route' in self.request.query_params:
        filter_backends = (GeoRouteFilter, CategoryFilter)
    elif 'geo_point' in self.request.query_params:
        filter_backends = (GeoPointFilter, CategoryFilter)

    for backend in list(filter_backends):
        queryset = backend().filter_queryset(self.request, queryset, view=self)

    return queryset
~~~~

[`get_serializer_class(self)`](http://www.django-rest-framework.org/api-guide/generic-views/#get_serializer_classself)

serializer에 사용해야하는 클래스를 반환합니다. Defaults는 <tt style="color: #FF0000">`serializer_class`</tt> attribute을 반환하는 것입니다.
읽기 및 쓰기 작업에 다른 serializer를 사용하거나 다른 유형의 사용자에게 다른 serializer를 제공하는 등의 동적 동작을 제공하기 위해 재정의될 수 있습니다.
<br>예:
~~~~
def get_serializer_class(self):
    if self.request.user.is_staff:
        return FullAccountSerializer
    return BasicAccountSerializer
~~~~

<b>Save and deletion hooks:</b>

다음과 같은 메소드가 mixin 클래스에서 제공되며 오브젝트 저장 또는 삭제 동작을 쉽게 overriding할 수 있습니다.
* <tt style="color: #FF0000">`perform_create(self, serializer)`</tt> - 새 객체 인스턴스를 저장할 때 <tt style="color: #FF0000">`CreateModelMixin`</tt>에 의해 호출됩니다.
* <tt style="color: #FF0000">`perform_update(self, serializer)`</tt> - 기존 객체 인스턴스를 저장할 때 <tt style="color: #FF0000">`UpdateModelMixin`</tt>에 의해 호출됩니다.
* <tt style="color: #FF0000">`perform_destroy(self, instance)`</tt> - 객체 인스턴스를 삭제할 때 <tt style="color: #FF0000">`DestroyModelMixin`</tt>에 의해 호출됩니다.

이러한 hooks는 request에 내포되어 있지만 request 데이터의 일부가 아닌 attributes를 설정하는 데 특히 유용합니다. 예를 들어 request 사용자를 기준으로 또는 URL 키워드 argument를 기반으로 개체의 attribute을 설정할 수 있습니다.
~~~~
def perform_create(self, serializer):
    serializer.save(user=self.request.user)
~~~~
이러한 override points는 또한 확인 이메일을 보내거나 업데이트를 logging하는 것과 같이 객체 저장 전후에 발생하는 동작을 추가할 때 특히 유용합니다.
~~~~
def perform_update(self, serializer):
    instance = serializer.save()
    send_email_confirmation(user=self.request.user, modified=instance)
~~~~
<tt style="color: #FF0000">`ValidationError()`</tt>를 발생시켜 추가 validation를 제공하기 위해 이러한 hooks을 사용할 수도 있습니다. 데이터베이스 저장 시점에 적용할 validation logic이 필요한 경우 유용할 수 있습니다. 예 :
~~~~
def perform_create(self, serializer):
    queryset = SignupRequest.objects.filter(user=self.request.user)
    if queryset.exists():
        raise ValidationError('You have already signed up')
    serializer.save(user=self.request.user)
~~~~
<b>Note:</b> 이 메소드는 이전 버전의 2.x <tt style="color: #FF0000">`pre_save`</tt>, <tt style="color: #FF0000">`post_save`</tt>, <tt style="color: #FF0000">`pre_delete`</tt> 및 <tt style="color: #FF0000">`post_delete`</tt> 메소드를 대체하며 더 이상 사용할 수 없습니다.

<b>Other methods:<b>

<tt style="color: #FF0000">`GenericAPIView`</tt>를 사용하여 custom view를 작성하는 경우 호출해야 할 수도 있지만 일반적으로 다음 메소드를 대체해야 할 필요는 없습니다.
* <tt style="color: #FF0000">`get_serializer_context(self)`</tt> - serializer에 제공되어야하는 추가 컨텍스트가 포함된 dictionary을 반환합니다. Defaults는 <tt style="color: #FF0000">`request`</tt>, <tt style="color: #FF0000">`view`</tt> 및 <tt style="color: #FF0000">`format`</tt>키를 포함합니다.
* <tt style="color: #FF0000">`get_serializer(self, instance=None, data=None, many=False, partial=False)`</tt> - serializer 인스턴스를 반환합니다.
* <tt style="color: #FF0000">`get_paginated_response(self, data)`</tt> - paginated 스타일의 <tt style="color: #FF0000">`Response`</tt> 객체를 반환합니다.
* <tt style="color: #FF0000">`paginate_queryset(self, queryset)`</tt> - 필요한 경우 쿼리 개체에 페이지 개체를 반환하거나 이 뷰에 pagination이 구성되지 않은 경우 <tt style="color: #FF0000">`None`</tt>을 매깁니다.
* <tt style="color: #FF0000">`filter_queryset(self, queryset)`</tt> - 주어진 쿼리 세트를 사용중인 필터 백엔드를 사용하여 새로운 쿼리 세트를 반환합니다.


## Mixins

mixin 클래스는 basic view behavior를 제공하는데 사용되는 작업을 제공합니다. mixin 클래스는 <tt style="color: #FF0000">`.get()`</tt> 및 <tt style="color: #FF0000">`.post()`</tt>와 같은 핸들러 메서드를 직접 정의하는 것이 아니라 작업 메서드를 제공합니다. 이것은 보다 flexible composition of behavior를 가능하게 합니다.
<br>mixin 클래스는 <tt style="color: #FF0000">`rest_framework.mixins`</tt>에서 가져올 수 있습니다.

### ListModelMixin

queryset listing을 구현하는 <tt style="color: #FF0000">`.list(request, * args, ** kwargs)`</tt> 메서드를 제공합니다.
<br>queryset이 ​​채워지면 response의 body로 queryset의 serialized representation과 함께 <tt style="color: #FF0000">`200 OK`</tt> response를 반환합니다. 응답 데이터는 선택적으로 페이징될 수 있습니다.

### CreateModelMixin

new model instance creating 및 saving을 구현하는 <tt style="color: #FF0000">`.create(request, * args, ** kwargs)`</tt> 메서드를 제공합니다.
<br>객체가 생성되면 객체의 serialized representation이 response의 body인 <tt style="color: #FF0000">`201 Created`</tt> response을 반환합니다. 표현에 <tt style="color: #FF0000">`url`</tt>이라는 키가 포함되어 있으면 응답의 <tt style="color: #FF0000">`Location`</tt> 헤더가 해당값으로 채워집니다.
<br>객체 생성을 위해 제공된 request 데이터가 invalid 경우 <tt style="color: #FF0000">`400 Bad Request`</tt> response이 반환되며 오류 내역은 response 본문으로 반환됩니다.

### RetrieveModelMixin

response에서 기존 모델 인스턴스를 반환하도록 구현하는 <tt style="color: #FF0000">`.retrieve(request, * args, ** kwargs)`</tt> 메서드를 제공합니다.
<br>객체를 검색 할 수 있는 경우 <tt style="color: #FF0000">`200 OK`</tt> response를 반환하며 객체를 response body로 serialized representation를 반환합니다. 그렇지 않으면 <tt style="color: #FF0000">`404 Not Found`</tt>을 반환합니다.

### UpdateModelMixin

기존 모델 인스턴스를 업데이트하고 저장하는 <tt style="color: #FF0000">`.update(request, * args, ** kwargs)`</tt> 메서드를 제공합니다.
<br>또한 update 메소드와 유사한 <tt style="color: #FF0000">`.partial_update(request, * args, ** kwargs)`</tt> 메소드를 제공합니다. 단, 업데이트의 모든 필드는 선택 사항입니다. 이렇게하면 HTTP <tt style="color: #FF0000">`PATCH`</tt> requests을 지원할 수 있습니다.
<br>객체가 업데이트되면 객체의 serialized representation이 response body와 함께 <tt style="color: #FF0000">`200 OK`</tt> response를 반환합니다.
<br>객체를 업데이트하기 위해 제공된 요청 데이터가 invalid 경우 <tt style="color: #FF0000">`400 Bad Request`</tt> response가 반환되고 오류 세부 정보가 response body로 사용됩니다.

### DestroyModelMixin

기존 모델 인스턴스의 삭제를 구현하는 <tt style="color: #FF0000">`.destroy(request, * args, ** kwargs)`</tt> 메서드를 제공합니다.
<br>객체가 삭제되면 <tt style="color: #FF0000">`204 No Content`</tt> response를 반환하고, 그렇지 않으면 <tt style="color: #FF0000">`404 Not Found`</tt>을 반환합니다.


## Concrete View Classes

다음 클래스는 concrete generic views입니다. 일반적으로 heavily 커스터마이징 된 동작이 필요하지 않으면 generic views를 사용하면 됩니다.
<br>뷰 클래스는 <tt style="color: #FF0000">`rest_framework.generics`</tt>에서 가져올 수 있습니다.

### CreateAPIView

<b>create-only</b> endpoints에 사용됩니다.
<br><tt style="color: #FF0000">`post`</tt> 메소드 핸들러를 제공합니다.
<br>Extends: GenericAPIView, CreateModelMixin

### ListAPIView

<b>read-only</b> endpoints가 <b>collection of model instances</b>를 나타내는데 사용됩니다.
<br><tt style="color: #FF0000">`get`</tt> 메소드 핸들러를 제공합니다.
<br>Extends: GenericAPIView, ListModelMixin

### RetrieveAPIView

<b>read-only</b> endpoints가 <b>single model instance</b>를 나타내는데 사용됩니다.
<br><tt style="color: #FF0000">`get`</tt> 메소드 핸들러를 제공합니다.
<br>Extends: GenericAPIView, RetrieveModelMixin

### DestroyAPIView

<b>single model instance</b>의 <b>delete-only</b> endpoints에 사용됩니다.
<br><tt style="color: #FF0000">`delete`</tt> 메서드 핸들러를 제공합니다.
<br>Extends: GenericAPIView, DestroyModelMixin

### UpdateAPIView

<b>single model instance</b>의 <b>update-only</b> endpoints에 사용됩니다.
<br><tt style="color: #FF0000">`put`</tt> 및 <tt style="color: #FF0000">`patch`</tt> 메소드 핸들러를 제공합니다.
<br>Extends: GenericAPIView, UpdateModelMixin

### ListCreateAPIView

<b>collection of model instances</b>를 나타내는 <b>read-write</b> endpoints에 사용됩니다.
<br><tt style="color: #FF0000">`get`</tt> 및 <tt style="color: #FF0000">`post`</tt> 메서드 핸들러를 제공합니다.
<br>Extends: GenericAPIView, ListModelMixin, CreateModelMixin

### RetrieveUpdateAPIView

<b>single model instance</b>를 나타내기 위해 <b>read or update</b> endpoints에 사용됩니다.
<br><tt style="color: #FF0000">`get`</tt>, <tt style="color: #FF0000">`put`</tt> 및 <tt style="color: #FF0000">`patch`</tt> 메소드 핸들러를 제공합니다.
<br>Extends: GenericAPIView, RetrieveModelMixin, UpdateModelMixin

### RetrieveDestroyAPIView

<b>single model instance</b>를 나타내는 <b>read or delete</b> endpoints에 사용됩니다.
<br><tt style="color: #FF0000">`get`</tt> 및 <tt style="color: #FF0000">`delete`</tt> 메서드 핸들러를 제공합니다.
<br>Extends: GenericAPIView, RetrieveModelMixin, DestroyModelMixin

### RetrieveUpdateDestroyAPIView

<b>single model instance</b>를 나타내기 위해 <b>read-write-delete</b> endpoints에 사용됩니다.
<br><tt style="color: #FF0000">`get`</tt>, <tt style="color: #FF0000">`put`</tt>, <tt style="color: #FF0000">`patch`</tt> 및 <tt style="color: #FF0000">`delete`</tt> 메소드 핸들러를 제공합니다.
<br>Extends: GenericAPIView, RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin


## Customizing the generic views

기존 generic views를 사용하지만 종종 slightly customized behavior를 사용할 경우도 있습니다. 여러 위치에서 customized behavior를 재사용하는 경우, behavior를 common class로 리팩토링하여 필요할 때 모든 view나 viewset에 적용할 수 있습니다.

### Creating custom mixins

예를 들어, URL conf 내의 복수의 필드에 근거해 오브젝트를 검색할 필요가 있는 경우, 다음과 같이 mixin 클래스를 작성할 수 있습니다.
~~~~
class MultipleFieldLookupMixin(object):
    """
    Apply this mixin to any view or viewset to get multiple field filtering
    based on a `lookup_fields` attribute, instead of the default single field filtering.
    """
    def get_object(self):
        queryset = self.get_queryset()             # Get the base queryset
        queryset = self.filter_queryset(queryset)  # Apply any filter backends
        filter = {}
        for field in self.lookup_fields:
            if self.kwargs[field]: # Ignore empty fields.
                filter[field] = self.kwargs[field]
        return get_object_or_404(queryset, **filter)  # Lookup the object
~~~~
그런 다음 custom behavior를 적용해야 할 때마다 이 mixin을 view 또는 viewset에 간단하게 적용할 수 있습니다.
~~~~
class RetrieveUserView(MultipleFieldLookupMixin, generics.RetrieveAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    lookup_fields = ('account', 'username')
~~~~
사용해야 하는 custom behavior이 있는 경우 custom mixins을 사용하는 것이 좋습니다.

### Creating custom base classes

multiple views에서 mixin을 사용하는 경우 이 단계를 더 진행하고 프로젝트 전체에서 사용할 수 있는 고유한 base views set을 만들 수 있습니다. 예 :
~~~~
class BaseRetrieveView(MultipleFieldLookupMixin,
                       generics.RetrieveAPIView):
    pass

class BaseRetrieveUpdateDestroyView(MultipleFieldLookupMixin,
                                    generics.RetrieveUpdateDestroyAPIView):
    pass
~~~~
프로젝트 전반에 걸쳐 많은 수의 뷰에서 일관되게 반복해야하는 custom behavior가 있는 경우 custom base classes를 사용하는 것이 좋습니다.


## PUT as create

버전 3.0 이전에는 객체가 이미 존재하는지 여부에 따라 REST framework mixins가 <tt style="color: #FF0000">`PUT`</tt>을 업데이트 또는 작성 작업으로 처리했습니다.
<br>생성 작업으로 <tt style="color: #FF0000">`PUT`</tt>을 허용하는 것은 객체의 존재 또는 부재에 대한 정보를 반드시 노출하므로 문제가 됩니다. 또한 이전에 삭제된 인스턴스를 투명하게 다시 만들 수 있다는 것이 단순히 <tt style="color: #FF0000">`404`</tt> responses를 반환하는 것보다 더 나은 default behavior임이 명하지 않습니다.
<br>"<tt style="color: #FF0000">`PUT`</tt> as 404"와 "<tt style="color: #FF0000">`PUT`</tt> as create"는 서로 다른 상황에서 유효할 수 있지만 버전 3.0부터는 더 간단하고 명확한 404 behavior를 default로 사용합니다.
<br>일반적인 PUT-as-create 동작이 필요한 경우이 <tt style="color: #FF0000">`AllowPUTAsCreateMixin`</tt> 클래스를 view에 mixin로 포함 할 수 있습니다.


## Third party packages

다음 타사 패키지는 추가 generic view 구현을 제공합니다.

### Django REST Framework bulk
 
[django-rest-framework-bulk package](https://github.com/miki725/django-rest-framework-bulk)는 API requests를 통해 대량 작업을 적용 할 수 있도록 generic view mixins뿐만 아니라 일반적인 concrete generic views를 구현합니다.

### Django Rest Multiple Models

[Django Rest Multiple Models](https://github.com/MattBroach/DjangoRestMultipleModels)은 단일 API request를 통해 multiple serialized 모델 and/or querysets를 전송하기 위한 generic view (and mixin)를 제공합니다.