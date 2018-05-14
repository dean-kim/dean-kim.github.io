title:  "Django settings secret key"
date:   2018-05-14 19:30:00
author: Dean Kim
categories: Docker
tags:	Django settings secret-key
cover:  "/assets/instacode.png"
---

Django 프로젝트 생성을 하면 settings.py라는 파일이 기본으로 생성됩니다.

Django 프로젝트와 관련된 많은 설정 항목들이 있는데 그 중에 secret key와 database 설정에 사용되는 설정 값들을 따로 분리해 관리하는 것에 대한 글을 작성했습니다.

그렇다면 Django secret key는 무엇인지부터 알아보겠습니다.

## SECRET_KEY

출처 : [Django 공식문서](https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-SECRET_KEY)

특정 장고 설치를 위한 비밀 키. 이것은 암호화 서명을 제공하는데 사용되며 고유한 예측할 수 없는 값으로 설정해야 합니다.

django-admin startproject는 새로운 프로젝트마다 무작위로 생성된 <tt style="color: #FF0000">`SECRET_KEY`</tt>를 자동으로 추가합니다.

키의 사용은 텍스트 또는 바이트라고 가정해서는 안됩니다. 모든 용도는 force_text() 또는 force_bytes()를 통해 원하는 유형으로 변환해야 합니다.

Django는 SECRET_KEY가 설정되어 있지 않으면 시작할 수 없습니다.

경고

이 값을 비밀로 유지하십시오.

Django를 알려진 SECRET_KEY로 실행하면 장고의 보안 보호 기능을 많이 무효화하고 권한 상승 및 원격 코드 실행 취약점이 발생할 수 있습니다.

비밀 키는 다음 용도로 사용됩니다.

* django.contrib.sessions.backends.cache 다른 세션 백엔드를 사용하거나 기본 get_session_auth_hash()를 사용하는 모든 세션.
* CookieStorage 또는 FallbackStorage를 사용하는 모든 메시지.
* 모든 PasswordResetView 토큰.
* 다른 키가 제공되지 않는 암호화 서명 사용.

비밀 키를 돌리면 위의 모든 내용이 무효화됩니다. 비밀 키는 사용자의 암호로 사용되지 않으며 키 순환은 영향을 미치지 않습니다.

Django에서 secret key는 Django 보안설정에 사용되는 중요한 설정이니만큼 보안을 고려해 사용하는 것이 좋을 것입니다.

<tt style="color: #FF0000">`Two Scoops of Django`</tt>라는 책에서는 이를 환경변수로 분리하거나 따로 파일로 분리하여 관리하는 방법을 권장하고 있습니다.

저는 따로 파일로 분리하여 관리하는 방법을 선택해 이 글을 작성했습니다.

## Database

settings.py를 보면 database에 대한 설정을 하는 항목이 있습니다.

어떤 port를 쓰는지 datebase 이름, 비밀번호 등을 설정하게 되는데 이 역시 보안을 고려해 공개하는 것보다는 따로 관리하는 것이 좋습니다.

## 파일로 분리하여 관리

저는 따로 <tt style="color: #FF0000">`secrets.json`</tt>이라는 파일에 secret key와 database 설정에 관련된 값들을 관리하고자 합니다.

먼저 secrets.json을 만들고 다음과 같이 작성합니다.

~~~~
# in secrets.json
{
  "DJANGO_SECRET_KEY" : "자신의 Django 프로젝트의 secret key",
  "DATABASE_NAME": "database의 이름",
  "DATABASE_USER": "database의 user",
  "DATABASE_PASSWORD": "database의 password"
}
~~~~

파일을 작성 후 저장한 뒤에 <tt style="color: #FF0000">`.gitignore`</tt>에 파일을 등록해 공개된 github 계정에 저장되지 않도록 합니다.

~~~~
# in .gitignore
...
# Secret
secrets.json
...
~~~~

그리고 Django settings의 해당 항목들을 수정합니다.

~~~~
# in settings.py
...
with open("./refrigerator_manager/settings/secrets.json") as f:
    secrets = json.loads(f.read())


# Keep secret keys in secrets.json
def get_secret(setting, secrets=secrets):
    try:
        return secrets[setting]
    except KeyError:
        error_msg = "Set the {0} environment variable".format(setting)
        raise ImproperlyConfigured(error_msg)

SECRET_KEY = get_secret("DJANGO_SECRET_KEY")

DATABASES = {
    'default': {
        ...
        'NAME': get_secret("DATABASE_NAME"),
        'USER': get_secret("DATABASE_USER"),
        'PASSWORD': get_secret("DATABASE_PASSWORD"),
        ...
    }
}
...
~~~~

이렇게 작성해 두면 본인의 github repo에 push했을 때 secrets.json을 제외하고 push가 되어 보안에 영향을 줄 수 있는 값들을 따로 관리할 수 있습니다.

## 앞으로 추가할 내용

현재 회사에서는 secret key를 AWS 배포 시 환경변수로 등록해 배포하고 있습니다.

개인 프로젝트를 AWS에서 배포할 시점에 AWS에서 환경변수로 등록해 배포하는 내용을 추가하도록 하겠습니다.