# URL Dispatcher

깨끗하고 우아한 URL 스키마는 고품질의 웹 애플리케이션에서 중요한 세부사항입니다. 장고는 프레임워크의 제한없이 원하는대로 URL을 디자인할 수 있게합니다.

URL이 깨끗하고 사용 가능해야하는 이유에 대한 훌륭한 논거는 World Wide Web 제작자 Tim Berners-Lee에 의해 작성된 `멋진 URL은 바뀌지 않는다`를 참조하십시오.

# Overview

앱의 URL을 디자인하려면 일상적으로 **URLconf**(URL configuration)라고 하는 파이썬 모듈을 만들게 됩니다. 이 모듈은 순수한 파이썬 코드이며 URL 경로 표현식과 피아썬 함수(당신의 views) 사이의 매핑입니다.

이 매핑은 필요한 만큼 짧거나 길 수 있고 다른 매핑을 참조할 수 있습니다. 그리고 순수한 파이썬 코드이기 때문에 동적으로 생성될 수 있습니다.

또한 장고는 언어에 따라 URL을 변환하는 방법을 제공합니다. 더많은 정보에 대해서는 `국제화 문서`을 참고하십시오.

---

# How Django processes a request

이것은 사용자가 장고 기반 사이트에서 페이지를 요청할 때, 실행될 파이썬 코드를 결정하기 위해 시스템이 따라야하는 알고리즘입니다 : 

1. 장고는 사용할 루트 URLconf 모듈을 결정합니다. 일반적으로 이것은 **`ROOT_URLCONF`** 설정의 값이지만 들어오는 **HttpRequest** 객체가 (미들웨어로 설정된) **`urlconf`** attribute를 가지고 있으면, **`ROOT_URLCONF`** 설정 대신 해당 값이 사용됩니다.
2. 장고는 파이썬 모듈을 불러와 **urlpatterns** 변수를 찾습니다. 이것은 **`django.url.path()`** 그리고/혹은 **`django.urls.re_path()`** 인스턴스의 파이썬 리스트여야 합니다.
3. 장고는 각 URL 패턴을 순서대로 실행하고 요청된 URL과 매치되는 첫 번째 위치에서 멈춥니다.
4. URL 패턴 중 하나가 매치되면 장고는 간단한 파이썬 함수(또는 `class-based view`)인 주어진 view를 import하여 호출합니다. view는 다음 arguments를 전달받습니다 : 
    - `**HttpRequest**` 인스턴스
    - 매치되는 URL 패턴이 명명된 그룹들을 반환하지 않으면, 정규표현식의 매치되는 항목이 지정된 위치의 arguments로 제공됩니다.
    - keyword arguments는 **`django.urls.path()`** 또는 **`django.urls.re_path()`** 에 대한 선택적 **kwargs** argument에 지정된 argument로 오버라이드된 경로표현식과 매치되는 모든 명명된 부분들로 구성됩니다.
5. URL 패턴이 매치되지 않거나 이 과정의 어느 시점에서 예외가 발생하면 장고는 적절한 error-handling view를 호출합니다. `Error handling` 을 참조하십시오.

---

# Example

다음은 URLconf 샘플입니다 : 

    from django.urls import path
    
    from . import views
    
    urlpatterns = [
    		path('articles/2003/', views.special_case_2003),
    		path('articles/<int:year>/', views.year_archive),
    		path('articles/<int:year>/<int:month>/', views.month_archive),
    		path('articles/<int:year>/<int:month>/<slug:slug>/', views.article_detail),
    ]

노트 : 

- URL에서 값을 캡처하려면, <>사용.
- 캡처된 값에는 선택적으로 변환형이 포함될 수 있습니다. 예를 들면 **<int:name>** 을 사용하여 정수 매개 변수를 캡처합니다. 변환형이 포함되어 있지 않으면 / 문자를 제외한 모든 문자열을 매치합니다.
- 모든 URL에 대해 첫번째 / 를 추가할 필요가 없습니다. 예를 들면 **/articles**이 아니라 **articles**입니다.

요청 예 : 

