---
title: "Making queries"
categories:
- django
tags:
- django reference
---

# Making queries

**본 글은 작성 중 입니다**

너가 `data models`를 만들고 나면, Django는 create, retrieve, update 그리고 delete object를 할 수 있는 database-abstraction API를 자동적으로 제공할 것이다. 이 문서는 이 API를 어떻게 사용하는지를 알려준다. 모든 다양한 model lookup options에 대한 자세한 내용은 `data model reference`를 참조하여라.

이 가이드를 설명하는 동안, 우리는 다음과 같이 Weblog application을 구성하는 models를 사용할 것이다:

    from django.db import models
    
    class Blog(models.Model):
    	name = models.CharField(max_length=100)
    	tagline = models.TextField()
    	
    	def __str__(self):
    		return self.name
    
    class Author(models.Model):
    	name = models.CharField(max_length=200)
    	email = models.EmailField()
    
    	def __str__(self):
    		return self.name
    
    class Entry(models.Model):
    	blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    	headline = models.CharField(max_length=255)
    	body_text = models.TextField()
    	pub_date = models.DateField()
    	mod_date = models.DateField()
    	authors = models.ManyToManyField(Author)
    	n_comments = models.IntegerField()
    	n_pingbacks = models.IntegerField()
    	rating = models.IntegerField()
    
    	def __str__(self):
    		return self.headline

# Creating objects

Python object의 database-table data를 나타내기 위해, Django는 직관적인 시스템을 사용한다: database class를 나타내는 model class, 그리고 그 class의 instance는 database table 안의 특정 기록들을 나타낸다.

object를 생성하기 위해, model class의 keyword arguments를 사용하여 그것을 instantiate 한 뒤, database에 저장하기 위해 **`save()`**를 호출해라.

models가 **mysite/blog/models.py** 파일 안에 있다고 가정하자. 예시는 다음과 같다:

    >>> from blog.models import Blog
    >>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
    >>> b.save()

이는 뒤에서 **INSERT** SQL statement를 실행한다. Django는 너가 명시적으로 **`save()를`** 호출하기 전까지 database를 건드리지 않는다.

**`save()`** method는 return 값이 없다.

- **See also**

    **`save()`**는 여기에서 설명하지 않은 다수의 advanced option들이 있다. 모든 세부 사항을 위해서 **`save()`** 문서를 참조하라

    object를 생성하고 저장하는 것을 한 번에 하기 위해서는 **`create()`** method를 사용하여라.

---

# Saving changes to objects

databse 에 이미 있는 object의 변경 사항을 저장하기 위해서는 **`save()`**를 사용해라. 

주어진 **Blog** instance **b5**는 이미 database에 저장되어 있다. 이 예시는 그것의 이름을 바꾸고, database에 그것의 record를 update하는 것이다:

    >>> b5.name = 'New name'
    >>> b5.save()

이것은 뒤에서 **UPDATE** SQL statement를 실행한다. Django는 너가 명시적으로 **`save()`** 호출하기 전까지 database를 건드리지 않는다.

## Saving ForeignKey and ManyToManyField fields

**`ForeignKey`** field를 update하는 것은 일반적인 field를 저장하는 것과 정확히 같은 방법으로 작동한다 - 단순히 옳은 타입의 object를 할당 해주는 것이다. 이 예시는 **Entry** instance **entry**의 **blog** attribute를 update하는 것이다. **Entry**와 **Blog**의 적당한 instances들이 이미 database에 저장되어 있다고 가정하자(따라서 우리는 다음과 같이 그들을 retrieve 할 수 있다):

    >>> from blog.models import Blog, Entry
    >>> entry = Entry.objects.get(pk=1)
    >>> chees_blog = Blog.objects.get(name="Cheddar Talk")
    >>> entry.blog = cheese_blog
    >>> entry.save()

**`ManyToManyField`**를 updating하는 것은 약간 다르게 작동한다 - relation에 record를 추가하기 위해 field에 **`add()`** method를 사용해라. 이 예시는 **Author** instance **joe**를 **entry** object에 추가한 것이다:

    >>> from blog.models import Author
    >>> joe = Author.objects.create(name="Joe")
    >>> entry.authors.add(joe)

