초록

Django에 Built-in으로 작성된 ArchiveIndexView와 같이 date_field만 있으면 자동으로 Archive 문서를 작성할 수 있는 내장 클래스는 분명 Blog를 만들때 유용한 기능이 될 수 있음에도, (물론 용도에 따라 다를 것이다) 공식문서에 정리되어 있지않다(21/02/03). 그래서 초보자가 Django CBV도 공부할겸 내용을 맛보기 형태로 정리해 보았다.
본문
개괄

이글은 "리액트와 함께 장고 시작하기 Complete 강의"를 듣다가 공식 Tutorial에서는 쉽게 보기 힘든 내용이 적혀져 있어서 한번 정리해보고, 강사님의 말처럼 직접 코드를 보고 공부하는 것이 도움이 많이 된다는 말에 약점인 CBV를 강화하고자 정리하였다. (강의는 강추한다)

글을 작성하는 도중에 관련내용이 적혀있는 Wiki Docs를 찾았는데, 자세히 적혀있진 않지만 View를 공부하는 도중에 도움이 되었다. 링크: Django 자습서

장고에 능숙한 분들에게는 만드는 것이 크게 어렵진 않겠지만, 나와 같은 초심자 입장에서는 실제로 어떤 속성을 가지고 작동하는지 보니 공부가 많이 된다.
참조코드

사용하는 버전은 Django=~3.1.5 이다.

django/views/generic/__init__.py에 정리된 내용에 따르면,

from django.views.generic.base import RedirectView, TemplateView, View
from django.views.generic.dates import (
    ArchiveIndexView, DateDetailView, DayArchiveView, MonthArchiveView,
    TodayArchiveView, WeekArchiveView, YearArchiveView,
)
from django.views.generic.detail import DetailView
from django.views.generic.edit import (
    CreateView, DeleteView, FormView, UpdateView,
)
from django.views.generic.list import ListView

__all__ = [
    'View', 'TemplateView', 'RedirectView', 'ArchiveIndexView',
    'YearArchiveView', 'MonthArchiveView', 'WeekArchiveView', 'DayArchiveView',
    'TodayArchiveView', 'DateDetailView', 'DetailView', 'FormView',
    'CreateView', 'UpdateView', 'DeleteView', 'ListView', 'GenericViewError',
]


class GenericViewError(Exception):
    """A problem in a generic view."""
    pass

generic view의 종류

    Base
        View
        TemplateView
        RedirectView
    dates
        ArchiveIndexView
        DateDetailView
        DayArchiveView
        MonthArchiveView
        TodayArchiveView
        WeekArchiveView
        YearArchiveView
    detail
        DetailView
    edit
        CreateView
        DeleteView
        FormView
        UpdateView
    list
        ListView

모두 능숙하게 사용하고 싶지만 ListView와 DetailView는 많이 쓰니 다음에 다루기로 하고, 문서에서 찾기 힘들었던 기능에 대해 먼저 다루려고 한다. tistory와 같은 블로그들에서 흔히 볼수 있는 archive와 관련된 기능들을 살펴본다
Format Codes

그전에 python 3.9.1 docs를 참조하여 Format Codes 를 정리하자. 순서는 임의로 바꾸었으니 알파벳순으로 정렬된 문서를 보고 싶다면 원문을 참조하자.
Directve	Meaning	Example
%y	%02d 형식으로 작성된 년도	15, 16, 21
%Y	%04d 형식으로 작성된 년도	2015, 2016, 2021
%m	%02d 형식으로 작성된 월	07, 09
%d	%02d 형식으로 작성된 일	21, 08
%H	%02d 형식으로 작성된 시(24)	00, 01, ..., 24
%I	%02d 형식으로 작성된 시(12)	00, 01, ..., 12
%M	%02d 형식으로 작성된 분	00, 01, ..., 60
%S	%02d 형식으로 작성된 초	00, 01, ..., 60
%f	%06d 형식으로 작성된 마이크로초	000000, 000001, ..., 999999

Directve	Meaning	Example
%B	월 (locale에 따라 다르다)	January, February, ..., December
%b	월의 약어 (locale에 따라 다르다)	Jan, Feb, ..., Dec
%w	일요일로 시작하는 주의 인덱스	0, 1, ..., 6
%A	요일 (locale에 따라 다르다)	Sunday, Monday, ..., Saturday
%a	요일의 약어 (locale에 따라 다르다)	Sun, Mon, ..., Sat
%U	그 해에서의 몇번째 주인지 (일요일 기준)	00, 01, ..., 53
%W	그 해에서의 몇번째 주인지 (월요일 기준)	00, 01, ..., 53
%j	그 해에서의 몇번째 날인지	001, 002, ..., 366

