---
layout: post
title:  "Django REST Framework -Responses"
date:   2017-05-12 12:43:59
author: Dean Kim
categories: Rest_Framework
tags:	Django Rest_framework
cover:  "/assets/instacode.png"
---

# Responses

기본 HttpResponse 객체와 달리 TemplateResponse 객체는 response를 처리하기 위해 뷰에서 제공 한 컨텍스트의 세부 정보를 유지합니다. response의 최종 출력은 나중에 response 프로세스에서 필요할 때까지 계산되지 않습니다.
"Django documentation"

REST framework는 클라이언트 요청에 따라 여러 콘텐츠 형식으로 렌더링 할 수 있는 콘텐츠를 반환 할 수있는 <tt style="color: #FF0000">`Response`</tt> 클래스를 제공하여 HTTP 콘텐츠 negotiation을 지원합니다.
<tt style="color: #FF0000">`Response`</tt> 클래스는 Django의 <tt style="color: #FF0000">`SimpleTemplateResponse`</tt> 하위 클래스입니다. <tt style="color: #FF0000">`Response`</tt> 객체는 native Python primitives로 구성되어야하는 데이터로 초기화됩니다. 그런 다음 REST framework는 표준 HTTP 내용 협상을 사용하여 최종 response 내용을 렌더링하는 방법을 결정합니다.
<tt style="color: #FF0000">`Response`</tt> 클래스를 사용할 필요는 없으며 필요에 따라 뷰에서 일반 <tt style="color: #FF0000">`HttpResponse`</tt> 또는 <tt style="color: #FF0000">`StreamingHttpResponse`</tt> 객체를 반환할 수도 있습니다. <tt style="color: #FF0000">`Response`</tt> 클래스를 사용하면 여러 가지 형식으로 렌더링 할 수있는 content-negotiated 웹 API responses을 반환하기에 더 좋은 인터페이스만 제공됩니다.
어떤 이유로 든 REST framework를 heavily customize하지 않으려면 tt style="color: #FF0000">`Response`</tt> 객체를 반환하는 보기에 항상 <tt style="color: #FF0000">`APIView`</tt> 클래스 또는 <tt style="color: #FF0000">`@api_view`</tt> 함수를 사용해야합니다. 이렇게하면 보기에서 content negotiation을 수행하고 response에 적합한 renderer를 선택하여 보기에서 반환 할 수 있습니다.


## Creating responses

### Response()

Signature : <tt style="color: #FF0000">`Response(data, status=None, template_name=None, headers=None, content_type=None)`</tt>
일반 <tt style="color: #FF0000">`HttpResponse`</tt> 개체와 달리 렌더링 된 콘텐츠로 <tt style="color: #FF0000">`Response`</tt> 개체를 인스턴스화하지 않습니다. 대신 Python primitives로 구성된 렌더링되지 않은 데이터를 전달합니다.
<tt style="color: #FF0000">`Response`</tt> 클래스에서 사용하는 렌더러는 Django 모델 인스턴스와 같은 복잡한 데이터 유형을 기본적으로 처리 할 수 ​​없으므로 <tt style="color: #FF0000">`Response`</tt> 객체를 만들기 전에 데이터를 원시 데이터 유형으로 serialize해야합니다.
REST framework의 <tt style="color: #FF0000">`Serializer`</tt> 클래스를 사용하여 이 데이터 serialization을 수행하거나 custom serialization을 사용할 수 있습니다.
Arguments:
* <tt style="color: #FF0000">`data`</tt> : The serialized data for the response.
* <tt style="color: #FF0000">`status`</tt> : response의 상태 코드입니다. Defaults 200입니다. [상태 코드를 참조](http://www.django-rest-framework.org/api-guide/status-codes/)
* <tt style="color: #FF0000">`template_name`</tt> : <tt style="color: #FF0000">`HTMLRenderer`</tt>가 선택된 경우 사용할 template 이름입니다.
* <tt style="color: #FF0000">`headers`</tt> : response에 사용할 HTTP 헤더 dictionary입니다.
* <tt style="color: #FF0000">`content_type`</tt> : response의 content 유형입니다. 일반적으로 content negotiation에 따라 렌더러에서 자동으로 설정되지만 content 유형을 명시적으로 지정해야하는 경우가 있습니다.


## Attributes

### .data

<tt style="color: #FF0000">`Request`</tt> 객체의 렌더링되지 않은 content입니다.

### .status_code

HTTP response의 숫자 상태 코드입니다.

### .content

response의 렌더링된 content입니다. <tt style="color: #FF0000">`.content`</tt>에 액세스하려면 먼저 <tt style="color: #FF0000">`.render()`</tt> 메서드를 호출해야합니다.

### .template_name

<tt style="color: #FF0000">`template_name`</tt>이 제공된 경우, <tt style="color: #FF0000">`HTMLRenderer`</tt> 또는 다른 custom template renderer가 response에 대해 허용된 renderer인 경우에만 필요합니다.

### .accepted_renderer

response을 렌더링하는 데 사용되는 renderer 인스턴스입니다.
뷰에서 response이 반환되기 직전에 <tt style="color: #FF0000">`APIView`</tt> 또는 <tt style="color: #FF0000">`@api_view`</tt>에 의해 자동으로 설정됩니다.

### .accepted_media_type

content negotiation 단계에서 선택한 미디어 유형입니다.
뷰에서 response이 반환되기 직전에 <tt style="color: #FF0000">`APIView`</tt> 또는 <tt style="color: #FF0000">`@api_view`</tt>에 의해 자동으로 설정됩니다

### .renderer_context

renderer의 <tt style="color: #FF0000">`.render()`</tt> 메소드에 전달될 추가 컨텍스트 정보의 dictionary입니다.
뷰에서 response이 반환되기 직전에 <tt style="color: #FF0000">`APIView`</tt> 또는 <tt style="color: #FF0000">`@api_view`</tt>에 의해 자동으로 설정됩니다.


## Standard HttpResponse attributes

<tt style="color: #FF0000">`Response`</tt> 클래스는 <tt style="color: #FF0000">`SimpleTemplateResponse`</tt>를 확장하고 모든 일반적인 attributes과 메서드를 response에서도 사용할 수 있습니다. 예를 들어 표준 방식으로 response에 헤더를 설정할 수 있습니다.
~~~~
response = Response()
response['Cache-Control'] = 'no-cache'
~~~~

### .render()

Signature: <tt style="color: #FF0000">`.render()`</tt>
다른 <tt style="color: #FF0000">`TemplateResponse`</tt>와 마찬가지로 이 메소드는 response의 serialized data를 최종 response content로 렌더링하기 위해 호출됩니다. <tt style="color: #FF0000">`.render()`</tt>가 호출되면 <tt style="color: #FF0000">`accept_renderer`</tt> 인스턴스에서 <tt style="color: #FF0000">`.render(data, accepted_media_type, renderer_context)`</tt> 메서드를 호출한 결과로 response content가 설정됩니다.
일반적으로 Django's standard response cycle에 의해 처리되므로 <tt style="color: #FF0000">`.render()`</tt>를 직접 호출할 필요가 없습니다.
