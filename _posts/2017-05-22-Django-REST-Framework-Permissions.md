---
layout: post
title:  "Django REST Framework -Permissions"
date:   2017-05-22 12:43:59
author: Dean Kim
categories: Rest_Framework
tags:	Django Rest_framework
cover:  "/assets/instacode.png"
---

# Permissions
- 원본 : [공식문서](http://www.django-rest-framework.org/api-guide/permissions/)


정보 또는 코드에 대한 액세스 권한을 얻으려면 일반적으로 authorization 또는 identification만으로는 충분하지 않습니다. 이를 위해서는 액세스를 요청하는 entity에 authorization이 있어야 합니다.
"Apple Developer Documentation"

[authentication](http://www.django-rest-framework.org/api-guide/authentication/) 및 [throttling](http://www.django-rest-framework.org/api-guide/throttling/)과 함께 사용 권한은 request에 액세스를 허용할지 또는 거부 할지를 결정합니다.
Permission 검사는 다른 코드가 진행되기 전에 view의 맨 처음에 항상 실행됩니다. Permission 검사는 일반적으로 들어오는 request을 허용해야 하는지를 결정하기 위해 <tt style="color: #FF0000">`request.user`</tt> 및 <tt style="color: #FF0000">`request.auth`</tt> properties를 등록 정보의 인증 정보를 사용합니다.
<br>Permission은 다른 클래스의 사용자가 API의 다른 부분에 액세스하는 것을 허용하거나 거부하는 데 사용됩니다.
<br>가장 간단한 permission 스타일은 인증된 사용자에게 액세스를 허용하고 인증되지 않은 모든 사용자에 대한 액세스를 거부하는 것입니다. 이것은 REST Framework의 <tt style="color: #FF0000">`IsAuthenticated`</tt> 클래스에 해당합니다.
<br>약간 덜 엄격한 permission 스타일은 인증된 사용자에게 전체 액세스를 허용하지만 인증되지 않은 사용자에게는 읽기 전용 액세스를 허용하는 것입니다. 이것은 REST Framework의 <tt style="color: #FF0000">`IsAuthenticatedOrReadOnly`</tt> 클래스에 해당합니다.

### How permissions are determined

REST Framework의 permission은 항상 permission 클래스 목록으로 정의됩니다.
<br>뷰의 본문을 실행하기 전에 목록의 각 permission이 검사됩니다. permission 검사가 실패하면 <tt style="color: #FF0000">`exceptions.PermissionDenied`</tt> 또는 <tt style="color: #FF0000">`exceptions.NotAuthenticated`</tt> 예외가 발생하고 뷰 본문이 실행되지 않습니다.
<br>permission 검사가 실패하면 다음 규칙에 따라 "403 Forbidden"또는 "401 Unauthorized" response가 반환됩니다.
* request가 성공적으로 인증되었지만 permission이 거부되었습니다. - HTTP 403 Forbidden response가 반환됩니다.
* request가 성공적으로 인증되지 않았고 최상위 우선 순위 인증 클래스가 <tt style="color: #FF0000">`WWW-Authenticate`</tt> 헤더를 사용하지 않습니다. - HTTP 403 Forbidden response가 반환됩니다.
* request가 성공적으로 인증되지 않았고 최상위 우선 순위 인증 클래스는 <tt style="color: #FF0000">`WWW-Authenticate`</tt> 헤더를 사용합니다. - 적절한 <tt style="color: #FF0000">`WWW-Authenticate`</tt> 헤더가 있는 HTTP 401 Unauthorized response가 반환됩니다.

### Object level permissions

REST Framework permissions은 또한 object-level permissioning을 지원합니다. Object level permissions은 사용자가 특정 개체 (일반적으로 모델 인스턴스)에 대한 작업을 허용해야하는지 여부를 결정하는데 사용됩니다.
<br>Object level permissions은 <tt style="color: #FF0000">`.get_object()`</tt>가 호출될 때 REST Framework의 generic views에 의해 실행됩니다. view level permissions의 경우와 같이, 유저가 지정된 객체를 조작할 수 없는 경우는 <tt style="color: #FF0000">`exceptions.PermissionDenied`</tt> 예외가 Throw됩니다.
<br>자신만의 뷰를 작성하고 object level permissions을 적용하려는 경우 또는 generic views에서 <tt style="color: #FF0000">`get_object`</tt> 메소드를 재정의하려면 개체를 가져오는 시점 뷰에서 <tt style="color: #FF0000">`.check_object_permissions(request, obj)`</tt> 메소드를 명시적으로 호출해야합니다.
<br><tt style="color: #FF0000">`PermissionDenied`</tt> 또는 <tt style="color: #FF0000">`NotAuthenticated`</tt> 예외가 발생하거나 뷰에 적절한 permissions이 있는 경우 반환됩니다.
<br>예시:
~~~~
def get_object(self):
    obj = get_object_or_404(self.get_queryset())
    self.check_object_permissions(self.request, obj)
    return obj
~~~~
#### Limitations of object level permissions

성능상의 이유로 generic views는 오브젝트 목록을 리턴할 때 queryset의 각 인스턴스에 object level permissions을 자동으로 적용하지 않습니다.
<br>object level permissions을 사용하는 경우 종종 적절하게 [filter the queryset](http://www.django-rest-framework.org/api-guide/filtering/)하여 사용자가 볼 수 있는 인스턴스만 볼 수 있도록 해야합니다.

### Setting the permission policy

default permission 정책은 <tt style="color: #FF0000">`DEFAULT_PERMISSION_CLASSES`</tt> 설정을 사용하여 전역으로 설정할 수 있습니다. 예를 들어.
~~~~
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    )
}
~~~~
지정되지 않은 경우 이 설정은 defaults로 제한되지 않은 액세스를 허용합니다.
~~~~
'DEFAULT_PERMISSION_CLASSES': (
   'rest_framework.permissions.AllowAny',
)
~~~~
또한 <tt style="color: #FF0000">`APIView`</tt> class-based views를 사용하여 per-view 또는 per-viewset basis별로 인증 정책을 설정할 수 있습니다.
~~~~
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView

class ExampleView(APIView):
    permission_classes = (IsAuthenticated,)

    def get(self, request, format=None):
        content = {
            'status': 'request was permitted'
        }
        return Response(content)
~~~~
또는 function based views와 함께 <tt style="color: #FF0000">`@api_view`</tt> 데코레이터를 사용하는 경우.
~~~~
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response

@api_view(['GET'])
@permission_classes((IsAuthenticated, ))
def example_view(request, format=None):
    content = {
        'status': 'request was permitted'
    }
    return Response(content)
~~~~
<b>Note:</b> 클래스 속성이나 데코레이터를 통해 새 permission 클래스를 설정하면 <b>settings.py</b> 파일을 통해 설정된 기본 목록을 무시하도록 뷰에 지시합니다.


## API Reference

### AllowAny

<tt style="color: #FF0000">`AllowAny`</tt> permission 클래스는 <b>request가 인증되었거나 인증되지 않았는지 여부에 관계없이</b> 제한되지 않은 액세스를 허용합니다.
<br>사용 permission 설정에 빈 목록이나 튜플을 사용하여 동일한 결과를 얻을 수 있기 때문에 이 사용 permission은 반드시 필요한 것은 아니지만 의도를 명시적으로 지정하기 때문에 이 클래스를 지정하는 것이 유용할 수 있습니다.

### IsAuthenticated

<tt style="color: #FF0000">`IsAuthenticated`</tt> permission 클래스는 인증되지 않은 사용자에게 permission을 거부하고 그렇지 않은 경우에는 permission을 허용합니다.
<br>이 permission은 등록된 사용자만 API에 액세스할 수 있게 하려는 경우 적합합니다.

### IsAdminUser

<tt style="color: #FF0000">`IsAdminUser`</tt> permission 클래스는 <tt style="color: #FF0000">`user.is_staff`</tt>가 <tt style="color: #FF0000">`True`</tt>인 경우를 제외하고 모든 사용자에 대한 사용 permission을 거부합니다. 이 경우 사용 permission이 허용됩니다.
<br>이 permission은 신뢰할 수 있는 관리자의 하위 집합에서만 API에 액세스 할 수 있게 하려는 경우 적합합니다.

### IsAuthenticatedOrReadOnly

<tt style="color: #FF0000">`IsAuthenticatedOrReadOnly`</tt>를 사용하면 인증된 사용자가 모든 request를 수행할 수 있습니다. 권한이 없는 사용자에 대한 request는 request 방법이 "안전한"방법 중 하나일 경우에만 허용됩니다. <tt style="color: #FF0000">`GET`</tt>, <tt style="color: #FF0000">`HEAD`</tt> 또는 <tt style="color: #FF0000">`OPTIONS`</tt>.
<br>이 권한은 API에서 익명 사용자에게 읽기 권한을 허용하고 인증된 사용자에게만 쓰기 권한을 허용하려는 경우에 적합합니다.

### DjangoModelPermissions

이 permission 클래스는 장고의 표준 <tt style="color: #FF0000">`django.contrib.auth`</tt> [model permissions](https://docs.djangoproject.com/en/1.11/topics/auth/customizing/#custom-permissions)과 관련이있다. 이 permission은 <tt style="color: #FF0000">`.queryset`</tt> 속성이 설정된 보기에만 적용되어야합니다. 권한 부여는 사용자가 인증되고 관련 모델 permission이 할당된 경우에만 부여됩니다.
* <tt style="color: #FF0000">`POST`</tt> 요청을 사용하려면 사용자에게 모델에 대한 <tt style="color: #FF0000">`add`</tt> permission이 있어야 합니다.
* <tt style="color: #FF0000">`PUT`</tt> 및 <tt style="color: #FF0000">`PATCH`</tt> 요청은 사용자가 모델에 대한 <tt style="color: #FF0000">`change`</tt> permission을 요구합니다.
* <tt style="color: #FF0000">`DELETE`</tt> 요청은 사용자에게 모델에 대한 <tt style="color: #FF0000">`delete`</tt> permission이 필요합니다.
<br>default 동작을 재정의하여 custom 모델 permission을 지원할 수도 있습니다. 예를 들어 <tt style="color: #FF0000">`GET`</tt> 요청에 대한보기 모델 permission을 포함할 수 있습니다.
<br>custom 모델 permission을 사용하려면 <tt style="color: #FF0000">`DjangoModelPermissions`</tt>를 재정의하고 <tt style="color: #FF0000">`.perms_map`</tt> 속성을 설정하십시오. 자세한 내용은 소스 코드를 참조하십시오.

#### Using with views that do not include a `queryset` attribute.

재정의된 <tt style="color: #FF0000">`get_queryset()`</tt> 메서드를 사용하는 보기에서 이 permission을 사용하는 경우 보기에 <tt style="color: #FF0000">`queryset`</tt> 특성이 없을 수 있습니다. 이 경우에는 sentinel 쿼리 세트로 뷰를 표시하여 이 클래스가 필요한 permission을 결정할 수 있도록 하는 것이 좋습니다. 예 :
~~~~
queryset = User.objects.none()  # Required for DjangoModelPermissions
~~~~

### DjangoModelPermissionsOrAnonReadOnly

<tt style="color: #FF0000">`DjangoModelPermissions`</tt>와 유사하지만 인증되지 않은 사용자는 API에 대한 read-only 액세스만 허용합니다.

### DjangoObjectPermissions

이 퍼미션 클래스는 모델에 대한 객체별 permission을 허용하는 장고의 표준 [object permissions framework](https://docs.djangoproject.com/en/1.11/topics/auth/customizing/#handling-object-permissions)와 관련이있다. 이 permission 클래스를 사용하려면 [django-guardian](https://github.com/django-guardian/django-guardian)과 같은 object-level permissions을 지원하는 permission 백엔드도 추가해야합니다.
<br><tt style="color: #FF0000">`DjangoModelPermissions`</tt>와 마찬가지로 이 permission은 <tt style="color: #FF0000">`.queryset`</tt> 속성 또는 <tt style="color: #FF0000">`.get_queryset()`</tt> 메소드가 있는 보기에만 적용되어야 합니다. 권한 부여는 사용자가 인증되고 relevant per-object permissions 및 relevant model permissions이 할당된 경우에만 부여됩니다.
* <tt style="color: #FF0000">`POST`</tt> requests는 사용자에게 모델 인스턴스에 대한 추가 권한이 필요합니다.
* <tt style="color: #FF0000">`PUT`</tt> 및 <tt style="color: #FF0000">`PATCH`</tt> requests는 사용자가 모델 인스턴스에 대한 <tt style="color: #FF0000">`change`</tt> permission을 요구합니다.
* <tt style="color: #FF0000">`DELETE`</tt> requests는 사용자에게 모델 인스턴스에 대한 <tt style="color: #FF0000">`delete`</tt> permission이 있어야합니다.
<br><tt style="color: #FF0000">`DjangoObjectPermissions`</tt>는 <tt style="color: #FF0000">`django-guardian`</tt> 패키지를 <b>필요로하지 않으며</b> 다른 object-level backends도 똑같이 잘 지원해야합니다.
<br><tt style="color: #FF0000">`DjangoModelPermissions`</tt>와 마찬가지로 <tt style="color: #FF0000">`DjangoObjectPermissions`</tt>를 재정의하고 <tt style="color: #FF0000">`.perms_map`</tt> 속성을 설정하여 custom 모델 permissions을 사용할 수 있습니다. 자세한 내용은 소스 코드를 참조하십시오.
<br><b>Note:</b> <tt style="color: #FF0000">`GET`</tt>, <tt style="color: #FF0000">`HEAD`</tt> 및 <tt style="color: #FF0000">`OPTIONS`</tt> requests에 대한 object level <tt style="color: #FF0000">`view`</tt> permissions이 필요한 경우, <tt style="color: #FF0000">`DjangoObjectPermissionsFilter`</tt> 클래스를 추가하여 목록 엔드 포인트가 사용자에게 적절한 view permissions이 있는 오브젝트를 포함하여 결과만 리턴하도록 해야합니다.


## Custom permissions

custom permission을 구현하려면, <tt style="color: #FF0000">`BasePermission`</tt>를 재정의해, 다음의 메소드의 어느 쪽인지, 또는 양쪽 모두를 구현합니다.
* <tt style="color: #FF0000">`.has_permission(self, request, view)`</tt>
* <tt style="color: #FF0000">`.has_object_permission(self, request, view, obj)`</tt>
<br>request에 액세스 권한이 부여되면 메서드는 <tt style="color: #FF0000">`True`</tt>를 반환하고 그렇지 않으면 <tt style="color: #FF0000">`False`</tt>를 반환해야합니다.
<br>request가 읽기 작업인지 아니면 쓰기 작업인지 테스트해야하는 경우 <tt style="color: #FF0000">`'GET'`</tt>, <tt style="color: #FF0000">`'OPTIONS'`</tt>및 <tt style="color: #FF0000">`'HEAD'`</tt>가 포함된 튜플인 <tt style="color: #FF0000">`SAFE_METHODS`</tt> 상수와 비교하여 request 메소드를 확인해야합니다. 예 :
~~~~
if request.method in permissions.SAFE_METHODS:
    # Check permissions for read-only request
else:
    # Check permissions for write request
~~~~
<b>Note:</b> view-level <tt style="color: #FF0000">`has_permission`</tt> 검사가 이미 통과된 경우에만 인스턴스 수준의 <tt style="color: #FF0000">`has_object_permission`</tt> 메소드가 호출됩니다. 또한 인스턴스 수준 검사를 실행하려면 보기 코드에서 <tt style="color: #FF0000">`.check_object_permissions(request, obj)`</tt>를 명시적으로 호출해야 합니다. generic views를 사용하는 경우 default로 이 옵션이 처리됩니다.

테스트가 실패할 경우 Custom permissions은 <tt style="color: #FF0000">`PermissionDenied`</tt> 예외를 발생시킵니다. 예외와 관련된 오류 메시지를 변경하려면 Custom permissions에 직접 <tt style="color: #FF0000">`message`</tt> 특성을 구현하십시오. 그렇지 않으면 <tt style="color: #FF0000">`PermissionDenied`</tt>의 <tt style="color: #FF0000">`default_detail`</tt> 특성이 사용됩니다.
~~~~
from rest_framework import permissions

class CustomerAccessPermission(permissions.BasePermission):
    message = 'Adding customers not allowed.'

    def has_permission(self, request, view):
         ...
~~~~

### Example

다음은 들어오는 request의 IP 주소를 블랙리스트와 비교하여 검사하고 IP가 블랙리스트에 올랐으면 request를 거부하는 permission 클래스의 예입니다.
~~~~
from rest_framework import permissions

class BlacklistPermission(permissions.BasePermission):
    """
    Global permission check for blacklisted IPs.
    """

    def has_permission(self, request, view):
        ip_addr = request.META['REMOTE_ADDR']
        blacklisted = Blacklist.objects.filter(ip_addr=ip_addr).exists()
        return not blacklisted
~~~~
들어오는 모든 requests에 ​​대해 실행되는 전역 permissions뿐만 아니라 특정 개체 인스턴스에 영향을 주는 작업에 대해서만 실행되는 object-level permissions을 만들 수도 있습니다. 예 :
~~~~
class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Object-level permission to only allow owners of an object to edit it.
    Assumes the model instance has an `owner` attribute.
    """

    def has_object_permission(self, request, view, obj):
        # Read permissions are allowed to any request,
        # so we'll always allow GET, HEAD or OPTIONS requests.
        if request.method in permissions.SAFE_METHODS:
            return True

        # Instance must have an attribute named `owner`.
        return obj.owner == request.user
~~~~
generic views는 적절한 object level permissions을 검사하지만 custom views를 작성하는 경우 object level permission 검사를 직접 확인해야합니다. 객체 인스턴스가 있으면 뷰에서 <tt style="color: #FF0000">`self.check_object_permissions(request, obj)`</tt>를 호출하여 그렇게 할 수 있습니다. object-level permission의 체크가 실패했을 경우, 이 호출은 적절한 <tt style="color: #FF0000">`APIException`</tt>를 발생시킵니다.
<br>또한 generic views는 단일 모델 인스턴스를 검색하는 뷰에 대한 object-level permissions만 검사합니다. 목록 뷰의 object-level filtering이 필요한 경우 별도로 쿼리 세트를 필터링해야합니다. 자세한 내용은 [filtering documentation](http://www.django-rest-framework.org/api-guide/filtering/)를 참조하십시오.


## Third party packages

다음의 third party package를 사용 가능합니다.

### Composed Permissions

[Composed Permissions](https://github.com/niwinz/djangorestframework-composed-permissions) 패키지는 작고 재사용 가능한 구성 요소를 사용하여 복잡한 객체와 multi-depth(with logic operators) permission 객체를 정의하는 간단한 방법을 제공합니다.

### REST Condition

[REST Condition](https://github.com/caxap/rest_condition) 패키지는 간단하고 편리한 방법으로 복잡한 permissions을 구축하기 위한 또 다른 확장입니다. 이 확장을 통해 사용 permissions을 논리 연산자와 결합할 수 있습니다.

### DRY Rest Permissions

[DRY Rest Permissions](https://github.com/dbkaplan/dry-rest-permissions) 패키지는 개별 기본 동작과 custom 동작에 대해 서로 다른 사용 permissions을 정의할 수 있는 기능을 제공합니다. 이 패키지는 앱의 데이터 모델에 정의된 관계에서 파생된 permissions을 가진 앱을 위해 만들어졌습니다. 또한 API serializer를 통해 클라이언트 응용 프로그램에 반환되는 permissions 검사도 지원합니다. 또한 기본 및 custom 목록 작업에 사용 permissions을 추가하여 사용자당 검색하는 데이터를 제한할 수 있습니다.

### Django Rest Framework Roles

[Django Rest Framework Roles](https://github.com/computer-lab/django-rest-framework-roles) 패키지를 사용하면 여러 유형의 사용자에 대해 API를 쉽게 parameterize 할 수 있습니다.

### Django Rest Framework API Key

[Django Rest Framework API Key](https://github.com/manosim/django-rest-framework-api-key) 패키지를 사용하면 서버에 대한 모든 request에 ​​API 키 헤더가 필요하도록 할 수 있습니다. 장고 admin 인터페이스에서 생성할 수 있습니다.