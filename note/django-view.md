# 001

호출 가능한 객체
FBV Function Based View
CBV Class Based View

HttpRequset - 인자: 현재 요청에 대한 모든 내역을 담는다
URL Capture Values
2번째 인자:

instagram/10/ 등의 주소에 대해 일일히 설정할 수 있지만
path, <int> 쓰면 형변환을 해서 넘어오게 됩니다.
Converter의 to_python을 이용해 변환
필히 HttpResponce를 리턴해야합니다.

특정 view를 감싸는 함수들 -> 미들웨어
request를 처리할때, 함수를 호출할때 등등
responece에 대해 처리

process_responce 에서 responce 객체는 HTTPREQUSET객체로 가정
django.shortcuts.render

파일 like 객체에 대한 str/bytes 타입도 지원
```python
def post_detail(reuset, pk):
    responce = HttpResonce()
    response.write("Hello World")
    response.write("Hello World")
    response.write("Hello World")
    return responce
```
스트리밍 구현하실때 활용도가 높다.
장고내 모든 문자열은 utf-8로 처리된다. (일일히 인코딩할 필요가 없다.)

HttpRequset
django는 타입이 다르다고 해서 에러가 나지 않지만, hint를 통해 linter가 잡아줄 수 있다.

CBV에서 접근하는 두가지 방법
as_view에서 (model=Item 하거나)
class를 상속한뒤, model=Item 하기

# 002

파일 다운로드와 관련해서 여러가지 처리가 필요하다
django.http.FileResponse 공부하면 수비다.
pandas를 이용해 CSV 응답을 생성하기

to_csv, to_excel(io),  io =  BytesIO()
엑셀을 활용할때는 pandas가 xlwt를 활용해서(랩핑해서)사용한다.
업로드를 했을때에 참고하여 처리 결괄ㄹ 응답으로 내어줄 수 있다.
HTTP 프로토콜만 지키면 지원가능
Pillow를 통한 이미지 응답 생성

canvas.save(response, format='PNG')
사원증에 들어가는 이미지를 만들어 볼 수 있겠다

# 003 

URL_DISPATCHER
최상의 URL CONF 는 settings에 URL CONF에 있다.
include 문자열로 지정하면 장고가 찾아서 import를 해주는 부분이 되겠구요
순차적으로 찾는다.
url이 없으면 404 not found 오류가 발생
```
path('archives/<int:year>/', views.archives_year),
# re_path(r'archives/(?P<year>\d+)/', views.archives_year),
# re_path(r'archives/(?P<year>\\d+)/', views.archives_year),
```
같은 표현

slug는 Unicode 지원이 안되니 custom을 만들자
app_name = '앱이름' ->  url reverse

# 004

FBV 공통 기능은 장식자
```
@api_view()
@throttle_classese
def my_vew(request):
    return rEspon
```

CBV
django.views.generic

써드파티 django-braces
as_view 를 통해 View 함수를 생성

```
# def post_detail(request: HttpRequest, pk: int) -> HttpResponse:
#     try:
#         post = Post.objects.get(pk=pk) # DoesNotExist
#     except Post.DoesNotExist:
#         raise Http404
#     return render(request, 'instagram/post_detail.html', {
#         'post': post,
#     })

def post_detail(request: HttpRequest, pk: int) -> HttpResponse:
    post = get_object_or_404(Post, pk=pk)
    return render(request, 'instagram/post_detail.html', {
        'post': post,
    })
```

```
class PostDetailView(DetailView):
    model = Post
    pk_url_kwarg = 'id'

# post_detail = PostDetailView.as_view()
```

# 005

Base views
내부 구현에 대해 이해하자
DetailView, ListView,
ArchiveIndexVieew, YearArchiveView, MonthArchiveView, WeekArchiveView, DayArchiveView
...

실제로 Mixin 클래스는 파이썬에 없고, 상속받는 용이라는 뜻으로 mixin을 썼음
View <-
TemplateView <-
RedirectView <-

