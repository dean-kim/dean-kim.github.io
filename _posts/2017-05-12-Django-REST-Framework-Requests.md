---
layout: post
title:  "Django REST Framework -Requests"
date:   2017-05-12 10:43:59
author: Dean Kim
categories: Rest_Framework
tags:	Django Rest_framework
cover:  "/assets/instacode.png"
---

# Requests

REST-based web service 작업을 하고 있다면 request.POST는 무시해야 합니다.
"Malcom Tredinnick, [Django developers group](https://groups.google.com/forum/#!topic/django-developers/dxI4qVzrBY4/discussion)"

REST framework의 <tt style="color: #FF0000">`Request`</tt> 클래스는 표준 <tt style="color: #FF0000">`HttpRequest`</tt>를 확장하여 REST framework의 유연한 request parsing 및 request authentication을 지원합니다.


## Request parsing

REST framework의 Request 객체는 유연한 request parsing을 제공하므로 사용자가 일반적으로 form data를 처리하는 것과 같은 방식으로 JSON data 또는 다른 media types으로 requests를 처리할 수 ​​있습니다.

### .data

<tt style="color: #FF0000">`request.data`</tt>는 request body의 parsed content을 반환합니다. 이는 다음을 제외하고 표준 <tt style="color: #FF0000">`request.POST`</tt> 및 <tt style="color: #FF0000">`request.FILES`</tt> attributes과 유사합니다.
* 여기에는 file 및 non-file 입력을 포함하여 모든 parsed content가 포함됩니다.
* <tt style="color: #FF0000">`POST`</tt>가 아닌 HTTP 메소드의 컨텐츠 분석을 지원합니다. 즉, <tt style="color: #FF0000">`PUT`</tt> 및 <tt style="color: #FF0000">`PATCH`</tt> 요청의 컨텐츠에 액세스 할 수 있습니다.
* 이는 form data를 지원하는 것보다 REST framework의 유연한 request parsing을 지원합니다. 예를 들어 incoming form data를 처리하는 것과 같은 방식으로 들어오는 JSON data를 처리 할 수 ​​있습니다.

