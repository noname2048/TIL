# 001

기본에 대해 생략

# 002

CSS Layout
```css
@media (max_width: 600px) {

}
```

* Bootstrap4
* Materialize CSS
* Semantic UI
* Material UI
* UIKit
* Foundation

getbootstrap.com

CDN
Content Delivered Network

게임개발을 위한 AzureCDN

Cerulean
Cosmo

# 003

부모가 정의한 block 내에서만 가능


# 004

jQuery 출시 2006
최근에는 ㅓquery를 쓰지 않는다.
웹 페이지 -> 웹 애플리케이션의 시대
2020년 2월 1일 기준으로 React 가 jquery 앞섬
장고 form이 jquery
장고 admin이 jquery
최근 브라우저 에서는 네이트브 구현으로 제공한다

R. V. A
Dom을 직접 조작하지 않는다.
jQuery 에 대한 별칭을 $ 로 정의한다
익명 함수를 리턴하면 모두 로딩이 되면 쓴다

$(function() {
    $('#btn-naver-1').click(function(){
        console.log();
    });
    $('#wow').submit(function() {
        e.preventDefault();
        console.log("form submit");
    })
});
appendTo
var tpl = _.template(raw_tempalte)
var rendered = tpl({
    numbers: sample.slice(0,5).sort(function(x, y){ return (x - y);})
})
lodash 라이브러리

동일 도메인 정책> 같은 프로토콜/호스트명/포트 내에서만 Ajax 요청 가능
CORS 지원 (Cross Origin Resource Sharing) -> 서버측 셋업이 필요
djagno CORS응닿 어용 헤더 
django-core-headers 라는 라이브러리 활용

해당 서버 설정에 CORS 설정을 허용해야 한다.
Access-Control-Allow-Origin
dataType: 'jsonp'
GET 만 되는 script태그로 요청 가능

# 005

pytho manage.py runserver 0.0.0.0:8000

IP가 여러개일 수 있다.

ngrok 서버-중개(프록시)

# 006

Static & Media 파일
앱/프로젝트 단위로 저장 /서빙

static 파일 경로

```python
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'static'
STATICFILES_DIRS = [
    BASE_DIR / 'askcompany' / 'static',
]
```
python manage.py collectstatic

STATICFILES_FINDERS = [
    'django.contrib.staticfiles.finders.FileSystemFinder',
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
]

{% loda static %}
"{% static "blog/title.png %}

Debug = True 일떄만 static 정적 파일을 지원한다.
왜냐하면 다른 웹서버가 많이 지원한다.
Static File Finder 

1. CDN tjqltm
2. apache/nginx 웹서버 등
3. whitenoise 등을 이용하여 

nginx 리버스를 통해서 IF /static/
읽어서 넘길수있다.
```
server {
    location /static {
        autoindex off;
        alias /var/www/staticfiles;
    }
    location /media {
        autoindex off;
        alias /var/www/media;
    }
}
```

django-storages 로 Amazon S3 등 지원

브라우저캐싱
