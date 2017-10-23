---
layout: post
title:  "Django REST Framework -Views"
date:   2017-05-22 13:43:59
author: Dean Kim
categories: Rest_Framework
tags:	Django Rest_framework
cover:  "/assets/instacode.png"
---

# Views
- 원본 : [공식문서](http://www.django-rest-framework.org/api-guide/views/)


Django의 class-based 뷰는 old-style 뷰에서 출발하는 것을 welcome합니다.
"[Reinout van Rees](http://reinout.vanrees.org/weblog/2011/08/24/class-based-views-usage.html)"

REST framework는 Django의 <tt style="color: #FF0000">`View`</tt> 클래스의 하위 클래스로 하는 <tt style="color: #FF0000">`APIView`</tt> 클래스를 제공합니다.
APIView 클래스는 다음과 같은 방식으로 일반 View 클래스와 다릅니다.
* 핸들러 메소드에 전달 된 Requests은 Django의 <tt style="color: #FF0000">`HttpRequest`</tt> 인스턴스가 아닌 REST framework의 <tt style="color: #FF0000">`Request`</tt> 인스턴스가됩니다.
* 핸들러 메서드는 Django의 <tt style="color: #FF0000">`HttpResponse`</tt> 대신 REST framework의 <tt style="color: #FF0000">`Response`</tt>를 반환할 수 있습니다. 뷰는 content negotiation을 관리하고 response에서 올바른 렌더러를 설정합니다.
* 모든 <tt style="color: #FF0000">`APIException`</tt> 예외가 발견되어 적절한 responses으로 조정됩니다.
* 들어오는 requests는 인증되고 적절한 permission and/or throttle 확인이 handler 메서드에 request을 보내기 전에 실행됩니다.

<tt style="color: #FF0000">`APIView`</tt> 클래스를 사용하는 것은 일반 <tt style="color: #FF0000">`View`</tt> 클래스를 사용하는 것과 거의 같습니다. 들어오는 request은 <tt style="color: #FF0000">`.get()`</tt> 또는 <tt style="color: #FF0000">`.post()`</tt>와 같은 적절한 핸들러 메소드로 전달됩니다. 또한 API policy의 다양한 aspects를 제어하는 ​​여러 attributes를 클래스에 설정할 수 있습니다.
예 : 
~~~~
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import authentication, permissions

class ListUsers(APIView):
    """
    View to list all users in the system.

    * Requires token authentication.
    * Only admin users are able to access this view.
    """
    authentication_classes = (authentication.TokenAuthentication,)
    permission_classes = (permissions.IsAdminUser,)

    def get(self, request, format=None):
        """
        Return a list of all users.
        """
        usernames = [user.username for user in User.objects.all()]
        return Response(usernames)
~~~~


## API policy attributes

다음 attributes는 API views의 pluggable aspects를 제어합니다.

* <b>.renderer_classes</b>
* <b>.parser_classes</b>
* <b>.authentication_classes</b>
* <b>.throttle_classes</b>
* <b>.permission_classes</b>
* <b>.content_negotiation_class</b>


## API policy instantiation methods

다음 메소드는 REST framework에서 다양한 pluggable API policies를 인스턴스화 하는데 사용됩니다. 일반적으로 이러한 메서드를 재정의할 필요는 없습니다.

* <b>.get_renderers(self)</b>
* <b>.get_parsers(self)</b>
* <b>.get_authenticators(self)</b>
* <b>.get_throttles(self)</b>
* <b>.get_permissions(self)</b>
* <b>.get_content_negotiator(self)</b>
* <b>.get_exception_handler(self)</b>


## API policy implementation methods

다음 메서드는 핸들러 메서드에 전달하기 전에 호출됩니다.

* <b>.check_permissions(self, request)</b>
* <b>.check_throttles(self, request)</b>
* <b>.perform_content_negotiation(self, request, force=False)</b>


## Dispatch methods

다음 메소드는 뷰의 <tt style="color: #FF0000">`.dispatch()`</tt> 메소드에 의해 직접 호출됩니다. 이들은 <tt style="color: #FF0000">`.get()`</tt>, <tt style="color: #FF0000">`.post()`</tt>, <tt style="color: #FF0000">`put()`</tt>, <tt style="color: #FF0000">`patch()`</tt> 및 <tt style="color: #FF0000">`.delete()`</tt>와 같은 핸들러 메소드를 호출하기 전후에 수행되어야 하는 모든 조치를 수행합니다.

### .initial(self, request, *args, **kwargs)

핸들러 메서드가 호출되기 전에 발생해야 하는 모든 작업을 수행합니다. 이 메서드는 사용 권한 및 제한을 적용하고 content negotiation을 수행하는데 사용됩니다.
일반적으로 이 메서드를 재정의할 필요는 없습니다.

### .handle_exception(self, exc)

핸들러 메서드에 의해 throw된 예외는 Response 인스턴스를 반환하거나 예외를 다시 발생시키는 이 메서드로 전달됩니다.
default implementation에서는 장고의 <tt style="color: #FF0000">`Http404`</tt> 및 <tt style="color: #FF0000">`PermissionDenied`</tt> 예외 뿐만 아니라 <tt style="color: #FF0000">`rest_framework.exceptions.APIException`</tt>의 하위 클래스를 처리하고 적절한 error response을 반환합니다.
API에서 반환하는 error responses을 customize해야하는 경우 이 메소드를 서브 클래스화 해야합니다.

### .initialize_request(self, request, *args, **kwargs)

핸들러 메소드에 전달된 request 객체가 일반적인 Django <tt style="color: #FF0000">`HttpRequest`</tt>가 아닌 <tt style="color: #FF0000">`Request`</tt>의 인스턴스인지 확인합니다.
일반적으로 이 메서드를 재정의할 필요는 없습니다.

### .finalize_response(self, request, response, *args, **kwargs)

핸들러 메서드에서 반환된 <tt style="color: #FF0000">`Response`</tt> 객체가 content negotiation에 의해 결정된대로 올바른 내용 유형으로 렌더링되도록 합니다.
일반적으로 이 메서드를 재정의할 필요는 없습니다.


## Function Based Views

(그 클래스 기반의 view들)이 항상 우월한 해결책은 실수라고 말하라.
"Nick Coghlan"

REST framework를 사용하면 regular function based views로 작업 할 수 있습니다. 그것은 간단한 Django <tt style="color: #FF0000">`HttpRequest`</tt>가 아닌 <tt style="color: #FF0000">`Request`</tt>의 인스턴스를 수신하고 Django <tt style="color: #FF0000">`HttpResponse`</tt> 대신 <tt style="color: #FF0000">`Response`</tt>을 리턴할 수 있도록 function based views를 래핑하는 간단한 데코레이터 세트를 제공하며, request가 처리됩니다.

### @api_view()

<b>Signature:</b> <tt style="color: #FF0000">`@api_view(http_method_names=['GET'], exclude_from_schema=False)`</tt>

이 기능의 핵심은 뷰가 응답해야하는 HTTP 메소드 목록을 사용하는 <tt style="color: #FF0000">`api_view`</tt> 데코레이터입니다. 예를 들어, 다음은 몇 가지 데이터를 수동으로 반환하는 매우 simple view를 작성하는 방법입니다.
~~~~
from rest_framework.decorators import api_view

@api_view()
def hello_world(request):
    return Response({"message": "Hello, world!"})
~~~~
이 뷰는 [Settings](http://www.django-rest-framework.org/api-guide/settings/)에 지정된 default renderers, parsers, authentication 클래스 등을 사용합니다.
기본적으로 <tt style="color: #FF0000">`GET`</tt> 메소드만 허용됩니다. 다른 방법은 "405 Method Not Allowed"로 응답합니다. 이 동작을 변경하려면 다음 보기에서 허용하는 방법을 지정하십시오.
~~~~
@api_view(['GET', 'POST'])
def hello_world(request):
    if request.method == 'POST':
        return Response({"message": "Got some data!", "data": request.data})
    return Response({"message": "Hello, world!"})
~~~~

<tt style="color: #FF0000">`exclude_from_schema`</tt> argument를 사용하여 API 뷰를 [auto-generated schema](http://www.django-rest-framework.org/api-guide/schemas/)에서 생략된 것으로 표시할 수도 있습니다.
~~~~
@api_view(['GET'], exclude_from_schema=True)
def api_docs(request):
    ...
~~~~


## API policy decorators

기본 설정을 재정의하기 위해 REST framework는 뷰에 추가 할 수 있는 일련의 추가 데코레이터를 제공합니다. 이들은 <tt style="color: #FF0000">`@api_view`</tt> 데코레이터 다음(below)에 와야합니다. 예를 들어 [throttle](http://www.django-rest-framework.org/api-guide/throttling/)을 사용하여 특정 사용자가 하루에 한 번만 호출 할 수 있도록 뷰를 만들려면 <tt style="color: #FF0000">`@throttle_classes`</tt> 데코레이터를 사용하여 throttle 클래스 목록을 전달합니다.
~~~~
from rest_framework.decorators import api_view, throttle_classes
from rest_framework.throttling import UserRateThrottle

class OncePerDayUserThrottle(UserRateThrottle):
        rate = '1/day'

@api_view(['GET'])
@throttle_classes([OncePerDayUserThrottle])
def view(request):
    return Response({"message": "Hello for today! See you tomorrow!"})
~~~~
이러한 데코레이터는 위에서 설명한 <tt style="color: #FF0000">`APIView`</tt> 하위 클래스에 설정된 attributes에 해당합니다.
사용 가능한 데코레이터는 다음과 같습니다.
* <tt style="color: #FF0000">`@renderer_classes(...)`</tt>
* <tt style="color: #FF0000">`@parser_classes(...)`</tt>
* <tt style="color: #FF0000">`@authentication_classes(...)`</tt>
* <tt style="color: #FF0000">`@throttle_classes(...)`</tt>
* <tt style="color: #FF0000">`@permission_classes(...)`</tt>