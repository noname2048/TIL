## 002

###### django-rest-framework를 쓰는 이유

* 시리얼라이져!

* 파서!

* API 관리 뷰 지원!

* 렌더러로 응답포맷 다양화!

* 인증과 권한을 위한 JWT 지원!

* 쓰롤링 지원! (유저에 따라 100회 지원등)

###### django-rest-framework 적용 순서

* 모델 만들기 
* 시리얼라이저 등록
* url router 등록

###### HTTP를 테스트하는 방법

* GUI
  postman

* CLI
  cURL, HTTPie
* 라이브러리 형태
  requests, 셀레니움

###### HTTPie와 httpbin 사이트를 이용한 예제 

```
❯ http --json GET name==value
❯ http --json POST name=value
```

Get Value 사용

```
❯ http GET httpbin.org/get x=1 y=2
HTTP/1.1 200 OK
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Length: 389
Content-Type: application/json
Date: Fri, 19 Feb 2021 01:48:51 GMT
Server: gunicorn/19.9.0

{
    "args": {},
    "headers": {
        "Accept": "application/json, */*;q=0.5",
        "Accept-Encoding": "gzip, deflate",
        "Content-Length": "20",
        "Content-Type": "application/json",
        "Host": "httpbin.org",
        "User-Agent": "HTTPie/2.4.0",
        "X-Amzn-Trace-Id": "Root=1-602f1903-1dd826881a41af6253da579c"
    },
    "origin": "39.115.28.12",
    "url": "http://httpbin.org/get"
}

```

Post value 사용

```
❯ http --form POST httpbin.org/post x=1 y=2
HTTP/1.1 200 OK
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Length: 491
Content-Type: application/json
Date: Fri, 19 Feb 2021 02:02:39 GMT
Server: gunicorn/19.9.0

{
    "args": {},
    "data": "",
    "files": {},
    "form": {
        "x": "1",
        "y": "2"
    },
    "headers": {
        "Accept": "*/*",
        "Accept-Encoding": "gzip, deflate",
        "Content-Length": "7",
        "Content-Type": "application/x-www-form-urlencoded; charset=utf-8",
        "Host": "httpbin.org",
        "User-Agent": "HTTPie/2.4.0",
        "X-Amzn-Trace-Id": "Root=1-602f1c3f-4bf505654b083e8a4822309d"
    },
    "json": null,
    "origin": "39.115.28.12",
    "url": "http://httpbin.org/post"
}

```

## 003

###### 통신을 위해 데이터는 바이트, 혹은 문자열 형식로 표현되어야한다.

* 직렬화가 대두된 이유!
* 직렬화에 쓰이는 파일 형식
  JSON, XML, Pickle(파이썬전용)

직접 직렬화 해보기

```
data = [
	{'id': post.id, 'title': post.title}, for post in Post.objects.all()
]
```

내장기능을 이용해 직렬화 해보기

```
json.dumps('한글', ensure_ascii=False).encode('utf8')

from django.core.serializer import DjangoJSONEncoder

class MyJSONEncoder(DjangoJSONEncoder):
	def default(self, obj):
		if isinstance(obj, QuerySet):
			return tuple(obj)
		elif issinstance(obj, Post):
			return {'id': obj.id, 'title': obj.title, 'content': obj.content}
		elif hasattr(obj, 'as_dict'):
			return obj.as_dict()
		
		return super().default(obj)
```

ensure_ascii 옵션을 False로 줄때 16진수로 표시되는 한글 예제 (True로 줘도 읽기 힘들다)

```
data = Post.objects.all()
json.dumps(data, cls=MyJSONEncoder, ensure_ascii=False)
```

json.JSONEncoder를 상속하여 구현한 reset_framework의 JSONRender

```
rest_framework,renderer.JSONRender
rest_framework.renderer.JSONRenderer
# json.dumps에 대한 래핑 클래스

JSONRenderer().render(data) -> JSONRender를 사용하여, 모델에 대한 직렬화 지원안됌
ModelSerializer 를 이용할것
```

