---
layout: post
title:  "Django REST Framework -Pagination"
date:   2017-05-12 08:43:59
author: Dean Kim
categories: Rest_Framework
tags:	Django Rest_framework
cover:  "/assets/instacode.png"
---

# Pagination

Django는 paginated data, 즉 "이전/다음"링크를 사용하여 여러 페이지로 나누어진 데이터를 관리하는데 도움이되는 몇 가지 클래스를 제공합니다.
"Django documentation"

REST framework는 customizable pagination styles을 지원합니다. 이렇게 하면 큰 결과 sets를 개별 데이터 페이지로 분할하는 방법을 수정할 수 있습니다.
pagination API는 다음을 지원합니다.
* response content의 일부로 제공되는 Pagination links.
* <tt style="color: #FF0000">`Content-Range`</tt> 또는 <tt style="color: #FF0000">`Link`</tt>와 같은 response headers에 포함 된 Pagination links.

built-in styles은 현재 response 내content의 일부로 포함된 링크를 사용합니다. 이 style은 browsable API를 사용할 때 더 쉽게 액세스 할 수 있습니다.
Pagination은 generic views 또는 viewsets를 사용하는 경우에만 자동으로 수행됩니다. 일반 <tt style="color: #FF0000">`APIView`</tt>를 사용하는 경우 페이지 paginated response을 반환하도록 pagination API를 직접 호출해야합니다. 예를 들어 <tt style="color: #FF0000">`mixins.ListModelMixin`</tt> 및 <tt style="color: #FF0000">`generics.GenericAPIView`</tt> 클래스의 소스 코드를 참조하십시오.
Pagination 클래스를 <tt style="color: #FF0000">`None`</tt>으로 설정하면 pagination을 끌 수 있습니다.


## Setting the pagination style

default pagination style은 <tt style="color: #FF0000">`DEFAULT_PAGINATION_CLASS`</tt> 및 <tt style="color: #FF0000">`PAGE_SIZE`</tt> setting keys를 사용하여 전체적으로 설정할 수 있습니다. 예를 들어 built-in limit/offset pagination을 사용하려면 다음과 같이하면 됩니다.
~~~~
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
    'PAGE_SIZE': 100
}
~~~~
pagination 클래스와 사용할 page size를 모두 설정해야합니다.
<tt style="color: #FF0000">`pagination_class`</tt> attribute을 사용하여 개별보기에서 pagination 클래스를 설정할 수도 있습니다. 일반적으로 API 전체에서 동일한 pagination style을 사용하고 싶지만 per-view마다 default or maximum page size와 같은 pagination의 개별적인 측면을 변경할 수도 있습니다.


## Modifying the pagination style

pagination style의 특정 측면을 수정하려면 pagination 클래스 중 하나를 재정의하고 변경하려는 attributes을 설정해야합니다.
~~~~
class LargeResultsSetPagination(PageNumberPagination):
    page_size = 1000
    page_size_query_param = 'page_size'
    max_page_size = 10000

class StandardResultsSetPagination(PageNumberPagination):
    page_size = 100
    page_size_query_param = 'page_size'
    max_page_size = 1000
~~~~
그런 다음 <tt style="color: #FF0000">`.pagination_class`</tt> attribute을 사용하여 뷰에 새 스타일을 적용 할 수 있습니다.
~~~~
class BillingRecordsView(generics.ListAPIView):
    queryset = Billing.objects.all()
    serializer_class = BillingRecordsSerializer
    pagination_class = LargeResultsSetPagination
~~~~
또는 <tt style="color: #FF0000">`DEFAULT_PAGINATION_CLASS`</tt> settings key를 사용하여 스타일을 global로 적용하십시오. 예 :
~~~~
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'apps.core.pagination.StandardResultsSetPagination'
}
~~~~


## API Reference

### PageNumberPagination

이 pagination style은 request query parameters에 단일 숫자 페이지 번호를 받습니다.

<b>Request:</b>
~~~~
GET https://api.example.org/accounts/?page=4
~~~~
<b>Response:</b>
~~~~
HTTP 200 OK
{
    "count": 1023
    "next": "https://api.example.org/accounts/?page=5",
    "previous": "https://api.example.org/accounts/?page=3",
    "results": [
       …
    ]
}
~~~~