**`ManyToManyField`**에 많은 수의 record를 한 번에 저장하기 위해, 많은 arguments를 **`add()`** 의 호출 안에 포함해라. 다음과 같다:

    >>> john = Author.objects.create(name="John")
    >>> paul = Author.objects.create(name="Paul")
    >>> george = Author.objects.create(name="George")
    >>> ringo = Author.objects.create(name="Ringo")
    >>> entry.authors.add(john, paul, george, ringo)

Django는 만약 너가 다른 종류의 object를 추가하거나 할당 할 때 error를 일으킬 것이다.

---

# Retrieving objects

너의 database로부터 objects를 retrieve하기 위해, 너의 model class의 **`Manager`**를 통해 **`QuerySet`**을 구성해라.

**`QuerySet`**은 너의 database의 objects의 모음을 나타낸다. 이것은 없거나 한 개일 수도 있고, 많은 *filter*일 수도 있다. Filters는 주어진 parmeters에 따라서 query results를 줄여 나간다. SQL 용어로는, **`QeurySet`**은 **SELECT** 명령어에 해당하고, filter은 제한하는 구절인 **WHERE** 또는 **LIMIT**이다.

너는 너의 model **`Manager`**를 사용함으로써 **`QuerySet`**을 얻는다. 각각의 model은 적어도 한 개의 **`Manager`**를 갖고 있고, 이는 기본적으로 **`objects`**라고 불리운다. 다음와 같이 model class를 통해 직접적으로 접근할 수 있다:

    >>> Blog.objects
    <django.db.models.manager.Manager object at ...>
    >>> b = Blog(name='Foo', tagline='Bar')
    >>> b.objects
    Traceback:
    	...
    AttributeError: "Manager isn't accessible via Blog instances."

- **Note**

    **`Managers`**는 model class를 통해서만 접근할 수 있고 model instances를 통해서는 안된다. 이는 "table-level" operations와 "record-level" operations를 분리하기 위함이다.

**`Manager`**은 한 model에 대한 **QuerySets**의 메인 소스이다. 예를 들어, **Blog.objects.all()**은 database에서 모든 **Blog** objects를 담고 있는 **`QuerySet`**을 반환한다.

## Retrieving all objects

table로 부터 objects를 retrieve하는 가장 쉬운 방법은 모든 것들을 받는 것이다. 이를 하기 위해서 **`Manager`**에 **`all()`** method를 사용해라:

    >>> all_entries = Entry.objects.all()

**`all()`** mothod는 database에 있는 모든 objects의 **`QuerySets`**을 반환한다.

---

## Retrieving specific objects with filters

**`all()`**에 의해 반환된 **`QuerySet`**은 database table의 모든 objects를 묘사한다. 그러나 보통, 너는 완전한 objects의 집합에서 일부만 선택해야 할 필요가 있을 것이다.

이러한 부분집합을 만들기 위해서, 너는 filter conditions를 추가하여 초기의 **`Queryset`**을 수정한다. 다음은 **`QuerySet`**을 refine하는 대표적인 두 가지 방법이다:

**filter(**kwargs)**

주어진 lookup parameters와 match하는 

**exclude(**kwargs)**

주어진 lookup parameters와 match하지 않는 objects를 포함하고 있는 새로운 **`QuerySet`**을 반환한다. 

lookup parameters (위 함수의 정의에서 ****kwargs**)는 `Field lookups` 아래에 기술된 포멧을 따라야 한다.

예를 들어, 2006년의 blog entries의 **`QuerySet`**을 얻고 싶다면 다음과 같이 **`filter()`**를 사용해라:

    Entry.objects.filter(pub_date__year=2006)

기본 manager class를 사용하면 이는 다음과 같다:

    Entry.objects.all().filter(pub_date__year=2006)

**Chaining filters**

**`QuerySet`**을 refine한 결과는 **`QuerySet`** 그 자신이기 때문에 연속적인 refinement을 함께 할 수 있다. 예시를 보자:

    >>> Entry.objects.filter(
    ...		headline__startswith='What'
    ...).exclude(
    ...		pub_date__gte=datetime.date.today()
    ...).filter(
    ...		pub_date__gte=datetime.date(2005, 1, 30)
    ...)

이는 모든 database의 entries를 시작 **`QuerySet`**으로 받고, filter를 추가하고, exclusion하고, 다른 filter를 추가했다. 최종 **`QuerySet`**의 결과는 "What"으로 시작하는 headline과 현재 날짜와 2005년 1월 30일 사이에 출판된 모든 entries를 갖고 있다.

