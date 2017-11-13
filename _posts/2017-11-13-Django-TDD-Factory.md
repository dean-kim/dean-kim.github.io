---
layout: post
title:  "Django TDD-Factory"
date:   2017-11-13 10:43:59
author: Dean Kim
categories: Django
tags:	Django TDD Factory
cover:  "/assets/instacode.png"
---

# Factory
- 원본 : [공식문서](http://factoryboy.readthedocs.io/en/latest/introduction.html)

TDD를 진행하는데 있어서 과정마다 dummy data를 만드는 것은 비효율적이라고 생각하여 instance를 생성해주는 Factory를 사용해 보기로 했습니다.


## Basic usage

팩토리는 객체를 인스턴스화하는 데 사용되는 속성 집합을 선언하며 클래스는 Meta 클래스의 모델 속성에 정의된 것으로 정의합니다.

예)
~~~~
import factory
from . import base

class UserFactory(factory.Factory):
    class Meta:
        model = base.User

    firstname = "John"
    lastname = "Doe"
~~~~
이제 base.User 인스턴스를 쉽게 얻을 수 있습니다.
~~~~
>>> john = UserFactory()
<User: John Doe>
~~~~
키워드 인수를 팩토리에 전달하여 정의된 특성을 재정의할 수도 있습니다.
~~~~
>>> jack = UserFactory(firstname="Jack")
<User: Jack Doe>
~~~~
지정된 클래스는 많은 팩토리 서브 클래스에 관련 지을 수 있습니다.
~~~~
class EnglishUserFactory(factory.Factory):
    class Meta:
        model = base.User

    firstname = "John"
    lastname = "Doe"
    lang = 'en'


class FrenchUserFactory(factory.Factory):
    class Meta:
        model = base.User

    firstname = "Jean"
    lastname = "Dupont"
    lang = 'fr'
~~~~
다음과 같이 사용할 수 있습니다.
~~~~
>>> EnglishUserFactory()
<User: John Doe (en)>
>>> FrenchUserFactory()
<User: Jean Dupont (fr)>
~~~~

중간의 내용은 추후로 조금 미루고, 지금 회사의 TDD 상황에서 먼저 만들어야 되는 through table에 관련된 Post-generation hooks에 관련된 내용을 먼저 공부하겠습니다.

### Post-generation hooks

일부 객체는 추가 정의 메소드 호출이나 복잡한 정의 처리를 기대합니다. 예를 들어 <tt style="color: #FF0000">`User`</tt>가 관련 <tt style="color: #FF0000">`Profile`</tt>을 가질 필요가 있을 수 있습니다. <tt style="color: #FF0000">`Profile`</tt>은 <tt style="color: #FF0000">`User`</tt> 객체에서 작성됩니다.

이 패턴을 지원하기 위해 factory_boy는 다음 도구를 제공합니다.
* <tt style="color: #FF0000">`PostGenerationMethodCall`</tt>: 특정 속성을 함수 호출에 연결할 수 있습니다.
* <tt style="color: #FF0000">`PostGeneration`</tt>: 이 클래스는 생성된 객체가 인자로 주어진 함수를 호출할 수 있습니다.
* <tt style="color: #FF0000">`post_generation()`</tt>: <tt style="color: #FF0000">`PostGeneration`</tt>과 동일한 기능을 수행하는 데코레이터입니다.
* <tt style="color: #FF0000">`RelatedFactory`</tt>: 첫 번째 팩토리를 작성 / 작성한 후 지정된 팩토리를 작성하거나 작성합니다.

Post-generation hooks는 팩토리 클래스에서 선언된 것과 동일한 순서로 호출되므로 함수는 이전 Post-generation hook에서 적용된 부작용에 의존할 수 있습니다.

### Extracting parameters

모든 post-building hooks는 <tt style="color: #FF0000">`Factory`</tt>로 전달된 속성 집합에서 매개 변수를 선택하는 공통 기반을 공유합니다.

예를 들어, <tt style="color: #FF0000">`PostGeneration`</tt> 훅은 <tt style="color: #FF0000">`post`</tt>로 선언됩니다.
~~~~
class SomeFactory(factory.Factory):
    class Meta:
        model = SomeObject

    @post_generation
    def post(obj, create, extracted, **kwargs):
        obj.set_origin(create)