#### Setup

<tt style="color: #FF0000">`PageNumberPagination`</tt> style을 global로 사용하려면 다음 구성을 사용하여 <tt style="color: #FF0000">`PAGE_SIZE`</tt>를 원하는대로 수정하십시오.
~~~~
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 100
}
~~~~
<tt style="color: #FF0000">`GenericAPIView`</tt> subclasses에서 <tt style="color: #FF0000">`pagination_class`</tt> attribute을 설정하여 페이지 단위로 <tt style="color: #FF0000">`PageNumberPagination`</tt>을 선택할 수도 있습니다.

#### Configuration

<tt style="color: #FF0000">`PageNumberPagination`</tt> 클래스는 pagination style을 수정하기 위해 재정의 될 수 있는 여러 attributes을 포함합니다.
이러한 attributes을 설정하려면 <tt style="color: #FF0000">`PageNumberPagination`</tt> 클래스를 재정의 한 다음 위와 같이 custom pagination 클래스를 활성화해야합니다.
* <tt style="color: #FF0000">`django_paginator_class`</tt> - Django Paginator 클래스를 사용합니다. 기본값은 <tt style="color: #FF0000">`django.core.paginator.Paginator`</tt>입니다. 대부분의 use cases에서 괜찮습니다.
* <tt style="color: #FF0000">`page_size`</tt> - 페이지 크기를 나타내는 숫자 value입니다. 설정된 경우 <tt style="color: #FF0000">`PAGE_SIZE`</tt> 설정보다 우선합니다. Defaults은 <tt style="color: #FF0000">`PAGE_SIZE`</tt> settings key와 동일한 값입니다.
* <tt style="color: #FF0000">`page_query_param`</tt> - pagination 컨트롤에 사용할 query parameter의 이름을 나타내는 문자열 value입니다.
* <tt style="color: #FF0000">`page_size_query_param`</tt> - 설정된 경우 클라이언트가 per-request로 페이지 크기를 설정할 수 있도록 하는 query parameter의 이름을 나타내는 문자열 value입니다. Defaults는 <tt style="color: #FF0000">`None`</tt>으로, 클라이언트가 요청된 페이지 크기를 제어 할 수 없음을 나타냅니다.
* <tt style="color: #FF0000">`max_page_size`</tt> - 설정되어 있는 경우 requested 최대 페이지 크기를 나타내는 숫자 value입니다. 이 attribute은 <tt style="color: #FF0000">`page_size_query_param`</tt>도 설정되어있는 경우에만 유효합니다.
* <tt style="color: #FF0000">`last_page_strings`</tt> - 집합에서 마지막 페이지를 요청하기 위해 page_query_param과 함께 사용할 수 있는 values을 나타내는 문자열 values의 목록 또는 tuple입니다. Defaults은 <tt style="color: #FF0000">`('last',)`</tt>
* <tt style="color: #FF0000">`template`</tt> - browsable API에서 pagination 컨트롤을 렌더링할 때 사용할 템플릿의 이름입니다. 렌더링 스타일을 수정하기 위해 재정의되거나 HTML pagination 컨트롤을 완전히 비활성화하려면 <tt style="color: #FF0000">`None`</tt>으로 설정될 수 있습니다. Defaults는 <tt style="color: #FF0000">`"rest_framework / pagination / numbers.html"`</tt>입니다.


### LimitOffsetPagination

이 pagination style은 여러 데이터베이스 레코드를 찾을 때 사용되는 구문을 반영합니다. 클라이언트에는 "limit"및 "offset" query parameter가 모두 포함됩니다. limit는 반환할 최대 항목 수를 나타내며 다른 스타일의 <tt style="color: #FF0000">`page_size`</tt>와 같습니다. offset은 unpaginated items의 전체 집합에 대한 쿼리의 시작 위치를 나타냅니다.

<b>Request:</b>
~~~~
GET https://api.example.org/accounts/?limit=100&offset=400
~~~~
<b>Response:</b>
~~~~
HTTP 200 OK
{
    "count": 1023
    "next": "https://api.example.org/accounts/?limit=100&offset=500",
    "previous": "https://api.example.org/accounts/?limit=100&offset=300",
    "results": [
       …
    ]
}
~~~~