###### 이러한 방식의 한계점

* django에서 지원하는 Model이나 Form에 대한 직렬화가 힘들다
* ModelSerializer 를 이용하여 해결한다

Serializer 예제

```
In [11]: from instagram.serializers import PostSerializer
In [12]: from instagram.models import Post
In [13]: serializer = PostSerializer(Post.objects.first())

In [14]: serializer.data
Out[14]: {'id': 1, 'message': '안녕하세요, 첫번째 포스팅.', 'created_at': '2021-02-19T01:34:13.065323Z', 'updated_at': '2021-02-19T01:34:13.065365Z'}

In [15]: serializer = PostSerializer(Post.objects.all(), many=True)

In [16]: serializer.data
Out[16]: [OrderedDict([('id', 1), ('message', '안녕하세요, 첫번째 포스팅.'), ('created_at', '2021-02-19T01:34:13.065323Z'), ('updated_at', T01:34:13.065365Z')]), OrderedDict([('id', 2), ('message', '으아악'), ('created_at', '2021-02-19T01:42:17.928105Z'), ('updated_at', '2021-09T01:42:17.928155Z')]), OrderedDict([('id', 3), ('message', '으아악2'), ('created_at', '2021-02-19T02:18:04.820296Z'), ('updated_at', '2021-19T02:18:04.820351Z')]), OrderedDict([('id', 4), ('message', '으아악2'), ('created_at', '2021-02-19T02:18:42.588459Z'), ('updated_at', '2002-19T02:18:42.588507Z')])]

```

qs(퀘리셋)을 ModelSerializer 로 변환할때는 many=True를 꼭 넣어야한다.

```
json.dumps(serializer.data, ensure_ascii=False)

from rest_framework.rendererss import JSONRenderer
json_utf8_string = JSONRenderer().render(serializer.data)
```

###### 실제 시리얼라이저는 django의 복잡한 관계에 대해서도 구현이 되어있음!

* rest_framework 만세!

###### Json을 HTTP를 넘기는 방법

* json.dumps를 이용해 HttpResponse로 응답

* 위의 방식을 내부적으로 구현한 JsonResponse 를 이용

###### 조회목적으로 Serializer를 사용해보자

```
from rest_framework.response import Response

qs = Post.objects.all()
serializer = PostModelserializer(qs, many=True)
response = Response(seralizer.data)
```

###### ListAPIView에서 시리얼라이즈 써보기 (시리얼라이즈 활용하기)

```
from rest_framework import generics

class PostListAPIView(generics.ListAPIView):
	queryset = Post.objects.all()
	serializer_class = PostModelSerializer
	
post_list = PostListAPIView.as_view()
```

* 참고로, rest_framework는 크게 두가지 기능에 대해 각각 두가지, 세가지 메소드를 지원한다.
  List(list, create), Detail(C, U, D), Viewset(list, CRUD - 5가지 기능 모두 지원)

###### 시리얼라이즈에서 관계에 대한 정보를 가져오는 방법 2가지 (post의 저자 가져오기)

* ReadOnlyField 만들기
* 또는, 새로운 시리얼라이즈 만들기

```
from rest_framework import serializers
from .models import Post
from django.contrib.auth import get_user_model


class AuthorSerializer(serializers.ModelSerializer):
    class Meta:
        model = get_user_model()
        fields = ["username", "email", "first_name"]


class PostSerializer(serializers.ModelSerializer):
    username = serializers.ReadOnlyField(source="author.username")
    author_email = serializers.ReadOnlyField(source="author.email")

    author = AuthorSerializer()

    class Meta:
        model = Post
        fields = [
            "pk",
            "author",
            "username",
            "author_email",
            "message",
            "created_at",
            "updated_at",
        ]

```

## 004

###### 조회방식 두가지

* 한개만 조회하기
* 여러개 조회하기
  Many=True 옵션을 꼭 줘야한다.

