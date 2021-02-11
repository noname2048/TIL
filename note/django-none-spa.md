## 002

https://2019.stateofcss.com/technologiess/css-frameworks/

jQuery, Bootstrap4

###### requirements 관리팁

```
requirements
| - commmon.txt
|	Django~=3.0.0
|
| - dev.txt
|	-r common.txt
|
| - prod.txt
|	-r common.txt
|
requiremnets.txt
	-r dev.txt

```

###### static 과 templates DIR 관리

```
Just-Folder (name : asklecture)
| - project (name : asklecture)
| - | - templates
| - | - static
| - | - settings.py 
| - manage.py
| - requirements.txt
```

```python
# settings.py
STATIC_URL = '/static/'
STATICFILES_DIR = [
    BASE_DIR / 'asklecture' / 'static' /
]
STATIC_ROOT = BASE_DIR / 'static' /

MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media' /

```

git은 빈 폴더를 저장하지 않는다.

window 초코렛 - 깃

맨처음 프로젝트는 migrate 만 해도 된다.

###### 개발환경이나 등등

```
manage.py -> setdefault("DJANGO_SETTINGS_MODULE", "<project_name>.settings.dev"))
asgi.py, wsgi.py -> setdefault("DJANGO_SETTINGS_MODULE", "<project_name>.settings.prod"))

```



###### 프로젝트 secret 관련 설정

```python
# SECURITY WARNING: keep the secret key used in production secret!
SECRET_DIR = Path(__file__).resolve().parent / "secrets.json"
try:
    SECRET_KEY = json.load(open(SECRET_DIR))["SECRET_KEY"]
except:
    raise ImproperlyConfigured("SECRET_KEY NOT FOUND!")
```

###### django tool bar 관련

pip install django-debug-toolbar

apps = [ debug_toolbar ]

middleware = ['...']

## 003

모든 path  에 대해서는 re_path('') 빈문자열을 넣는다

###### from django.contrib.auth import get_user_model

```
In [1]: from django.contrib.auth import get_user_model

In [2]: User = get_user_model()

In [3]: user = User.objects.first()

In [4]: user
Out[4]: <User: noname>

In [6]: user.password = '1234'

In [7]: user.set_password('1234')

In [8]: user.password
Out[8]: 'pbkdf2_sha256$216000$LUV4PV1hVoeV$yjtIYGgdenPFNlEhk6ilCuPWfN1XJtTkjbLxPKCRKpk='
```