#### Setup

<tt style="color: #FF0000">`LimitOffsetPagination`</tt> 스타일을 global로 사용하려면 다음 구성을 사용하십시오.
~~~~
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination'
}
~~~~
선택적으로 <tt style="color: #FF0000">`PAGE_SIZE`</tt> 키를 설정할 수도 있습니다. <tt style="color: #FF0000">`PAGE_SIZE`</tt> parameter도 사용되는 경우 <tt style="color: #FF0000">`limit`</tt> query parameter는 선택 사항이며 클라이언트가 생략 할 수 있습니다.
<tt style="color: #FF0000">`GenericAPIView`</tt> subclasses에서는 <tt style="color: #FF0000">`pagination_class`</tt> attribute을 설정하여 각 뷰별로 <tt style="color: #FF0000">`LimitOffsetPagination`</tt>을 선택할 수 있습니다.

#### Configuration

<tt style="color: #FF0000">`LimitOffsetPagination`</tt> 클래스에는 pagination style을 수정하기 위해 재정의할 수 있는 많은 attributes이 포함되어 있습니다.
이러한 attributes을 설정하려면 <tt style="color: #FF0000">`LimitOffsetPagination`</tt> 클래스를 재정의한 다음 위와 같이 custom pagination 클래스를 활성화해야합니다.
* <tt style="color: #FF0000">`default_limit`</tt> - query parameter에서 클라이언트가 클라이언트를 제공하지 않을 경우 사용할 limit를 나타내는 숫자 value입니다. Defaults는 <tt style="color: #FF0000">`PAGE_SIZE`</tt> settings key와 동일한 value입니다.
* <tt style="color: #FF0000">`limit_query_param`</tt> - "limit" query parameter의 이름을 나타내는 문자열 value입니다. Defaults는 <tt style="color: #FF0000">`'limit'`</tt>입니다.
* <tt style="color: #FF0000">`offset_query_param`</tt> - "offset" query parameter의 이름을 나타내는 문자열 value입니다. Defaults는 <tt style="color: #FF0000">`'offset'`</tt>입니다.
* <tt style="color: #FF0000">`max_limit`</tt> - 설정된 경우 이는 클라이언트가 요청할 수 있는 최대 허용 limit를 나타내는 숫자 값입니다. Defaults는 <tt style="color: #FF0000">`None`</tt>입니다.
* <tt style="color: #FF0000">`template`</tt> - browsable API에서 pagination 컨트롤을 렌더링 할 때 사용할 템플릿의 이름입니다. 렌더링 스타일을 수정하기 위해 재정의되거나 HTML pagination 컨트롤을 완전히 비활성화하려면 없음으로 설정될 수 있습니다. Defaults는 <tt style="color: #FF0000">`"rest_framework / pagination / numbers.html"`</tt>입니다.


### CursorPagination

cursor-based pagination은 클라이언트가 결과 세트를 통해 페이지할 때 사용할 수 있는 불투명한 "cursor"표시기를 제공합니다. 이 pagination 스타일은 정방향 및 역방향 컨트롤만 제공하며 클라이언트가 임의의 위치로 이동할 수 없도록 합니다.
Cursor based pagination을 수행하려면 결과 집합에 항목의 순서가 변경되지 않아야합니다. 일반적으로 이 순서는 creation timestamp on the records 일 수 있습니다. 이는 ordering to paginate를 일관되게 유지하기 때문입니다.
Cursor based pagination은 다른 스키마보다 복잡합니다. 또한 결과 세트가 고정 된 순서를 제공해야하며 클라이언트가 임의로 결과 세트에 index를 작성할 수 없도록 해야합니다. 그러나 다음과 같은 이점을 제공합니다.
* 일관된 pagination view를 제공합니다. 적절하게 사용하면 <tt style="color: #FF0000">`CursorPagination`</tt>는 레코드를 페이징 할 때 새 pagination 과정, 심지어 다른 클라이언트에 의해 새로운 item을 insert하더라도 클라이언트가 같은 item을 두 번 보지 않을 것이라는 점을 보장합니다.
* 매우 큰 데이터 세트 사용을 지원합니다. 극도로 큰 데이터 세트의 경우 오프셋 기반 pagination 스타일을 사용하는 pagination이 비효율적이거나 사용할 수 없게 될 수 있습니다. 대신 Cursor based pagination 스키마는 fixed-time properties을 가지며 데이터 집합 크기가 커질수록 속도가 느려지지 않습니다.