```ipython
In [5]: from instagram.serializers import PostSerializer

In [6]: from instagram.models import Post

In [8]: PostSerializer(Post.objects.first()).data
Out[8]: {'pk': 1, 'author': OrderedDict([('username', 'noname'), ('email', ''), ('first_name', '')]), 'username': 'noname', 'author_email': '', 'message': '안녕하세요, 첫번째 포스팅.', 'created_at': '2021-02-19T01:34:13.065323Z', 'updated_at': '2021-02-19T01:34:13.065365Z'}

In [9]: PostSerializer(Post.objects.all(), many=True).data
Out[9]: [OrderedDict([('pk', 1), ('author', OrderedDict([('username', 'noname'), ('email', ''), ('first_name', '')])), ('username', 'noname'), ('author_email', ''), ('message', '안녕하세요, 첫번째 포스팅.'), ('created_at', '2021-02-19T01:34:13.065323Z'), ('updated_at', '2021-023.065365Z')]), OrderedDict([('pk', 2), ('author', OrderedDict([('username', 'noname'), ('email', ''), ('first_name', '')])), ('username', 'noname'), ('author_email', ''), ('message', '으아악'), ('created_at', '2021-02-19T01:42:17.928105Z'), ('updated_at', '2021-02-19T01:42:17.955Z')]), OrderedDict([('pk', 3), ('author', OrderedDict([('username', 'noname'), ('email', ''), ('first_name', '')])), ('username', 'noname'), ('author_email', ''), ('message', '으아악2'), ('created_at', '2021-02-19T02:18:04.820296Z'), ('updated_at', '2021-02-19T02:18:04.820351]), OrderedDict([('pk', 4), ('author', OrderedDict([('username', 'noname'), ('email', ''), ('first_name', '')])), ('username', 'noname'), ('author_email', ''), ('message', '으아악2'), ('created_at', '2021-02-19T02:18:42.588459Z'), ('updated_at', '2021-02-19T02:18:42.588507Z')]), OrderedDict([('pk', 5), ('author', OrderedDict([('username', 'noname'), ('email', ''), ('first_name', '')])), ('username', 'noname'), ('author_email', ''), ('message', '비행기 타고가요~'), ('created_at', '2021-02-19T11:25:03.737147Z'), ('updated_at', '2021-02-19T11:25:03.7371]
```

Form 처럼 View 에서 데이터 받아오기가 가능

###### APIView에 대해 알아보자

* 모체가 된다.
* GeneralVPIView를 구현한다.

* 생각보다 구현된 덩치가 크지 않다고 한다. => 시간날때마다 읽어보기

* 데코레이터 뷰는 동적으로 APIView를 이용하여 로직을 처리한다 (상속한다)

* render
  JSON과 TEMPLATE_HTML
* parser
  JSON, FORM, MULTIPART
* authentication
  세션, 베이직
* throttle
* permission
  유저를 식별하고 나서의 리소스 접근레벨의 정의 - AllowAny
* content_negotiation
  DefaultContentNegotiation
* metadata
  SimpleMetadata
* versinoing

* 이 모든걸 커스텀 가능!



세션인증에 관해서는 프론트엔드단을 구현하면 세션 인증은 힘들 수 있다.



###### DRF의 기본개념

* APIView  - 클래스 기반뷰 (django는 View 상속, rest_framework는 APIView 상속)

*  @api_view 함수 기반 뷰를 위한 장식자

###### APIView => Generic => Viewsets

* Generic 까지는 1개의 URL을, Viewsets는 여러개의 URL을 처리한다.

###### VPIView 내의 dispatch 를 살펴보면 메소드를 찾는 방법이 나와있다.

* http_method_names 를 확인한뒤에 dispatch로 구현 확인

generics.ListAPIView
generics.CreateAPIView
generics.ListCreateAPIView
generics.DestoryAPIView

###### 장식자 사용 두가지 방법

* @를 통해
* 함수 호출