Directve	Meaning	Example
%x	날짜 표현 (locale에 따라 다르다)	08/16/88 (None);
08/16/1988(en_US)
%X	시간 표현 (locale에 따라 다르다)	21:30:00 (en_US);
21:30:00 (de_DE)
%c	날짜 + 시간 (locale에 따라 다르다)	Tue Aug 16 21:30:00 1988 (en_US);
Di 16 Aug 21:30:00 1988 (de_DE)
%Z	타임존 이름	(empty), UTC, GMT
%z	타임존에서의 오프셋	(empty), +0000, -0400, +1030, +063415, -030712.345216

수많은 상속

상속이 조금 복잡하다. 잘 공부하면 다중 상속에 대해 잘 알 수 있을듯. 관계도를 통한 정리가 필요해보인다.

    YearMixin
    year_format = '%Y'
    year = None
    MonthMixin
    month_format = '%b'
    month = None
    DayMixin
    day_format = '%d'
    day = None
    WeekMixin
    week_format = %U
    week = None
    DateMixin
    date_field=None
    allow_future=False
    BaseDateListView(MultipleObjectMixin, DateMixin, View)
    allow_empty = False
    date_list_period = 'year'
    BaseArchiveIndexView(BaseDateListView)
    context_object_name = 'latest'
    ArchiveIndexView(MultipleObjectTemplateResponseMixin, BaseArchiveIndexView)
    template_name_suffix = '_archive'

사용하려는 ArchiveIndexView 까지 모체 3개를 이용하여 하나를 만들고, BaseDateListView->BaseArchvieIndexView->ArchiveIndexView까지 3번의 상속이 이루어 진다. 그 와중에, DateMixin과 TemplateResonseMixin도 끼어든다.

    View
    http_method_names = ['get', 'post', 'put', 'patch', 'delete', 'head', 'options', 'trace']
    MultipleObjectMixin(ContextMixin)
    allow_empty = True
    queryset = None
    model = None
    paginate_by = None
    paginate_orphans = 0
    context_object_name = None
    paginator_class = Paginator
    page_kwarg = 'page'
    ordering = None
    ContextMixin
    extra_context = None

View (MVC - Controlloer, MTV - View)의 역할에 집중해서 보기

상당히 복잡해 보이지만, View의 역할은 모델에서 데이터를 불러와 template 단에 context로 넘겨주는 역할을 하고 있으므로 그와 관련된 과정을 살펴보자.

(Django는 전통적으로 알려져있는 Model View Controller 디자인 패턴에서 이름만 변경된 그래서 헷갈린다. Model Template View 패턴을 사용한다.)

MultipleOjbectMixin 에서
1. queryset 확인
2. model 확인
둘다 없으면 error

DateMixin 에서
1. date_field를 확인한다.
2. _make_date_lookup_arg 함수에서 data를 Combine할 표를 date_field 기준으로 만든다

BaseDateListView 에서
1. allow_empty = False
2. dat_list_period = 'year'로 통일
3. get_dated_queryset 함수로 queryset을 호출한다.
4. context로 object_list, date_list를 넘긴다.
5. get_dated_items함수 미구현시 에러.

BaseArchiveIndexView 에서
1. context에 latest 라는 이름이 추가된다.
2. get_dated_items 함수를 내림차순으로 dated_queryset 정렬

MultipleObjectTemplateResponseMixin 에서
1. 참고할 template name을 (template_name_suffix): _archive로
정리하면

    다른 CBV와 마찬가지로 views.py에
    ArchiveIndexView.as_view(model=Post, date_field='created_at', paginated_by=10) 로 작성한다. 혹은 클래스 상속으로 구현한다.
    이때 model 이나(or) queryset이 꼭 있어야한다.
    또, date_field도 꼭 필요하다.
    기본적인 template 이름은 소문자 모델명 + _archive.html이 된다. (싫으면 suffix 이름을 바꿔주자)
    context는 objet_list, date_list, latest 가 넘어간다.
    날짜를 통한 아카이브화 (개념) 구현 완성

의문점

    %j는 366까지 표기된다. 왜?
    velog는 markdown 각주를 제공하지 않는다. 왜? -> LYNMP(beta)라는 사이트를 찾았다. 어라. (아니면 markdown extentions engine을 찾아보자)
    Archive는 좋은 기능임에도 docs에 따로 정리된 문서가 없다. 왜?
    django docs(ko)가 잘 없다. (그럼 다른 잘 정리된건 뭐가 있을까) 개발자는 영어랑 친하다. 튜토리얼만 한글이면 된다.

이글을 수정한다면

    관계도로 표현된 모델들 정리. 일러스트로 해보고 싶다.
    이해하기 쉽게 바꾸어보고 싶다. (입문(X), 초급(O))
