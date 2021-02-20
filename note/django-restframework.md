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