###### DRF의 두가지 구현 (함수, 클래스)

* @api_view, ["GET", "POST"]

* APIView 상속

###### Readonly를 적절히 사용하자

* AuthorSerializer 잘못하면 모두 정보를입력해야할 수 도 있따.

## 05 DRF에서 지원하는 Mixins

###### 대표적인 믹스인

* CreateModel-Mixin(이하 믹스인 생략)
* ListModel
* Retrieve
* Update
* Destroy

###### 기능에 따른 믹스인 분류

* get/post

* get/post,put,patch/delete

###### DRF 에서의 form.is_valid 처리 방식

* Django는 if
* DRF 는 raise

###### method 별 로직 연결 예제

```
class PostListAPIView(mixins.ListModelMixin, mixins.CreateModelMixin, generics.GenericAPIView):
	
	queryset = Post.objects.all()
	serializer_class = PostSerializer
	
	def get(self, request, *args, **kwargs):
		return self.list(request, *args, **kwargs)
		
	def post(self, request, *args, **kwargs):
		return self.create(request, *args, **kwargs)
		
```

Detail = Retrieve, Update, Destroy, GenericAPI

###### 위 의 내용을 재사용성을 높이자면

* APIView => Mixin => Gererics => Viewsets

Generics 는 종류가 정말정말 많다

ViewSet 도 상속해서 직접구현하거나, 상속해서 끝내는 2가지 종류가 존재

* ModelViewSet
* viewsets.ModelViewSet

URL Patterns 에 매핑하기

* ```
  post_list = PostViewSet.as_view({"get": "list"})
  post_detail = PostViewSet.as_view({"get": "retrieve"})
  ```

* 라우터를 사용하면 포멧형식도 지원한다! P?<format> => 라우터를 쓰는 이유1

* 라우터를 사용하면 또 무엇을 지원하는지에 대한 것도 알려준다 => 라우터를 쓰는이유 2

새로운 EndPoint 추가하기

```
@action(detail=False, methods=['GET'])
def public(self, request):
	qs = self.queryset.filter(is_public=True)
	...
	
@action(detail=True, methods=["PATCH"])
def set_public(self, request, pk):
	instance = self.get_object()
	instance.is_public = True
	...
```

여기서 detail은 url 두개중 어떤것을 탈것이냐를 묻는다 (list / create, update, delete, patch)



self.queryset 보다 self.get_queryset을 써라?
self.get_serializer(qs, many=True)

######  해당 뷰셋은 list + CRUD 에 추가적으로 2개의 방법을 더 지원한다.

* ```
  class PostViewSet(ModelViewSet):
      queryset = Post.objects.all()
      serializer_class = PostSerializer
  
      # def dispatch(self, request, *args, **kwargs):
      #     logger.info(
      #         f"request.body {request.body},\n" f"request.POST :{request.POST},\n"
      #     )
      #     return super().dispatch(request, *args, **kwargs)
  
      @action(detail=False, methods=["GET"])
      def public(self, request):
          qs = self.get_queryset().filter(is_public=True)
          serializer = self.get_serializer(qs, many=True)
          return Response(serializer.data)
  
      @action(detail=True, methods=["PATCH"])
      def set_public(self, request, pk):
          instance = self.get_object()
          instance.is_public = True
          instance.save(update_fields=["is_public"])
          serializer = self.get_serializer(instance, many=False)
          return Response(serializer.data)
  ```

* ```
  ❯ http --json PATCH http://localhost:8000/posts/4/ message="응? 세번ㅉ"
  HTTP/1.1 200 OK
  Allow: GET, PUT, PATCH, DELETE, HEAD, OPTIONS
  Content-Length: 177
  Content-Type: application/json
  Date: Sun, 21 Feb 2021 01:04:40 GMT
  Referrer-Policy: same-origin
  Server: WSGIServer/0.2 CPython/3.9.1
  Vary: Accept, Cookie
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  
  {
      "author_email": "",
      "created_at": "2021-02-19T02:18:42.588459Z",
      "is_public": false,
      "message": "응? 세번ㅉ",
      "pk": 4,
      "updated_at": "2021-02-21T01:04:40.226062Z",
      "username": "noname"
  }
  ```