---

**Filtered QuerySets are unique**

너가 **`QuerySet`**을 refine할 때마다, 너는 이전의 **`QuerySet`**과 전혀 연결되지 않은 새로운 **`QuerySet`**을 을 얻는다. 각각의 refinement는 저장되고, 사용되고, 다시 쓸 수 있는 분리되고 별개의 **`QuerySet`**을 생성한다. 

다음은 예시이다:

    >>> q1 = Entry.objects.filter(headline__startswith="What")
    >>> q2 = q1.exclude(pub_date__dte=datetime.date.today())
    >>> q3 = q1.filter(pub_date__gte=datetime.date.today())

이 세 가지 **QuerySets**는 각각 별개이다. 첫 번째 것은 "What"으로 시작하는 headline을 갖고 있는 모든 entries를 포함하는 **`QuerySet`**이다. 두 번째 것은 첫 번째 것의 하위 집합으로, **pub_date**가 오늘이거나 미래인 것들의 records를 제외하는 추가적인 기준을 갖고 있다. 세 번째 것은 첫 번째의 하위 집합으로, **pub_date**가 오늘이거나 미래인 것들만 고르는 추가적인 기준을 갖고 있다. 초기 **`QuerySet`**(**q1**)은 refinement 과정에 영향을 받지 않는다.

---

**QuerySets are lazy**

QuerySets는 게으르다 - **`QuerySet`**을 생성하는 작업은 어떠한 database activity도 포함하지 않는다. 너는 매우 길게 filters를 쌓을 수 있지만, Django는 **`QuerySet`**가 *evaluated* 되기 전까지 실제로 query를 실행하지 않는다. 다음 예시를 보자:

    >>> q = Entry.objects.filter(headline__startswith="What")
    >>> q = q.filter(pub_date__lte=datetime.date.today())
    >>> q = q.exclude(body_text__icontains="food")
    >>> print(q)

이는 세 번의 database hit처럼 보이지만, 사실 이것은 마지막 line의 (print(q))에서 database를 단 한 번만 hit한다. 일반적으로, 너가 **`QuerySet`**의 결과를 요구하지 않는 이상, 그것들은 database로 부터 도출되지 않을 것이다. 너가 그것을 할 때, **`QuerySet`**은 database에 접근하여 *evaluated*된다. 언제 evaluation이 일어나는지에 대한 자세한 디테일은 `When QuerySets are evalutated`를 보아라.

---

## Retrieving a single object with get()

**`filter()`**는 너에게 항상 **`QuerySet`**을 줄 것이다, 단 하나의 object가 query에 해당할 지라도 - 이 경우에, 이는 하나의 요소를 갖고 있는 **`QuerySet`**이 될 것이다.

만약 너가 query에 해당하는 하나의 object가 있다는 것을 안다면, 그 object를 직접적으로 반환하는 **`Manager`**의 **`get()`** mehtod를 사용해라:

    >>> one_entry = Entry.objects.get(pk=1)

너는 마치 **`filter()`**처럼 어떠한 query expression과도 함께 **`get()`**을 사용할 수 있다 - 다시 말하지만, `Field looups` 아래를 참고해라.

**`get()`**을 사용하는 것과 **[0]** 슬라이싱과 함께 **`filter()`**를 사용하는 것에 차이가 있다는 점을 알고 있어라. 만약 query와 맞는 결과가 없다면, **`get()`**은 **DoesNotExist** exception을 일으킬 것이다. 이 exception은 query가 수행되고 있는 model class의 attribute이다 - 따라서 위의 코드에서 만약 primary key가 1인 **Entry** object가 없다면, Django는 **Entry.DoesNotExist**를 일으킬 것이다.

비슷하게, Django는 **`get()`** query에 한 개 이상의 해당하는 것이 있을 경우 에러를 일으킬 것이다. 이 경우, 그것은 **`MultipleObjectsReturned`**를 일으킬 것이고, 이 역시 model class 자체의 attribute이다.

---

## Other QuerySet methods

너가 **`all()`**, **`get()`**, **`filter()`**, 그리고 **`exclude()`**를 사용할 대부분의 경우는 너가 database로부터 objects를 찾아볼 필요가 있을 때이다. 그러나 이 외에도 많은 것이 있다; 다양한 **`QuerySet`** method의 전체 리스트를 보기 위해서는 `QuerySet API Reference`를 보아라.