#### Details and limitations

cursor based pagination을 올바르게 사용하려면 세부 사항에 조금 주의를 기울여야합니다. 당신은 scheme에 대해 적용 할 ordering에 대해 생각해야 합니다. 기본값은 <tt style="color: #FF0000">`"-created"`</tt>에 의해 order하는 것입니다. 이이 모델 인스턴스에 <b>'created' timestamp field이어야하며</b>, 먼저 가장 최근에 추가된 항목으로는 "timeline" style paginated view를 제공한다고 가정합니다.
pagination 클래스의 <tt style="color: #FF0000">`'ordering'`</tt> attribute를 무시하거나 <tt style="color: #FF0000">`CursorPagination`</tt>과 함께 <tt style="color: #FF0000">`OrderingFilter`</tt> 필터 클래스를 사용하여 순서를 수정할 수 있습니다. <tt style="color: #FF0000">`OrderingFilter`</tt>와 함께 사용하는 경우 사용자가 order할 수 있는 입력란을 제한하는 것이 좋습니다.
cursor pagination을 올바르게 사용하려면 다음을 만족시키는 ordering field가 있어야 합니다.
* 생성 시 timestamp, slug 또는 한 번만 설정되는 다른 field와 같은 변경되지 않은 value이어야 합니다.
* 고유하거나 거의 고유해야합니다. Millisecond precision timestamps가 좋은 예입니다. 이 cursor pagination의 implementation은 smart "position plus offset" 스타일을 사용하여 not-strictly-unique values을 순서대로 올바르게 지원할 수 있습니다.
* 문자열로 강제 변환 될 수 있는 nullableable 값이어야합니다.
* field에는 데이터베이스 index가 있어야합니다.

이러한 제약 조건을 만족시키지 못하는 ordering field를 사용하면 일반적으로 작동하지만 cursor pagination의 이점을 일부 상실하게 됩니다.
cursor pagination에 사용되는 implementation에 대한 자세한 기술 정보는 "[Building cursors for the Disqus API](http://cramer.io/2011/03/08/building-cursors-for-the-disqus-api)" 블로그 게시물에서 기본 접근 방법에 대한 개요를 제공합니다.

#### Setup

<tt style="color: #FF0000">`CursorPagination`</tt> 스타일을 globally 사용하려면 다음 구성을 사용하여 <tt style="color: #FF0000">`PAGE_SIZE`</tt>를 원하는대로 수정하십시오.
~~~~
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.CursorPagination',
    'PAGE_SIZE': 100
}
~~~~

<tt style="color: #FF0000">`GenericAPIView`</tt> 서브 클래스에서 <tt style="color: #FF0000">`pagination_class`</tt> 속성을 설정하여 뷰 단위로 <tt style="color: #FF0000">`CursorPagination`</tt>을 선택할 수도 있습니다.

#### Configuration

<tt style="color: #FF0000">`CursorPagination`</tt> 클래스에는 pagination 스타일을 수정하기 위해 재정의 될 수 있는 많은 attributes가 포함되어 있습니다.
이러한 attributes를 설정하려면 <tt style="color: #FF0000">`CursorPagination`</tt> 클래스를 재정의 한 다음 위와 같이 custom pagination 클래스를 활성화해야합니다.
* <tt style="color: #FF0000">`page_size`</tt> = 페이지 크기를 나타내는 숫자 값입니다. 설정된 경우 <tt style="color: #FF0000">`PAGE_SIZE`</tt> 설정보다 우선합니다. Defaults는 <tt style="color: #FF0000">`PAGE_SIZE`</tt> 설정 키와 동일한 value입니다.
* <tt style="color: #FF0000">`cursor_query_param`</tt> = "cursor" query parameter의 이름을 나타내는 문자열 값입니다. Defaults는 'cursor'입니다.
* <tt style="color: #FF0000">`ordering`</tt> = 이것은 cursor based pagination이 적용될 필드를 나타내는 문자열 또는 문자열 목록이어야합니다. 예 : <tt style="color: #FF0000">`ordering = 'slug'`</tt>. Defaults는 <tt style="color: #FF0000">`-created`</tt>입니다. 뷰에서 <tt style="color: #FF0000">`OrderingFilter`</tt>를 사용하여이 value를 재정의 할 수도 있습니다.
* <tt style="color: #FF0000">`template`</tt> = browsable API에서 pagination 컨트롤을 렌더링 할 때 사용할 템플릿의 이름입니다. 렌더링 스타일을 수정하기 위해 재정의되거나 HTML pagination 컨트롤을 완전히 비활성화하려면 <tt style="color: #FF0000">`None`</tt>으로 설정 될 수 있습니다. Defaults는 <tt style="color: #FF0000">`"rest_framework/pagination/previous_and_next.html"`</tt> 입니다.