## 007 Renderer 를 통한 다양한 응답포맷 지원

같은 Endpoint에서 요청받은 타입에 맞춰 다양한 응답포맷을 지원

Content-Type, URL 방법을 통해 Renderer 지정가능

###### Content-Type

* JSONRenderer (디폴트)
* BrowsableAPIRenderer (디폴트)
* TemplateHTMLRenderer: 지정 템플릿을 통한 렌더링

###### TemplateHTMLRenderer

* 템플릿을 통해 Renderer를 수행하기 때문에, 별도의 Serializer가 불필요하다

###### StaticHTMLRenderer

* 강사님도 용도는 아직 떠오르지 않는다

###### 써드파티 Renderer

* drf-renderer-xlsx
* djangorestframework-yaml
* djangorestframework-xml
* djangorestframework-jsonp
* djangorestframework-msgpack
* djangorestframework-csv
* rest-freamework-latex

###### 렌더러 지정방법

* 전역
  settings - REST_FRAMEWORK - DEFAULT_RENDERER_CLASSES
* APIView 클래스 마다
* @api_view 마다

rest_framework.response.Response

* api, json 지원

응답 포맷

* accept 헤드
* .api .json
* format?

###### format_suffix_patterns 지정가능

* 그렇지 않을때 @api_view에 대해 작성할때는 format? 에 대한 지원을 작성할것

###### HTTPie 를 통한 format 지정

- Accept:application/json

## 008 form과 serializer 에 대한 비교

###### 공통점

* 데이터변환/직렬화
* 유효성 검사

차이점

* Form/ModelForm
  * 주로 HTML form 에 대해
  * 장고 어드민에서
  * CBV 단일뷰

* Serializer / ModelSerializer
  * 주로 json포맷에 대한 유효성
  * List/Create 및 특정 Record에 대한 Retrieve/Edit/Delete 등에서 활용
  * API 로 단일
  * ViewSet 을 2개 뷰

request.META.get(HTTP_USER_AGENT)

###### 유효성검사에서의 차이

invalid_form(form), raise_except=True

clean\_, validate\_

## 009 serializer 를 통한 유효성 검사

###### Serializer 의 생성자

* instance

* data

###### data가 주어지면

* is_valid 가 호출되고 나서야 
  * initial_data 필드에 접근가능
  * validated_data를 통해 유효성 검증에 통과한 값이 ..save() 사용
  * .errorss 유효성 검증 수행 후에 오류 내역을 보여줌
  * .data는 갱신된 인스턴스에 대한 필드값

###### serializer.save(**kwargs) 호출을 할때

* DB에 저장한 관련 instance를 리턴

* .validated_data와 kwargs사전 을 합친 데이터를 업데이트하려고 시도

* form의 호출

  ```
  form = PostForm(request.POST)
  if form.is_valid():
  	post = form.save(commit=False)
  	post.author = requset.user
  	post.ip = request.META['REMOTE_ADDR']
  	post.save()
  ```

* serailizer의 호출

  ```
  if serializer.is_valid():
  	serializer.save(author=request.user, ip=request.META["REMOTE_ADDR"])
  ```

.update() .create()

###### 인스턴스를 지정하지 않았을때는 create 호출!

###### rest_framework.validators 로직들

* Uniquevalidator 지정1개가 지정 QuerySet 범우에서 유일성 여부 체크
* UniqueTogetherValidator
* UniqueForDate, Month, Year ...

###### UniqueValidator

* 모델필ㄷ에 unique=True 라고 한다면 자동 지정
* queryset(필수!)
* message
* lookup

UniqueForDateValidator - 어느 기간내에 어떤 필드가 유니크 하다!

