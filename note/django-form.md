## 001 HTML Form

###### form 의 필수 속성

* action: 주소
* method: 전송방식
* enctype: 인코딩방식 (Get은 고정이라 붙이지 않는다)

###### Get 에서의 enctype 방식

* application/x-www-form-urlencoded (디폴트)
  URL인코딩을 수행하여 QueryString 형태로 전달
  파일 업로드가 불가능 하다!
* multipart/form-data
  파일업로드 가능. 파일 안보내도 상관이 없다.
* text/plan
  스펙에는 있지만 안쓰는 기능

###### urlencoded

* 공백은 + 로 인코딩 되고, Special문자는 ascii 16진수로 문자열, UTF8 인코딩 16진수 문자열로 변환
* 방 -> 0xEB, 0xB0, 0xA9
* 브라우저가 디코딩해서 보여주는것일 뿐이다.

###### form 요청에서 인자 보내는 2가지 방법

1. Get 처럼 Query String 으로 보내기.
2. Body에 인코딩의 인자를 보내기

HTML에서 Header 와 Body의 구분은 CR/LF 로 구분한다.

action칸에 /나 host가 오면 절대경로, 아니라면 상대경로이다

장고에서는 POST와 GET을 같이 받는다. action칸을 비워도 된다.

파일요청을 시도할 경우에는 반드시 enctype을 multipart/form-data로 지정한다.

###### 장고뷰에서의 인자 접근

* request.GET
* request.POST
* request.FILES(MultiValueDict 객체)

## 002

###### HttpRequest 객체

클라이언트로부터의 모든 요청 내용을 갈무리하여 가지고 있다.

###### Form 관련 속성

* .GET, .POST, .FILES [^1]왜 대문자로 지정하였을까
* QueryDict 타입은 일반 Dict과 다르게 유일한 key-value가 아닌 key-values 값 가능

###### MultiValueDict

* name=Tom&name=Tomi&name=Peter
  세개의 이름 모두 서버로 전송된다.

* d = MultivalueDict({'name': ['Adrian', 'Simon']})
  와 같이 설정한다면

* d['name']
  'Simon' (마지막값만 리턴한다)

* d.getlist('name')
  ['Adrian', 'Simon'] (모두 리턴한다)
* d.getlist('none')
  [] (빈 리스트가 리턴된다.)
* 기본적으로 immutable 속성을 가진다.
  기본적으로 mutable=False 라, 속성을 바꾸면 `__setitem__` 에서 `__asset_mutable()` 함수로 오류를 `raise`한다.

###### HttpResponse

* 다양한 응답을 Wrapping 한다.

* View에서 반환값으로 이 객체를 리턴하기를 기대한다.

* MIDDLEWARE1 -> MIDDLEWARE2 -> MIDDLEWARE3 -> View 함수 -> MIDDELWARE4 -> MIDDELWARE5
  요즘 MiDDLEWARE는 함수형식으로 만드는 것을 선호한다.

* dict-like 인터페이스로 응답의 커스텀 헤더를 추가/삭제 가능하다

  ```python
  response = HttpResponse()
  response['Age'] = 120
  ```

###### django.http.JsonResponse

* `from django.core.serializers.json import DjangoJSONEncoder` 와 같이 쓰인다.
  파이썬의 특별한 룰이 포함되어있다. 
* json.dumps(data) 를 통해 json을 통해 리턴 가능하다!

###### django.httpStreamingHttpResponse

* 메모리를 많이 차지하는 객체에 대한 효율적인 응답
* django는 short-lived 요청에 맞게 디자인 되어있기 때문에, 사용을 유의하자.
* HttpResponse를 상속받지 않는다.
* `__iter__`을 이용하는데, 파이썬에서는 generator을 응답하게 되어있다.

###### django.http.FileResponse

* StreamingHttpResponse를 상속받았다.
* Content-Length, Content_type, Content_Disposition 헤더 자동지정
* 인자는 open_file: Streaming Content
* as_attachment: Content_disposition 헤더 지정여부

## 003

###### Form

* 장고 폼이나 Serailzer을 활용하지 않으면 Django를 쓸 이유가 없다.
* 주요역할
  * HTML
  * Validation
  * dict 형태로 제공