handler 로 처리,
method가 결정되어있지 않으면 ... 다른 응답 반환
setattr로 구현

view.as_view()에 넘기는 kwargs 들은,
class에서 사전형태로 __init__ 에서 다뤄진다.
class TemplateResponseMixin: 에서 다룬다

```python
from django.views.generic import TemplateView


class Rootview(TemplateView):
    template_name = 'root.html'

urlpatterns = [
    # path('', TemplateView.as_view(template_name='root.html'), name='root'),
    path('', Rootview.asview(), name='root.html'),
```

template 로더
파일 시스템 로더 _> 지정한 경로에서

location.href = "http://naver.com"
RedirectView
permanent 
defaul=False
url=None (호스트가 같으면 호스트를 생략하고 절대경로로 줄것)
pattern_name=None (url_reverse)
query_string=False (QueryString 을 그대로 넘길 것인지?)
장고는 pattern_name을 지정하는 것을 선호함

# 006 generic view
List/DetailView

from django.views.generic import DetailView
from .models import Post

Detail view가 내부적으로 소문자로 넘긴다.
pk 혹은 등등

SingleObjectMixin에 pk-url_kwarg = 'pk' 가 있기 때문에
Url Capture value에 pk 가 있어야한다.
queryset=Post.objects.filter(is_public=True)

# 007

ListView는 페이징 처리를 지원한다.
template_name 이 지정되지 않았다면 역시나 찾는다.
allow_query
queryset
model
paginate_by
pagianate_orphans
context_object_name
paginator_classs
page_kwarg
ordering

paginate_queryset
소괄호를 감사도 튜플, 감싸지 않아도 튜플
get_context_data
if object_list is not None
self.object_list
값이 있으면 쓰고 아니면 sefl.object_list가 쓰인다.

page_size가 있다면 pagging 처리를 한다
is_paginated: False
템플릿 내에서 페이징 처리가 되어있느냐 아니냐 

```
post_list = ListView.as_view(model=Post, paginate_by=10)
```
```
{{ is_paginated }}
{{ page_obj }}
```
# 008

* pip install django-bootstrap4
* settings > bootstrap4
* load bootstrap4
* bootstrap_pagination page_obj

django-bootstrap4
bootstrap_pagination
템플릿 태그의 경우 콤마와 소괄호가 없다.

# 009

장식자

django 의 기본 장식자
django.views.decorators.http
require_http_methods, require_GET, require_POST, requrie_safe
axios -> react가 HEAD 요청을 응답을 받는다

django.contrib.auth.decorators
user_passes_test
login_required
permission_required

django.contrib.admin.views.decorators
staff member requjjirements

from django.contrib.auth.mixins import LoginRequiredMixin
을 다중상속하여 view를 만들수있다.
class의 경우에는
@method_decorator()
를 사용한다.

멤버함수에다가 method decorator를 사용하여
class 자체에 직접 적용을 하는 방법도 있어요
name = 을 줘야한다.


```
@method_decorator(login_required, name='dispatch')
class PostListView(ListView):
    model = Post
    paginate_by = 10

post_list = PostListView.as_view()
```

```
class PostListView(ListView):
    model = Post
    paginate_by = 10

    @method_decorator(login_required)
    def dispatch(self, *args, **kwargs):
        pass

post_list = PostListView.as_view()
```

```
class PostListView(LoginRequiredMixin, ListView):
    model = Post
    paginate_by = 10
```

# 010

Generic date Views
ArchiveIndexView
YearArchiveView
MonthArchiveView
WeekArchiveView
Day + ArchiveView
공통 옵션 : allow_future = False

