django의 세팅은

from django.conf import global_settings 와

from **\<project\>** import settings 가 합쳐져야 한다.
따라서 from django.conf import settings

MEDIA_URL 은 URL로 접근할때
MEDIA_ROOT 는 실제 저장하는 곳

from django.conf.urls.static import static 을 하면
\* 근데 static은 debug 모드에서 활성화 안됌(빈리스트)

url을 보내도 장고의 보안기능으로 esacpe 처리가 되어있다.

```bash
❯ python manage.py shell
Python 3.9.1 (default, Jan 28 2021, 01:00:29) 
[GCC 7.5.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from instagram.models import Post
>>> Post.objects.get(pk=3)
<Post: 3333333333>
>>> Post.objects.get(pk=3).photo
<ImageFieldFile: cloud.jpeg>
>>> Post.objects.get(pk=3).photo.url
'/media/cloud.jpeg'
>>> Post.objects.get(pk=3).photo.path
'/home/swook/Desktop/web/askcompany/media/cloud.jpeg'
>>>
```
debug 일때 media 서빙 미지원
```
urlpattrerns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT) 
```
등으로 추가하기

```
어디에 잠시 저장할 건가
settings.FILE_UPLOAD_MAX_MEMORY_SIZE
```

### 05
1. 기본파이썬 쉘
2. IPython
3. Jupyter Notebook
4. BPython

-i 쉘지정

-c 코드지정

파이프로 넘겨줄 수 도 있다. (echo ""  | python)

manage.py에 보면 os.environ.setdefault('DJANGO_SETTINGS_MODULE' ...) 로 설정하고 나면

django.setup() 로 셋업하고,

Post.objects.all() 로 정상 작동가능

jupyter 에서 쓰려면

```python
import os
os.environ['DJANGO_SETTINGS_MODULE'] = 'askcompany.settings'
os.environ["DJANGO_ALLOW_ASYNC_UNSAFE"] = "true"

import django
django.setup()
```

os.path(저수준)
pathlib(고수준)
# https://python.flowdas.com/library/pathlib.html

# 006

`<ModelClass>.objecst.all()`

create
정렬으로 지정하는 것이 좋다
Default 정렬로 지정할 수 있다.
기본 LAZY로 동작한다. 실제 데이터가 필요할때 동작한다.
filter < - > exclude
1개 획득을 시도한다면 first나 last를 쓰도록하자. 인덱스를 쓰다가 인덱스 에러가 날 수 있다. get을 쓰면 없다면 DoesNotExist 에러 / MultipleObjectsReturned
get__lte=2 lessthanequal,graterthan, greterthanequal
lte, gt, gte

qs.first()
qs.last()
qs.none()

# 008
django-extensions
python manage.py 
패키지는 변수명과 마찬가지로 하이픈이 못와서 언더바로 쓴다.
```
 python manage.py shell_plus --ipython --print-sql
# Shell Plus Model Imports
from instagram.models import Post
from django.contrib.admin.models import LogEntry
from django.contrib.auth.models import Group, Permission, User
from django.contrib.contenttypes.models import ContentType
from django.contrib.sessions.models import Session
# Shell Plus Django Imports
from django.core.cache import cache
from django.conf import settings
from django.contrib.auth import get_user_model
from django.db import transaction
from django.db.models import Avg, Case, Count, F, Max, Min, Prefetch, Q, Sum, When
from django.utils import timezone
from django.urls import reverse
from django.db.models import Exists, OuterRef, Subquery
Python 3.9.1 (default, Jan 28 2021, 01:00:29) 
Type 'copyright', 'credits' or 'license' for more information
IPython 7.19.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: from instagram.models import Post

In [2]: Post.objects.all()
Out[2]: SELECT "instagram_post"."id",
       "instagram_post"."message",
       "instagram_post"."created_at",
       "instagram_post"."updated_at",
       "instagram_post"."is_public",
       "instagram_post"."photo"
  FROM "instagram_post"
 LIMIT 21

Execution time: 0.000802s [Database: default]
<QuerySet [<Post: 11111111111>, <Post: 222222222>, <Post: 3333333333>]>

In [3]: 
```

```
class Meta:
    ordering = ['-id']
```