* queryset (필수)
* field (필수)
* date_field (필수)
* message (필수)

rest_framework.exceptionsValidationError 사용 - django 기본과 이름은 같지만 실상은 다르다



serializer 에서의 유효성 검사

* validators 지정하거나 (validator_title)
* Meta.validators 지정
* (clean := valitdate)

perform_ 계열 함수

###### 실제 모델 구현은 perform_create, perfom_update, perform_destroy를 통해 이루어집니다.

* CreateModelMixins

ip라는 필드를 추가했다고 했을때





###### Model 의 editable=False 를 통해 보여주기만 하고 수정은 컨트롤 내부에서만 진행 할 수 있다.

하하

## 010 Authentication 과 permision

* Authentication 유저 식별

* Permissions 각 요청에 대한 허용/거부

* Throttling 일정기간 동안에 요청 가능한 수



###### 인증처리 순서

1. 매 요청 시마다 APIView 의 dispatch 호출
2. APIView 의 initail 호출
3. APIView의 perform_authentication 호출
4. request 의 user의 속성 호출 (rest_framework.request.Request 타입)
5. request의 _authenticate() 호출

###### 매번 토큰을 검사한다.



###### 지원하는 인증의 종류

* sessionAuthentication
  세션을 통한 인증

* BasicAuthentication
  basic 인증 헤더를 통한 인증
  `https --auth username:password --form POST`

  > 반드시 basic 인증시에는 https 를 사용하세요

* TokenAuthentication
  Token: 랜덤에 가까운 무언가를 작성한다 (expire 가 없다, 유출되면 큰일난다)
* RemoteUserAuthentication



rest_framework는 django를 통해 구현이 되어있음 (url만 다른방향으로 도어있다)

###### DRF의 permission 시스템, APIview 단위로 지정

* `from rest_framework.permissions`

* AllowAny
  적합하지 않다.
* IsAuthenticated
* IsAdminuser
  is_staff 체크
* IsAuthenticatedOrReadOnly
  디폴트로 생각해보자
* DjangoModelPermissions
  인증된 요청에 한해 뷰호출 허가, 추가로 장고의 모델단위 Permissions 체크
* DjangoModelPermissionsOrAnonReadOnly
  위과 유사하나, 비인증 요청에게는 읽기만 허용
* DjangoObjectPermissions
  비인증 요청은 거부, 인증된 요청은 Object에 대한 권한 체크



@api_view에는 @permission_classes 장식자 설정



###### 커스텀 Permission

* has_permissions(request, view)
  대부분의 permission 클래스에서 구현

* has_object_permission(request, view, obj)
  Form 노출시에 체크!

dry-rest-permissions 를 통해서 룰기반의 퍼미션 제공



`User.objects.create_user?`

## 011 Filtering

###### Filtering

self.kwargs (장고기본과 같다)

```
class PostViewSet(ModelViewSet):
	...
    filter_backends = [SearchFilter, OrderingFilter]
    search_fields = ["message"]  # ?search=
    ordering_fields = ["pk", "created_at"]  # ?ordering= 정렬을 허용할 필드의 화이트 스트
    ordering = ["pk"]  # 디폴트
```

문자열 패턴 지정

* ^ 시작

* = 정확히 일치

* @ 단어/구문에 대한 검색 (MySQL 만 지원)

* $ Regex Search

## 012 Pagination

* PageNumverPagination
  page/page_size
* LimitOffsetPagination
  offset/limit

###### 디폴트 지정

* ```
  REST_FRAMEWORK={
  	"PAGE_SIZE": 10,
  	"DERAULT_PAGINATION_CLASS": "rest_framework.pagination.PageNumberPagination"
  }
  ```

  rest_framework.pagination.pagination

* ```
  "PAGE_SIZE": 3,
  "DEFAULT_PAGINATION_CLASS": "rest_framework.pagination.PageNumberPagination",
  ```

## 013 Throttling (최대 호출 횟수/제한하기)

