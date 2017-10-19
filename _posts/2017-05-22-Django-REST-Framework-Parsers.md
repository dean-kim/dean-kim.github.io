---
layout: post
title:  "Django REST Framework -Parsers"
date:   2017-05-22 11:43:59
author: Dean Kim
categories: Rest_Framework
tags:	Django Rest_framework
cover:  "/assets/instacode.png"
---

# Parsers
- 원본 : [공식문서](http://www.django-rest-framework.org/api-guide/parsers/)


웹 서비스를 상호 작용하는 기계는 단순 형식보다 복잡한 데이터를 전송하기 때문에 form-encoded보다 데이터 전송에 더 많은 구조화된 형식을 사용하는 경향이 있습니다.
"Malcom Tredinnick"

REST framework에는 Parser 클래스가 내장되어있어 다양한 미디어 유형으로 requests를 수락할 수 있습니다. 또한 custom 파서를 정의할 수 있으므로 API에서 허용하는 미디어 유형을 유연하게 디자인할 수 있습니다.

### How the parser is determined

View에 대한 valid 파서 세트는 항상 클래스 목록으로 정의됩니다. <tt style="color: #FF0000">`request.data`</tt>에 액세스하면 REST framework는 들어오는 request의 <tt style="color: #FF0000">`Content-Type`</tt> 헤더를 검사하고 request 내용을 구문 분석하는데 사용할 구문 분석기를 결정합니다.
<b>Note:</b> 클라이언트 applications을 개발할 때는 항상 HTTP 요청으로 데이터를 보낼 때 <tt style="color: #FF0000">`Content-Type`</tt> 헤더를 설정해야합니다.
콘텐츠 유형을 설정하지 않으면 대부분의 클라이언트는 <tt style="color: #FF0000">`'application / x-www-form-urlencoded'`</tt>을 default로 사용합니다. 이는 원하지 않을 수 있습니다.
예를 들어 [.ajax() method](http://api.jquery.com/jQuery.ajax/)로 jQuery를 사용하여 json으로 인코딩된 데이터를 보내는 경우 <tt style="color: #FF0000">`contentType: 'application/json'`</tt>설정을 포함해야합니다.


### Setting the parsers

파서의 기본 세트는 <tt style="color: #FF0000">`DEFAULT_PARSER_CLASSES`</tt> 설정을 사용하여 전역으로 설정할 수 있습니다. 예를 들어, 다음 설정은 default JSON 또는 양식 데이터 대신 <tt style="color: #FF0000">`JSON`</tt> content가 있는 requests만 허용합니다.
~~~~
REST_FRAMEWORK = {
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework.parsers.JSONParser',
    )
}
~~~~
APIView class-based views를 사용하여 individual view 또는 viewset에 사용되는 파서를 설정할 수도 있습니다.
~~~~
from rest_framework.parsers import JSONParser
from rest_framework.response import Response
from rest_framework.views import APIView

class ExampleView(APIView):
    """
    A view that can accept POST requests with JSON content.
    """
    parser_classes = (JSONParser,)

    def post(self, request, format=None):
        return Response({'received data': request.data})
~~~~
또는 function based views와 함께 <tt style="color: #FF0000">`@api_view`</tt> 데코레이터를 사용하는 경우.
~~~~
from rest_framework.decorators import api_view
from rest_framework.decorators import parser_classes

@api_view(['POST'])
@parser_classes((JSONParser,))
def example_view(request, format=None):
    """
    A view that can accept POST requests with JSON content.
    """
    return Response({'received data': request.data})
~~~~


## API Reference

### JSONParser

Parses <tt style="color: #FF0000">`JSON`</tt> request content.
<b>.media_type:</b> <tt style="color: #FF0000">`application/json`</tt>

### FormParser

HTML 양식 내용을 파싱합니다. <tt style="color: #FF0000">`request.data`</tt>는 데이터의 <tt style="color: #FF0000">`QueryDict`</tt>로 채워집니다.
일반적으로 HTML 양식 데이터를 완벽하게 지원하기위해 <tt style="color: #FF0000">`FormParser`</tt>와 <tt style="color: #FF0000">`MultiPartParser`</tt>를 함께 사용하려고합니다.
<b>.media_type:</b> <tt style="color: #FF0000">`application/x-www-form-urlencoded`</tt>

### MultiPartParser

파일 업로드를 지원하는 multipart HTML 양식 콘텐츠를 파싱합니다. 두 <tt style="color: #FF0000">`request.data`</tt> 모두 <tt style="color: #FF0000">`QueryDict`</tt>로 채워집니다.
일반적으로 HTML 양식 데이터를 완벽하게 지원하기위해 <tt style="color: #FF0000">`FormParser`</tt>와 <tt style="color: #FF0000">`MultiPartParser`</tt>를 함께 사용하려고합니다.
<b>.media_type:</b> <tt style="color: #FF0000">`multipart/form-data`</tt>

### FileUploadParser

raw 파일 업로드 내용을 구문 분석합니다. <tt style="color: #FF0000">`request.data`</tt> property은 업로드된 파일을 포함하는 single key <tt style="color: #FF0000">`'file'`</tt>이 포함된 dictionary입니다.
<tt style="color: #FF0000">`FileUploadParser`</tt>와 함께 사용된 뷰가 <tt style="color: #FF0000">`filename`</tt> URL 키워드 argument로 호출되면 해당 argument가 filename으로 사용됩니다.
<tt style="color: #FF0000">`filwname`</tt> URL 키워드 argument없이 호출되면 클라이언트는 <tt style="color: #FF0000">`Content-Disposition`</tt> HTTP 헤더에 filename을 설정해야합니다. 예를 들면 <tt style="color: #FF0000">`Content-Disposition: attachment; filename=upload.jpg`</tt>.
<b>.media_type:</b> <tt style="color: #FF0000">`*/*`</tt>
</br>Note: 
* <tt style="color: #FF0000">`FileUploadParser`</tt>는 파일을 raw 데이터 request로 업로드할 수 있는 native 클라이언트에서 사용하기 위한 것입니다. 웹 기반 업로드 또는 multipart 업로드가 지원되는 native 클라이언트의 경우 <tt style="color: #FF0000">`MultiPartParser`</tt> 파서를 대신 사용해야합니다.
* 이 파서의 <tt style="color: #FF0000">`media_type`</tt>은 모든 콘텐츠 형식과 일치하므로 <tt style="color: #FF0000">`FileUploadParser`</tt>는 일반적으로 API view에 설정된 유일한 파서여야합니다.
* <tt style="color: #FF0000">`FileUploadParser`</tt>는 Django의 표준 <tt style="color: #FF0000">`FILE_UPLOAD_HANDLERS`</tt> 설정과 <tt style="color: #FF0000">`request.upload_handlers`</tt> attribute를 고려합니다. 자세한 내용은 [Django 문서](https://docs.djangoproject.com/en/1.11/topics/http/file-uploads/#upload-handlers)를 참조하십시오.

Basic usage example:
~~~~
# views.py
class FileUploadView(views.APIView):
    parser_classes = (FileUploadParser,)

    def put(self, request, filename, format=None):
        file_obj = request.data['file']
        # ...
        # do some stuff with uploaded file
        # ...
        return Response(status=204)

# urls.py
urlpatterns = [
    # ...
    url(r'^upload/(?P<filename>[^/]+)$', FileUploadView.as_view())
]
~~~~


## Custom parsers

custom parser를 implement하려면 <tt style="color: #FF0000">`BaseParser`</tt>를 오버라이드하고 <tt style="color: #FF0000">`.media_type`</tt> property을 설정하고 <tt style="color: #FF0000">`.parse(self, stream, media_type, parser_context)`</tt> 메소드를 implement해야합니다.
메서드는 <tt style="color: #FF0000">`request.data`</tt> property를 채우는 데 사용할 데이터를 반환해야합니다.
<tt style="color: #FF0000">`.parse()`</tt>에 전달된 arguments는 다음과 같습니다.

#### stream

request의 body를 나타내는 스트림과 같은 representing object입니다.

#### media_type

Optional. 제공되는 경우 들어오는 request 콘텐츠의 미디어 유형입니다.
request의 <tt style="color: #FF0000">`Content-Type`</tt>에 따라 : 헤더에 따라 렌더러의 <tt style="color: #FF0000">`media_type`</tt> attribute보다 더 구체적일 수 있으며 미디어 유형 parameters가 포함될 수 있습니다. 예: <tt style="color: #FF0000">`"text/plain; charset=utf-8"`</tt>

#### parser_context

Optional. 이 argument가 제공되면 request 내용을 구문 분석하는 데 필요할 수 있는 추가 컨텍스트를 포함하는 dictionary가 됩니다.
default로 이 키에는 <tt style="color: #FF0000">`view`</tt>, <tt style="color: #FF0000">`request`</tt>, <tt style="color: #FF0000">`args`</tt>, <tt style="color: #FF0000">`kwargs`</tt> 키가 포함됩니다.

### Example

다음은 request 본문을 나타내는 문자열로 <tt style="color: #FF0000">`request.data`</tt> property을 채우는 일반 텍스트 파서의 예입니다.
~~~~
class PlainTextParser(BaseParser):
    """
    Plain text parser.
    """
    media_type = 'text/plain'

    def parse(self, stream, media_type=None, parser_context=None):
        """
        Simply return a string representing the body of the request.
        """
        return stream.read()
~~~~


## Third party packages

다음의 third party packages들이 이용 가능합니다.

### YAML

[REST framework YAML](http://jpadilla.github.io/django-rest-framework-yaml/)은 [YAML](http://www.yaml.org/) 파싱 및 렌더링 지원을 제공합니다. 이전에 REST framework 패키지에 직접 포함되어 있었으며 현재는 third-party package로 지원됩니다.

#### Installation & configuration

install using pip
~~~~
$ pip install djangorestframework-yaml
~~~~

REST framework settings를 수정하십시오.
~~~~
REST_FRAMEWORK = {
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework_yaml.parsers.YAMLParser',
    ),
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework_yaml.renderers.YAMLRenderer',
    ),
}
~~~~

### XML

[REST framework XML](http://jpadilla.github.io/django-rest-framework-xml/)은 간단한 비공식 XML 형식을 제공합니다. 이전에 REST framework 패키지에 직접 포함되어 있었으며 현재는 third-party package로 지원됩니다.

#### Installation & configuration

install using pip
~~~~
$ pip install djangorestframework-xml
~~~~

REST framework settings를 수정하십시오.
~~~~
REST_FRAMEWORK = {
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework_xml.parsers.XMLParser',
    ),
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework_xml.renderers.XMLRenderer',
    ),
}
~~~~

### MessagePack

[MessagePack](https://github.com/juanriaza/django-rest-framework-msgpack)은 빠르고 효율적인 binary serialization format입니다. [Juan Riaza](https://github.com/juanriaza)는 MessagePack 렌더러와 REST framework에 대한 파서 지원을 제공하는 [djangorestframework-msgpack](https://github.com/juanriaza/django-rest-framework-msgpack) 패키지를 유지 관리합니다.

### CamelCase JSON

[djangorestframework-camel-case](https://github.com/vbabiy/djangorestframework-camel-case)는 REST framework용 camel case JSON 렌더러와 파서를 제공합니다. 이를 통해 serializers는 Python-style의 underscored 필드 이름을 사용할 수 있지만 Javascript-style의 camel case 이름으로 API에 표시됩니다. 그것은 [Vitaly Babiy](https://github.com/vbabiy)에 의해 관리됩니다.