음수의 슬라이싱을 지원하지 않는다
주위: step 을 사용하게 되면... LAZY가 적용되지 않는다.
```
In [5]: Post.objects.all()[1:3]
Out[5]: SELECT "instagram_post"."id",
       "instagram_post"."message",
       "instagram_post"."created_at",
       "instagram_post"."updated_at",
       "instagram_post"."is_public",
       "instagram_post"."photo"
  FROM "instagram_post"
 ORDER BY "instagram_post"."id" DESC
 LIMIT 2
OFFSET 1

Execution time: 0.000232s [Database: default]
<QuerySet [<Post: 222222222>, <Post: 11111111111>]>

In [6]: Post.objects.all()[1:3:1]
SELECT "instagram_post"."id",
       "instagram_post"."message",
       "instagram_post"."created_at",
       "instagram_post"."updated_at",
       "instagram_post"."is_public",
       "instagram_post"."photo"
  FROM "instagram_post"
 ORDER BY "instagram_post"."id" DESC
 LIMIT 2
OFFSET 1

Execution time: 0.000133s [Database: default]
Out[6]: [<Post: 222222222>, <Post: 11111111111>]

In [7]: Post.objects.all()[1:3:2]
SELECT "instagram_post"."id",
       "instagram_post"."message",
       "instagram_post"."created_at",
       "instagram_post"."updated_at",
       "instagram_post"."is_public",
       "instagram_post"."photo"
  FROM "instagram_post"
 ORDER BY "instagram_post"."id" DESC
 LIMIT 2
OFFSET 1

Execution time: 0.000176s [Database: default]
Out[7]: [<Post: 222222222>]

```

# 009
django-debug-toolbar
-다양한 패널중에 SQLPanel
(html 응답만 가능함. 따라서 ajax 요청에 대한 지원은 불가능)

필히 body 태그가 있어야한다.
body 태그가 있어야만 나온다. (주입부분이 body라서. INSERT_BEFORE)

-> 메모리에 누적
개발 서버 재시작( 자동 재시작, 파이썬 파일이 변경되면 초기화)

settings.DEBUG = True 시에만 쿼리 실행내역을 메모리에 누적 장고 기본지원
쿼리 수동초기화 가능
django-querycount 를 이용하여 개발서버 콘솔에 표준출력 가능. ajax도 출력가능

# 010
ORM 이 DB에 대해 모든것을 처리하지 않는다
admin의 경우 auth에 대해 usermodel이 있다
custom user model - 있..다..

User model은 변경 될 수 있다.
```
# AUTH_USER_MODEL = 'instagram.User'
# AUTH_USER_MODEL = 'auth.User' -> default
```

settings.AUTH_USER_MODEL

# 011

OneToOneField 
1. reverse_name 이 set이 빠짐
2. 대신 없으면 오류남 (first)

시그널...을 이용해서 호출되서 함수 지정 -> Profile이 User생성이후에 항상 생성
DoesNotExist 에러 방지..

# 012

스크립트 - 모두 코드영역
M:N M(활용) N(피용) M에다가 하는걸 선호 (개별로)
DB Brower for SQLite
python unpack 문법
RDBMS지만 DB따라 NoSQL 기능도 지원
하나의 Post 안에 다수의 댓글 저장 가능
django.contrib.postgres.fields.JSONField

# 013

마이그레이션을 할때 앱이름을 지정하라 (명확한 의도, 속도가 빠르다)
DBA나 장고이전의 DB를 사용하면 migration을 사용할 수 없다
처음에 만 migrate를 기본앱 없이 지정해서 사용한다.
createsuperuser로 계정을 생성했다.
python manage.py showmigrations <앱이름>
makemigrations > sqlmigrate > migrate
만들고, 확인하고, 변경하고

migrations 파일
테이블에 데이터를 부을때 별도의 명령을 통해서 할 수 있다. > migrations
데이터를 바로 부을 수 있다.
대게 모델로부터 자동생성
DB는 수시로 백업을 해줘야 한다.
예전 DB마다 다 다름 -> 새로운 테이블을 만들어서 데이터를 카피 하는 식으로 할 수 있다.
적용된 마이그레이션 파일은 절대 삭제하시면 안됩니다.
squashmigrations 로 압축가능
showmigrations 로 어디까지 적용되었는지 확인가능
마이그레이션을 생략하면 현재 버전까지 순차적으로 적용한다

마이그렛션의 이름을 지정할때, 가지고 있는 문자열로 단독 마이그레이션의 파일을 찾을 수 있을때 사용가능
최초의 상황을 zero라고 부른다.
python manage.py showmigrations <>
python manage.py migrate <> zero
모두 해제
적용사항이 되면 절때 삭제하시면 안됩니다.
다른 ORM 에서도 id를 Primary_key로 사용한다(주소개념)
다른 필드를 기본키로 하려면 primaray_key = True 지정하기

새로운 필드에 대해 필수 필드. (디폴트, blank && null = False)
makemigrations 에 대해서 기존 Record에 어떤 값을 채워넣을지 묻는다. (1. 입력. 2. 중단)
개발 DB도 실제 Production 과 비슷한 DB를 사용하라.
Docker Composed를 활용하라. MYSQL, POSTGRESQL 등등
필드에 대해서 막 옴

## 협업 Tips
절대 하지 말아야할일 - 팀원 각자가 마이그레이션 파일을 생성한다
미적용 마이그레이션 _> 롤백 _> 마이그레이셔 제거 _> 새로운 마이그레이션 파일 생성
2) squashmigrations 활용하기
   