## Custom pagination styles

custom pagination serializer 클래스를 생성하려면 <tt style="color: #FF0000">`pagination.BasePagination`</tt>을 서브 클래스 화하고 <tt style="color: #FF0000">`paginate_queryset(self, queryset, request, view = None)`</tt> 및 <tt style="color: #FF0000">`get_paginated_response(self, data)`</tt> 메소드를 재정의해야합니다.
* <tt style="color: #FF0000">`paginate_queryset`</tt> 메소드는 initial queryset를 전달받고 requested 페이지의 데이터만 포함하는 iterable object를 반환해야합니다.
* <tt style="color: #FF0000">`get_paginated_response`</tt> 메소드는 serialized된 페이지 데이터를 전달 받고 <tt style="color: #FF0000">`Response`</tt> 인스턴스를 반환해야합니다.

<tt style="color: #FF0000">`paginate_queryset`</tt> 메소드는 pagination 인스턴스에 상태를 설정할 수 있으며 나중에 <tt style="color: #FF0000">`get_paginated_response`</tt> 메소드에서 사용할 수 있습니다.

### Example

default pagination 출력 스타일을 nested '링크'키 아래의 다음 링크와 이전 링크를 포함하는 수정된 형식으로 바꾸려고 한다고 가정합니다. 다음과 같이 custom pagination 클래스를 지정할 수 있습니다.
~~~~
class CustomPagination(pagination.PageNumberPagination):
    def get_paginated_response(self, data):
        return Response({
            'links': {
               'next': self.get_next_link(),
               'previous': self.get_previous_link()
            },
            'count': self.page.paginator.count,
            'results': data
        })
~~~~
그런 다음 구성에서 custom 클래스를 설정해야합니다.
~~~~
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'my_project.apps.core.pagination.CustomPagination',
    'PAGE_SIZE': 100
}
~~~~
browsable API의 responses에서 키의 순서가 어떻게 표시되는지 신경쓰면 paginated responses 본문을 작성할 때 <tt style="color: #FF0000">`OrderedDict`</tt>를 사용하도록 선택할 수 있지만 선택 사항입니다.

### Header based pagination

built-in <tt style="color: #FF0000">`PageNumberPagination`</tt> 스타일을 수정하여 응답 본문에 pagination 링크를 포함하는 대신 [GitHub API와 비슷한 스타일](https://developer.github.com/guides/traversing-with-pagination/)로 <tt style="color: #FF0000">`Link`</tt> header를 포함합니다.
~~~~
class LinkHeaderPagination(pagination.PageNumberPagination):
    def get_paginated_response(self, data):
        next_url = self.get_next_link()
        previous_url = self.get_previous_link()

        if next_url is not None and previous_url is not None:
            link = '<{next_url}>; rel="next", <{previous_url}>; rel="prev"'
        elif next_url is not None:
            link = '<{next_url}>; rel="next"'
        elif previous_url is not None:
            link = '<{previous_url}>; rel="prev"'
        else:
            link = ''

        link = link.format(next_url=next_url, previous_url=previous_url)
        headers = {'Link': link} if link else {}

        return Response(data, headers=headers)
~~~~

### Using your custom pagination class

custom pagination 클래스를 기본적으로 사용하려면 <tt style="color: #FF0000">`DEFAULT_PAGINATION_CLASS`</tt> 설정을 사용하세요.
~~~~
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'my_project.apps.core.pagination.LinkHeaderPagination',
    'PAGE_SIZE': 100
}
~~~~
이제 list endpoints에 대한 API responses에 pagination 링크를 responses 본문의 일부로 포함하는 대신 <tt style="color: #FF0000">`Link`</tt> header가 포함됩니다. 예 :

