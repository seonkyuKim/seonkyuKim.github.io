---
title: "Django 2.1 reference 번역: Introduction to models 2"
categories: reference
tags:
- django 2.1
- reference
toc: true
---
>들어가며: 본 글은 장고 공식 문서 중 Introduction to models의 뒷부분의 번역본입니다. 이전 글은 [Introduction-to-models-1](https://seonkyukim.github.io/reference/Introduction-to-models-1)을 보십시오.
>원본 링크: [https://docs.djangoproject.com/en/2.1/topics/db/models/](https://docs.djangoproject.com/en/2.1/topics/db/models/#model-inheritance)

# Model inheritance

Django에서의 model inheritance는 Python에서 일반적인 class inheritance와 거의 동일하게 작동하지만, [Introduction-to-models-1](https://seonkyukim.github.io/reference/Introduction-to-models-1)의 시작 부분 기초 사항들은 따라야 합니다. 즉, base class 역시 **`django.db.models.Model`**의 하위 클래스여야 합니다.

당신은 부모 모델이 (자신의 데이터베이스 테이블을 갖고 있는) 고유의 모델이 될 것인지, 아니면 부모 모델이 단지 자식 class들을 통해서만 볼 수 있는 공통된 정보들을 담고 있을 것인지만 결정하면 됩니다.

다음은 Django에서 가능한 세 가지 유형의 inheritance입니다.

1. 종종 부모 클래스가 각 하위 모델에 입력하고 싶지 않은 정보들을 보유하기를 원할 것입니다. 이 class는 따로 분리하여 사용하지 않으므로 `Abstract base classes` 를 보시면 됩니다.
2. (아마 완전히 다른 application으로 부터) 이미 존재하는 모델을 상속 받기 원하고 각각의 model이 그들의 database table을 갖고 있기를 원하는 경우, `Multi-table inheritnace`를 살펴보십시오.
3. 마지막으로 model field를 수정하지 않고 단지 model의 Python-level behavior를 수정하기 원하는 경우, `Proxy models`를 사용할 수 있다.

## Abstract base classes

abstract base classes는 다수의 다른 모델들에게 공통된 정보들을 담고 싶을 때 유용합니다. **base class**를 작성하고 `Meta` class에 **abstract=True**라고 설정하면 됩니다. 그러면 이 모델은 데이터베이스를 만드는데 사용되지 않습니다. 대신에 이것이 다른 모델들을 위한 base class로 사용될 때, 이것의 field들이 child class의 필드에 추가됩니다.

다음 예시를 보십시오 :

```python
from django.db import models
    
class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()
    
    class Meta:
            abstract= True
    
class Student(CommonInfo):
    home_group = models.CharField(max_length=5)
```

**Student** 모델은 세 가지 필드를 갖게 될 것입니다: **name**, **age**, 그리고 **home_group**입니다. **CommonInfo** 모델은 abstract base class이기 때문에 보통의 장고 모델처럼 사용될 수 없습니다.  이 모델은 데이터베이스 테이블을 생성하지 않고 manager를 갖고 있지 않으며 instantiated 될 수 없고 직접적으로 저장 될 수 없습니다.

abstract base classes로부터 상속된 필드들은 또 다른 필드나 값으로 override 할 수 있고 **None**으로 제거 할 수 있습니다.

많은 용도로 이런 종류의 model inheritance 원하실 것입니다. 이것은 Python level에서 공통 정보를 분석하는 방법을 제공하면서 동시에 database level에서는 자식 모델 당 하나의 데이터베이스 테이블을 만듭니다. 

### Meta inheritance

abstract base class가 생성될 때, Django는 base class에서 선언 한 `Meta` inner class를 attribute로 사용할 수 있게 합니다. 만약 자식 class가 그것 고유의 `Meta` class를 선언하지 않았을 경우, 그것은 부모의 `Meta`를 상속받습니다. 만약 자식이 부모의 Meta class를 extend하고 싶다면, 상속받으면 됩니다. 예시를 보십시오:

```python
from django.db import models
    
class CommonInfo(models.Model):
    # ...
    class Meta:
        abstract = True
        ordering = ['name']
    
class Student(CommonInfo):
    # ...
    class Meta(CommonInfo.Meta):
        db_table = 'student_info'
```

Django는 abstract base class의 `Meta` class에 단 하나의 값을 조정합니다: `Meta` attribute를 installing하기 전에, **abstract=False**로 설정합니다. 이것은 abstract base classes의 자식들이 자동적으로 abstract class가 되지 않는다는 것을 의미합니다. 물론, 다른 abstract base class로부터 상속받는 또 다른 abstract base class를 만들 수 있습니다. 매번 명시적으로 **abstract=True**라고 설정해야 한다는 것을 기억하십시오.

몇몇 attribute들은 abstract base class의 `Meta` class 안에 있기에 부적합합니다. 예를 들어 **db_table**가  `Meta` class 안에 있을 경우, (그들 고유의 `Meta`를 명시하지 않은) 모든 자식 class들이 모두 같은 데치터베이스 테이블을 이용한다는 것인데, 이것은 원하는 작업 아닐 것입니다.



## Be careful with related_name and related_query_name

**ForeignKey** 또는 **ManyToManyField**에서 **related_name** 또는 **related_query_name**을 사용할 경우, 항상 필드에 대한 *unique* reverse name과 query name을 반드시 명시해 주어야 합니다. 이것은 보통 abstract base class에서 문제를 유발합니다. 왜냐하면 이 class에 있는 필드들이 각각의 자식 클래스에 포함되는데, 매번 같은 값의 (**related_name**과 **related_query_name**을 포함하는) attribute를 갖게 되기 때문입니다..

abstract base class에서 **related_name** 또는 **related_query_name**을 사용할 때 이 문제를 해결하기 위해서는, value의 일부분이 **%(app_label)s**와 **%(class)s**를 포함하고 있어야 합니다. 

- **%(class)s**는 필드가 사용되는 자식 class의 소문자 이름으로 대체됩니다.
- **%(app_label)s**는 자식 class가 포함되어 있는 app의 소문자 이름으로 대체됩니다. 각각의 installed applicatoin은 유일하고 app 안에 있는 model class 이름도 유일하기 때문에, 최종 이름은 서로 다르게 될 것입니다.

예를 들어 **comon/models.py**에 다음과 같은 app이 있습니다:

```python
from django.db import models
    
class Base(models.Model):
    m2m = models.ManyToManyField(
        OtherModel,
        related_name="%(app_label)s_%(class)s_related",
        related_query_name="%(app_label)s_%(class)ss",
    )
    	
    class Meta:
        abstract = True
    
class ChildA(Base):
    pass
    
class ChildB(Base):
    pass
```

**rare/models.py**에도 다음 app이 있습니다:

```python
from common.models import Base
    
class ChildB(Base):
    pass
```

**common.ChildA.m2m** 필드의 reverse name은 **common_childa_related**가 될 것이고 reverse query name은 **common_childas**가 될 것입니다. **common.childB.m2m** 필드의 reverse name은 **common_childb_related**가 될 것이고 reverse query name은 **common_chlidbs**가 될 것입니다. 마지막으로 **rare.ChildB.m2m** 필드의 reverse name은 **rare_childb_related**가 될 것이고 reverse query name은 **rare_childbs**가 될 것이다. related name이나 related query name을 구성하기 위해 **%(class)s**와 **%(app_label)s** 부분들을 어떻게 사용하는지는 당신에게 달려있지만, 이것을 사용하지 않을 경우, Django는 system check(또는 **migrate**)를 사용할 때 error를 일으킬 것입니다.

abstract base class의 필드에 대한 **related_name** attribute를 명시하지 않은 경우, default reverse name은 child class 이름 뒤에 **_set**이 붙은 것이 될 것이고, child class에 직접적으로 선언한 것처럼 정상적으로 작동할 것입니다. 예를 들어 위의 코드에서, **related_name** attribute가 생략 되었다면, **m2m** 필드의 reverse name은 **ChildA**의 경우 **childa_set**이 될 것이고 **ChildB**의 경우 **childb_set**이 될 것입니다.



## Multi-table inheritance

Django에서 지원하는 두 번째 타입의 model inheritance는 계층 구조의 각 모델이 모두 그것 고유의 모델일 때입니다. 각각의 모델은 그들 고유의 데이터베이스 테이블에 대응되고 독립적으로 queried 되고 생성될 수 있습니다. inheritance relationship은 자식 모델과 그들의 부모 사이의 links를 (자동적으로 생성되는 **OneToOneField**를 통해서) 줍니다. 예시를 보십시오:

 ```python
from django.db import models
    
class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)
    
class Restaurant(Place):
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)
```

데이터들은 서로 다른 데이터베이스 테이블에 있음에도 불구하고 **Place**의 모든 필드들은 **Restaurant**에서도 모두 이용 가능합니다. 따라서 다음 두 가지가 모두 가능합니다:

```shell
>>> Place.objects.filter(name="Bob's Cafe")
>>> Restaurant.objects.filter(name="Bob's Cafe")
```

**Restaurant** 이기도 한 **Place**를 갖고 있을 경우, **Place** 객체로부터 소문자 모델 이름을 이용하여 **Restaurant** 객체를 얻을 수 있습니다:

```shell
>>> p = Place.objects.get(id=12)
# If p is a Restaurant object, this will give the child class:
>>> p.restaurant
<Restaurant: ...>
```

그러나 위 예시에서의 **p**가 **Restaurant**이 아닐 경우(그것이 **Place** 객체로부터 직접적으로 생성되었거나 다른 class의 부모였다면), **p.restaurant**를 호출하는 것은 **Restaurant.DoesNotExist** exception을 일으킬 것입니다.

**Restaurant**에 자동적으로 생성된 **Place**로 link해주는 **OneToOneField**는 다음과 같이 생겼습니다:

```python
place_ptr = models.OneToOneField(
    Place, on_delete=models.CASCADE,
    parent_link=True,
)
```

해당 필드를 **Restaurant**에 **parent_link=True** 와 함께 고유의 **OneToOneField**를 선언함으로써 override할 수 있습니다.

### Meta and multi-table inheritance

multi-table inheritance의 경우에, 자식 class에서 부모의 `Meta` class를 상속받는 것은 말이 되지 않습니다. 모든 `Meta` option들은 이미 부모 class에 적용되었고, 그들을 다시 적용하는 것은 보통 모순적인 행동입니다(abstract base class와는 반대인데, base class는 그들 스스로는 존재하지 않습니다).

따라서 자식 model은 그들 부모의 `Meta` class에 접근할 수 없다. 그러나 부모로부터 몇 가지 행동을 상속받는 경우가 있다: 만약 자식이 **`ordering`** attribute 또는 **`get_latest_by`** attribute를 명시하지 않는다면, 자식은 부모로부터 이들을 상속받을 것이다.

만약 부모가 ordering을 갖고 있고 너는 자식이 아무런 ordering을 갖지 않기를 원하면, 명시적으로 사용을 중단 할 수 있다:

    class ChildModel(ParentModel):
    	# ...
    	class Meta:
    		# Remove parent's ordering effect
    		ordering = []

---

**Inheritance and reverse relations**

multi-table inheritance가 자식과 부모를 link하기 위해 암묵적인 **`OneToOneField`**를 사용했기 때문에, 위의 예시처럼 부모에서 자식으로 내려가는 것이 가능하다. 하지만, 이는 **`ForeignKey`** 와 **`ManyToManyField`**의 default **`related_name`** 값을 사용해 나간다. 만약 너가 부모의 자식 class에 이런 종류의 relations를 둔다면, 너는 반드시 각각의 field에 **`related_name`** attribute를 명시해야 한다. 만약 그렇지 않으면, Django는 validation error를 일으킬 것이다.

예를 들어, 위의 **Place** class 를 다시 사용하여 **`ManyToManyField`**를 이용한 다른 자식 class를 만들자:

    class Supplier(Place):
    	customers = models.ManyToManyField(Place)

다음은 error 결과이다:

    Reverse query name for 'Supplier.customers'
    clashes with reverse query
    name for 'Supplier.place_ptr'.
    
    HINT: Add or Change a related_name argument to the fefinition for
    'Supplier.customers' or 'Supplier.place_ptr'.

다음과 같이 **customers** field에 **related_name**을 추가 해주는 것은 error를 해결할 수 있다: **models.ManyToManyField(Place, related_name='provider')**.

---

**Specifying the parent link field**

위에서 말했듯이, Django는 너의 자식 class와 그 어떤 non-abstract parent model과 연결해주는 **`OneToOneField`**를 자동적으로 생성할 것이다. 만약 다시 부모로 연결해주는 attribute 이름을 제어하고 싶은 경우, 너는 너만의 **`OneToOneField`**를 생성하고 너의 field가 부모 class를 다시 가리키도록 **`parent_link=True`**라고 설정하면 된다.

---

## Proxy models

`multi-table inheritance`를 사용할 때, 새로운 database table은 한 model의 각각의 하위 class에 대해 생긴다. 이는 보통 이상적인 행동인데, 이는 하위 class가 base class에 존재하지 않는 추가적인 data field를 저장할 공간을 필요로 하기 때문이다. 하지만 때때로, 너는 단지 한 model의 Python 동작을 바꾸고 싶을 것이다 - 아마 default manager를 바꾸거나 새로운 method를 추가할 것이다.

이는 proxy model inheritance을 위한 것이다: 원래 model에 *proxy*를 생성하는 것이다. 너는 proxy model의 instances를 생성하고 삭제하고 업데이트 할 수 있으며, 모든 data들은 너가 원래의 (non-proxied) model을 사용하는 것과 같이 저장된다. 차이점은 원래의 것을 변경하지 않고, proxy에서 default model ordering 또는 default manager과 같은 것들을 바꿀 수 있다는 점이다.

Proxy model들은 보통의 model들과 같이 선언될 수 있다. 너는 Django에게 **Meta** class의 **`proxy`** attribute을 **True**로 설정함으로써 알려줄 수 있다.

예를 들어, Person model에 method를 추가하고 싶다고 가정하자. 너는 다음과 같이 할 수 있다:

    from django.db import models
    
    class Person(models.Model):
    	first_name = models.CharField(max_length=30)
    	last_name = models.CharField(max_length=30)
    
    class MyPerson(Person):
    	class Meta:
    		proxy = True
    
    	def do_something(self):
    		# ...
    		pass

**MyPerson** class는 그것의 부모 **Person** class와 같은 database table에서 작동한다. 특히, 모든 새로운 Person instance들 또한 MyPerson을 통해 접근 가능하고, 반대의 경우도 가능하다:

    >>> p = Person.objects.create(first_name="foobar")
    >>> MyPerson.objects.get(first_name="foobar")
    <MyPerson: foobar>

너는 또한 model에서 다른 default ordering을 정의하기 위해 proxy model을 사용할 수 있다. 너는 아마 항상 **Person** model을 정렬하고 싶지 않을 수 있지만, proxy를 사용할 때 규칙적으로 **last_name** attribute를 이용하여 정렬할 수 있다. 이것은 쉽다:

    class OrderedPerson(Person):
    	class Meta:
    		ordering = ["last_name"]
    		proxy = True

이제 보통의 **Person** queries는 정렬되지 않지만, **OrderedPerson** queries는 **last_name**에 의해 정렬될 것이다.

Proxy model은 `보통의 model들과 같은 방법으로` **Meta** attribute를 상속 받는다.

**QuerySets still return the model that was requested**

말하자면, 너가 **Person** objects를 query 할 때마다 Django가 **MyPerson** object를 반환하게 하는 방법은 없다. **Person** objects의 queryset은 그 타입의 objects를 반환할 것이다. proxy objects의 요점은 원래의 Person에 의존적인 코드는 proxy objects를 사용할 것이라는 점과 너 고유의 코드는 너가 추가한 extension들을 사용할 수 있다는 점이다(다른 코드는 그다지 의존적이지 않다). 이는 Person (또는 어떠한 다른) model을 너가 만든 다른 것으로 대체할 수 있는 것이 아니다.

---

**Base class restrictions**

proxy model은 반드시 하나의non-abstract model class로
 부터 상속을 받아야 한다. 너는 여러 개의 non-abstract models로부터 상속받을 수 없는데, 이는 proxy model은 다른 database table에 있는 rows들 간의 어떤 연결도 제공하지 않기 때문이다. abstract model classes이 어떠한 model fields도 정의하지 않았다면, proxy model은 많은 수의 abstract model classes을 상속할 수 있다. 한 proxy model은 공통의 한 non-abstract 부모 class를 공유하고 있는 여러 proxy models로부터 상속 받을 수 있다.

---

**Proxy model managers**

만약 너가 한 proxy model에 어떠한 model manager도 명시하지 않는다면, 그것은 그것의 model parents로부터 managers를 상속한다. 만약 너가 proxy model에 manager를 정의한다면, 그것은 default가 될 것이지만,  부모 classes에 정의된 어떠한 manager도 여전히 사용 가능하다.

위의 예시를 이어 가자면, 너는 다음과 같이 **Person** model을 query할 때 default manager를 바꿀 수 있다:

    from django.db import models
    
    class NewManager(models.Manager):
    	# ...
    	pass
    
    class MyPerson(Person):
    	objects = NewManager()
    
    	class Meta:
    		proxy = True

만약 proxy에 이미 존재하는 default를 대체하지 않고 새로운 manager를 추가하고 싶다면, 너는 `custom manager` 문서에 기술된 방법들을 사용할 수 있다: 새로운 manager를 갖고 있는 base class를 생성한 뒤에 primary base class 이후에 상속 받아라.

    # Create an abstract class for the new manager.
    class ExtraManagers(models.Model):
    	secondary = NewManager()
    	
    	class Meta:
    		abstract = True
    
    class MyPerson(Person, ExtraManagers):
    	class Meta:
    		proxy = True

너는 이를 자주 필요로 하지는 않을 것이지만, 너가 하고 싶을 때 이는 가능하다.

---

**Differences between proxy inheritance and unmanaged models**

proxy model inheritance는 model의 **Meta** class에 **`managed`** attribute를 사용하여 unmanaged model을 생성하는 것과 비슷하게 보인다.

조심스럽게 **`Meta.db_table`**를 설정하여 너는 기존의 model을 가리고 Python method를 거기에 추가할 수 있는 unmanaged model을 생성할 수 있다. 그러나 변경 작업을 수행하면 두 복사본을 동기화 된 상태로 유지해야 하므로 반복적이고 오류가 발생할 수 있다.

다른 한편으로는 proxy model은 proxing 하고 있는 모델과 똑같이 행동하도록 만들어졌다. 그들은 부모 model과 항상 동기화 되어 있는데 그들이 직접적으로 그들의 field와 manager를 상속 받았기 때문이다.

일반적인 규칙은 다음과 같다:

1. 만약 기존의 model과 database table을 mirroring하고 기존의 모든 database table column들을 원하지 않는다면, Meta.managed=False를 써라. 이 option은 Django가 제어하지 않는 database view와 table을 modeling 할 때 유용하다.
2. 만약 너가 model의 Python-only 동작을 변경하고 싶지만, 기존 model의 field를 유지하고 싶다면, Meta.proxy=True를 사용해라. 이는 data가 저장될 때 proxy model의 저장소 구조와 정확히 일치하도록 설정된다.

---

## Multiple inheritance

Python의 하위 class와 마찬가지로, Django model이 여러 개의 부모 model로부터 상속 받는 것이 가능하다. 일반적인 Python name resolution rule이 적용된다는 것을 명심해라. 특정 이름(e.g. `Meta`)가 나타나는 첫 base class가 사용될 것이다. 예를 들어, 이것은 만약 여러 부모 class가 `Meta` class를 포함하고 있다면, 첫 번째 것이 사용될 것이며 다른 것들은 모두 무시될 것이다.

일반적으로, 너는 다수의 부모로부터 상속 받을 필요가 없다. 일반적으로 유용하게 사용되는 경우는 "mix-in" class를 사용할 때이다: mix-in을 상속하고 있는 모든 class에 특정 추가적 field와 method를 추가하는 것이다. 너의 inheritance hierarchies를 가능한 간단하고 직관적으로 유지하여 특정 정보가 어디로부터 왔는지 헷갈리지 않게 해라.

공통된 **id** primary key field를 갖고 있는 다양한 model로부터 상속을 받는 것은 error를 일으킬 것이다. 올바르게 multiple inheritance를 사용하기 위해, 너는 base model에 명시적으로 **`AutoField`**를 쓸 수 있다:

    class Article(models.Model):
    	article_id = models.AutoField(primary_key=True)
    	...
    
    class Book(models.Model):
    	book_id = models.AutoField(primary_key=True)
    	...
    
    class BookReview(Book, Article):
    	pass

또는 **`AutoField`**를 갖고 있는 공통의 ancestor를 사용해라. 이는 각각의 부모 model에서 공통의 ancestor로의 OneToOneField를 명시적으로 사용하는 것을 요구하는데, 자동으로 생성되고 자식으로부터 상속받는 field들 간의 충돌을 피하기 위해서이다:

    class Piece(models.Model):
    	pass
    
    class Article(Piece):
    	article_piece = models.OneToOneField(Piece, on_delete=models.CASCADE, parent_link=True)
    	...
    
    class Book(Piece):
    	book_piece = models.OneToOneField(Piece, on_delete=models.CASCADE, parent_link=True)
    	...
    
    class BookReivew(Book, Article):
    	pass

---

## Field name "hiding" is not permitted

보통의 Python inheritance에서, 자식 class는 부모 class로부터 어떠한 attribute를 override하는 것이 허용된다. Django에서 이것은 model fields에 대하여 주로 허락되지 않는다. 만약 non-abstract class가 **author**이라 된 field를 갖고 있다면, 너는 그 base class로부터 상속 받은 다른 class에 **author**이라 불리는 다른 model field나 attribute를 생성할 수 없다.

이 제약은 abstract model로부터 상속 받은 model field에는 적용되지 않는다. 이러한 field들을 다른 field나 값으로 override 할 수 있고, **field_nam = None**으로 설정하여 제거 할 수 있다.

- **Warning**

    model manager는 abstract base class로 부터 상속된다. 상속된 **`Manager`**에 의해 참조되는 상속된 field를 override하는 것은 미묘한 bug를 일으킬 수 있다. `custom managers and model inheritance`를 보아라.

- **Note**

    몇몇의 fields들은 model에 추가적인 attribute를 정의한다. 예를 들어 **`ForeignKey`**는 field 이름에 **_id**가 추가된 별도의 attribute를 정의할 뿐만 아니라 **related_name** 과 **relate_query_name** 을 foreign model에 정의한다. 

    이런 추가적인 attributes들은 그것을 정의한 field가 바뀌거나 제거되어 더 이상 추가적인 attribute를 정의하지 않는 이상 override할 수 없다.

부모 model의 field를 override하는 것은 (**Model.__init__**에서 어떤 field가 initialized 될지를 명시하는) 새로운 instance를 initializing 하는 것과 serialization과 같은 영역에 어려움이 있다. 이것들은 Python class inheritance가 똑같은 방식으로 처리 할 필요가 없는 기능이므로, Django model inheritance와 Python class inheritance는 임의적이지 않다.

이런 제약은 단지 **`Field`** instances인 attribute에만 적용된다. 보통의 Python attribute는 너가 원한다면 Override할 수 있다. 또한 이것은 Python 이 인식하는 attribute 이름에만 적용된다: 만약 너가 직접 database column 이름을 설정한다면, 너는 mutli-table inheritance의 child와 ancestor model에 나타나는 같은 column name을 가질 수 있다(이들은 서로 다른 database table의 column들이다).

Django는 만약 너가 ancestor model에 있는 어떠한 model field를 override하면 **`FieldError`**를 일으킬 것이다.

---

# Organizing models in a package

**`manage.py startapp`** command는 **models.py** 파일을 갖고 있는 application 구조를 생성한다. 만약 너가 많은 models을 갖고 있다면, 그들을 다른 파일에 구성하는 것이 효과적일 것이다.

이렇게 하기 위해서는 **models** package를 만들어라. **models.py**를 삭제하고 **myapp/models/** directory를 **__init__.py** 파일과 너의 models를 저장할 파일들과 함께 만들어라. 너는 model들을 **__init__.py** 파일에 import 해야 한다.

예를 들어, 만약 너가 models directory에 **organic.py**와 **synthetic.py**를 갖고 있다면 다음과 같다:

myapp/models/__init__.py

    from .organic import Person
    from .synthetic import Robot

from .models import *를 사용하는 것보다 명시적으로 각각의 model을 import하는 것이 namespace를 어질러놓지 않을 수 있고, 코드를 더욱 읽기 쉽게 만들어주며, code analysis tools를 효과적으로 만들어준다.

- **See also**

    `The Models Reference`

    model fields, related objects, 그리고 **QuerySet**을 포함해서 모든 model 관련 API를 다루고 있다.