---

## Limiting QuerySets

너의 **`QuerySet이`** 특정 개수의 결과를 갖게 제한하고 싶다면 Python의 array-slicing syntax를 이용해라. 이는 SQL의 **LIMIT** 과 **OFFSET** clauses와 같다.

예를 들어 다음은 첫 5개의 objects를 반환한다(**LIMIT 5**):

    >>> Entry.objects.all()[:5]

다음은 6번째 부터 10번째 objects를 반환한다(**OFFSET 5 LIMIT 5**):

    >>> Entry.objects.all()[5:10]

Negative indexing(i.e. **Entry.objects.all()[-1]**)은 지원되지 않는다.

일반적으로, slicing QuerySet은 새로운 QuerySet을 반환한다 - 이것은 query를 evaluate하지 않는다. 예외는 너가 Python slice syntax에서 "step" parameter를 사용할 경우이다. 예를 들어, 이것은 첫 10개의 매 두 번째 objects를 반환하기 위해 query를 실행한다:

    >>> Entry.objects.all()[:10:2]

sliced queryset에 filtering을 추가하거나 순서를 매기는 것은 그것이 어떻게 작동할지에 대한 모호성 때문에 금지된다.

리스트가 아니라 하나의 단일 object를 얻고 싶다면(e.g. **SELECT foo FROM bar LIMIT 1**), slice 대신에 간단한 index를 사용해라. 예를 들어, 이는 headline을 알파벳 순서로 정렬한 후, database의 첫 번째 **Entry**를 반환할 것이다. 

    >>> Entry.objects.order_by('headline')[0] 

이것은 대략 다음과 같다:

    >>> Entry.objects.order_by('headline')[0:1].get()

그러나, 만약 기준에 합당한 objects가 없다면 첫 번째 것은 **IndexError**를 일으키는 반면, 두 번째 것은 **DoesNotExist**를 일으킬 것이다. 자세한 사항은 **`get()`**을 보아라.

---

## Field lookups

Field lookups는 SQL **WHERE** clause의 항목을 지정하는 방법이다. 그들은 **QuerySet** method인 **`filter()`**, **`exclude()`**, 그리고 **`get()`**에 keyword arguments로 명시돼 있다.

기본 lookups keyword arguments들은 **field__lookuptype=value**와 같은 형식을 취한다. (두 개의 underscore이다). 예시를 보자:

    >>> Entry.objects.filter(pub_date__lte='2006-01-01')

대략 다음과 같은 SQL로 바뀐다:

    SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';

- **How this is possible**

    Python은 runtime에 names와 values가 evalutated되는 임의의 name-value arguments를 허용하는 함수를 정의 할 수 있다. 자세한 내용은 공식 Python 튜토리얼의 `Keyword Arguments`를 참조하여라.

lookup에 명시된 field는 model field의 이름이어야 한다. 한 가지 예외가 있는데, **`ForeignKey`**의 경우, 너는 접미사로 **_id**가 추가된 field name을 명시할 수 있다. 이 경우에 value parameter는 foreign model의 primary key의 raw value를 포함할 것을 기대된다. 예시를 보자:

    >>> Entry.objects.filter(blog_id=4)

만약 너가 유효하지 않은 keyword argument를 넘겨준다면, lookup 함수는 **TypeError**를 일으킬 것이다.

datase API는 약 24개의 lookup type들을 소개한다; 완전한 reference는 `field lookup reference`를 참조하여라. 무엇이 가능한지 잠깐 소개를 하기 위해 여기에는 너가 자주 사용하게 될 몇 가지를 소개하겠다:

**`exact`**

''exact'' match이다. 예시를 보자:

    >>> Entry.objects.get(headline__exact="Cat bites dog")

이는 다음과 같은 SQL lines이다:

    SELECT ... WHERE headline = 'Cat bites dog';

만약 너가 lookup type을 제공하지 않는다면 - 즉, 너의 keyword argument가 double underscore을 포함하지 않는다면 - lookup type은 **exact**라고 가정된다.