~~~~
팩토리를 호출 할 때 이 메소드에 대해 몇 가지 인수가 추출됩니다.
* <tt style="color: #FF0000">`post`</tt>인수가 전달되면 <tt style="color: #FF0000">`extracted`</tt> 필드로 전달됩니다.
* <tt style="color: #FF0000">`post__XYZ`</tt>로 시작하는 모든 인수가 추출되고 <tt style="color: #FF0000">`post__`</tt> 접두어가 제거되고 post-generation hook에 전달된 kwargs에 추가됩니다.

추출 된 인수는 <tt style="color: #FF0000">`model`</tt> 클래스에 전달되지 않습니다.

따라서 다음 호출에서 :
~~~~
>>> SomeFactory(
    post=1,
    post_x=2,
    post__y=3,
    post__z__t=42,
)
~~~~

<tt style="color: #FF0000">`post`</tt> hook는 <tt style="color: #FF0000">`1`</tt>을 추출하고 <tt style="color: #FF0000">`{'y': 3, 'z__t': 42}`</tt>를 키워드 인수로 받습니다. <tt style="color: #FF0000">`{'post_x': 2}`</tt>는 <tt style="color: #FF0000">`SomeFactory._meta.model`</tt>에 전달됩니다.

### RelatedFactory

class factory.RelatedFactory(factory, factory_related_name='', **kwargs) [source](http://factoryboy.readthedocs.io/en/latest/_modules/factory/declarations.html#RelatedFactory)

<tt style="color: #FF0000">`RelatedFactory`</tt>는 <tt style="color: #FF0000">`SubFactory`</tt>와 거의 유사하게 동작합니다. 기본 차이점과 관련하여 관련 <tt style="color: #FF0000">`Factory`</tt>가 기본 <tt style="color: #FF0000">`Factory`</tt> 후에 생성됩니다.

<tt style="color: #FF0000">`factory`</tt>
<tt style="color: #FF0000">`SubFactory`</tt>의 경우 <tt style="color: #FF0000">`factory`</tt> 인수는 다음과 같습니다.
* A <tt style="color: #FF0000">`Factory`</tt> subclass
* 또는 <tt style="color: #FF0000">`Factory`</tt> 서브 클래스에 대한 정규화된 경로 (자세한 내용은 [Circular imports](http://factoryboy.readthedocs.io/en/latest/reference.html#subfactory-circular) 참조)

<tt style="color: #FF0000">`name`</tt>
<tt style="color: #FF0000">`factory_related_name`</tt> 매개 변수가 설정된 경우 <tt style="color: #FF0000">`RelatedFactory`</tt> 속성이 설정되는 생성된 객체가 관련 팩토리에 전달될 수 있습니다.
<tt style="color: #FF0000">`name`</tt> 값을 키워드로 사용하여 키워드 인수로 전달됩니다.

<b>Note</b>
<tt style="color: #FF0000">`factory`</tt> 인수에 실제 <tt style="color: #FF0000">`Factory`</tt>를 전달할 때 클래스가 아니라 인스턴스 (즉, 클래스 다음에 ()를 쓰지 않음.)를 전달해야 합니다.
~~~~
class FooFactory(factory.Factory):
    class Meta:
        model = Foo

    bar = factory.RelatedFactory(BarFactory)  # Not BarFactory()
~~~~
<b>Note End</b>

~~~~
class CityFactory(factory.Factory):
    class Meta:
        model = City

    capital_of = None
    name = "Toronto"

class CountryFactory(factory.Factory):
    class Meta:
        model = Country

    lang = 'fr'
    capital_city = factory.RelatedFactory(CityFactory, 'capital_of', name="Paris")
~~~~

~~~~
>>> france = CountryFactory()
>>> City.objects.get(capital_of=france)
<City: Paris>
~~~~

여분의 kwargs는 일반적인 <tt style="color: #FF0000">`ATTR__SUBATTR`</tt> 구문을 통해 관련 factory로 전달될 수 있습니다.
~~~~
>>> england = CountryFactory(lang='en', capital_city__name="London")
>>> City.objects.get(capital_of=england)
<City: London>
~~~~

<tt style="color: #FF0000">`RelatedFactory`</tt> 속성에 값이 전달되면 <tt style="color: #FF0000">`RelatedFactory`</tt> 생성이 비활성화됩니다.
~~~~
>>> france = CountryFactory()
>>> paris = City.objects.get()
>>> paris
<City: Paris>
>>> reunion = CountryFactory(capital_city=paris)
>>> City.objects.count()  # No new capital_city generated
1
>>> guyane = CountryFactory(capital_city=paris, capital_city__name='Kourou')
>>> City.objects.count()  # No new capital_city generated, ``name`` ignored.
1
~~~~

<b>Note</b>
<tt style="color: #FF0000">`RelatedFactory`</tt>의 타겟은, 초기 팩토리가 인스턴스화된 후에 평가됩니다. 그러나 빌드 컨텍스트는 해당 팩토리로 전달됩니다. 즉, <tt style="color: #FF0000">`factory.SelfAttribute`</tt>에 대한 호출이 호출하는 factory의 컨텍스트로 돌아갈 수 있습니다.
~~~~
class CountryFactory(factory.Factory):
    class Meta:
        model = Country

    lang = 'fr'
    capital_city = factory.RelatedFactory(CityFactory, 'capital_of',
        # Would also work with SelfAttribute('capital_of.lang')
        main_lang=factory.SelfAttribute('..lang'),
    )
~~~~

### PostGeneration

class factory.PostGeneration(callable) [source](http://factoryboy.readthedocs.io/en/latest/_modules/factory/declarations.html#PostGeneration)

<tt style="color: #FF0000">`PostGeneration`</tt> 선언문은 생성 작업을 수행합니다.
유일한 인수는 호출 가능 객체이며 일단 기본 객체가 생성되면 호출됩니다.
기본 객체가 생성되면 제공된 호출 가능 객체는 <tt style="color: #FF0000">`callable(obj, create, extracted, ** kwargs)`</tt>로 호출됩니다. 여기서 :
* <tt style="color: #FF0000">`obj`</tt>는 이전에 생성된 기본 객체입니다.
* <tt style="color: #FF0000">`create`</tt>는 사용 된 전략을 나타내는 boolean입니다.
* <tt style="color: #FF0000">`extracted`</tt>이 <tt style="color: #FF0000">`None`</tt>임 <tt style="color: #FF0000">`Factory`</tt> 선언시 <tt style="color: #FF0000">`PostGeneration`</tt> 선언에 값이 전달되지 않으면
* <tt style="color: #FF0000">`kwargs`</tt>는 <tt style="color: #FF0000">`Factory`</tt>를 호출할 때 <tt style="color: #FF0000">`attr__key=value`</tt>로 전달되는 추가 매개 변수입니다.
~~~~
class UserFactory(factory.Factory):
    class Meta:
        model = User

    login = 'john'
    make_mbox = factory.PostGeneration(
            lambda obj, create, extracted, **kwargs: os.makedirs(obj.login))
~~~~

### Decorator

factory.post_generation() [source](http://factoryboy.readthedocs.io/en/latest/_modules/factory/helpers.html#post_generation)
데코레이터가 제공되어 동일한 <tt style="color: #FF0000">`obj`</tt>, <tt style="color: #FF0000">`created`</tt>, <tt style="color: #FF0000">`extracted`</tt> 및 키워드 인수를 <tt style="color: #FF0000">`PostGeneration`</tt>으로 받아들이는 단일 메서드를 꾸미기도 합니다.

~~~~
class UserFactory(factory.Factory):
    class Meta:
        model = User

    login = 'john'

    @factory.post_generation
    def mbox(self, create, extracted, **kwargs):
        if not create:
            return
        path = extracted or os.path.join('/tmp/mbox/', self.login)
        os.path.makedirs(path)
        return path
~~~~

~~~~
>>> UserFactory.build()                  # Nothing was created
>>> UserFactory.create()                 # Creates dir /tmp/mbox/john
>>> UserFactory.create(login='jack')     # Creates dir /tmp/mbox/jack
>>> UserFactory.create(mbox='/tmp/alt')  # Creates dir /tmp/alt
~~~~

### PostGenerationMethodCall

class factory.PostGenerationMethodCall(method_name, *arg, **kwargs) [source](http://factoryboy.readthedocs.io/en/latest/_modules/factory/declarations.html#PostGenerationMethodCall)
<tt style="color: #FF0000">`PostGenerationMethodCall`</tt> 선언은 인스턴스 생성 직후 생성된 객체에 대한 메소드를 호출합니다. 이 선언 클래스는 초기화하는 동안 팩토리 인스턴스의 속성을 생성하는 친숙한 방법을 제공합니다. 선언은 다음 인수를 사용하여 작성됩니다.

<tt style="color: #FF0000">`method_name`</tt>
<tt style="color: #FF0000">`model`</tt> 객체에서 호출 할 메소드의 이름입니다.

<tt style="color: #FF0000">`arg`</tt>
<tt style="color: #FF0000">`method_name`</tt>에 지정된 메소드에 전달할 기본 선택적 선택적 위치 인수

<tt style="color: #FF0000">`kwargs`</tt>
<tt style="color: #FF0000">`method_name`</tt>에 주어진 메소드에 전달할 키워드 인수의 기본 세트

factory 인스턴스가 생성되면, 기본적으로 <tt style="color: #FF0000">`PostGenerationMethodCall`</tt> 선언에 지정된 인수를 사용하여 <tt style="color: #FF0000">`method_name`</tt>에 지정된 메소드가 생성된 오브젝트에 호출됩니다.

예를 들어 인스턴스 생성 중에 생성 된 User 인스턴스에 기본 password를 설정하려면 다음과 같은 <tt style="color: #FF0000">`password`</tt> 속성에 대한 선언을 만들 수 있습니다.
~~~~
class UserFactory(factory.Factory):
    class Meta:
        model = User

    username = 'user'
    password = factory.PostGenerationMethodCall('set_password',
                                                'defaultpassword')
~~~~

<tt style="color: #FF0000">`UserFactory`</tt>에서 사용자를 인스턴스화하면 팩토리는 <tt style="color: #FF0000">`User.set_password('defaultpassword')`</tt>를 호출하여 암호 속성을 작성합니다. 따라서 기본적으로 사용자는 <tt style="color: #FF0000">`'defaultpassword'`</tt>로 설정된 암호를 갖게됩니다.
~~~~
>>> u = UserFactory()                             # Calls user.set_password('defaultpassword')
>>> u.check_password('defaultpassword')
True
~~~~

<tt style="color: #FF0000">`PostGenerationMethodCall`</tt> 선언에 인수나 인수가 하나도 없으면 속성 이름과 일치하는 키워드 인수를 통해 재정의된 값을 메서드에 직접 전달할 수 있습니다. 예를 들어, 위의 선언에 지정된 기본 암호를 인스턴스화 중에 원하는 암호를 키워드 인수로 전달하여 무시할 수 있습니다.
~~~~
>>> other_u = UserFactory(password='different')   # Calls user.set_password('different')
>>> other_u.check_password('defaultpassword')
False
>>> other_u.check_password('different')
True
~~~~

<b>Note</b>
Django 모델의 경우 <tt style="color: #FF0000">`PostGenerationMethodCall`</tt>에 의해 호출된 객체 메소드가 객체를 데이터베이스에 저장하지 않는 한 <tt style="color: #FF0000">`create()`</tt>를 수행하면 객체를 다시 저장해야한다는 것을 명심해야 합니다.
~~~~
>>> u = UserFactory.create()  # u.password has not been saved back to the database
>>> u.save()                  # we must remember to do it ourselves
~~~~
<tt style="color: #FF0000">`DjangoModelFactory`</tt>에서 하위 클래스로 분류하여 이를 피할 수 있습니다. 예 :
~~~~
class UserFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = User

    username = 'user'
    password = factory.PostGenerationMethodCall('set_password',
                                                'defaultpassword')
~~~~
<b>Warnings</b>
<tt style="color: #FF0000">`PostGenerationMethodCall`</tt>은 일관되고 간단한 API를 유지하기 위해 최대 하나의 위치 인수를 허용합니다. 다른 모든 매개 변수는 키워드 인수로 전달되어야합니다.
</b>Warnings End</b>
<b>Note End</b>

팩토리 인수에서 추출한 키워드는 <tt style="color: #FF0000">`PostGenerationMethodCall`</tt> 선언에 있는 기본값으로 병합됩니다.
~~~~
>>> UserFactory(password__disabled=True)    # Calls user.set_password('', 'sha1', disabled=True)
~~~~

### Module-level functions

<tt style="color: #FF0000">`Factory`</tt> 클래스와 다양한 [Declarations](http://factoryboy.readthedocs.io/en/latest/reference.html#declarations) 클래스 및 메소드 외에도 factory_boy는 모듈 수준의 함수 몇 개를 제공합니다. 이 함수는 대부분 경량 팩토리 생성에 유용합니다.