- **/articles/2005/03/** 에 대한 요청은 항목의 3번째 pattern과 매치됩니다. 장고는 **view.month_archive(request, year = 2005, month = 3 )** 함수를 호출합니다.
- **/articles/2003/** 은 2번째 패턴이 아닌 1번째 패턴과 매치되는데, 패턴이 순서대로 테스트되므로 처음 매치되는 것이 1번째 패턴이기 때문입니다. 이런 특수한 경우를 넣기 위해 이러한 형태를 자유롭게 이용해보아도 좋을 것입니다. 여기서 장고는 **views.special_case_2003(request)**를 호출합니다.
- **/articles/2003**은 위 패턴 중 어떠한 것도 매치되지 않을 것입니다. 왜냐하면 각 패턴은 URL이 / 로 끝나야 하기 때문입니다.
- **/articles/2003/03/building-a-django-site/**는 마지막 패턴과 매치됩니다. 장고는 **views.articles_detail(request, year = 2003, month = 3, slug = "building-a-django-site")** 함수를 호출합니다.

---

# Path converters

기본적으로 다음 경로 변환기를 사용할 수 있습니다 : 

- **str** - 경로 구분 기호인 **'/'**를 제외한 비어 있지 않은 문자열과 매치합니다. 이것은 변환기가 표현식에 포함되지 않은 경우의 기본값입니다.
- **int** - 0 또는 임의의 양의 정수와 매치합니다. int형의 반환값을 가집니다.
- **slug** - ASCII 문자 또는 숫자와 - 및 _ 문자로 구성된 모든 slug 문자열과 매치합니다. ( 예 : **building-your-1st-django-site** )
- **uuid** - 형식이 지정된 UUID와 매치합니다. 여러 URL이 같은 페이지에 매핑되지 않도록 하려면 -를 포함하고 문자는 소문자여야합니다. ( 예 : **075194d3-6885-417e-a8a8-6c931e272f00. `UUID`** 인스턴스를 반환합니다. )
- **path** - 경로 구분 기호인 **'/'**를 포함한 비어있지 않은 문자열과 매치합니다. **str**과 같은 URL 경로의 구획이 아닌 완전한 URL 경로와 비교할 수 있습니다.

---

# Registering custom path converters

보다 복잡한 매칭 요구 사항의 경우 당신만의 경로 변환기를 정의할 수 있습니다.

변환기는 다음을 포함하는 클래스입니다 : 

- 정규식 클래스 attribute (문자열).
- 매치되는 문자열을 view 함수로 전달되어야 하는 유형으로 변환하는 **to_python(self, value)** 메소드. 단, 주어진 값을 변환할 수 없는 경우 **ValueError**를 발생시켜야 합니다.
- 파이썬 형식을 URL에서 사용할 문자열로 변환시키는 **to_url(self, value)** 메소드.

예 : 

    class FourDigitYearConverter:
        regex = '[0-9]{4}'
    
        def to_python(self, value):
            return int(value)
    
        def to_url(self, value):
            return '%04d' % value

**`regiter_converter()`**를 사용하여 URLconf에 사용자 정의 변환기 클래스를 등록하세요 : 

     from django.urls import path, register_converter
    
    from . import converters, views
    
    register_converter(converters.FourDigitYearConverter, 'yyyy')
    
    urlpatterns = [
        path('articles/2003/', views.special_case_2003),
        path('articles/<yyyy:year>/', views.year_archive),
        ...
    ]

---

# Using regular expressions

만약 경로 및 변환기 구문으로 URL 패턴을 정의하는 데 충분하지 않으면 정규표현식을 사용할 수도 있습니다. 그렇게 하기위해선 **`path()`** 대신 **`re_path()`**를 사용하여야합니다.

파이썬 정규표현식에서 명명된 정규표현식 그룹의 구문은 **(?P<name>pattern)**입니다. 여기서 **name**은 그룹의 이름이며 **pattern**은 매치시킬 패턴입니다.

다음은 정규표현식을 사용하여 재 작성된 이전의 URLconf 예제입니다 : 

    from django.urls import path, re_path
    
    from . import views
    
    urlpatterns = [
        path('articles/2003/', views.special_case_2003),
        re_path(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
        re_path(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/$', views.month_archive),
        re_path(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/(?P<slug>[\w-]+)/$', views.article_detail),
    ]

이것은 다음을 제외하고 이전 예제와 거의 동일합니다 : 

- 매치시킬 정확한 URL이 약간 더 제한됩니다. 예를 들자면, 연도가 정확히 4자리 길이로 제한되므로 연도 10000은 더이상 매치되지 않습니다.
- 캡처된 각 argument는 정규표현식의 매치하는 유형에 관계없이 문자열로 view에 전송됩니다.

**`path()`** 에서 **`re_path()`**로 또는 그 반대로 전환할 때 view argument의 유형이 변경될 수 있으므로, view를 수정해야 할 수도 있다는 점을 알고 있어야 합니다.

## **Using unnamed regular expression groups**

명명된 그룹 구문 뿐만 아니라 ( e.g. **(?P<year>[0-9]{4})** ), 당신은 더 짧은 명명되지 않은 그룹을 사용할 수 있습니다 ( e.g. **([0-9]{4})** ).

 이 사용법은 의도하려는 매치의 의미와 view의 argument사이에 오차가 우발적으로 발생하기 쉬워지므로 특별히 권장되지는 않습니다.

두 경우 모두 주어진 정규표현식 내에서 하나의 스타일만 사용하는 것이 좋습니다. 두 스타일이 섞인 경우 명명되지 않은 그룹은 무시되며, 명명된 그룹만 view 함수에 전달됩니다.

---

## Nested arguments

정규표현식은 nested arguments를 허용하고, Django는 이를 해결하고 view로 전달합니다. 리버싱 시 Django는 캡처된 nested argument를 모두 무시하고, 캡처된 argument 이외의 모든 값을 채우려고 시도합니다. 선택적으로 페이지 argument를 가지는 다음 URLpatterns를 고려하십시오 : 

    from django.urls import re_path
    
    urlpatterns = [
        re_path(r'^blog/(page-(\d+)/)?$', blog_articles),                  # bad
        re_path(r'^comments/(?:page-(?P<page_number>\d+)/)?$', comments),  # good
    ]

두 패턴 모두 nested argument를 사용하여 해결됩니다 : 얘를 들자면, **/blog/page-2/**는 **page-2/**와 **2**의 두가지 argument가 있는 **blog_articles**와 매치됩니다. **comments**의 두번째 패턴은 **comment/page-2/**와 매치시키며 keyword argument인 **page_number**는 2로 설정됩니다. 이 경우  외부 argument는 캡처할 수 없는 argument**(?:...)**입니다.

**blog_articles**의 view는 가장 바깥쪽에서 캡처된 argument를 리버스시켜야 하며, 이 경우 **page-2/**또는 arguments를 리버스시킬 수 없고, **comments**는 argument 또는 **page_number** 값을 사용하여 리버스할 수 있습니다.

Nested captured arguments는 view argument와 **blog_articles**로 표시된 URL 사이의 강력한 커플링을 만듭니다 : view는 view에 있는 값만이 아니라 URL(**page-2/**)의 일부분도 받습니다. 이 커플링은 리버싱 시 더욱 뚜렷해집니다. 이 때 view를 리버스한 후 페이지 번호 대신 URL의 일부분을 전달해야 합니다.

경험에 비추어 볼 때, 정규표현식이 argument를 필요로 하지만 view가 이를 무시하는 경우 view와 함께 사용할 값만 캡처하고 캡처되지 않은 arguments를 사용합니다.

---

# What the URLconf searches against

URLconf는 요청된 URL을 일반 파이썬 문자열로 검색합니다. 여기에서 GET 또는 POST 매개변수 또는 도메인 이름은 포함되지 않습니다.

예를 들면, **[https://www.example.com/myapp](https://www.example.com/myapp)**에 대한 요청에서 URLconf는 **myapp/**을 찾습니다.

**[https://www.example.com/myapp/?page=3](https://www.example.com/myapp/?page=3)**에 대한 요청에서 URLconf는 **myapp/**을 찾습니다.

URLconf는 요청 메소드를 보지 않습니다. 다시 말해, 모든 요청 메소드 - **POST, GET, HEAD** 등 - 는 동일한 URL에 대해 동일한 함수를 전송합니다.

 

---

# Specifying defaults for view arguments

views의 arguments에 대한 기본 매개변수를 지정하는 것은 편리한 트릭입니다. 다음은 URLconf와 view의 예시입니다 :

    # URLconf
    from django.urls import path
    
    from . import views
    
    urlpatterns = [
        path('blog/', views.page),
        path('blog/page<int:num>/', views.page),
    ]
    
    # View (in blog/views.py)
    def page(request, num=1):
        # Output the appropriate page of blog entries, according to num.
        ...

위의 예시에서, 두 URL 패턴은 동일한 view - **[views.page](http://views.page)** -를 가리키고 있지만 첫 번째 패턴은 URL에서 아무것도 캡처하지 않습니다. 만약 처 번째 패턴이 매치되면, **page()** 함수는 **num**, **1**에 대한 기본 argument를 사용합니다. 만약 두 번째 패턴이 매치되면, **page()**는 캡처된 모든 **num**값을 사용합니다.

---

# Performance

**urlpatterns**의 각 정규표현식은 처음 접속할 때 컴파일됩니다. 이것은 시스템을 놀랍도록 빠르게 만듭니다.

---

# Syntax of the **urlpatterns variable**

**urlpatterns**는 **`path()`** 및/또는 **`re_path()`** 인스턴스의 파이썬 리스트여야 합니다.

---

# Error handling

장고는 요청된 URL에 대한 매치되는 항목을 찾지 못하거나 예외가 발생하면 error-handling view를 호출합니다.

이러한 경우에 사용할 views는 4개의 변수로 특정됩니다. 그것의 기본값들은 대부분의 프로젝트에 충분해야 하지만, 추가적인 사용자 설정은 기본값을 오버라이드하는 것으로 가능합니다.

자세한 정보는 `error views 사용자 설정`을 참고하세요.

이러한 값은 root URLconf에서 설정할 수 있습니다. 이러한 변수를 다른 URLconf에 설정하는 것은 아무런 영향을 미치지 않습니다.

값은 호출가능하거나 에러 상황을 처리하기 위해 호출해야하는 view에 대한 전체 파이썬 import 경로를 나타내는 문자열이여야 합니다.

변수는 다음과 같습니다 : 

- **handler400** - **`django.conf.urls.handler400`**
- **handler403** - **`django.conf.urls.handler403`**
- **handler404** - **`django.conf.urls.handler404`**
- **handler500** - **`django.conf.urls.handler500`**

---

# Including other URLconfs

어떤 시점에서든 당신의 **urlpatterns**는 다른 URLconf 모듈들을 "include"할 수 있습니다. 이것은 기본적으로 다른 것들보다 아래의 URL들의 집합을 "roots"합니다.

예를 들어, 여기 `장고 웹사이트`를 위한 URLconf의 발췌부분이 있습니다. 여기에는 다른 많은 URLconfs가 include됩니다 : 

    from django.urls import include, path
    
    urlpatterns = [
        # ... snip ...
        path('community/', include('aggregator.urls')),
        path('contact/', include('contact.urls')),
        # ... snip ...
    ]

장고가 **`include()`**를 만날 때 마다 URL의 매치된 부분을 해당 지점까지 잘라내고 추가 처리를 위해 나머지 문자열을 include된 URLconf로 전송합니다.

또 다른 가능성으로는 **`path()`** 인스턴스 리스트를 사용하여 추가 URL 패턴을 include시키는 것입니다. 예를 들어 다음 URLconf를 고려하십시오 : 

    from django.urls import include, path
    
    from apps.main import views as main_views
    from credit import views as credit_views
    
    extra_patterns = [
        path('reports/', credit_views.report),
        path('reports/<int:id>/', credit_views.report),
        path('charge/', credit_views.charge),
    ]
    
    urlpatterns = [
        path('', main_views.homepage),
        path('help/', include('apps.help.urls')),
        path('credit/', include(extra_patterns)),
    ]

이 예제에서 **/credit/reports/** URL은 **credit_views.report()** Django view에 의해 처리하게 됩니다.

이것은 단일 패턴 접두사를 반복적으로 사용하는 URLconf에서 중복을 제거하는 데 사용될 수 있습니다. 예를 들어 다음 URLconf를 고려하십시오 : 

    from django.urls import path
    from . import views
    
    urlpatterns = [
        path('<page_slug>-<page_id>/history/', views.history),
        path('<page_slug>-<page_id>/edit/', views.edit),
        path('<page_slug>-<page_id>/discuss/', views.discuss),
        path('<page_slug>-<page_id>/permissions/', views.permissions),
    ]

우리는 공통 경로 접두사를 한번만 표시하고 다른 접미어를 그룹화하여 이 문제를 개선할 수 있습니다 : 

    from django.urls import path
    from . import views
    
    urlpatterns = [
        path('<page_slug>-<page_id>/history/', views.history),
        path('<page_slug>-<page_id>/edit/', views.edit),
        path('<page_slug>-<page_id>/discuss/', views.discuss),
        path('<page_slug>-<page_id>/permissions/', views.permissions),
    ]

## Captured parameters

include된 URLconf는 부모 URLconfs에서 캡처된 매개변수를 수신하므로 다음 예제가 유효합니다 : 

    # In settings/urls/main.py
    from django.urls import include, path
    
    urlpatterns = [
        path('<username>/blog/', include('foo.urls.blog')),
    ]
    
    # In foo/urls/blog.py
    from django.urls import path
    from . import views
    
    urlpatterns = [
        path('', views.blog.index),
        path('archive/', views.blog.archive),
    ]

위의 예제에서 캡처된 **"username"** 변수는 예상대로 include된 URLconf로 전달됩니다.

---

# Passing extra options to view functions

URLconfs에는 추가 arguments를 파이썬 사전으로서 view함수에 전달할 수 있는 후크가 있습니다.

`path()`함수는 view함수에 전달할 추가 keyword argument의 사전이어야하는 선택적인 제 3의 argument를 가질 수 있습니다.

예 : 

    from django.urls import path
    from . import views
    
    urlpatterns = [
        path('blog/<int:year>/', views.year_archive, {'foo': 'bar'}),
    ]

이 예제에서 **/blog/2005/**에 대한 요청으로 장고는 **views.year_archive(request, year=2005, foo= 'bar')**를 호출합니다.

이 기술은 `syndication framework`에서 메타데이터 및 옵션을 views에 전달하는데 사용됩니다.

> **Dealing with conflicts**

명명된 keyword argument를 캡처하고 추가 argument의 사전에서 동일한 이름의 argument를 전달하는 URL 패턴을 가질 수 있습니다. 이 경우 URL에서 캡처한 argument 대신 사전의 argument가 사용됩니다.

## Passing extra options to include()

마찬가지로 **`include()`**에 추가 옵션을 전달할 수 있으며 include된 URLconf의 각 행에는 추가 옵션이 전달됩니다.

예를 들어, 이 두 URLconf sets는 기능적으로 동일합니다 : 

첫 번째 Set : 

    # main.py
    from django.urls import include, path
    
    urlpatterns = [
        path('blog/', include('inner'), {'blog_id': 3}),
    ]
    
    # inner.py
    from django.urls import path
    from mysite import views
    
    urlpatterns = [
        path('archive/', views.archive),
        path('about/', views.about),
    ]

두 번째 Set : 

    # main.py
    from django.urls import include, path
    from mysite import views
    
    urlpatterns = [
        path('blog/', include('inner')),
    ]
    
    # inner.py
    from django.urls import path
    
    urlpatterns = [
        path('archive/', views.archive, {'blog_id': 3}),
        path('about/', views.about, {'blog_id': 3}),
    ]

추가 옵션은 include된 URLconf의 모든 행에 항상 전달되며, 행의 view가 해당 옵션을 실제로 유효한 것으로 수용하는지에 대한 여부와는 관계가 없습니다. 이러한 이유로 이 기술은 include된 URLconf의 모든 view가 통과하고 있는 추가 옵션을 확실히 받아들일 경우에만 유용합니다.

---

# Reverse resolution of URLs

장고 프로젝트를 진행할 때 공통적으로 필요한 것은 생성된 컨텐츠에 내장(view 및 에셋 URLs, 사용자에게 보여지는 URL 등) 또는 서버측의 네비게이션 플로우 처리(리디렉션 등)를 위해 최종 형태의 URL들을 얻을 수 있는 가능성입니다.

이러한 URL들을 하드 코딩하지 않는 것이 바람직합니다(힘들고 확정성이 없으며 오류가 발생하기 쉬운 방법). 마찬가지로 위험한 것은 URLconf에서 설명하는 디자인과 평행한 URL들을 생성하는 ad-hoc 메커니즘을 고안하는 것으로 시간이 지남에 따라 오래된 URL이 생성될 수 있습니다.

즉, 필요한 것은 DRY 메카니즘입니다. 이것은 여러 장점들 중 오래된 URL을 검색하고 대체하기 위해 모든 프로젝트 소스코드를 검토할 필요 없이 URL 디자인의 진화를 가능하게 할 것입니다.

URL을 얻기 위해 우리가 이용할 수 있는 정보의 최초의 부분은 그것을 다루는 view의 식별(e.g. 이름)입니다. 반드시 필요한 URL의 조회에 참여해야하는 정보의 다른 부분으로는 view argument의 유형(positional, keyword)과 값이 있습니다.

장고는 URL 매퍼가 URL 디자인의 유일한 저장소라고 할 수 있는 솔루션을 제공합니다. URLconf로 피드를 보낸 다음 양방향으로 사용할 수 있습니다 : 

- 사용자/브라우저가 요청한 URL을 시작으로, URL에서 추출한 값과 함께 필요할 수 있는 argument를 제공하는 적잘한 장고 view를 호출합니다.
- 해당 장고 view의 식별자와 전달된 argument의 값부터 시작하여, 연관된 URL을 얻습니다.

첫 번째는 이전 섹션에서 논의하였던 사용법입니다. 두 번째는 URL 분석 리버스, URL 매칭 리버스, URL 조회 리버스 또는 단순히 URL 리버싱이라 하는 것입니다.

장고는 URL이 필요한 여러 레이어와 일치하는 URL 리버싱을 수행하는 도구를 제공합니다 : 

- 탬플릿에서 : **`url`** 탬플릿 태그 사용
- 파이썬 코드에서 : **`reverse()`** 함수 사용
- 장고 모델 인스턴스의 URL처리와 관련된 상위 레벨 code에서 : **`get_absolute_url()`** 메소드

## Examples

이 URLconf항목을 다시 고려하세요 : 

    from django.urls import path
    
    from . import views
    
    urlpatterns = [
        #...
        path('articles/<int:year>/', views.year_archive, name='news-year-archive'),
        #...
    ]

이 설계에 따른면 연도 nnnn에 해당하는 아카이브의 URL은 **/article/<nnnn>/**입니다.

이 코드는 템플릿 코드에서 다음을 사용하여 얻을 수 있습니다 : 

    <a href="{% url 'news-year-archive' 2012 %}">2012 Archive</a>
    {# Or with the year in a template context variable: #}
    <ul>
    {% for yearvar in year_list %}
    <li><a href="{% url 'news-year-archive' yearvar %}">{{ yearvar }} Archive</a></li>
    {% endfor %}
    </ul>

또는 파이썬 코드에서 : 

    from django.http import HttpResponseRedirect
    from django.urls import reverse
     
    def redirect_to_year(request):
        # ...
        year = 2006
        # ...
        return HttpResponseRedirect(reverse('news-year-archive', args=(year,)))

어떠한 이유로 인해 연간 기사 아카이브의 내용이 게시된 URL을 변경해야 하는 경우 URLconf에서 항목을 변경하기만 하면 됩니다.

views가 일반적인 성격인 일부의 시나리오에서는 URL과 views 사이에 다대일 관계가 존재할 수 있습니다. 이러한 경우 view이름은 리버싱 URLs을 할 때 식별자로서 충분하지 않습니다. 다음을 읽고 장고가 제공하는 솔루션에 대해 알아보십시오.

---

# Naming URL patterns

URL 리버싱을 실시하기 위해서는 위의 예제에서와 같이 **명명된 URL 패턴**을 사용하여야 합니다. URL 이름에 사용되는 문자열에는 원하는 모든 문자가 포함될 수 있습니다. 유효한 파이썬 이름들에 의해 제한되지 않습니다.

URL 패턴의 이름을 지정할 때 다른 애플리케이션이 사용하는 이름과 충돌할 가능성이 없는 이름을 선택합니다. URL 패턴 **comment**를 호출하고 다른 애플리케이션이 동일한 작업을 수행하는 경우 **`reverse()`**가 찾는 URL은 프로젝트의 **urlpatterns** 리스트의 마지막 패턴에 따라 다릅니다.

URL이름에 접두사를 추가하면, 애플리케이션 이름(예 : **comment** 대신 **myapp-comment** 등)에서 파생되어 충돌할 가능성이 감소합니다.

view를 오버라이드하고 싶은 경우 다른 애플리케이션과 동일한 URL이름을 의도적으로 선택할 수 있습니다. 예를 들어, 일반적인 사용 사례는 **`LoginView`**를 오버라이드하는 것입니다. 장고의 일부와 대부분의 서드파티 앱들은 이 view가 **login**이라는이름의 URL패턴을 가지고 있다고 가정합니다. 커스텀 로그인 view를 가지고 있고 그것의 URL에 **login**이라는 이름을 붙이면, **`reverse()`**는 **django.contrib.auth.urls**가 include된 후(그것이 전부 포함된 경우) 그것이 **urlpatterns**에 있는 한 커스텀 로그인 view를 찾습니다.  

또한 argument가 각각 다른 경우 여러 URL 패턴에 동일한 이름을 사용할 수도 있습니다.  URL 이름 외에도 **`reverse()`**는 argument 수와 keyword argument의 이름을 매치시킵니다.

---

# URL namespaces

## Introduction

URL 네임스페이스를 사용하면 다른 애플리케이션이 동일한 URL 이름을 사용하더라도 `명명된 URL 패턴`을 고유하게 리버스할 수 있습니다. 서드 파티 앱은 항상 네임스페이스가 있는 URLs을 사용하는 것이 좋습니다(튜토리얼에서 처럼). 마찬가지로 애플리케이션의 여러 인스턴스가 배포된 경우 URL을 리버스할 수 있습니다. 즉, 단일 애플리케이션의 여러 인스턴스가 명명된 URL을 공유하므로 네임스페이스는 이러한 명명된 URL을 구분할 수 있는 방법을 제공합니다.

URL 네임스페이스를 적절하게 사용하는 장고 애플리케이션은 특정 사이트에 두 번 이상 배포할 수 있습니다. 예를 들어 **`django.contrib.admin`**에는 `하나 이상의 admin 인스턴스를 배포`할 수 있는 **`AdminSite`**클래스가 있습니다. 다음 예제에서는 서로 다른 두 명의 독자(작성자 및 게시자)에게 동일한 기능을 제공할 수 있도록 다른 위치에 있는 2개의 튜토리얼에서 polls 애플리케이션을 배치하는 아이디어를 설명할 것입니다.

URL 네임스페이스는 두 부분으로 나뉘며 둘 다 문자열입니다 : 

**application namespace**

이것은 배포되고 있는 애플리케이션의 이름을 설명합니다. 단일 어플리케이션의 모든 인스턴스는 같은 애플리케이션 네임스페이스를 가집니다. 예를 들어, 장고의 관리자 애플리케이션에는 다소 예측 가능한 애플리케이션 네임스페이스인 **'admin'**이 있습니다.

**instance namespace**

이것은 애플리케이션의 특정 인스턴스를 식별합니다. 인스턴스 네임스페이스는 전체 프로젝트에서 유일해야 합니다. 그러나 인스턴스 네임스페이스는 애플리케이션 네임스페이스와 동일할 수 있습니다. 이것은 애플리케이션의 기본 인스턴스를 지정하는데 사용됩니다. 예를 들어, 기본 장고 관리자 인스턴스는 **'admin'** 인스턴스 네임스페이스를 가집니다.

네임스페이스가 있는 URL은 **':'** 연산자를 사용하여 지정됩니다. 예를 들어, 관리자 애플리케이션의 기본 색인 페이지는 **'admin:index'**를 사용하여 참조됩니다. 이것은 **'admin'**의 네임스페이스와 **'index'**라는 이름의 URL을 나타냅니다.

네임스페이스는 nested될 수 있습니다. 명명된 URL **'sports:polls:index'**는 최상위 네임스페이스 **'sports'** 내에 정의된 **'polls'**네임스페이스에서 **'index'**라는 패턴을 찾습니다.

---

## Reversing namespaced URLs

해결하고자하는 네임스페이스가 있는 URL (e.g. **'polls:index'**)이 주어지면 장고는 정규화된 이름을 분할한 후 다음과 같은 조회를 시도합니다 :  

1. 첫 번째로, 장고는 매치되는 애플리케이션 네임스페이스(이 예에서는 **'polls'**)를 찾습니다. 그러면 해당 애플리케이션의 인스턴스 리스트가 생성됩니다.
2. current 애플리케이션이 정의된 경우, 장고는 해당 인스턴스의 URL 확인자를 찾아 반환합니다. current 애플리케이션은 **`reverse()`**함수에 대한 **current_app** argument로 지정할 수 있습니다. **`url`** 탬플릿 태그는 현재 해결된 view의 네임스페이스를 **`RequestContext`**의 current 애플리케이션으로 사용됩니다. **`request.current_app`** attribute에 current 애플리케이션을 설정하는 것으로 기본값을 오버라이드 할 수 있습니다.
3. current 애플리케이션이 없으면 장고는 기본 애플리케이션 인스턴스를 찾습니다. 기본 어플리케이션 인스턴스는 `application namespace`와 매치되는 `instance namespace`가 있는 인스턴스입니다(이 예제에서는 **'polls'**라는 **polls** 인스턴스).
4. 기본 애플리케이션 인스턴스가 없으면 장고는 인스턴스 이름이 무엇이든 마지막으로 배포된 애플리케이션의 인스턴스를 선택합니다.
5. 제공된 네임스페이스가 1단계에서 `application namespace`와 매치되지 않으면 장고는 네임스페이스를 `instance namespace`로 직접 조회합니다.

만약 nested 네임스페이스가 있으면, view 이름이 확인 될 때까지만 네임스페이스의 각 부분에 대해 이러한 단계가 반복됩니다. view 이름은 확인된 네임스페이스의 URL로 해석됩니다.

### Example

이 해결 방법을 실제로 적용하려면 튜토리얼의 polls 애플리케이션의 두 인스턴스(하나는 **'author-polls'**이고, 다른 하나는 **'publisher-polls'**이다)에 대한 예제를 고려하십시오. **polls**를 만들고 표시 할 때 인스턴스 네임스페이스를 고려하도록 해당 애플리케이션을 향상시켰다고 가정합니다.

    # urls.py
    from django.urls import include, path
    
    urlpatterns = [
        path('author-polls/', include('polls.urls', namespace='author-polls')),
        path('publisher-polls/', include('polls.urls', namespace='publisher-polls')),
    ]

    # polls/urls.py
    from django.urls import path
    
    from . import views
    
    app_name = 'polls'
    urlpatterns = [
        path('', views.IndexView.as_view(), name='index'),
        path('<int:pk>/', views.DetailView.as_view(), name='detail'),
        ...
    ]

이 설정을 사용하여 다음 조회가 가능합니다 : 

- 인스턴스 중 하나가 current인 경우 - 예를 드러 **'author-polls'**인스턴스에서 디테일한 페이지를 렌더링하는 경우 - **'polls:index'**인스턴스의 색인 페이지로 해석됩니다. 즉, 다음 두 경우 모두 **"/author-polls/'**가 됩니다.

클래스 기반 view의 매소드 : 

    reverse('polls:index', current_app=self.request.resolver_match.namespace)

탬플릿에서 : 

    {% url 'polls:index' %}

- current 인스턴스가 없다면 - 예를 들어 사이트의 다른 페이지를 렌더링하는 경우 - **'polls:index'**는 마지막으로 등록된 **polls**인스턴스로 해석됩니다. 기본 인스턴스 (**'polls'**의 instance namespace)가 없으므로 등록된 마지막 인스턴스가 사용됩니다. 이것은 **urlpatterns**에서 마지막으로 선언되었기 때문에 **'publisher-polls'**가 됩니다.
- **'author-polls:index'**는 항상 **'author-polls'**인스턴스의 색인 페이지(그리고 **'publisher-polls'**도 마찬가지)로 해석됩니다.

만일 기본 인스턴스(즉, **'polls'**라는 인스턴스)가 존재한다면, current 인스턴스가 없는 경우(위의 목록에서 2번째 항목)에만 위에서 변경됩니다. 이 경우 **'polls:index'**는 **urlpatterns**에서 마지막으로 선언된 인스턴스 대신 기본 인스턴스의 색인 페이지로 해석됩니다.

---

## URL namespaces and included URLconfs

include된 URLconfs의 애플리케이션 네임스페이스는 두 가지 방법으로 지정할 수 있습니다.

첫째, include된 URLconf 모듈에서 **urlpatterns** attribute와 동일한 수준으로 **app_name** attribute를 설정할 수 있습니다. **urlpatterns** 자체의 리스트가 아니라 **`include()`**하기 위해 실제 모듈이나 문자열 레퍼런스를 모듈에 전달해야 합니다.

    # polls/urls.py
    from django.urls import path
    
    from . import views
    
    app_name = 'polls'
    urlpatterns = [
        path('', views.IndexView.as_view(), name='index'),
        path('<int:pk>/', views.DetailView.as_view(), name='detail'),
        ...
    ]

    # urls.py
    from django.urls import include, path
    
    urlpatterns = [
        path('polls/', include('polls.urls')),
    ]

**polls.urls**에 정의된 URL에는 애플리케이션 네임스페이스 **polls**가 있습니다.

둘째, 네임스페이스 데이터가 포함된 개체를 include할 수 있습니다. **`path()`** 혹은 **`re_path()`** 인스턴스의 리스트를 **include()**하면 해당 객체에 포함된 URL이 전역 네임스페이스에 추가됩니다. 그러나 다음을 포함하는 2-튜플을 **include()**할 수도 있습니다 : 

    (<list of path()/re_path() instances>, <application namespace>)

예 : 

    from django.urls import include, path
    
    from . import views
    
    polls_patterns = ([
        path('', views.IndexView.as_view(), name='index'),
        path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    ], 'polls')
    
    urlpatterns = [
        path('polls/', include(polls_patterns)),
    ]

여기에는 주어진 애플리케이션 네임스페이스에 지정된 URL 패턴이 include됩니다.

인스턴스 네임스페이스는 **`include()`**에 대한 **네임스페이스** argument를 사용하여 지정할 수 있습니다. 인스턴스 네임스페이스가 특정되지 않으면 include된 URLconf의 애플리케이션 네임스페이스가 기본값으로 사용됩니다. 즉, 해당 네임스페이스의 기본 인스턴스이기도 합니다.