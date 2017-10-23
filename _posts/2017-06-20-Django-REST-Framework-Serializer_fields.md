---
layout: post
title:  "Django REST Framework -Serializer fields"
date:   2017-06-20 10:43:59
author: Dean Kim
categories: Rest_Framework
tags:	Django Rest_framework
cover:  "/assets/instacode.png"
---

# Serializer fields
- 원본 : [공식문서](http://www.django-rest-framework.org/api-guide/fields/)



Form 클래스의 각 필드는 데이터의 유효성을 검사할 뿐만 아니라 일관된 형식으로 정규화하는 "cleaning"작업도 담당합니다. - 장고 공식문서

Serializer 필드는 primitive values, 내부 datatypes 간의 변환을 처리합니다. 또한 입력 값의 유효성 검사와 부모 오브젝트에서 값을 검색하고 설정하는 작업도 처리합니다.

<b>Note:</b> serializer 필드는 <tt style="color: #FF0000">`fields.py`</tt>에 선언되어 있지만, 규칙에 따라 <tt style="color: #FF0000">`from rest_framework import serializer`</tt> 가져와 <tt style="color: #FF0000">`serializers.<FieldName>`</tt>로 필드를 참조해야합니다.

### Core arguments

각 serializer 필드 클래스 constructor는 최소한 다음의 인수들을 취합니다. 일부 Field 클래스는 field-specific 인수를 추가로 가져 오지만 항상 다음 사항을 받아 들여야합니다.

`read_only`
<br>
읽기 전용 필드는 API 출력에 포함되지만 create 또는 update operations 중 입력에 포함되면 안됩니다. serializer 입력에 잘못 포함된 'read_only'필드는 무시됩니다.
<br>representation을 serializing 할 때 필드를 사용하지만 deserialization 중에 인스턴스를 작성하거나 업데이트 할 때 필드를 사용하지 않으려면 이 값을 <tt style="color: #FF0000">`True`</tt>로 설정하십시오.
<br>Defaults는 <tt style="color: #FF0000">`False`</tt> 입니다.

`write_only`
<br>
이 값을 <tt style="color: #FF0000">`True`</tt>로 설정하면 인스턴스를 업데이트하거나 만들 때 필드가 사용될 수 있지만 representation을 serialize 할 때는 필드가 포함되지 않습니다.
<br>Defaults는 <tt style="color: #FF0000">`False`</tt> 입니다.

`required`
<br>
deserialization 중에 필드가 제공되지 않으면 일반적으로 오류가 발생합니다. deserialization 중에 이 필드가 필요하지 않은 경우 false로 설정하십시오.
이 값을 <tt style="color: #FF0000">`False`</tt>로 설정하면 인스턴스를 serializing할 때 object attribute or dictionary key가 출력에서 ​​생략될 수 있습니다. 키가 없으면 출력 representation에 포함되지 않습니다.
<br>Defaults는 <tt style="color: #FF0000">`True`</tt> 입니다.

`allow_null`
<br>
보통 serializer 필드에 <tt style="color: #FF0000">`None`</tt>이 전달되면 오류가 발생합니다. <tt style="color: #FF0000">`None`</tt>을 유효한 값으로 간주해야하는 경우 이 키워드 인수를 <tt style="color: #FF0000">`True`</tt>로 설정하십시오.
<br>Defaults는 <tt style="color: #FF0000">`False`</tt> 입니다.

`default`
<br>
설정되면 input value이 제공되지 않는 경우, 필드에 사용될 default value가 제공됩니다. 설정되지 않은 경우 default behaviour는 속성을 전혀 채우지 않는 것입니다.
부분 갱신 조작 중에는 <tt style="color: #FF0000">`default`</tt>가 적용되지 않습니다. 부분 업데이트의 경우 들어오는 데이터에서 제공되는 필드만 유효한 값이 반환됩니다.
함수 또는 다른 호출 가능 객체로 설정될 수 있습니다. 이 경우 값은 사용될 때마다 평가됩니다. 호출되면 인수를 받지 않습니다. 호출 가능 객체가 <tt style="color: #FF0000">`set_context`</tt> 메소드를 가지고 있으면 필드 인스턴스를 인수로 사용하기 전에 매번 호출됩니다. 이는 validators와 동일한 방식으로 작동합니다.
인스턴스를 serializing 할 때 오브젝트 속성 또는 사전 키가 인스턴스에 없는 경우 default가 사용됩니다.
<tt style="color: #FF0000">`default value`</tt>을 설정하면 해당 필드가 필요하지 않음을 의미합니다. <tt style="color: #FF0000">`default`</tt> 및 <tt style="color: #FF0000">`required`</tt> 키워드 인수를 모두 포함하면 올바르지 않으며 오류가 발생합니다.

`source`
<br>
필드를 채우는 데 사용할 속성의 이름입니다. <tt style="color: #FF0000">`URLField(source='get_absolute_url')`</tt>와 같은 <tt style="color: #FF0000">`self`</tt> 인수만 사용하거나 <tt style="color: #FF0000">`EmailField(source='user.email')`</tt>와 같은 속성을 통과하는 dotted notation을 사용할 수 있습니다.
value <tt style="color: #FF0000">`source='*'`</tt>는 특별한 의미를 가지며 전체 객체가 필드로 전달되어야 함을 나타내기 위해 사용됩니다. nested representations을 작성하거나 output representation을 결정하기 위해 전체 오브젝트에 액세스해야하는 필드에 유용합니다.
Defaults는 필드의 이름입니다.

`validators`
<br>
입력 필드 입력에 적용되어야하고 validation 오류를 발생시키거나 단순히 반환해야 하는 validator 함수의 목록입니다. Validator 함수는 일반적으로 <tt style="color: #FF0000">`serializers.ValidationError`</tt>를 발생시켜야 하지만 Django에 내장된 <tt style="color: #FF0000">`ValidationError`</tt>는 Django 코드베이스 또는 third party Django 패키지에 정의된 validators와의 호환성을 위해 지원됩니다.

`error_messages`
<br>
오류 메시지에 대한 오류 코드 dictionary.

`label`
<br>
HTML 양식 필드 또는 기타 descriptive 요소의 필드 이름으로 사용할 수 있는 짧은 텍스트 문자열입니다.

`help_text`
<br>
HTML 양식 필드나 다른 기술적 요소의 필드에 대한 description으로 사용될 수 있는 텍스트 문자열입니다.

`initial`
<br>
HTML 양식 필드의 값을 미리 채우는데 사용해야 하는 값입니다. 일반 Django <tt style="color: #FF0000">`Field`</tt>에서와 마찬가지로 호출 대상을 전달할 수 있습니다.
~~~~
import datetime
from rest_framework import serializers
class ExampleSerializer(serializers.Serializer):
    day = serializers.DateField(initial=datetime.date.today)
~~~~

`style`
<br>

렌더러가 필드를 렌더링하는 방법을 제어하는 ​​데 사용할 수 있는 key-value pairs dictionary입니다.
다음은 <tt style="color: #FF0000">`'input_type'`</tt>과 <tt style="color: #FF0000">`'base_template'`</tt>의 두 가지 예입니다.
~~~~
# Use <input type="password"> for the input.
password = serializers.CharField(
    style={'input_type': 'password'}
)

# Use a radio input instead of a select input.
color_channel = serializers.ChoiceField(
    choices=['red', 'green', 'blue'],
    style={'base_template': 'radio.html'}
)
~~~~
공식문서 참조 [HTML & Forms](http://www.django-rest-framework.org/topics/html-and-forms/)


## Boolean fields

### BooleanField

boolean representation.
<br>HTML로 인코딩된 양식 입력을 사용할 때 <tt style="color: #FF0000">`default=True`</tt> 옵션이 지정되어 있어도 값을 생략하면 항상 필드를 <tt style="color: #FF0000">`False`</tt>로 설정하는 것으로 처리됩니다. 이것은 HTML 체크 상자 입력이 값을 생략하여 선택되지 않은 상태를 나타내므로 REST framework는 누락을 빈 체크 박스 입력으로 처리하기 때문입니다.
Corresponds to <tt style="color: #FF0000">`django.db.models.fields.BooleanField`</tt>
<br><b>Signature</b>: <tt style="color: #FF0000">`BooleanField()`</tt>

### NullBooleanField

유효한 값으로 <tt style="color: #FF0000">`None`</tt>도 허용하는 boolean representation입니다.
<br>Corresponds to <tt style="color: #FF0000">`django.db.models.fields.NullBooleanField`</tt>.
<br><b>Signature</b>: <tt style="color: #FF0000">`BooleanField()`</tt>


## String fields

### CharField

텍스트 representation. 선택적으로 텍스트가 <tt style="color: #FF0000">`max_length`</tt>보다 짧고 <tt style="color: #FF0000">`min_length`</tt>보다 긴가에 대한 validate 합니다.
<br>Corresponds to <tt style="color: #FF0000">`django.db.models.fields.CharField`</tt> or <tt style="color: #FF0000">`django.db.models.fields.TextField`</tt>.
<br><b>Signature</b>: <tt style="color: #FF0000">`CharField(max_length=None, min_length=None, allow_blank=False, trim_whitespace=True)`</tt>
- <tt style="color: #FF0000">`max_length`</tt> : 입력이 이 값을 초과하지 않는지 확인합니다.
- <tt style="color: #FF0000">`min_length`</tt> : 입력이 이 값보다 작은지 확인합니다.
- <tt style="color: #FF0000">`allow_blank`</tt> : <tt style="color: #FF0000">`True`</tt>로 설정하면 빈 문자열을 유효한 값으로 간주해야합니다. <tt style="color: #FF0000">`False`</tt>로 설정하면 빈 문자열이 유효하지 않은 것으로 간주되어 유효성 검사 오류가 발생합니다. Defaults는 <tt style="color: #FF0000">`False`</tt>입니다.
- <tt style="color: #FF0000">`trim_whitespace`</tt> : <tt style="color: #FF0000">`True`</tt>로 설정하면 앞뒤 공백이 잘립니다. Defaults는 <tt style="color: #FF0000">`True`</tt>.

<tt style="color: #FF0000">`allow_null`</tt> 옵션은 문자열 필드에도 사용할 수 있지만 <tt style="color: #FF0000">`allow_blank`</tt>를 사용하면 사용하지 않는 것이 좋습니다. <tt style="color: #FF0000">`allow_blank=True`</tt>와 <tt style="color: #FF0000">`allow_null=True`</tt>를 모두 설정하는 것은 유효하지만 문자열 representations에 허용되는 두 가지 유형의 빈 값이 있으므로 데이터 불일치와 미세한 응용 프로그램 버그가 발생할 수 있습니다.

### EmailField

텍스트 representation, 유효한 전자 메일 주소인지 텍스트의 validate 합니다.
<br>Corresponds to <tt style="color: #FF0000">`django.db.models.fields.EmailField`</tt>
<br><b>Signature</b>: <tt style="color: #FF0000">`EmailField(max_length=None, min_length=None, allow_blank=False)`</tt>

### RegexField

텍스트 representation, 주어진 값을 특정 정규식과 비교하여 validate 합니다.
<br>Corresponds to <tt style="color: #FF0000">`django.forms.fields.RegexField`</tt>.
<br><b>Signature</b>: <tt style="color: #FF0000">`RegexField(regex, max_length=None, min_length=None, allow_blank=False)`</tt>
<br>필수 <tt style="color: #FF0000">`regex`</tt> 인자는 문자열이거나 컴파일된 파이썬 regular expression 객체일 수 있습니다.
<br>validation에 장고의 <tt style="color: #FF0000">`django.core.validators.RegexValidator`</tt> 사용

### SlugField

패턴 <tt style="color: #FF0000">`[a-zA-Z0-9 _-]+`</tt> 에 대한 입력의 validates 하는 <tt style="color: #FF0000">`RegexField`</tt>입니다.
<br>Corresponds to <tt style="color: #FF0000">`django.db.models.fields.SlugField`</tt>.
<br><b>Signature</b>: <tt style="color: #FF0000">`SlugField(max_length=50, min_length=None, allow_blank=False)`</tt>

### URLField

URL 일치 패턴에 대해 입력의 validates 하는 <tt style="color: #FF0000">`RegexField`</tt>입니다. <tt style="color: #FF0000">`http://<host>/<path>`</tt> 형식의 정규화된 URL이 필요합니다.
<br>Corresponds to <tt style="color: #FF0000">`django.db.models.fields.URLField`</tt>
<br>validation에 장고의 <tt style="color: #FF0000">`django.core.validators.URLValidator`</tt> 사용
<br><b>Signature</b>: <tt style="color: #FF0000">`URLField(max_length=200, min_length=None, allow_blank=False)`</tt>

### UUIDField

입력이 유효한 UUID 문자열임을 보장하는 필드입니다. <tt style="color: #FF0000">`to_internal_value`</tt> 메소드는 <tt style="color: #FF0000">`uuid.UUID`</tt> 인스턴스를 리턴합니다. 출력 시 필드는 정규 하이픈 형식의 문자열을 반환합니다. 예 :
~~~~
"de305d54-75b4-431b-adb2-eb6b9e546013"
~~~~
<br><b>Signature</b>: <tt style="color: #FF0000">`UUIDField(format='hex_verbose')`</tt>
* <tt style="color: #FF0000">`format`</tt>: uuid 값의 representation 형식을 결정합니다.
  * <tt style="color: #FF0000">`'hex_verbose'`</tt>: 하이픈을 포함한 16진수 representation : <tt style="color: #FF0000">`"5ce0e9a5-5ffa-654b-cee0-1238041fb31a"`</tt>
  * <tt style="color: #FF0000">`'hex'`</tt>: 하이픈을 제외하고 UUID의 압축된 16진수 representation : <tt style="color: #FF0000">`"5ce0e9a55ffa654bcee01238041fb31a"`</tt>
  * <tt style="color: #FF0000">`'int'`</tt>: UUID의 128 비트 정수 representation : <tt style="color: #FF0000">`"123456789012312313134124512351145145114"`</tt>
  * <tt style="color: #FF0000">`'urn'`</tt>: UUID의 RFC 4122 URN representation : <tt style="color: #FF0000">`"urn : uuid : 5ce0e9a5-5ffa-654b-cee0-1238041fb31a"`</tt> <tt style="color: #FF0000">`format`</tt> parameters를 변경하면 representation 값에만 영향을 줍니다. 모든 형식은 <tt style="color: #FF0000">`to_internal_value`</tt>에서 허용됩니다.

### FilePathField

파일 시스템의 특정 디렉토리에 있는 파일 이름으로 제한된 필드
<br>Corresponds to <tt style="color: #FF0000">`django.forms.fields.FilePathField`</tt>.
<br><b>Signature</b>: <tt style="color: #FF0000">`FilePathField(path, match=None, recursive=False, allow_files=True, allow_folders=False, required=None, **kwargs)`</tt>
* <tt style="color: #FF0000">`path`</tt>: 이 FilePathField가 선택해야 하는 디렉토리에 대한 절대 파일 시스템 경로.
* <tt style="color: #FF0000">`match`</tt>: FilePathField가 파일 이름을 필터링하는데 사용할 regular expression as a string 입니다.
* <tt style="color: #FF0000">`recursive`</tt>: path의 모든 하위 디렉토리가 포함되어야하는지 여부를 지정합니다. Default는 <tt style="color: #FF0000">`False`</tt>입니다.
* <tt style="color: #FF0000">`allow_files`</tt>: 지정된 위치의 파일을 포함할지 여부를 지정합니다. Default는 <tt style="color: #FF0000">`True`</tt>입니다. 이 필드 또는 <tt style="color: #FF0000">`allow_folders`</tt>는 <tt style="color: #FF0000">`True`</tt> 여야합니다.
* <tt style="color: #FF0000">`allow_folders`</tt>: 지정된 위치의 폴더를 포함할지 여부를 지정합니다. Default는 <tt style="color: #FF0000">`False`</tt>입니다. 이 필드 또는 <tt style="color: #FF0000">`allow_files`</tt>는 <tt style="color: #FF0000">`True`</tt> 여야합니다.

### IPAddressField

입력이 유효한 IPv4 또는 IPv6 문자열인지 확인하는 필드입니다.
<br>Corresponds to <tt style="color: #FF0000">`django.forms.fields.IPAddressField`</tt> and <tt style="color: #FF0000">`django.forms.fields.GenericIPAddressField`</tt>.
<br><b>Signature</b>: <tt style="color: #FF0000">`IPAddressField(protocol='both', unpack_ipv4=False, **options)`</tt>
* <tt style="color: #FF0000">`protocol`</tt>: 유효한 입력을 지정된 프로토콜로 제한합니다. 허용되는 값은 'both'(default), 'IPv4'또는 'IPv6'입니다. 일치는 대소 문자를 구분하지 않습니다.
* <tt style="color: #FF0000">`unpack_ipv4`</tt>: ::ffff:192.0.2.1과 같은 IPv4 매핑 주소의 압축을 풉니다. 이 옵션을 사용하면 주소가 192.0.2.1로 압축 해제됩니다. 기본값은 사용하지 않습니다. 프로토콜이 'both'로 설정된 경우에만 사용할 수 있습니다.


## Numeric fields

### IntegerField

정수 representation.
<br>Corresponds to <tt style="color: #FF0000">`django.db.models.fields.IntegerField`</tt>, <tt style="color: #FF0000">`django.db.models.fields.SmallIntegerField`</tt>, <tt style="color: #FF0000">`django.db.models.fields.PositiveIntegerField`</tt> and <tt style="color: #FF0000">`django.db.models.fields.PositiveSmallIntegerField`</tt>.
<br><b>Signature</b>: <tt style="color: #FF0000">`IntegerField(max_value=None, min_value=None)`</tt>
* <tt style="color: #FF0000">`max_value`</tt>: 제공된 숫자가 이 값보다 크지 않은지 Validate합니다.
* <tt style="color: #FF0000">`min_value`</tt>: 제공된 숫자가 이 값보다 작지 않은지 Validate합니다.

### FloatField

부동 소수점 representation.
<br>Corresponds to <tt style="color: #FF0000">`django.db.models.fields.FloatField`</tt>
<br><b>Signature</b>: <tt style="color: #FF0000">`FloatField(max_value=None, min_value=None)`</tt>
* <tt style="color: #FF0000">`max_value`</tt>: 제공된 숫자가 이 값보다 크지 않은지 Validate합니다.
* <tt style="color: #FF0000">`min_value`</tt>: 제공된 숫자가 이 값보다 작지 않은지 Validate합니다.

### DecimalField

10진수 representation으로, Python에서 <tt style="color: #FF0000">`Decimal`</tt> 인스턴스로 나타냅니다.
<br>Corresponds to <tt style="color: #FF0000">`django.db.models.fields.DecimalField`</tt>.
<br><b>Signature</b>: <tt style="color: #FF0000">`DecimalField(max_digits, decimal_places, coerce_to_string=None, max_value=None, min_value=None)`</tt>
* <tt style="color: #FF0000">`max_digits`</tt>: 숫자에 허용되는 최대 자릿수입니다. <tt style="color: #FF0000">`None`</tt> 또는 <tt style="color: #FF0000">`decimal_places`</tt>보다 크거나 같은 정수여야합니다.
* <tt style="color: #FF0000">`decimal_places`</tt>: 숫자와 함께 저장할 소수 자릿수입니다.
* <tt style="color: #FF0000">`coerce_to_string`</tt>: representation에 문자열 값을 반환해야 하는 경우 <tt style="color: #FF0000">`True`</tt>로 설정하고 <tt style="color: #FF0000">`Decimal`</tt> 객체를 반환해야하는 경우 <tt style="color: #FF0000">`False`</tt>로 설정합니다. Defaults는 <tt style="color: #FF0000">`COERCE_DECIMAL_TO_STRING`</tt> 설정 키와 동일한 값이며, 재정의하지 않으면 <tt style="color: #FF0000">`True`</tt>입니다. serializer가 <tt style="color: #FF0000">`Decimal`</tt> 객체를 반환하면 최종 출력 형식은 렌더러에 의해 결정됩니다. <tt style="color: #FF0000">`localize`</tt>를 설정하면 값이 <tt style="color: #FF0000">`True`</tt>로 설정됩니다.
* <tt style="color: #FF0000">`max_value`</tt>: 제공된 숫자가 이 값보다 크지 않은지 Validate합니다.
* <tt style="color: #FF0000">`min_value`</tt>: 제공된 숫자가 이 값보다 작지 않은지 Validate합니다.
* <tt style="color: #FF0000">`localize`</tt>: 현재 로케일을 기반으로 입력 및 출력의 localization을 사용하려면 <tt style="color: #FF0000">`True`</tt>로 설정하십시오. 또한 <tt style="color: #FF0000">`coerce_to_string`</tt>을 <tt style="color: #FF0000">`True`</tt>로 설정합니다. Defaults는 <tt style="color: #FF0000">`False`</tt>입니다. 설정 파일에서 <tt style="color: #FF0000">`USE_L10N=True`</tt>로 설정한 경우 데이터 형식이 사용 가능합니다.

#### Example usage

소수점 이하 2자릿수를 포함하여 999까지의 숫자를 validate하려면 다음을 사용합니다.
~~~~
serializers.DecimalField(max_digits=5, decimal_places=2)
~~~~
소수점 이하 10자릿수를 포함하여 10억 미만의 숫자를 검증하려면 다음을 수행하십시오.
~~~~
serializers.DecimalField(max_digits=19, decimal_places=10)
~~~~
이 필드에는 선택적 인수 <tt style="color: #FF0000">`coerce_to_string`</tt>도 사용됩니다. <tt style="color: #FF0000">`True`</tt>로 설정하면 representation이 문자열로 출력됩니다. <tt style="color: #FF0000">`False`</tt>로 설정하면 representation이 <tt style="color: #FF0000">`Decimal`</tt> 인스턴스로 남게되고 최종 representation은 렌더러에 의해 결정됩니다.
<br>설정을 해제하면 default는 <tt style="color: #FF0000">`COERCE_DECIMAL_TO_STRING`</tt> 설정과 동일한 값으로 설정되며, 그렇지 않은 경우 <tt style="color: #FF0000">`True`</tt>로 설정됩니다.


## Date and time fields

### DateTimeField

date and time representation.
<br>Corresponds to <tt style="color: #FF0000">`django.db.models.fields.DateTimeField`</tt>.
<br><b>Signature</b>: <tt style="color: #FF0000">`DateTimeField(format=api_settings.DATETIME_FORMAT, input_formats=None)`</tt>
* <tt style="color: #FF0000">`format`</tt>: 출력 형식을 나타내는 문자열입니다. 지정하지 않으면 default는 <tt style="color: #FF0000">`DATETIME_FORMAT`</tt> 설정 키와 동일한 값으로 설정되며, 설정하지 않으면 <tt style="color: #FF0000">`'iso-8601'`</tt>이됩니다. 형식 문자열로 설정하면 <tt style="color: #FF0000">`to_representation`</tt> 반환값을 문자열 출력으로 강제 변환해야합니다. 형식 문자열은 아래에 설명되어 있습니다. 이 값을 <tt style="color: #FF0000">`None`</tt>으로 설정하면 Python <tt style="color: #FF0000">`datetime`</tt> 객체가 <tt style="color: #FF0000">`to_representation`</tt>에 의해 반환되어야합니다. 이 경우 datetime 인코딩은 렌더러에 의해 결정됩니다.
* <tt style="color: #FF0000">`input_formats`</tt>: 날짜를 구문 분석하는 데 사용할 수있는 입력 형식을 나타내는 문자열 목록입니다. 지정하지 않으면 <tt style="color: #FF0000">`DATETIME_INPUT_FORMATS`</tt> 설정이 사용되며 default는 <tt style="color: #FF0000">`['iso-8601']`</tt>입니다.

#### `DateTimeField` format strings.

형식 문자열은 명시적으로 형식을 지정하는 [Python strftime formats](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior)이거나 [ISO 8601](https://www.w3.org/TR/NOTE-datetime) 스타일 datetimes이 사용되어야 함을 나타내는 특수 문자열 <tt style="color: #FF0000">`'iso-8601'`</tt>일 수 있습니다. (예 : <tt style="color: #FF0000">`'2013-01-29T12:34:56.000000Z'`</tt>)
<br>형식의 값으로 <tt style="color: #FF0000">`None`</tt> 값이 사용되면 datetime 객체는 <tt style="color: #FF0000">`to_representation`</tt>에 의해 반환되고 최종 출력 representation은 렌더러 클래스에 의해 결정됩니다.

#### `auto_now` and `auto_now_add` model fields.

<tt style="color: #FF0000">`ModelSerializer`</tt> 또는 <tt style="color: #FF0000">`HyperlinkedModelSerializer`</tt>를 사용할 때 <tt style="color: #FF0000">`auto_now=True`</tt> 또는 <tt style="color: #FF0000">`auto_now_add=True`</tt> 인 모델 필드는 기본적으로 <tt style="color: #FF0000">`read_only=True`</tt> 인 <tt style="color: #FF0000">`serializer`</tt> 필드를 사용합니다.
이 동작을 재정의하려면 serializer에서 명시적으로 <tt style="color: #FF0000">`DateTimeField`</tt>를 선언해야합니다. 예 :
~~~~
class CommentSerializer(serializers.ModelSerializer):
    created = serializers.DateTimeField()

    class Meta:
        model = Comment
~~~~

### DateField

A date representation.

<br>Corresponds to <tt style="color: #FF0000">`django.db.models.fields.DateField`</tt>.
<br><b>Signature</b>: <tt style="color: #FF0000">`DateField(format=api_settings.DATE_FORMAT, input_formats=None)`</tt>
* <tt style="color: #FF0000">`format`</tt>: 출력 형식을 나타내는 문자열입니다. 지정하지 않으면 defaults는 <tt style="color: #FF0000">`DATE_FORMAT`</tt> 설정 키와 동일한 값으로 설정되며, 설정하지 않으면 <tt style="color: #FF0000">`'iso-8601'`</tt>이 됩니다. 형식 문자열로 설정하면 <tt style="color: #FF0000">`to_representation`</tt> 반환 값을 문자열 출력으로 강제 변환해야합니다. 형식 문자열은 아래에 설명되어 있습니다. 이 값을 <tt style="color: #FF0000">`None`</tt>으로 설정하면 Python 날짜 객체가 <tt style="color: #FF0000">`to_representation`</tt>에 의해 반환되어야 합니다. 이 경우 날짜 인코딩은 렌더러에 의해 결정됩니다.
* <tt style="color: #FF0000">`input_formats`</tt>: 날짜를 구문 분석하는데 사용할 수 있는 입력 형식을 나타내는 문자열 목록입니다. 지정하지 않으면 <tt style="color: #FF0000">`TIME_INPUT_FORMATS`</tt> 설정이 사용되며 defaults는 <tt style="color: #FF0000">`['iso-8601']`</tt>입니다.

#### `TimeField` format string

형식 문자열은 명시적으로 형식을 지정하는 [Python strftime formats](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior)이거나 [ISO 8601](https://www.w3.org/TR/NOTE-datetime) 스타일 시간이 사용되어야 함을 나타내는 특수 문자열 <tt style="color: #FF0000">`'iso-8601'`</tt>일 수 있습니다. (예: <tt style="color: #FF0000">`'12:34:56.000000'`</tt>)

### DurationField

A Duration representation.
<br>Corresponds to <tt style="color: #FF0000">`django.db.models.fields.DurationField`</tt>.
<br>이 필드의 <tt style="color: #FF0000">`validated_data`</tt>에는 <tt style="color: #FF0000">`datetime.timedelta`</tt> 인스턴스가 포함됩니다. representation은 <tt style="color: #FF0000">`'[DD] [HH:[MM:]]ss[.uuuuuu]'`</tt> 형식의 문자열입니다.
<br><b>Note:</b> 이 필드는 Django 버전 1.8 이상에서만 사용 가능합니다.
<br><b>Signature</b>: <tt style="color: #FF0000">`DurationField()`</tt>


## Choice selection fields

#### Parsers and file uploads.

<tt style="color: #FF0000">`FileField`</tt> 및 <tt style="color: #FF0000">`ImageField`</tt> 클래스는 <tt style="color: #FF0000">`MultiPartParser`</tt> 또는 <tt style="color: #FF0000">`FileUploadParser`</tt>에서만 사용하기에 적합합니다. 대부분의 파서, 예를 들어. JSON은 파일 업로드를 지원하지 않습니다. Django의 일반 [FILE_UPLOAD_HANDLERS](https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-FILE_UPLOAD_HANDLERS)는 업로드된 파일을 처리하는데 사용됩니다.

### FileField

A file representation.
<br>Django의 표준 FileField validation을 수행합니다.
<br>Corresponds to <tt style="color: #FF0000">`django.forms.fields.FileField`</tt>.
<br><b>Signature</b>: <tt style="color: #FF0000">`FileField(max_length=None, allow_empty_file=False, use_url=UPLOADED_FILES_USE_URL)`</tt>
* <tt style="color: #FF0000">`max_length`</tt>: 파일 이름의 최대 길이를 지정합니다.
* <tt style="color: #FF0000">`allow_empty_file`</tt>: 빈 파일이 허용되는지 여부를 지정합니다.
* <tt style="color: #FF0000">`use_url`</tt>: <tt style="color: #FF0000">`True`</tt>로 설정하면 URL 문자열 값이 출력 representation에 사용됩니다. <tt style="color: #FF0000">`False`</tt>로 설정하면 파일 이름 문자열 값이 출력 representation에 사용됩니다. <tt style="color: #FF0000">`UPLOADED_FILES_USE_URL`</tt> 설정 키의 Defaults로 설정되며, 그렇지 않은 경우 <tt style="color: #FF0000">`True`</tt>로 설정됩니다.

### ImageField

An image representation.
<br>업로드된 파일 내용을 알려진 이미지 형식과 일치하는지 Validates합니다.
<br>Corresponds to <tt style="color: #FF0000">`django.forms.fields.ImageField`</tt>.
<br><b>Signature</b>: <tt style="color: #FF0000">`ImageField(max_length=None, allow_empty_file=False, use_url=UPLOADED_FILES_USE_URL)`</tt>
* <tt style="color: #FF0000">`max_length`</tt>: 파일 이름의 최대 길이를 지정합니다.
* <tt style="color: #FF0000">`allow_empty_file`</tt>: 빈 파일이 허용되는지 여부를 지정합니다.
* <tt style="color: #FF0000">`use_url`</tt>: <tt style="color: #FF0000">`True`</tt>로 설정하면 URL 문자열 값이 출력 representation에 사용됩니다. <tt style="color: #FF0000">`False`</tt>로 설정하면 파일 이름 문자열 값이 출력 representation에 사용됩니다. <tt style="color: #FF0000">`UPLOADED_FILES_USE_URL`</tt> 설정 키의 Defaults로 설정되며, 그렇지 않은 경우 <tt style="color: #FF0000">`True`</tt>로 설정됩니다.

<tt style="color: #FF0000">`Pillow`</tt> 패키지 또는 <tt style="color: #FF0000">`PIL`</tt> 패키지가 필요합니다. <tt style="color: #FF0000">`PIL`</tt>은 더 이상 적극적으로 관리하지 않으므로 <tt style="color: #FF0000">`Pillow`</tt> 패키지를 사용하는 것이 좋습니다.


## Composite fields

### ListField

오브젝트의 리스트를 validates하는 필드 클래스입니다.
<br><b>Signature</b>: <tt style="color: #FF0000">`ListField(child, min_length=None, max_length=None)`</tt>
* <tt style="color: #FF0000">`child`</tt>: 목록의 객체의 validating을 검사하는 데 사용해야하는 필드 인스턴스입니다. 이 인수가 제공되지 않으면 목록의 객체에 대한 validated되지 않습니다.
* <tt style="color: #FF0000">`min_length`</tt>: 리스트에 이 수보다 적은 수의 요소가 포함되고 있는지를 Validates합니다.
* <tt style="color: #FF0000">`max_length`</tt>: 리스트에이 수보다 많은 요소가 포함되지 않는지를 Validates합니다.

예를 들어, 정수 목록을 validate하려면 다음과 같이 사용할 수 있습니다.
~~~~
scores = serializers.ListField(
   child=serializers.IntegerField(min_value=0, max_value=100)
)
~~~~
또한 <tt style="color: #FF0000">`ListField`</tt> 클래스는 재사용 가능한 목록 필드 클래스를 작성할 수 있는 선언 스타일을 지원합니다.
~~~~
class StringListField(serializers.ListField):
    child = serializers.CharField()
~~~~
이제는 <tt style="color: #FF0000">`child`</tt> argument를 제공할 필요없이 응용 프로그램 전체에 custom <tt style="color: #FF0000">`StringListField`</tt> 클래스를 재사용 할 수 있습니다.

### DictField

objects dictionary를 validates하는 필드 클래스입니다. <tt style="color: #FF0000">`DictField`</tt>의 키는 항상 문자열 값으로 간주됩니다.
<br><b>Signature</b>: <tt style="color: #FF0000">`DictField(child)`</tt>
* <tt style="color: #FF0000">`child`</tt>: dictionary의 값의 validating하는데 사용해야하는 필드 인스턴스입니다. 이 인수가 제공되지 않으면 매핑의 값이 validated하지 않습니다.

예를 들어 문자열과 문자열의 매핑을 validates하는 필드를 만들려면 다음과 같이 작성합니다.
~~~~
document = DictField(child=CharField())
~~~~
<tt style="color: #FF0000">`ListField`</tt>와 마찬가지로 선언 스타일을 사용할 수도 있습니다. 예 :
~~~~
class DocumentField(DictField):
    child = CharField()
~~~~

### JSONField

들어오는 데이터 구조가 validates한 JSON primitives로 구성되었는지 확인하는 필드 클래스입니다. 대체 바이너리 모드에서는 JSON으로 인코딩된 바이너리 문자열을 나타내고 validate합니다.
<br><b>Signature</b>: <tt style="color: #FF0000">`JSONField(binary)`</tt>
* <tt style="color: #FF0000">`binary`</tt>: <tt style="color: #FF0000">`True`</tt>로 설정하면 필드는 primitive 데이터 구조가 아닌 JSON 인코딩된 문자열을 출력하고 validate합니다. Defaults는 <tt style="color: #FF0000">`False`</tt>입니다.


## Miscellaneous fields

### ReadOnlyField

수정하지 않고 단순히 필드의 값을 반환하는 필드 클래스입니다.
<br>이 필드는 모델 필드가 아닌 속성과 관련된 필드 이름을 포함할 때 <tt style="color: #FF0000">`ModelSerializer`</tt>에서 default로 사용됩니다.
<br><b>Signature</b>: <tt style="color: #FF0000">`ReadOnlyField()`</tt>
<br>예를 들어 <tt style="color: #FF0000">`has_expired`</tt>가 <tt style="color: #FF0000">`Account`</tt> 모델의 속성인 경우 다음 serializer가 이를 자동으로 <tt style="color: #FF0000">`ReadOnlyField`</tt>로 생성합니다.
~~~~
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ('id', 'account_name', 'has_expired')
~~~~

### HiddenField

사용자 입력에 따라 값을 사용하지 않고 대신 default value 또는 호출 가능 값에서 값을 가져오는 필드 클래스입니다.
<br><b>Signature</b>: <tt style="color: #FF0000">`HiddenField()`</tt>
<br>예를 들어 serializer validated 데이터의 일부로 항상 현재 시간을 제공하는 필드를 포함시키려면 다음을 사용합니다.
~~~~
modified = serializers.HiddenField(default=timezone.now)
~~~~
<tt style="color: #FF0000">`HiddenField`</tt> 클래스는 사전 정의된 일부 필드 값을 기반으로 실행해야하는 validation이 있지만 해당 필드를 모두 최종 사용자에게 표시하지 않으려는 경우에만 필요합니다.
<br><tt style="color: #FF0000">`HiddenField`</tt>에 대한 추가 예제는 [validators](http://www.django-rest-framework.org/api-guide/validators/) 설명서를 참조하십시오.

### ModelField

임의의 모델 필드에 묶을 수 있는 일반 필드입니다. <tt style="color: #FF0000">`ModelField`</tt> 클래스는 serialization/deserialization 작업을 관련 모델 필드에 위임합니다. 이 필드는 new custom serializer 필드를 만들지 않고도 custom 모델 필드에 대한 serializer 필드를 만드는데 사용할 수 있습니다.
<br>이 필드는 <tt style="color: #FF0000">`ModelSerializer`</tt>에서 custom 모델 필드 클래스에 해당하는 데 사용됩니다.
<br><b>Signature</b>: <tt style="color: #FF0000">`ModelField(model_field=<Django ModelField instance>)`</tt>
<br><tt style="color: #FF0000">`ModelField`</tt> 클래스는 일반적으로 내부 사용을 위한 클래스이지만 필요한 경우 API에서 사용할 수 있습니다. <tt style="color: #FF0000">`ModelField`</tt>를 올바르게 인스턴스화하려면 인스턴스화된 모델에 첨부된 필드를 전달해야합니다. 예: <tt style="color: #FF0000">`ModelField(model_field=MyModel()._meta.get_field('custom_field'))`</tt>

### SerializerMethodField

이것은 읽기 전용 필드입니다. 이 클래스는 첨부된 serializer 클래스에서 메서드를 호출하여 값을 가져옵니다. 객체의 serialized representation에 모든 종류의 데이터를 추가하는데 사용할 수 있습니다.
<br><b>Signature</b>: <tt style="color: #FF0000">`SerializerMethodField(method_name=None)`</tt>
* <tt style="color: #FF0000">`method_name`</tt>: serializer에서 호출할 메서드의 이름입니다. 포함되지 않은 경우 기본값은 <tt style="color: #FF0000">`get_<field_name>`</tt>입니다.

<tt style="color: #FF0000">`method_name`</tt> 인수로 참조되는 serializer 메소드는 serialized 될 오브젝트인 단일 argument (<tt style="color: #FF0000">`self`</tt> 외)를 채택해야합니다. 그것은 당신이 객체의 serialized representation에 포함되기를 원하는 것을 반환해야한다. 예 :
~~~~
from django.contrib.auth.models import User
from django.utils.timezone import now
from rest_framework import serializers

class UserSerializer(serializers.ModelSerializer):
    days_since_joined = serializers.SerializerMethodField()

    class Meta:
        model = User

    def get_days_since_joined(self, obj):
        return (now() - obj.date_joined).days
~~~~


## Custom fields

사용자 정의 필드를 만들려면 <tt style="color: #FF0000">`Field`</tt>를 서브 클래스화한 다음 <tt style="color: #FF0000">`.to_representation()`</tt> 및 <tt style="color: #FF0000">`.to_internal_value()`</tt> 메소드 중 하나 또는 모두를 재정의해야합니다. 이 두 메소드는 초기 데이터 유형과 primitive, serializable 데이터 유형간에 변환하는데 사용됩니다. 기본 데이터 유형은 일반적으로 숫자, 문자열, 부울, <tt style="color: #FF0000">`date`</tt>/<tt style="color: #FF0000">`time`</tt>/<tt style="color: #FF0000">`datetime`</tt> 또는 <tt style="color: #FF0000">`None`</tt> 중 하나입니다. 그것들은 다른 기본 객체만을 포함하는 객체와 같은 list이나 dictionary일 수도 있습니다. 사용중인 렌더러에 따라 다른 유형이 지원될 수 있습니다.
<br><tt style="color: #FF0000">`.to_representation()`</tt> 메서드는 초기 데이터 유형을 원시의 serializable 데이터 유형으로 변환하기 위해 호출됩니다.
<br><tt style="color: #FF0000">`to_internal_value()`</tt> 메소드는 기본 데이터 유형을 내부 파이썬 representation으로 복원하기 위해 호출됩니다. 데이터가 invalid인 경우이 메소드는 serializers.ValidationError를 발생시켜야합니다.
<br>버전 2.x에 있던 <tt style="color: #FF0000">`WritableField`</tt> 클래스는 더 이상 존재하지 않습니다. <tt style="color: #FF0000">`Field`</tt>가 데이터 입력을 지원하면 필드의 하위 클래스를 만들고 <tt style="color: #FF0000">`to_internal_value()`</tt>를 재정의해야합니다.

### Examples

RGB 색상 값을 나타내는 클래스를 serializing하는 예를 살펴 보겠습니다.
~~~~
class Color(object):
    """
    A color represented in the RGB colorspace.
    """
    def __init__(self, red, green, blue):
        assert(red >= 0 and green >= 0 and blue >= 0)
        assert(red < 256 and green < 256 and blue < 256)
        self.red, self.green, self.blue = red, green, blue

class ColorField(serializers.Field):
    """
    Color objects are serialized into 'rgb(#, #, #)' notation.
    """
    def to_representation(self, obj):
        return "rgb(%d, %d, %d)" % (obj.red, obj.green, obj.blue)

    def to_internal_value(self, data):
        data = data.strip('rgb(').rstrip(')')
        red, green, blue = [int(col) for col in data.split(',')]
        return Color(red, green, blue)
~~~~
기본적으로 필드 값은 객체의 속성에 매핑으로 처리됩니다. 필드 값에 액세스하고 설정하는 방법을 customize해야하는 경우 <tt style="color: #FF0000">`.get_attribute()`</tt> and/or <tt style="color: #FF0000">`.get_value()`</tt>를 재정의해야합니다.
<br>예를 들어, serialized되는 객체의 클래스 이름을 나타내는데 사용할 수 있는 필드를 만듭니다.
~~~~
class ClassNameField(serializers.Field):
    def get_attribute(self, obj):
        # We pass the object instance onto `to_representation`,
        # not just the field attribute.
        return obj

    def to_representation(self, obj):
        """
        Serialize the object's class name.
        """
        return obj.__class__.__name__
~~~~

#### Raising validation errors

위의 <tt style="color: #FF0000">`ColorField`</tt> 클래스는 현재 데이터 validation을 수행하지 않습니다. 잘못된 데이터를 나타내려면 <tt style="color: #FF0000">`serializers.ValidationError`</tt>를 발생시켜야합니다.
~~~~
def to_internal_value(self, data):
    if not isinstance(data, six.text_type):
        msg = 'Incorrect type. Expected a string, but got %s'
        raise ValidationError(msg % type(data).__name__)

    if not re.match(r'^rgb\([0-9]+,[0-9]+,[0-9]+\)$', data):
        raise ValidationError('Incorrect format. Expected `rgb(#,#,#)`.')

    data = data.strip('rgb(').rstrip(')')
    red, green, blue = [int(col) for col in data.split(',')]

    if any([col > 255 or col < 0 for col in (red, green, blue)]):
        raise ValidationError('Value out of range. Must be between 0 and 255.')

    return Color(red, green, blue)
~~~~
<tt style="color: #FF0000">`.fail()`</tt> 메소드는 <tt style="color: #FF0000">`error_messages`</tt> dictionary에서 메시지 문자열을 취하는 <tt style="color: #FF0000">`ValidationError`</tt>를 발생시키는 지름길입니다. 예 :
~~~~
default_error_messages = {
    'incorrect_type': 'Incorrect type. Expected a string, but got {input_type}',
    'incorrect_format': 'Incorrect format. Expected `rgb(#,#,#)`.',
    'out_of_range': 'Value out of range. Must be between 0 and 255.'
}

def to_internal_value(self, data):
    if not isinstance(data, six.text_type):
        self.fail('incorrect_type', input_type=type(data).__name__)

    if not re.match(r'^rgb\([0-9]+,[0-9]+,[0-9]+\)$', data):
        self.fail('incorrect_format')

    data = data.strip('rgb(').rstrip(')')
    red, green, blue = [int(col) for col in data.split(',')]

    if any([col > 255 or col < 0 for col in (red, green, blue)]):
        self.fail('out_of_range')

    return Color(red, green, blue)
~~~~
이 스타일은 오류 메시지를 코드에서보다 명확하게 분리하고 preferred해야합니다.


## Third party packages

다음 third party 패키지도 제공됩니다.

### DRF Compound Fields

[drf-compound-fields](https://drf-compound-fields.readthedocs.io/en/latest/) 패키지는 <tt style="color: #FF0000">`many=True`</tt> 옵션을 사용하는 serializer가 아닌 다른 필드에서 설명할 수 있는 단순 값 list와 같은 "compound" serializer 필드를 제공합니다. 또한 특정 유형 또는 해당 유형의 항목 list일 수 있는 입력된 dictionaries 및 값에 대한 필드가 제공됩니다.

### DRF Extra Fields

[drf-extra-fields](https://github.com/Hipo/drf-extra-fields) 패키지는 <tt style="color: #FF0000">`Base64ImageField`</tt> 및 <tt style="color: #FF0000">`PointField`</tt> 클래스를 포함하여 REST framework에 대한 추가 serializer 필드를 제공합니다.

### djangrestframework-recursive

[djangorestframework-recursive](https://github.com/heywbj/django-rest-framework-recursive) 패키지는 recursive 구조의 serializing and deserializing를 위한 <tt style="color: #FF0000">`RecursiveField`</tt>를 제공한다.

### django-rest-framework-gis

[django-rest-framework-gis](https://github.com/djangonauts/django-rest-framework-gis) 패키지는 <tt style="color: #FF0000">`GeometryField`</tt> 필드와 GeoJSON 시리얼 라이저 같은 django REST framework를 위한 geographic 추가 기능을 제공합니다.

### django-rest-framework-hstore

[django-rest-framework-hstore](https://github.com/djangonauts/django-rest-framework-hstore) 패키지는 [django-hstore](https://github.com/djangonauts/django-hstore) <tt style="color: #FF0000">`DictionaryField`</tt> 모델 필드를 지원하는 <tt style="color: #FF0000">`HStoreField`</tt>를 제공합니다.