* USER < form > VIEW < Model > Database
* forms.CharField() 를 쓴다 ORM 처럼 TextField가 없다.
* widget
* Get: 입력폼
* Post 데이터가 있으면 유효성 검증 시행, 실패시 입력 폼 보여주기
* **self.cleaned_data (성공한 데이터만 남는다)

###### get_absolute_url이 구현이 되어있어야 redirect를 사용할 수 있다!

###### ModelForm

* 모델로 부터 필드정보를 가져와서 구성해준다.

* 모델이 변경이 되면 필드정보도 변경이 되니 편리하다

* 폼과 모델을 연결하는 것으로 save form을 지원한다.

* ```
  class PostForm(forms.ModelForm):
  	class Meta:
  		model = Post
  		fields = '__all__'
  ```

###### Form

* 유효성 검사 필드를 만들어서 넣을 수 있다.
* validators=[min_length_3_validator]
* 주의, 모델이 아니라 폼에서 작동한다. 
* 모델에서 설계하면 모델폼으로 가져올 수 있다.

```
from django.core.validators import MinLengthValidator

class Post(models.Model):
    message = models.TextField(
        validators=[MinLengthValidator(10, "길이가 달라요")]
    )
```



어드민에서는 모델폼을 자체적으로 생성한다

###### form.as_table form.as_ul 등을 가능 (기본적으로는 table사용)

required 란은 꼭 채워넣어야한다

```python
<form action="" method="post" enctype="multipart/form-data">
    {% csrf_token %}
    <table>
        {{ form }}
    </table>
    <input type="submit" value="저장" />

</form>
```



## 004 CSRF 사이트 간 요청 위조 공격

Cross Site Request Forgery

* POST 요청에 한해 CsrfViewmiddleware를 통한 체크
* Token 값이 없거나 유효하지 않으면 403 Forbidden 응답
* 폼을 요청할때 CSRF Token 값이 할당된다.
* CSRF Token != 유저인증(Token,) JWT (Json  Web Token)

* @csrf_exempt 로 특정 POST를 요청하는 뷰에서 해제 가능
* APP-API의 경우에 끈다
* django csrf jquery 공식문서를 활용해서 
* ajax : 비동기 자바스크팁트 xml 요청

###### Jquery를 활용할때 쿠키에서 csrf 토큰을 가져와서 사용한다. (폼태그에서 가져올 수도 있다.)

```var csrftoken = getCookie('csrftoken')```



## 005 ModelForm

###### ModelForm

* 장고 Form을 상속하여 Model로 부터 필드정보를 읽어들여 Form Fields를 세팅한다.

- 지정 model instance로의 저장(save) 지원

- ```python
  class PostModelForm(forms.ModelForm):
  	class Meta:
  		model = Post
  		fields = '__all__'
  ```

- ```
  post = form.save(commit=True)
  ```

  commit은 모델플래그를 호출 할 것인가를 묻는다. 즉, 모델에도 저장할것인가?
  form.save_m2m() 등을 호출한다

- form.save() != instance.save()

- commit=False는 지연시키고자 할 떄 사용한다.

```
path('<int:pk>/edit/', views.post_edit, name='post_edit'),
```

```
{{ post.message|linebreaks }}
{{ post.message|linebreaksbr }}
```

###### exclude = []

- exclude 는 쓰지 말자 whitelist를 쓰도록 합시다.

- 유효성 검사는 fields 에 지정된 경우에만 사용한다. 

###### user 정보 넣기

```
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            return redirect(post)
```

###### django.request.meta

* REMOTE_ADDR 참조가능

* ```
  # models.GenericIPAddressField
  ```

```
post.ip = request.META['REMOTE_ADDR']
post.author = request.user
```

###### message_framework 이용가능

* django.contrib.messages

###### Form을 끝까지 쓰자

* form.cleaned_data['message'] 로 접근
  request.POST['message'] 보다 안전하다.

## 006

Form 유효성 검사가 수행되는 시점은 is_valid를 호출할 때이다.

###### 검사 호출 로직

* cls.full_clean() 호출
  * 각 필드 Type에 맞춰 유효성 검사
* form 객체 내에서 cls.clean_필드명() 있는지 검사하여 호출 (Form 클래스에 구현)
* cls.clean() 함수가 있다면 호출해서 유효성을 검사한다
* validator

###### Validator 함수를 통한 유효성 검사

