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

# 004

로그인 구현

* settings.py > AUTH_USER_MODLE = ""
* AbstractUser
* UserCreationForm
* 

## 005

이메일 보내기

1. SMTP (이메일을 많이 보낼경우 전문적인 서비스를 써라)
   sendgird (5만건 무료, 1천건 월별, mailgun, mailjet, Amazon SES

2. django-sendgrid-v5



```
from django.core.mail import send_mail
send_mail("", "", "", [""], fail_siltently=False)
```



여러가지가 섞여서 SMTP 따로 구현

```python
 # models.py
 from django.core.mail import send_mail
 
 def send_welcom_email(self):
     subject = "회원가입을 환영합니다 test"
     content = "환영환영~"
     sender_email = "sungwook@noname2048.dev"
     return send_mail(subject, content, sender_email, [self.email], fail_silently=False)
```

```python
def signup(request):
    if request.method == "POST":
        form = SignupForm(request.POST)
        if form.is_valid():
            signed_user = form.save(commit=False)

            messages.success(request, f"Welcome {signed_user.username}")

            if signed_user.send_welcom_email() == 1:  # FIXME: selary 로 하기
                next_url = request.GET.get("next", "/")
                return redirect(next_url)
            else:
                form = SignupForm()
    else:
        form = SignupForm()
    return render(
        request,
        "accounts/signup_form.html",
        {
            "form": form,
        },
    )
```

```python
# settings.py
try:
    SENDGRID_API_KEY = json.load(open(SECRET_DIR))["SENDGRID_API_KEY"]
except:
    raise ImproperlyConfigured("SECRET_KEY NOT FOUND!")


SENDGRID_SANDBOX_MODE_IN_DEBUG = False

# EMAIL_BACKEND = "sendgrid_backend.SendgridBackend"
EMAIL_HOST = "smtp.sendgrid.net"
EMAIL_PORT = "587"
EMAIL_HOST_USER = "apikey"
EMAIL_HOST_PASSWORD = SENDGRID_API_KEY
EMAIL_USE_TLS = True
DEFAULT_FROM_EMAIL = "sungwook.csw@noname2048.dev"

```

###### 가입한 서비스

* sendgrid - 이메일 SMTP
* namecheap - 도메인구매 및 메일박스관리 (POP3?)
* porkpun

## 10

PasswordChangeForm

PasswordChangeView

등등 으로 해결

```
class AcountLoginView(LoginView):
    template_name = "accounts/login_form.html"
    next_page = reverse_lazy("accounts:root") # 왜 reverse 쓰면 안되고 reverse_lazy를 써야할까

```



큰 실수: 

django 3.1.5 버전으로 개발 중에 3.2.문서를 보고 개선된 기능에 대해 함수를 작성했다.

작동하지 않는건 당연하다... (한참 삽질했다.)

LoginView 에 next_page 달아보겠다고 한참...



######  내가 쓴 log 는 왜 server time 이 다르게 나올까

```
logger = logging.getLogger("django.server")
logger.info("logout!")
```



```
[13/Feb/2021 07:16:09] "POST /accounts/login/?next=/accounts/ HTTP/1.1" 302 0
[13/Feb/2021 07:16:09,749] logout!
[13/Feb/2021 07:16:09] "GET /accounts/ HTTP/1.1" 200 14859
[13/Feb/2021 07:16:12] "GET /static/bootstrap-5.0.0-beta1-dist/css/bootstrap.css.map HTTP/1.1" 304 0
[13/Feb/2021 07:16:40,680] logout!
[13/Feb/2021 07:16:40] "GET /accounts/ HTTP/1.1" 200 14858
```



3자리 숫자가 붙어 있다.

## 11

```python
Python 3.9.1 (default, Jan 28 2021, 01:00:29) 
Type 'copyright', 'credits' or 'license' for more information
IPython 7.19.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: Post.objects.all()
Out[1]: <QuerySet [<Post: 하하 #태그>]>

In [2]: post = _
```

언더바는 전의 결과값을 가지고 있다!

```python
def extract_tag_list(self):
    return re.findall(r"#([a-zA-Z\dㄱ-힣]+)", self.caption)
```

소괄호로 묶으면 # 이 제거된다!~

###### extract_tag_list 를 python shell 에서 실험한 내용

``` ipython
Python 3.9.1 (default, Jan 28 2021, 01:00:29) 
Type 'copyright', 'credits' or 'license' for more information
IPython 7.19.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: post = Post.objects.first()

In [2]: post.extract_tag_list()
Out[2]: [<Tag: 태그>]

In [3]: post.tag_set.all()
Out[3]: <QuerySet []>

In [4]: post.tag_set.add(*post.extract_tag_list())

In [5]: post.tag_set.all()
Out[5]: <QuerySet [<Tag: 태그>]>

In [6]: 

```

###### save 하고 tag_set.add 하기 -> m2m field는 pk 가 실제로 필요하다!

```python
post.author = get_user(request)
post.save()
post.tag_set.add(*post.extract_tag_list())
```

## 013 유저페이지 구현

re_path에 달러를 꼭 붙일것!

```python
re_path(r"^(?P<username>[\w.@+-]+)/$", views.user_page, name="user_page"), # 잘못됨
re_path(r"^(?P<username>[\w.@+-]+)/", views.user_page, name="user_page"),
```