예를 들어, 다음 두 문장은 동일하다:

    >>> Blog.objects.get(id__exact=14)   # Explicit form
    >>> Blog.objects.get(id=14           # __exact is implied

**exact** lookups는 일반적인 경우이므로, 이는 편의를 위한 것이다.

**`iexact`**

case-insensitive match이다. 따라서 다음 query:

    >>> Blog.objects.get(name__iexact="beatles blog")

는 "Beatles Blog", "beatles blog", 또는 심지어 "BeAtlES blOG"의 제목을 가진 Blog를 match한다.

**`contains`**

case-sensitive한 포함 테스트이다. 예시를 보자:

    Entry.objects.get(haadline__contains='Lennon')

이는 대략 다음 SQL과 같다:

    SELECT ... WHERE headline LIKE '%Lennon%';

이는 'Today Lennon honored' 의 headline을 match하지만, 'today lennon honored'는 match하지 않는다는 점을 주의해라.

또한 case-insensitive 버전인 **`icontains`**가 있다.

**`startswith`**, **`endswith`**

각각 Starts-with와 ends-with search이다. 또한 case-insensitive 버전인 **`istartswith`**와 **`iendswith`**가 있다.

다시 말하지만, 이는 오직 표면만 다룬 것이다. 전체 reference는 `field lookup reference`에서 볼 수 있다.

---

## Lookups that span relationships

Django는 뒤에서 자동으로 SQL **JOIN**s를 다루면서, lookups에서 relatioships를 "follow"할 수 있는 강력하고 직관적인 방법을 제공한다. relationship을 확장하기 위해, 너가 원하는 field에 도달할 때까지, double underscore으로 분리하여, model 사이의 related fields의 이름을 사용해라.

다음 예시는 **Blog**의 **name**이 **'Beatles Blog'**인 모든 **Entry** objects를 받는다:

    >>> Entry.objects.filter(blog__name='Beatles Blog')

이런 확장은 너가 원하는 만큼 깊이 할 수 있다.

이는 또한 뒤 방향으로도 진행 가능하다. "reverse" relationship을 참조하고 싶다면, 단지 model의 소문자 이름을 사용해라.

다음 예시는 **headline**이 **'Lennon'**을 포함하고 있는 **Entry**가 적어도 하나 있는 모든 **Blog** objects를 받는다:

    >>> Blog.objects.filter(entry__headline__contains='Lennon')

만약 너가 multiple relationships를 가로질러 filtering하고 있고 중간 model이 filter 조건에 맞는 value를 갖고 있지 않다면, Django는 그것을 거기에 마치 유효하지만 비어있는(모든 values가 **NULL**) object가 있다고 다룰 것이다. 즉, error가 발생하지 않을 것이라는 의미이다. 예를 들어, 다음 filter를 보자:

    Blog.objects.filter(entry__authors__name='Lennon')

(만약, 연관된 **Author** model이 있고), 만약 한 entry와 연관된 **author**이 없다면, error를 일으키는 대신에 그것은 연관된 **name** 또한 없다고 다뤄진다. 보통 이는 너가 원하는 것일 것이다. 헷갈리는 유일한 경우는 만약 너가 **`isnull`**을 사용할 경우이다. 즉:

    Blog.objects.filter(entry__authors__name__isnull=True)

은 **author**에 비어있는 **name**을 갖고 있는 **Blog** objects 뿐만 아니라, **entry**에 비어있는 **author**을 갖고 있는 **Blog** objects 또한 반환할 것이다. 만약 너가 후자를 원하지 않는다면 이렇게 써야 한다:

    Blog.objects.filter(entry__authors__isnull=False,
    entry__authors__name__isnull=True)

---

---

# Related objects

만약 너가 model에 relationship을 정의하고 싶다면(i.e. **`ForeignKey`**, **`OneToOneField`**, 또는 **`ManyToManyField`**), model의 instances는 related object에 접근할 수 있는 편리한 API를 갖고 있다.

이 페이지의 맨 위에 있는 model들을 예시로 삼는다면, **Entry** object인 **e**는 그것과 연관된 **Blog** obejct를 blog attribute를 통해서 얻을 수 있다: **e.blog.**

(배후에서 이 기능은 Python `descriptors`에 의해 구현된다. 이 내용은 사용자에게 중요하지는 않지만 호기심을 유발하기 위해 여기에서 설명한다.)

Django는 또한 API accessors를 relationship의 다른 쪽에 생성한다 - related model에서 그 relationship을 정의한 모델로의 link이다. 예를 들어 **Blog** object **b**는 모든 related **Entry** objects의 리스트를 **entry_set** attribute를 이용하여 접근할 수 있다:**b.entry_set.all()**.

## One-to-many relationships

**Forward**

만약 model이 **`ForeignKey`**를 갖고 있다면, 그 model의 instances는 간단히 그 model의 attribute를 통하여 related (foreign) object에 접근할 수 있다.

예시를 보자:

    >>> e = Entry.objects.get(id=2)
    >>> e.blog # Returns the related Blog object.

너는 foreign-key attribute를 통해 가져오고 설정할 수 있다. 너가 예상한 대로, foreign key의 변화는 **`save()`**를 호출하기 전까지 너의 database에 저장되지 않는다. 예시를 보자:

    >>> e = Entry.objects.get(id=2)
    >>> e.blog = some_blog
    >>> e.save()

만약 **`ForeignKey`** field가 **null=True** 설정을 갖고 있다면(i.e. 그것이 **NULL** 값을 허락한다면), 너는 **None**을 설정하여 relation을 제거할 수 있다. 예시이다:

    >>> e = Entry.objects.get(id=2)
    >>> e.blog = None
    >>> e.save() # "UPDATE blog_entry SET blog_id = NULL...;"

one-to-many relationships에서 forward access는 related object에 처음 접근할 때 cached된다. 같은 object instance에 있는 foreign key로의 후속 접근들은 cached된다. 예시이다:

    >>> e = Entry.objects.get(id=2)
    >>> print(e.blog) # Hits the database to retrieve the associated Blog.
    >>> print(e.blog) # Doesn't hit the database; use cached version.

`**select_related() QuerySet**` method 모든 one-to-many relationships의 cache를 미리 recursive하게 미리 채운다. 예시이다:

    >>> e = Entry.objects.selected_related().get(id=2)
    >>> print(e.blog) # Doesn't hit the database; uses cached version.
    >>> print(e.blog) # Doesn't hit the database; uses cached version.

---

**Following relationships "backward"**

만약 model이 **`ForeignKey`**를 갖고 있다면, foreign-key model의 instances는 첫 번째 model(ForeignKey를 갖고 있는 앞의 model)의 모든 instances를 반환하는 **`Manager`**에 접근할 수 있다. 기본적으로, 이 **`Manager`**의 이름은 **F00_set**인데, **F00**는 source model의 소문자 이름이다. 이 **`Manager`**는 **QuerySets**를 반환하는데, 이것은 위의 "Retrieving objects" 부문에 기술되어 있는 대로 filtered 되고 manipulated 될 수 있다.

다음은 예시이다:

    >>> b = Blog.objects.get(id=1)
    >>> b.entry_set.all() # Returns all Entry objects related to Blog.
    
    # b.entry_set is a Manager that returns QuerySets.
    >>> b.entry_set.filter(headline__conatains='Lennon')
    >>> b.entry_set.count()

**F00_set** 이름을 **`ForeignKey`** 정의에 있는 **`related_name`** parameter를 설정함으로써 override할 수 있다. 예를 들어, 만약 **Entry** model이 **blog = ForeignKey(Blog, on_delete=models.CASCADE, related_name='entries')**로 바뀌었을 경우, 위의 코드는 다음과 같이 바뀐다:

    >>> b = Blog.objects.get(id=1)
    >>> b.entries.all() # Returns all Entry objects related to Blog.
    
    # b.entries is a Manager that returns QuerySets.
    >>> b.entries.filter(headline__contains='Lennon')
    >>> b.entries.count()

---

**Using a custom reverse manager**

기본적으로 reverse relations로 사용되는 **`RelatedManager`**는 해당 model의 `default manager`의 하위 class이다. 만약 너가 주어진 query에 대하여 다른 manager를 사용하고 싶다면 너는 다음과 같은 syntax를 사용하면 된다:

    from django.db import models
    
    class Entry(models.Model):
    	# ...
    	objects = models.Manager() # Default Manager
    	entries = EntryManager()   # Custom Manager
    
    b = Blog.objects.get(id=1)
    b.entry_set(manager='entries').all()

만약 **EntryManager**의 해당 **get_queryset()** method에서 기본 filtering을 수행하면 해당 filtering이 **all()** 호출에 적용된다. 

물론, custom reverse manager를 명시하는 것은 너가 그것의 custom method의 사용도 가능하게 한다.

    b.entry_set(manager='entries').is_published()

---