* 값이 원하는 조건에 맞지 않을 때, ValidationError 예외를 발생한다.
* 리턴값은 사용되지 않는다.
* Form 클래스내 Clean은 값을 반환하여 그 값을 사용한다.

```
    def clean_message(self):
        message = self.cleaned_data.get('message')

        if message:
            message = re.sub(r'[a-zA-Z]+', '', message)
            return message
```

```
hasattr(self, 'clean_%s' % name)()
self.add_error(name, e)
```

###### Validator

* 함수형 - 값인자를 1개 받은 callable object
* 클래스형, 인스턴스가 callable object



모델폼은 필드조건을 가져오는것 뿐, 이미 채워지고 나면 그냥 폼이다.

###### 빌트인 Validators

* RegexValidator
* EmailValidator
* URLValidator
* validate_email
* validate_slug



###### FileExtensionValidator: 파일 확장자 허용, 파일확장자만으로 그 포맷임을 확정할 수 없다.

* models.EmailField (CharField)
* models.GenericValidators



필드별 Error 기록 은 clean_필드명()

혹은 Non 필드 Error 기록 clean이 기록



###### validators vs clean

* 가급적 model fat
* clean은 form에서 1회성 유효성 검사 루틴에서
* clean은 값을 변경하고 싶을 때



###### unique_together = ['a', 'b']

* 로 묶어서 사용가능

* model meta option 에 설명이 되어있다.  [https://docs.djangoproject.com/en/3.1/ref/models/options/](https://docs.djangoproject.com/en/3.1/ref/models/options/)

## 007

###### Messages Framework

- HttpRequest를 통해 메세지를 남긴다. 메시지 1회 노출후 사라진다.

파이썬 로깅 모듈의 Level을 차용

레벨에 따라 로깅 여부 판단

```
messages.add_message(request, messages.SUCCESS, '새글이 등록되었습니다.')
messages.success(request, '새 글이 등록되었습니다.')
```

메세지는 쌓이면 소비를 해야함

```
messages.info(request, 'messages 테스트')
```

```
{% for message in messages %}
    <div class="alert alert-{{ messages.tags }}">
        {{ message.message }}
    </div>
{% endfor %}
```

###### Context Processors

* messages가 따로 지정하지 않아도 template에서 쓸수 있는이유 (settings에 정의되어있다.)

함수를 살펴보면 messages안에 적혀있다.



###### bootstrap4와 대응하기

* alert-primary

* debug, success, error, warning, info

```
from django.contrib.messages import constants as messages_constants

MESSAGE_TAGS = {
    messages_constants.DEBUG: 'secondary',
    messages_constants.ERROR: 'danger',
}
```

```
# MESSAGE_LEVEL = messages_constants.INFO
MESSAGE_LEVEL = messages_constants.DEBUG
```



###### bootstrap4/messages.html 활용가능

{% load i18n bootstrap4 %}

```
{% load bootstrap4 static %}
```

```
{% bootstrap_messages %}
```



## 008

confirm 은 get 요청으로 한번 더 받아서 삭제

###### post_confirm_delete.html

```
{% extends 'instagram/layout.html' %}


{% block content %}
<form action="" method="post">
    {% csrf_token %}
    <div id="alert alert-danger">
        정말 삭제하시겠습니까?
    </div>
    <a href="{{ post.get_absolute_url }}" class="btn btn-secondary">내용보기</a>
    <input type="submit" value="삭제하겠습니다" class="btn btn-danger" />
</form>
{% endblock %}
```

```
@login_required
def post_delete(request, pk):
    post = get_object_or_404(Post, pk=pk)

    if request.method == "POST":
        post.delete()
        messages.success(request, '게시글을 삭제하였습니다.')
        return redirect('instagram:post_list')

    return render(request, 'instagram/post_confirm_delete.html', {
        'post': post,
    })
```



## 009

FormView(모체), CreateView, UpdateView, DeleteView

###### ProcessFormView

* 했던 로직들을 사용한다.



###### FormMixin

form이 등록되어있지 않다면 기본 인스턴스를 만들어서 활용한다.

get_context_data(self, **kwargs):



###### FormView

* 상속받으면 get_absolute_url 구현이 필수이다.
* CBV ModelFormMixin 에서 구현



class PostCreateView(FormView):

form_class = PostForm

template_name = 'myapp/post_form.html'