```
In [4]: for post in post_list:
   ...:     year = random.choice(range(1990, 2020))
   ...:     month = random.choice(range(1, 13))
   ...:     post.created_at = post.created_at.replace(year=year, month=month)
   ...:     post.save()
   ...: 

In [5]: Post.objects.all().values_list('created_at')
Out[5]: <QuerySet [(datetime.datetime(2016, 10, 2, 19, 50, 24, 305024, tzinfo=<UTC>),), (datetime.datetime(1993, 12, 2, 19, 50, 24, 300968, tzinfo=<UTC>),), (datetime.datetime(1997, 6, 2, 19, 50, 24, 296903, tzinfo=<UTC>),), (datetime.datetime(1995, 4, 2, 19, 50, 24, 292909, tzinfo=<UTC>),), (datetime.datetime(2013, 12, 2, 19, 50, 24, 288961, tzinfo=<UTC>),), (datetime.datetime(2001, 9, 2, 19, 50, 24, 284869, tzinfo=<UTC>),), (datetime.datetime(2005, 3, 2, 19, 50, 24, 280678, tzinfo=<UTC>),), (datetime.datetime(2006, 7, 2, 19, 50, 24, 276678, tzinfo=<UTC>),), (datetime.datetime(2001, 1, 2, 19, 50, 24, 272939, tzinfo=<UTC>),), (datetime.datetime(2007, 12, 2, 19, 50, 24, 269207, tzinfo=<UTC>),), (datetime.datetime(2009, 3, 2, 19, 50, 24, 265378, tzinfo=<UTC>),), (datetime.datetime(2002, 2, 2, 19, 50, 24, 261748, tzinfo=<UTC>),), (datetime.datetime(1993, 4, 2, 19, 50, 24, 257989, tzinfo=<UTC>),), (datetime.datetime(1997, 5, 2, 19, 50, 24, 254297, tzinfo=<UTC>),), (datetime.datetime(2009, 4, 2, 19, 50, 24, 250508, tzinfo=<UTC>),), (datetime.datetime(2003, 4, 2, 19, 50, 24, 246341, tzinfo=<UTC>),), (datetime.datetime(1996, 10, 2, 19, 50, 24, 242252, tzinfo=<UTC>),), (datetime.datetime(1991, 5, 2, 19, 50, 24, 238024, tzinfo=<UTC>),), (datetime.datetime(1997, 6, 2, 19, 50, 24, 234200, tzinfo=<UTC>),), (datetime.datetime(1993, 4, 2, 19, 50, 24, 230394, tzinfo=<UTC>),), '...(remaining elements truncated)...']>
```

```
In [7]: Post.objects.all().values_list('created_at__year', flat=True)
Out[7]: <QuerySet [2016, 1993, 1997, 1995, 2013, 2001, 2005, 2006, 2001, 2007, 2009, 2002, 1993, 1997, 2009, 2003, 1996, 1991, 1997, 1993, '...(remaining elements truncated)...']>
```
```
{% for date in date_list %}
    {{ date|date:"" }}

{% endfor %}
```
필터에서 제공되는 함수


```
class PostDetailView(DetailView):
    model = Post
    pk_url_kwarg = 'id'

# post_detail = PostDetailView.as_view()
# class PostDetailView(DetailView):
#     model = Post
#
#     def get_queryset(self):
#         qs = super().get_queryset()
#         if self.request.user.is_authenticated:
#           qs = qs.filter(is_public=True)
#         qs = qs.filter()
#         return qs

post_detail = PostDetailView.as_view()

post_archive = ArchiveIndexView.as_view(model=Post, date_field='created_at', paginate_by=10)
post_archive_year = YearArchiveView.as_view(model=Post, date_field='created_at', make_object_list=True)
post_archive_year_month_day = DayArchiveView.as_view(model=Post, date_field='created_at')

```
'%m'
_archive_day.html

TodayArchiveView

# 011

적절한 상태코드로 응답하기
return HttpResponse(status=201)
status_code
200 성공
300 요청을 마치기 위해 추가 조치
400 클라이언트측 오류
500 서버측 오류 발생