* Rate: 지정 기간 내에 허용할 최대 호출 회수

* Scope:각 Rate에 대한 별칭 (alias)

* Throttle: 특정 조건 하에 최대 호출 횟수를 결정하는 로직이 구현된 클래스

###### 기본제공 Throttle

* Anon
  인증 요청에는 제한 (X) 비인증시 IP 단위
  디폴트 scope: anon
* User
  인증 요청에는 유저 단위 / 비인증시 IP 단위
* Scoped
  인증 요청에는 유저단위 / 비인증시 IP
  APIView 마다 throttle_scope 설정 가능

```
"DEFAULT_THROTTLE_CLASSES": [],
"DEFAULT_THROTTLE_RATES": {
	"anon": None,
	"user": None,
},
```

```
"DEFAULT_THROTTLE_CLASSES": [
	"rest_framework.throttling.UserRateThrottle"
],
"DEFAULT_THROTTLE_RATES": {
	"user": "10/day",
},
```



429 Too Many Requests 응답 (Retry-After 헤더 리턴)



### Cache

장고 기본에 Cache가 있다.

* Memcached 서버
* 데이터베이스
* 파일시스템
* 로컬 메모리
* 더미
* redis를 활용한 캐시 (django-redis-cache)

디폴트 캐시: 로컬 메모리 캐시

주로 memcache 나 redis를 사용한다 => django가 리스타트 되어도 남는다

redis가 더 안전하다



캐시 성능이 중요!

cache_format => 스코프 별로 따로 한다. ident = key



SimpleRateThrottle 에서는 다음과 같이 디폴트 캐시 설정

```
from django.core.cache import cache as default_cache (단수))

class ...(AnonRateThrottle):
	cache = caches['alternate']
```

###### 다수의 캐시서버를 캐시 가능!

* from django.core.cache import caches (복수)

Rates 포맷

* 포맷 숫자/간격
* 첫글자만 사용
* timestamps list를 가져온다
* 체크 범위 밖의 timestamp 값을 버린다
* 크기가 허용범위보다 클경우 요청 거부
* 허용범위보다 작을 경우현재 timestamp를 list에 추가하고 cache에 다시 저장



###### 클라이언트 IP를 결정짓는 방법

* X-Forwarded-For 헤드와 REMOTE_ADDR 헤더를 참조해서 확정
* 일반적인 로드 벨런서가 X-Forwarded-For 활용
* Nginx 에 대해 헤더로 넘겨줄 수 있다.
* (없다면 REMOTE_ADDR 활용)



###### custom scope

```
"DEFAULT_THROTTLE_CLASSES": [],
"DEFAULT_THROTTLE_RATES": {
	"contact": 1000/day,
	"upload": 20/day,
},
```

```
class CotactRateThrottle(UserRateThrottle):
	scope = 'contact'
	
class UploadRateThrottle(UserRateThrottle):
	scope = 'upload'
	
```

```
class ContactListView(APIView):
	throttle_classes = [ContactRateThrottle]
```

###### 한번에 적용하기

```
"DEFAULT_THROTTLE_CLASSES": [
	"rest_framework.throttling.ScopedRateThrottle"
],
"DEFAULT_THROTTLE_RATES": {
	"contact": 1000/day,
	"upload": 20/day,
},
```

```
class ContactlistView(APIView):
	throttle_scope = "contact"
```

###### 유저별로 다르게 적용하기

PremiumThrottle 에 대해 구현

```
class PremiumThrottle(UserRateThrottle):
	def allow_request(self, request, view):
		premium_scope = getattr(view, "premium_scope", None)
		
		if request.user.profile.is_premium_user:
			if not premium_scope:
				return True
			self.scope = premium_scope
		else:
			if not light_scope:
				return True
			self.scope = light_scope
			
		
		self.rate = self.get_rate()
		self.num_requests, self.duration = self.parse_rate(self.rate)
		
		return super().allow_request(request, view)
```

## 014 Token 인증