더 자세한 내용은 [parsers documentation 참조](http://www.django-rest-framework.org/api-guide/parsers/)

### .query_params

<tt style="color: #FF0000">`request.query_params`</tt>는 <tt style="color: #FF0000">`request.GET`</tt>보다 정확하게 명명된 동의어입니다.
코드내에서 명확성을 위해 Django의 standard <tt style="color: #FF0000">`request.GET`</tt> 대신 <tt style="color: #FF0000">`request.query_params`</tt>를 사용하는 것이 좋습니다. 이렇게하면 코드베이스를 보다 정확하고 명확하게 유지할 수 있습니다. 모든 HTTP 메소드 유형에는 <tt style="color: #FF0000">`GET`</tt> 요청뿐만 아니라 query parameters가 포함될 수 있습니다.

### .parsers

<tt style="color: #FF0000">`APIView`</tt> 클래스 또는 <tt style="color: #FF0000">`@api_view`</tt> decorator는 뷰에 설정된 <tt style="color: #FF0000">`parser_classes`</tt> 또는 <tt style="color: #FF0000">`DEFAULT_PARSER_CLASSES`</tt> 설정에 따라이 property이 자동으로 <tt style="color: #FF0000">`Parser`</tt> 인스턴스 목록으로 설정되도록 합니다.
일반적으로 이 property에 액세스할 필요는 없습니다.

참조 : 
클라이언트가 기형의 콘텐츠를 보낸 경우 <tt style="color: #FF0000">`request.data`</tt>에 액세스하면 <tt style="color: #FF0000">`ParseError`</tt>가 발생할 수 있습니다. 기본적으로 REST framework의 <tt style="color: #FF0000">`APIView`</tt> 클래스 또는 <tt style="color: #FF0000">`@api_view`</tt> decorator는 오류를 catch하고 <tt style="color: #FF0000">`400 Bad Request`</tt> response를 반환합니다.
클라이언트가 파싱할 수 없는 content-type을 가진 request을 보내면 <tt style="color: #FF0000">`UnsupportedMediaType`</tt> 예외가 발생합니다. 이 예외는 기본적으로 포착되어 <tt style="color: #FF0000">`415 Unsupported Media Type`</tt> response를 반환합니다.


## Content negotiation

request는 콘텐츠 negotiation 단계의 결과를 결정할 수 있는 몇 가지 properties을 제공합니다. 이를 통해 다양한 미디어 유형에 대해 다른 serialisation schemes를 선택하는 것과 같은 동작을 구현할 수 있습니다.

### .accepted_renderer

renderer 인스턴스는 컨텐츠 negotiation 단계에서 선택된 인스턴스입니다.

### .accepted_media_type

content negotiation 단계에서 수락 한 미디어 유형을 나타내는 문자열 representing 입니다.


## Authentication

REST framework는 다음과 같은 기능을 제공하는 유연한 per-reques authentication을 제공합니다.
* API의 다른 부분에 대해 각각 서로 다른 authentication policies을 사용하십시오.
* multiple authentication policies의 사용을 지원합니다.
* incoming request와 관련된 사용자 및 token 정보를 제공합니다.

### .user

<tt style="color: #FF0000">`request.user`</tt>는 일반적으로 <tt style="color: #FF0000">`django.contrib.auth.models.User`</tt>의 인스턴스를 반환하지만 behavior는 사용되는 authentication policy에 따라 다릅니다.
요청이 인증되지 않은 경우 <tt style="color: #FF0000">`request.user`</tt>의 default value은 <tt style="color: #FF0000">`django.contrib.auth.models.AnonymousUser`</tt>의 인스턴스입니다.
더 자세한 정보는 [authentication documentation 참조](http://www.django-rest-framework.org/api-guide/authentication/)

### .auth

<tt style="color: #FF0000">`request.auth`</tt>는 추가 authentication 컨텍스트를 리턴합니다. <tt style="color: #FF0000">`request.auth`</tt>의 정확한 behavior은 사용되는 authentication policy에 따라 다르지만 대개 요청이 authenticated된 token의 인스턴스일 수 있습니다.
request이 unauthenticated거나 추가 컨텍스트가없는 경우, <tt style="color: #FF0000">`request.auth`</tt>의 기본값은 <tt style="color: #FF0000">`None`</tt>입니다.
더 자세한 정보는 [authentication documentation 참조](http://www.django-rest-framework.org/api-guide/authentication/)

### .authenticators

<tt style="color: #FF0000">`APIView`</tt> 클래스 또는 <tt style="color: #FF0000">`@api_view`</tt> decorator는 뷰에 설정된 <tt style="color: #FF0000">`authentication_classes`</tt> 또는 <tt style="color: #FF0000">`DEFAULT_AUTHENTICATORS`</tt> 설정에 따라 이 property이 자동으로 Authentication 인스턴스 목록으로 설정되도록합니다.
일반적으로 이 property에 액세스할 필요는 없습니다.


## Browser enhancements

REST framework는 브라우저 기반 <tt style="color: #FF0000">`PUT`</tt>, <tt style="color: #FF0000">`PATCH`</tt> 및 <tt style="color: #FF0000">`DELETE`</tt> forms과 같은 몇 가지 브라우저 개선 사항을 지원합니다.

### .method

<tt style="color: #FF0000">`request.method`</tt>는 request의 HTTP 메소드의 <b>uppercased</b> 된 문자열 표현을 리턴합니다.
브라우저 기반 <tt style="color: #FF0000">`PUT`</tt>, <tt style="color: #FF0000">`PATCH`</tt> 및 <tt style="color: #FF0000">`DELETE`</tt> forms이 명확하게 지원됩니다.
자세한 내용은 [browser enhancements documentation 참조](http://www.django-rest-framework.org/topics/browser-enhancements/)

### .content_type

<tt style="color: #FF0000">`request.content_type`</tt>, HTTP 요청 본문의 미디어 유형을 나타내는 문자열 객체를 반환하거나 미디어 유형이 제공되지 않은 경우 빈 문자열을 반환합니다.
일반적으로 REST framework의 default request parsing behavior에 의존하므로 일반적으로 요청의 콘텐츠 형식에 직접 액세스할 필요가 없습니다.
request의 콘텐츠 형식에 액세스해야하는 경우 browser-based non-form 콘텐츠에 대한 명확한 지원을 제공하므로 <tt style="color: #FF0000">`request.META.get('HTTP_CONTENT_TYPE')`</tt>을 사용하는 것보다 <tt style="color: #FF0000">`.content_type`</tt> property을 사용해야합니다.
자세한 내용은 [browser enhancements documentation 참조](http://www.django-rest-framework.org/topics/browser-enhancements/)

### .stream

<tt style="color: #FF0000">`request.stream`</tt>은 request body의 내용을 나타내는 스트림을 반환합니다.
일반적으로 REST framework의 default request parsing behavior에 의존하므로 대개 request의 콘텐츠에 직접 액세스할 필요가 없습니다.


## Standard HttpRequest attributes

REST framework의 <tt style="color: #FF0000">`Request`</tt>는 Django의 <tt style="color: #FF0000">`HttpRequest`</tt>를 확장하므로 다른 모든 표준 속성과 메소드도 사용할 수 있습니다. 예를 들어 <tt style="color: #FF0000">`request.META`</tt> 및 <tt style="color: #FF0000">`request.session`</tt> dictionaries은 정상적으로 사용 가능합니다.
implementation reasons로 인해 <tt style="color: #FF0000">`Request`</tt> 클래스는 <tt style="color: #FF0000">`HttpRequest`</tt> 클래스에서 상속하지 않고 대신 composition을 사용하여 클래스를 확장합니다.