### Pagination & schemas

또한 <tt style="color: #FF0000">`get_schema_fields()`</tt> 메소드를 구현하여 REST framework가 제공하는 스키마 자동 생성에서 pagination 컨트롤을 사용할 수 있도록 할 수 있습니다. 이 메소드의 signature는 다음과 같습니다.</br>
<tt style="color: #FF0000">`get_schema_fields(self, view)`</tt></br>
메서드는 <tt style="color: #FF0000">`coreapi.Field`</tt> 인스턴스의 목록을 반환해야합니다.


## HTML pagination controls

기본적으로 pagination 클래스를 사용하면 browsable API에 HTML pagination 컨트롤이 표시됩니다. 두 가지 내장 디스플레이 스타일이 있습니다. <tt style="color: #FF0000">`PageNumberPagination`</tt> 및 <tt style="color: #FF0000">`LimitOffsetPagination`</tt> 클래스는 이전 및 다음 컨트롤이 포함된 페이지 번호 목록을 표시합니다. <tt style="color: #FF0000">`CursorPagination`</tt> 클래스는 이전 및 다음 컨트롤만 표시하는 보다 단순한 스타일을 표시합니다.

### Customizing the controls

HTML pagination 컨트롤을 렌더링하는 템플릿을 재정의 할 수 있습니다. 두 가지 기본 제공 스타일은 다음과 같습니다.
* <tt style="color: #FF0000">`rest_framework/pagination/numbers.html`</tt>
* <tt style="color: #FF0000">`rest_framework/pagination/previous_and_next.html`</tt>

global 템플릿 디렉토리에 이러한 경로 중 하나가 포함 된 템플릿을 제공하면 관련 pagination 클래스의 기본 렌더링이 무시됩니다.
또는 기존 클래스를 하위 클래스로 분류하고 클래스의 특성으로 <tt style="color: #FF0000">`template=None`</tt>을 설정하여 HTML pagination 컨트롤을 완전히 비활성화 할 수 있습니다. 그런 다음 맞춤 클래스를 기본 pagination 스타일로 사용하려면 <tt style="color: #FF0000">`DEFAULT_PAGINATION_CLASS`</tt> 설정 키를 구성해야합니다.

#### Low-level API


pagination 클래스가 컨트롤을 표시할지 어떨지를 결정하기위한 low-level API는, pagination 인스턴스의 <tt style="color: #FF0000">`display_page_controls`</tt> attribute으로서 공개되고 있습니다. HTML pagination 컨트롤을 표시해야하는 경우 <tt style="color: #FF0000">`paginate_queryset`</tt> 메서드에서 Custom pagination 클래스를 <tt style="color: #FF0000">`True`</tt>로 설정해야합니다.
<tt style="color: #FF0000">`.to_html()`</tt> 및 <tt style="color: #FF0000">`.get_html_context()`</tt> 메소드는 컨트롤이 렌더링되는 방식을 추가로 customize하기 위해 custom pagination 클래스에서 재정의 될 수도 있습니다.


## Third party packages

다음의 third party packages가 사용 가능합니다.

### DRF-extensions

[<tt style="color: #FF0000">`DRF extensions`</tt> 패키지](http://chibisov.github.io/drf-extensions/docs/)에는 [<tt style="color: #FF0000">`PaginateByMaxMixin`</tt> mixin 클래스](http://chibisov.github.io/drf-extensions/docs/#paginatebymaxmixin)가 포함되어있어 API 클라이언트가 허용된 최대 페이지 크기를 얻기 위해 <tt style="color: #FF0000">`?page_size=max`</tt>를 지정할 수 있습니다.

### drf-proxy-pagination

[<tt style="color: #FF0000">`drf-proxy-pagination`</tt> 패키지](https://github.com/tuffnatty/drf-proxy-pagination)는 query parameter로 pagination 클래스를 선택할 수 있는 <tt style="color: #FF0000">`ProxyPagination`</tt> 클래스를 포함합니다.