DRF 가 지원하는 인증

* rest_framework.authentication.SessionAuthentication
  프론트 엔드와 장고가 같은 호스트면 세션인증 가능!(nginx)
  외부 서비스/앱에서는 세션 인증 불가

* rest_framework.authenticationi.BasicAuthentication
  보안상 못할짓

* rest_framework.authentication.TokenAuthentication
  초기에 id Token 발급받아 API 로 보내 처리

`rest_framework.authtoken.models.Token`



###### Token 생성 3가지

* ```
  token, created = Token.objects.get_or_create(user=user)
  ```

* ```
  @receiver(post_save, sender=settings.AUTH_USER_MODEL)
  def create_auth_token(sender, instance=None, created=False, **kwargs):
  	if created:
  		Token.objects.create(user=instance)
  ```

* ```
  python3 manage.py drf_create_token [-r:재성성] <username>
  ```



```
❯ http POST http://localhost:8000/posts/1/ "Authorization: Token $TOKEN"
```

```
❯ http GET  http://localhost:8000/posts/1/ "Authorization: Token $TOKEN"
```

```
import requests

TOKEN = "asdfasegasrysfhdsfg"
headers = {
	"Authorization": f"Token {TOKEN}"
}

requests.get("http://localhost:8000/posts/1/", headers=headers)
print(res.json())
```

안드로이드 IOS  API 호출 가능!

## 015 Token 인증과 JWT 인증

토큰만 통과 된다면 유저임을 인증 받을 수 있따.

DRF 의 Token 은 랜덤 문자열에 1:1 매칭, 유효기간이 없다.

###### JWT

* 내부에 id 를 가지고 있음 -> DB 조회 없이 검사 가능
* 비밀 키로 서명한다 (위변조 불가)
* 포맷: 헤더, 내용, 서명 (서버만 하고, 암호화가 아니라 모두 열어볼 수 있다)

* 유효기간이 존재
* Refresh 메커니즘을 지원한다 (expire 전에 refresh 하는 기능)
* 이미 발급된 Token을 폐기(Revoke) 하는 것은 불가능하다

JWT 가 더 복잡하다!

* Header base64 인코딩

* Payload base64 인코딩

* Signature 위의 두항목에 비밀키로 서명한뒤에 + base64 인코딩

`from base64 import b64decode`



###### 일반 Token / JWT 토큰 여부에 상관없이 Token은 안전한 장소에 보관해야한다

* 스마트폰의 경우 안드로이드/IOS 에서 안전한 저장공간을 제공한다
* 웹브라우저에는 없다...

https - > SSL (Let's Encrypt)



djangorestframework-jwt

```
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework.authentication.SessionAuthentication",
        # "rest_framework.authentication.TokenAuthentication",
        "rest_framework_jwt.authentication.JSONWebTokenAuthentication",
    ],
}

JWT_AUTH = {
    "JWT_ALLOW_REFRESH": True,
}

```

```
    path("api-jwt-auth/", obtain_jwt_token),
    path("api-jwt-auth/refresh/", refresh_jwt_token),
    path("api-jwt-auth/verify/", verify_jwt_token),
]

```



```
class RefreshJSonWebToken(JSONWebTokenAPIView):
	serializer_class = RefreshJSONWebTokenSerializer
```

```
# 정상 처리시 응답
{
	"token": token,
	"user": user
}
```



직렬화

```
b64endcode(b'{""}')
```

```
JWT_AUTH 
JWT_EXPIRATION_DELTA
JWT_ALLOW_REFRESH (디폴트=False, orig_)
```



b64decode == 등호로 채워준다 (패딩이 안맞을 경우가 있다)



###### 주요세팅

JWT+SECRET_KEY

JWT_ALGORITHM: "HS256"

JWT_EXPIRATION_DELTA

JWT_ALLOW_REFRESH: False

JWT_REFRESH_EXPIRATION_DELTA: datetime.timdedelta(days=7)

