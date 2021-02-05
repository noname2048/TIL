# 장고 기본인증

## 001

###### django 기본인증 구현을 accounts 에 하는이유

django.conf.global_settings.py

```
LOGIN_URL = '/accounts/login/'
LOGIN_REDIRECT_URL = '/accounts/profile/'
```

django.contrib.auth.urls.py

* 기본세팅이 다 되어있지만 (settings에 넣어주기만 하면 된다)

LoginView

* authentication_form = None
* form_class = AuthenticationForm
*  template_name = 'registration/login.html'

###### 로그인후 RedirectURL을 Get으로 지정하는법

```
/accounts/login/?next=
```

## 002

Profile 페이지 만들기

context_processors 중에 auth단을 통해 전달되는 user context

user

* 로그인시에는 User 모델 인스턴스
* 로그아웃시에는 AnonymousUser (인스턴스)

```python
# profile_form.html
{% extends "accounts/layout.html" %}

{% load bootstrap4 %}

{% block title %}
    HI
{% endblock title %}

{% block content %}

<form action="" method="post">
    {% csrf_token %}
    {% bootstrap_form form %}
    {% buttons %}
        <button class="btn btn-primary">프로필 수정</button>
    {% endbuttons %}
    <input type="submit" value="Login" />
</form>

{% endblock content %}

```



```python
# profile.html
{% extends "accounts/layout.html" %}

{% block content %}
    <h2>User: {{ user }}</h2>
{#    {{ user.is_authenticated }}#}

    {% if user.profile %}
        <ul>
            <li>{{ user.profile.address }}</li>
            <li>{{ user.profile.zipcode }}</li>
        </ul>
    {% endif %}

    <a href="{% url "accounts:profile_edit" %}" class="btn btn-primary">
        수정하기
    </a>
{% endblock %}

```

## 003

```python
{%  if not user.is_authenticated %}

    <li class="nav-item">
    	<a class="nav-link" href="{% url 'accounts:signup' %}">회원가입</a>
    </li>
    <li class="nav-item">
    	<a class="nav-link" href="{% url 'accounts:login' %}">로그인</a>
    </li>

{% else %}

    <li class="nav-item">
    	<a class="nav-link" href="{% url 'accounts:profile' %}">프로필</a>
    </li>
    <li class="nav-item">
    	<a class="nav-link" href="{% url 'accounts:logout' %}?next=">로그아웃</a>
    </li>

{% endif %}

```

## 004

```python
class LoginForm(AuthenticationForm):
    answer = forms.IntegerField(help_text='3 + 3 = ?')

    def clean_answer(self):
        answer = self.cleaned_data.get('answer')
        if answer != 6:
            raise forms.ValidationError("땡~!")
        return answer

```

## 005

로그인 할때에는 "패스워드"에 "소금"을 붙여서 "알고리즘"을 반복하여 "해쉬"를 저장한다.

auth.form 에 살펴보면 set_password로 저장해야한다. !를 붙일경우 다른 시스템 없이 로그인 할 수 없다.

장고의 모든 인증의 기본은 auth이다.

settings.AUTH_USER_MODEL과  관계를 맺으면 된다.

절대 

```
from django.contrib.auth.models import User
```

로 쓰면 안되고

```
from django.contrib.auth import get_user_model
```

로 써야한다. (비슷한 예료 from django.conf import global_settings(X) import settings(O)) 가 있다.



```
signup = CreateView.as_view(
    model=get_user_model(),
    form_class=UserCreationForm,
    success_url=settings.LOGIN_URL,
    template_name='accounts/signup_form.html',
)

```

## 006

회원 가입후에 로그인 처리하기

LoginView에 보면 auth_login이라는 함수가 있다.

auth_login으로 가져와서



```
class SignupView(CreateView):
    model = get_user_model()
    form_class = UserCreationForm
    # success_url = settings.LOGIN_URL
    success_url = settings.LOGIN_REDIRECT_URL
    template_name = 'accounts/signup_form.html'

    def form_valid(self, form):
        response = super().form_valid(form)
        user = self.object
        auth_login(self.request, user)

        return response


signup_view = SignupView.as_view()

```



## 007

```python
{%  if not user.is_authenticated %}

<li class="nav-item">
<a class="nav-link" href="{% url 'accounts:signup' %}">회원가입</a>
</li>
<li class="nav-item">
<a class="nav-link" href="{% url 'accounts:login' %}?next={{ request.get_full_path }}">로그인</a>
</li>

{% else %}

<li class="nav-item">
<a class="nav-link" href="{% url 'accounts:profile' %}">프로필</a>
</li>
<li class="nav-item">
<a class="nav-link" href="{% url 'accounts:logout' %}?next={{ request.get_full_path }}">로그아웃</a>
</li>

{% endif %}
```



###### ?next= 를 이용한 다음경로 지정하기

해당 경로를 통해 적절히 처리 가능

next_page 인자를 지정하거나, LOGOUT_REDIRECT_URL 확인하기