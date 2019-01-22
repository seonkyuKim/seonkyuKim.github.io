---
title: "Introduction to models"
categories:
- django
tags:
- django reference
---

# Introduction to models

# Models

model은 너의 데이터에 대한 하나의, 결정적인 정보이다. 그것은 필수적인 field들과 너가 저장하고 있는 데이터들의 행동을 결정한다. 일반적으로, 각각의 model은 하나의 database table로 mapping된다.

기초 사항:

- 각각의 model은 **`django.db.models.Model`**의 하위 python 클래스이다.
- model 클래스의 각각의 attribute는 database field를 나타낸다.
- 이것들과 함께, Django는 너에게 자동으로 만들어진 database-access API를 제공한다; [Making queries](https://www.notion.so/Making-queries-d1ee592133a24ef48fa9394d3276dedd) 참고하라.

# Quick example

다음 예시는 **first_name**과 **last_name**을 갖고 있는 **Person** model이다:

    from django.db import models
    
    class Person(models.Model):
    	first_name = models.CharField(max_length=30)
    	last_name = models.CharField(max_length=30)

**first_name**과 **last_name**은 model의 `field`이다. 각각의 field는 class attribute로 구체화되며, 각각의 attribute는 하나의 database column으로 mapping된다.

위의 **Person** model은 다음과 같은 database 테이블을 생성한다:

    CREATE TABLE myapp_person (
    	"id" serial NOT NULL PRIMARY KEY,
    	"first_name" varchar(30) NOT NULL,
    	"last_name" varchar(30) NOT NULL
    );

Some technical notes:

- 테이블의 이름인 **myapp_person**은 몇 개의 model metadata로부터 자동적으로 파생되지만, override할 수 있다. 자세한 사항은 `Table names`를 참조하라.
- **id** field는 자동적으로 추가되지만 이 역시 override할 수 있다. `Automatic primary key fields`를 참조하라.
- 위 예시의 **CREATE TABLE**은 PostgreSQL 문법으로 쓰여 있는데, Django가 너의 `settings file`에 명시된 database backend 맞춤 SQL 을 사용한다는 것을 알고 있어라.

---

# Using models

너가 너의 models를 정의한 후, 너는 Django에게 앞으로 그 models를 사용할 것이라고 알려야 한다. 이를 위해 너의 settings file을 수정해야 하는데, 파일 안의 **`INSTALLED_APPS`**에 너의 **models.py**을 포함하고 있는 module이름을 추가해라.

예를 들어, 만약 너의 application의 models가 **myapp.models** module 안에 있다면 (application을 위해 **`manage.py startapp`** 명령어를 사용하여 만든 package structure 안에 있다면), **`INSTALLED_APPS`**는 다음과 같다:

    INSTALLED_APPS = [
    	#...
    	'myapp',
    	#...
    ]

너가 새로운 app들을 **`INSTALLED_APPS`**에 추가할 때, **`manage.py makemigrations`**를 이용하여 부분적으로 migrations를 만든 후, **`manage.py migrate`**를 꼭 실행하도록 하라

---

# Fields

model에서 가장 중요한 부분이다 - 그리고 model에서 요구되는 유일한 부분이다 - 그것은 바로 정의된 database field 들의 리스트를 아는 것이다. Fields는 class attirbute에 의해 명시된다. **clean, save** 또는 **delete**와 같은 `models API`와 field 이름이 충돌하지 않도록 주의하라

예시 :

    from django.db import models
    
    class Musician(models.Model):
    	first_name = models.CharField(max_length=50)
    	last_name = models.CharField(max_length=50)
    	instrument = models.CharField(max_length=100)
    
    class Album(models.Model):
    	artist = models.ForeignKey(Musician, on_delete=models.CASCADE)
    	name = models.CharField(max_length=100)
    	release_date = models.DateField()
    	num_starts = models.IntegerField()

## Field types

너의 model 안의 각각의 field는 적당한 **`Field`** class안의 instance이어야 한다. Django는 몇 가지를 결정하기 위해 field class types를 사용한다:

- Column type은 어떤 종류의 database를 저장할지 알려 준다.(e.g.**INTEGER, VARCHAR, TEXT**)
- Default HTML `widget`은 form field들을 rendering할 때 사용된다.(e.g. **<input type="text"><select>**)
- Minimal validation requirements는 Django의 admin과 자동으로 생성된 form에서 사용된다.

Django는 수십 개의 내장 field-type들을 갖고 있다; 너는 전체 목록을 model field reference에서 확인할 수 있다. 만약 Django의 내장 field가 제 기능을 못 할 경우, 너는 너만의 field를 쉽게 작성할 수 있다; `Writing custom model fields`를 참조해라.

---

## Field options

각각의 field는 field-specific arguments를 갖고 있다(`model field reference`에 작성되어 있다). 예를 들어, **`CharField`**(그리고 그것의 subclasses)는 데이터를 저장하는데 사용되는 **VARCHAR** database field의 사이즈를 명시해주는 **`max_length`** 인자를 필요로 한다.

또한 모든 field type에 적용되는 공통 argument들이 있다. 모두 optional이다. 이들은 `reference`에서 전부 설명되어 있지만, 여기에서는 가장 많이 사용하는 몇 가지만 요약을 하겠다.

**`null`**

만약 **True**라면, Django는 database에 빈 값인 **NULL**값을 저장한다. 초기 값은 **False**다.

**`blank`**

만약 True라면, field는 빈 칸이 있을 수 있다. 초기 값은 False이다.

이는 null과 다르다. null은 database-related인 반면, blank는 validation-related이다. 만약 field에서 blank=True라면, form validation은 entry의 비어있는 값을 허락한다. 만약 field에서 blank=False라면, 그 field의 값은 항상 요구된다.

**`choices`**

2-tuples로 구성된 iteralbe(e.g. list or tuple)은 이 field의 choices로 사용할 수 있다. 만약 이것이 주어지면, default form widget은 기본적인 text field가 아니라 select box가 될 것이고 주어진 choices로 선택권이 제한될 것이다.

choices list는 다음과 같다:

    YEAR_IN_SCHOOL_CHOICES = (
    	('FR', 'Freshman'),
    	('SO', 'Sophomore'),
    	('JR', 'Junior'),
    	('SR', 'Senior'),
    	('GR', 'Graduate'),
    )

각각의 tuple의 첫 번째 요소는 database에 저장되는 값이다. 두 번째 요소는 field의 form widget에 나타나는 값(display value)이다.

model instance가 주어졌을 때, **choices**를 갖고 있는 field에서 display value는 **`get_F00_display()`** method를 이용하여 접근할 수 있다. 예시를 보자:

    from django.db import models
    
    class Person(models.Model):
    	SHIRT_SIZES = (
    		('S', 'Small'),
    		('M', 'Medium'),
    		('L', 'Large'),
    	)
    	name = models.CharField(max_length=60)
    	shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)

    >>> p = Person(name="Fred Flintsone", shirt_size='L')
    >>> p.save()
    >>> p.shirt_size
    'L'
    >>> p.get_shirt_size_display()
    'Large'

**`default`**

field의 초기 값이다. 이는 한 값이나 callable object가 될 수 있다. 만약 callable하다면 이것은 새로운 object가 생성될 때마다 실행될 것이다.

**`help_text`**

form widget에 표시될 추가적인 "help" text이다. 이는 field가 form에서 사용되지 않더라도 documentation하는데 유용하다.

**`primary_key`**

만약 **True**라면, 이 field는 model의 primary key가 된다.

만약 너가 model에서 어떤 field에도 **`primary_key=True`**로 설정해 두지 않았다면, Django는 자동적으로 primary key로 사용할 `**IntegerField**`를 추가하기 때문에 너가 default primary-key를 override하고 싶지 않은 이상 어떤 field에 **`primary_key=True`**를 설정할 필요없다. `Automatic primary key fields`에 더 자세히 기술되어 있다.

primary key field는 읽을 수만 있다. 만약 너가 기존 object의 primary key 값을 변경한다면, 기존 object옆에 새로운 object가 생성될 것이다. 예시를 보자:

    from django.db import models
    
    class Fruit(models.Model):
    	name = models.CharField(max_length=100, primary_key=True)

    >>> fruit = Fruit.objects.create(name='Apple')
    >>> fruit.name = 'Pear'
    >>> fruit.save()
    >>> Fruit.objects.values_list('name', flat=True)
    <QuerySet ['Apple', 'Pear']>

**`unique`**

만약 True라면, 이 field는 table 전체에서 유일해야 한다.

다시 한 번 말하지만, 이들은 모두 공통된 field option들에 대한 짧은 설명이다. 자세한 내용은 `common model field option reference`에 기술되어 있다.

---

## Automatic primary key fields

초기 값으로, Django는 각각의 model에 다음과 같은 field를 생성한다:

    id = models.AutoField(primary_key=True)

이는 auto-incrementing primary key이다.

만약 너가 custom primary key를 명시하고 싶다면, 너의 field들 중 하나에 **`primary_key=True`**를 명시하여라. 만약 명시적으로 **`Field.primary_key`**를 설정 했다면, Django는 자동적으로 생성된 **id** column을 추가하지 않을 것이다.

각각의 model은 오직 하나의 field만이 primary_key=True를 갖고 있어야 한다(명시적으로 추가 되었든지 아니면 자동적으로 추가 되었든지).

---

## Verbose field names

**`ForeignKey`**, **`ManyToManyField`**와 **`OneToOneField`**를 제외하고, 각각의 field type들은 선택적인 첫 번째 위치의 argument들을 갖는다 - 바로 verbose name이다. 만약 verbose name이 주어지지 않았다면, Django는 자동적으로 field의 attribute name에 있는 underscores를 space로 바꾸어 verbose name을 생성한다.

이 예시에서 verbose name은 "**person's first name**"이다:

    first_name = models.CharField("person's first name", max_length=30)

이 예시에서 verbose name은 "**first name**"이다:

    first_name = models.CharField(max_length=30)

**`ForeignKey`**, **`ManyToManyField`**와 **`OneToOneField`**는 첫 번째 argument로 model class가 필요로 하기 때문에 **`verbose_name`** keyword argument를 사용한다.

    poll = models.ForeignKey(
    	Poll,
    	on_delete=models.CASCADE,
    	verbose_name="the related poll",
    )
    sites = models.ManyToManyField(Site, verbose_name="list of sites")
    place = models.OneToOneField(
    	Place,
    	on_delete=models.CASCADE,
    	verbose_name="related place",
    )

관습적으로 **`verbose_name`**의 첫 글자는 대문자를 쓰지 않는다. Django가 자동적으로 첫 번째 글자를 대문자로 바꾸어주기 때문이다.

---

## Relationships

명확하게, relational database의 강점은 table간에 relating이 되어있다는 점이다. Django는 가장 많이 사용되는 세 가지 database relationship을 정의해준다: many-to-one, many-to-many 그리고 one-to-one이다.

**Many-to-one relationships**

many-to-one relationship을 정의하기 위해, **`django.db.models.ForeignKey`**를 사용해라. 너는 이것을 다른 **`Field`** type처럼 사용하면 된다: 너의 model에 class attribute처럼 추가하여라.

**`ForeignKey`**는 지정된 위치의 argument를 요구한다: 모델과 relate된 class를 지정하는 것이다.

예를 들어, **Car** model이 **Manufacturer**을 갖고 있다면 - 즉, **Manufacturer**은 다양한 cars를 만들지만 각각의 **Car**는 단 하나의 **Manufacturer**를 갖고 있다 - 다음과 같은 정의를 써라:

    from django.db import models
    
    class Manufacturer(models.Model):
    	# ...
    	pass
    
    class Car(models.Model):
    	manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
    	# ...

너는 또한 `recursive relationship`(자기 자신에게 many-to-one relationship을 갖고 있는 object)을 만들 수 있고 `아직 정의되지 않은 model에게 relationship`을 만들 수 있다; 자세한 사항은 `model field reference`를 참고하라.

필수적인 것은 아니지만, **`ForeignKey field`**의 이름(위의 예에서는 **manufacturer**)은 소문자의 model 이름을 사용하는 것이 좋다. 물론, 너는 너가 원하는 대로 field 이름을 정해도 된다. 예시를 보자:

    class Car(models.Model):
    	company_that_makes_it = models.ForeignKey(
    		Manufacturer,
    		on_delete=models.CASCADE,
    	)

- **See also**

    **`ForeignKey`** field는 `model field references`에 설명되어 있는 다른 arguements도 사용할 수 있다. 이 option들은 relationship이 어떻게 작동할지 규정한다; 모두 optional이다. 

    backward-related objects에 접근하는 방법에 대해서는 Following relationships backward example을 참고하라.

    sample code를 보고 싶다면 Many-to-one relationship model example을 참고하라.

---

**Many-to-many relationships**

Many-to-many relationships를 정의하기 위해, **`ManyToManyField`**를 사용해라. 너는 이것을 다른 **`Field`** type처럼 사용하면 된다: 너의 model에 class attribute처럼 추가하여라.

**`ManyToManyField`**는 지정된 위치의 argument를 요구한다: relate할 class를 지정하는 것이다.

예를 들어, **Pizza** 가 다양한 **Topping** objects를 갖고 있다면 - 즉, 한 **Topping**이 다양한 pizzas에 있을 수 있고 각각의 **Pizza**는 다양한 toppings를 가질 수 있다 - 다음과 같이 이를 나타내면 된다:

    from django.db import models
    
    class Topping(models.Model):
    	# ...
    	pass
    
    class Pizza(models.Model):
    	# ...
    	toppings = models.ManyToManyField(Topping)

**`ForeignKey`**와 마찬가지로, 너는 `recursive relationship`(자기 자신에게 many-to-many relationship을 갖고 있는 object)을 만들 수 있고 `아직 정의되지 않은 model에게 relationship`을 만들 수 있다.

필수적인 것은 아니지만, **`ManyToManyField`**의 이름(위 예에서는 **toppings**)을 related model objects를 묘사하고 있는 복수형으로 하는 것이 좋다.

어떤 model이 **`ManyToManyField`**를 갖고 있는지는 중요하지 않지만, 너는 두 쪽 모두가 아니라 한 쪽에만 이것을 설정해야 한다.

일반적으로, **`ManyToManyField`** instance는 form에서 수정되어야 하는 object에 있어야 한다. 위의 예에서는 **toppings**는 **Pizza**에 있는데(**Topping**이 많은 **pizzas `ManyToManyField`**를 갖고 있는 것이 아니다) 그 이유는 pizza가 많은 topping들이 갖고 있다고 생각하는 것이 topping이 많은 pizza들 위에 있다고 생각하는 것보다 자연스럽기 때문이다. 위에서는 이와 같이 설정되어 있으며, Pizza는 사용자들이 topppings들을 선택할 수 있게 해준다.

- **See also**

    전체 예시를 보기 위해서는 `Many-to-many relationship model example`을 참조하라.

**`ManyToManyField`** field는 `model field references`에 설명되어 있는 다른 arguements도 사용할 수 있다. 이 option들은 relationship이 어떻게 작동할지 규정한다; 모두 optional이다.

---

**Extra fields on many-to-many relationships**

pizza와 topping과 같은 단순한 many-to-many relationship을 다루고 있다면, **`ManyToManyField`**가 너가 필요한 전부이다. 하지만, 때때로 너는 relationship이 있는 두 models간의 데이터들을 연결지어야 한다.

예를 들어 musicians이 속해있는 musical group을 추적하는 경우를 생각해보자. 여기에는 person과 그들이 멤버로서 속해있는 group들 간의 many-to-many relationship이 있고, 너는 이 관계를 나타내기 위해 **`ManyToManyField`**를 사용할 수 있다. 그러나, 너가 얻고 싶어하는 membership과 관련된 세부 사항들, 예를 들어 person들이 group에 들어간 날짜 등이 많이 있다. 

이러한 상황들에서, Django에서 너는 many-to-many relationship을 관리하는 model을 명시할 수 있다. 또한 너는 이 intermidate model에 추가적인 field들을 넣을 수 있다. intermediary 역할을 할 model을 알려주는  **`through`** argument를 이용하여 intermediate model은 **`ManyToManyField`**와 연결될 수 있다. 우리의 musician 예시는 다음 코드와 같이 쓸 수 있다:

    from django.db import models
    
    class Person(models.Model):
    	name = models.CharField(max_length=128)
    	
    	def __str__(self):
    		return self.name
    
    class Group(models.Model):
    	name = models.CharField(max_length=128)
    	members = models.ManyToManyField(Person, through='Membership')
    	
    	def __str__(self):
    		return self.name
    
    class Membership(models.Model):
    	person = models.ForeignKey(Person, on_delete=models.CASCADE)
    	group = models.ForeignKey(Group, on_delete=models.CASCADE)
    	date_joined = models.DateField()
    	invite_reason = models.CharField(max_length=64)

너가 intermediary model을 만들 때, 너는 many-to-many relationship에 포함될 model들에게 foreign key를 명시적으로 지정해주어야 한다. 이 명시적 선언은 두 model들이 어떻게 relate되는지 정의한다.

다음은 intermediate model에서 몇 가지 제약 사항이다:

- 너의 intermediate model은 source model(우리의 예에서는 **Group**)로 향하는 foreign key는 단 하나만 갖고 있어야 한거나, **`ManyToManyField.through_fields`**를 사용하여 Django가 relationship을 위해 사용해야 하는 foreign key를 명시적으로 지정해야 한다. 만약 한 개 이상의 foreign key를 갖고 있거나 through_fields가 명시되어 있지 않으면 validation error가 발생할 것이다. 비슷한 제약은 target model(우리의 예에서는 **Person**)에서도 적용된다.
- intermediary model을 이용하여 자기 자신에게 many-to-many relationship을 갖고 있는 model에 대해서는, 같은 model로 두 개의 foreign key가 있을 수 있다. 하지만 그들은 many-to-many relationship에서 서로 다른 측면에 있는 것으로 간주된다. 만약 두 개 이상의 foreign key가 있다면, 너는 역시 **through_fields**를 위와 같이 명시해 주어야 validation error가 발생하지 않을 것이다.
- intermediary model을 이용하여 자기 자신에게 **`many-to-many relationship`**을 정의할 때, 너는 `symmetrical=False` 를 사용해야 한다(model field reference를 참조하라).

이제 너의 intermediatry model(우리의 경우 **Membership**)을 이용하기 위한 **`ManyToManyField`** 준비 작업을 마쳤고, 너는 many-to-many relationship을 사용할 준비가 되었다. 너는 이것을 intermediate model의 instances를 만들어 사용할 수 있다:

    >>> ringo = Person.objects.create(name="Ringo Starr")
    >>> paul = Person.objects.create(name="Paul McCartney")
    >>> beatles = Group.objects.create(name="The Beatles")
    >>> m1 = Membership(person=ringo, group=beatles,
    ...			date_joined=date(1962, 8, 16),
    ...	 	 	invite_reason="Needed a new drummer.")
    >>> m1.save()
    >>> beatles.members.all()
    <QuerySet [<Person: ringo Starr>]>
    >>> ringo.group_set.all()
    <QuerySet [<Group: The Beatles>]>
    >>> m2 = Membership.objects.create(person=paul, group=beatles,
    ...			date_joined=date(1960, 8, 1),
    ...			invite_reason="Wanted to form a band.")
    >>> beatles.memebers.all()
    <QuerySet [<Person: Ringo starr>, <Person: Paul McCartney>]>

보통의 many-to-many field과는 다르게, 너는 **add()**, **create()**, 또는 **set()** 을 relationship을 만들기 위해 사용할 수 없다.

    >>> # The following statements will not work
    >>> beatles.members.add(john)
    >>> beatles.members.create(name="George Harrison")
    >>> beatles.members.set([john, paul, ringo, george])

왜 그럴까? 너는 단순히 **Person**과 **Group**사이의 relationship을 만들 수 없다 - 너는 **Membership** model이 필요로 하는 모든 세부 사항들을 명시해 주어야 한다. 단순히 **add**, **create** 그리고 assignment call으로는 이러한 추가적인 세부 사항들을 알려줄 방법이 없다. 결과적으로, 이들은 intermediate model을 이용하는 many-to-many relationship에 사용 불가능하다. 이런 종류의 relationship을 만드는 유일한 방법은 intermeidate model의 instance들을 생성하는 것이다.

**`remove()`** method도 비슷한 이유로 사용 불가능하다. 예를 들어, 만약 intermediate model을 사용하여 정의한 custom들이 **(model1, model2)**의 유일성을 보장하지 않는다면, **remove()** 호출은 어떤 어떤 intermediate model의 instance를 삭제해야 하는지 충분한 정보를 제공하지 않는다:

    >>> Membership.objects.create(person=ringo, group=beatles,
    ...			date_joined=date(1968, 9, 4),
    ...			invite_reason="You've been gone for a month and we miss you.")
    >>> beatles.members.all()
    <QuerySet [<Person: Ringo Starr>, <Person: Paul McCartney>, 
    <Person: Ringo Starr>]>
    >>> # This will not work because it cannot tell which membership to remove
    >>> beatles.members.remove(ringo)

그러나, **clear()** method는 모든 many-to-many relationships의 instance들을 삭제하기 위해 사용할 수 있다:

    >>> # Beatles have broken up
    >>> beatles.members.clear()
    >>> # Note that this deletes the intermediate model instances
    >>> Membership.objects.all()
    <QuerySet []>

일단 너가 너의 intermediate model의 instance를 생성하여 many-to-many relationship을 형성했다면, 너는 queries를 볼 수 있다. 평범한 many-to-many relationship과 마찬가지로, 너는 many-to-many로 연결된 model들의 attribute들을 이용하여 query할 수 있다.

    # Find all the groups with a member whose name starts with 'Paul'
    >>> Group.objects.filter(members__name__startswith='Paul')
    <QuerySet [<Group: The Beatles>]>

너는 또한 intermediate model들을 attributes들을 이용하여 query 할 수 있다.

    # Find all the members of the Beatles that joined after 1 Jan 1961
    >>> Person.objects.filter(
    ...			group__name='The Beatles',
    ...			membership__date_joined__gt=date(1961,1,1))
    <QuerySet [<Person: Ringo Starr]>

만약 너가 membership의 정보들에 접근할 필요가 있으면 너는 **Membership** model을 직접적으로 querying함으로써 할 수 있다:

    >>> ringos_membership = Membership.objects.get(group=beatles, person=ringo)
    >>> ringos_membership.date_joined
    datetime.date(1962, 8, 16)
    >>> ringos_membership.invite_reason
    'Needed a new drummer.'

같은 정보에 접근할 수 있는 또 다른 방법은 **Person** object로부터 `many-to-many reverse relationship`을 querying하는 것이다.

    >>> ringos_membership = ringo.membership_set.get(group=beatles)
    >>> ringos_membership.date_joined
    datetime.date(1962, 8, 16)
    >>> ringos_membership.invite_reason
    'Needed a new drummer.'

---

**One-to-one relationships**

one-to-one relationship을 정의하기 위해, **`OneToOneField`**를 사용해라. 너는 어느 다른 **Field** type처럼 이용하면 된다: 너의 model의 class attribute로 추가해라.

이것은 한 object가 다른 object를 "extends" 하고 있을 때 그 object의 primary key에서 가장 유용하다.

**`OneToOneField`**는 지정된 위치의 argument를 요구한다: 모델과 relate된 class를 지정하는 것이다.

예를 들어, 만약 너가 "places"라는 database를 만들고 있다면, 너는 address, phone number 등과 같은 평범한 것들을 데이터 베이스에 포함할 것이다. 그리고 나서, 너가 그 place들 위에 restaurant들의 데이터 베이스를 세우고 싶다면, **Restaurant** model에 그 fields들을 복제하는 것 대신에, **Restaurant**가 **Place**로 **`OneToOneField`**를 갖게 만들어 주면 된다(왜냐하면 reataurant "is a" place이기 때문이다; 사실, 이것을 다루기 위해 전형적으로 inheritance를 사용하는데, 이것은 암묵적으로 one-to-one relation을 갖는다).

**`ForeignKey`** 와 함께 `recursive relationship`은 정의될 수 있고 `references to as-yet undefined model`도 만들어 질 수 있다.

- **See also**

    전체 예시를 보기 위해 `One-to-one relationship model example`을 참고하라.

**`OneToOneField`**는 또한 optional `**parent_link**` argument를 사용할 수 있다.

**`OneToOneField`** class들은 자동적으로 model의 primary key가 되곤 했다. 이는 더 이상 사실이 아니다(너가 직접 `**primary_key**` argument들을 넘겨주어도 말이다). 그러므로, 이제 하나의 model에서 OneToOneField type의 다양한 field를 갖는 것이 가능하다.

---

## Models across files

한 model에서 다른 app에 있는 model로 relate을 하는 것은 완벽히 괜찮다. 이를 하기 위해서, 너의 모델이 정의되어 있는 곳 파일 가장 위에 related model을 import해야 한다. 이후, 필요한 곳 어디든지 그 다른 model class를 참조하면 된다. 예를 보자:

    from django.db import models
    from geography.models import ZipCode
    
    class Restaurant(models.Model):
    	# ...
    	zipe_code = models.ForeignKey(
    		ZipCode,
    		on_delete=models.SET_NULL,
    		blank=True,
    		null=True,
    	)

---

## Field name restrictions

Django는 model field 이름에 단 두 가지 제약만이 있다. 

1. filed name은 Python reserved word가 될 수 없다. 왜냐하면 이는 Python syntax error를 발생시킬 것이다. 예시를 보자:

        class Example(models.Model):
        	pass = models.IntegerField() # 'pass' is a reserved word!

2. field name은 두 개 이상의 underscore을 가질 수 없는데, 이는 Django의 query lookup syntax words이기 때문이다. 예시를 보자:

        class Example(models.Model):
        	foo__bar = models.IntegerField() # 'foo__bar' has two underscores!

이러한 제약 사항들은 해결될 수 있는데, 왜냐하면 너의 field name은 너의 database column 이름과 꼭 어울릴 필요가 없기 때문이다. `db_column` option을 참조하여라.

SQL reserved words, 예를 들어 **join**, **where** 또는 **select**는 model field name으로 사용할 수 있는데, 왜냐하면 Django는 기본 SQL query에 있는 모든 database table 이름과 column 이름들을 직접적으로 사용하지 때문이다. 그것은 너의 특정 database 엔진의 quoting syntax를 사용한다.

---

## Custom field types

만약 기존에 존재하는 model fields들이 너의 목적에 맞게 사용할 수 없다면, 또는 너가 일반적으로 사용되지 않는 database column type을 이용하려면 너만의 자체 field class를 만들 수 있다. 고유의 field를 만드는 전체 방법은 `Writing custom model fields`에 나와 있다.

---

# Meta options

inner **class Meta**를 다음과 같이 사용하여 너의 model에게 metadata를 줄 수 있다:

    from django.db import models
    
    class Ox(models.Model):
    	horn_length = models.IntegerField()
    
    	class Meta:
    		ordering = ["horn_length"]
    		verbose_name_plural = "oxen"

Model의 metadata는 "field가 아닌 모든 것"이다. 예를 들어 ordering options(**ordering**), database table name(**db_table**), 또는 사람이 읽을 수 있는 복수형 또는 단수형 이름들(**verbose_name** 그리고 **verbose_name_plural**)이 있다. 모두 필수는 아니고 **class Meta**를 model에 추가하는 것은 전적으로 optional이다.

**Meta** option에 대한 전체 목록은 model option reference에서 확인 할 수 있다.

---

# Model attributes

**objects**

model의 가장 중요한 attribute는 **`Manager`**이다. 이는 interface인데 database query operation들이 이를 통해 Django model로 제공되고 database로부터 `retrieve the instances` 하는데 사용된다. 만약 custom **Manager**가 정의되지 않는다면, 초기 이름은 **`objects`**이다. Managers은 model instance가 아니라, 단지 model class를 통해서만 접근 할 수 있다.

---

# Model methods

너의 obejcts를 위한 custom "row-leve" functionality를 model에 추가하기 위해 custom method를 정의해라. `**Manager**` mehods들은 "table-wide" 관련 작업들을 하기 위해 만들어진 반면, model methods는 특정 model instance에 대해 작동한다.

이는 business logic을 한 곳에 유지하기 위한 값진 기술이다 - the model

예를 들어, 이 model은 몇 개의 custom method를 갖고 있다:

    from django.db import models
    
    class Person(models.Moel):
    	first_name = models.CharField(max_length=50)
    	last_name = models.CharField(max_length=50)
    	birth_date = models.DateField()
    
    	def baby_boomer_status(self):
    		"Returns the person's baby-boomer status."
    		import datetime
    		if self.birth_date < datetime.date(1945, 8, 1):
    			return "Pre-boomer"
    		elif self.birth_date < datetime.date(1965, 1, 1):
    			return "Baby boomer"
    		else:
    			return "Post-boomer"
    
    	@property
    	def full_name(self):
    		"Returns the person's full name."
    		return '%s %s' % (self.first_name, self.last_name)

이 예시의 마지막 method는 `property`이다.

model i`nstance reference`에는 `각각의 model에게 자동적으로 주어진 mothod`들의 전체 리스트가 있다. 너는 이들을 override할 수 있다 - `overriding predefined model methods`를 참고하라 - 그러나 너가 거의 항상 정의해야 할 몇 가지 method들이 있다:

**__str__()**

모든 object의 문자 표현을 반환하는 Python의 "magic method"이다. model instance가 순수 string으로 표현되어야 할 때마다 Python과 Django이 이를 호출할 것이다. 제일 알아 두어야 할 것은, 이는 너가 대화형 console이나 admin에 obejct를 표시해야 할 때 일어난다.

너는 항상 이 mothod를 정의하고 싶어할 것이다; 초기 값이 전혀 도움이 되지 않기 때문이다.

**get_absolute_url()**

이는 Django에게 한 object에 대해 URL을 계산하라 말한다. Django는 이를 그것의 admin interface 안에서 사용하고 한 object의 URL을 알아내야 할 때 필요하다.

object를 고유하게 식별하는 URL을 가진 object는 이 method를 정의해야 합니다.

## Overriding predefined model methods

customize 할 database 동작을 encapsulate하고 있는 또 다른 `model method` 집합이 있다. 특히 너는 **`save()`**와 **`delete()`**가 동작하는 방법을 종종 바꾸고 싶을 것이다.

너는 자유롭게 이 methods(그리고 다른 model methods) 역시 override할 수 있고 동작을 바꿀 수 있다.

내장 method들을 override하는 전형적인 경우는 너가 object를 저장할 때마다 어떤 일이 일어나게 하고 싶을 때이다. 예시를 보자(`**save()**`에서 받아들이는 parameter의 문서를 참조하라):

    from django.db import models
    
    class Blog(models.Model):
    	name = models.CharField(max_length=100)
    	tagline = models.TextFeild()
    
    	def save(self, *args, **kwargs):
    		do_something()
    		super().save(*args, **kwargs)
    		do_something_else()

너는 또한 잘못된 저장을 방지할 수 있다:

    from django.db import models
    
    class Blog(models.Model):
    	name = models.CharField(max_length=100)
    	tagline = models.TextField()
    
    	def save(self, *args, **kwargs):
    		if self.name == "Yoko One's blog":
    			return # Yoko shall never have her own blog!
    		else:
    			super().save(*args, **kwargs) # Call the "real" save() method

database에 object가 저장되는 것을 확실히 하기 위해 superclass의 method를 호출해야 하는 것을 기억해야 한다 - 바로 **super.save(*args, **kwargs)** 이다. 만약 너가 superclass의 method를 호출하는 것을 잊는다면, default behavior은 발생하지 않을 것이고 database는 변하지 않을 것이다.

또한 model method 에 전달할 수 있는 parameter를 전달하는 것이 중요하다 - 이것이 바로 ***args, **kwargs**들이 하는 것이다. Django는 수시로 새로운 argument를 추가하면서 내장된 model method 기능들을 확장한다. 만약 너가 method를 정의할 때 ***args**, ****kwargs**를 사용한다면, 너의 코드는 argument들이 추가될 때 자동적으로 이들을 지원할 것이다.

- **Overriden model methods are not called on bulk operations**

    `deleting objects in bulk using a QuerySet`을 하거나 **`cascading delete`**의 결과로 삭제 될 때 object의 **`delete()`** method는 필수적으로 호출 될 필요 없다. customized delete logic이 실행 되었는지 확인하고 싶다면, 너는 **`pre_delete`** 또는 **`post_delete`**를 사용할 수 있다.

    불행히도 bulk로 object를 creating과 updating 할 때는 해결할 수 있는 방법이 없다. 왜냐하면 **save()**, **pre_save**, 그리고 **post_save** 그 어느 것도 호출되지 않기 때문이다.

---

## Executing custom SQL

다른 일반적인 패턴은 custom SQL statements를 model method와 module-level method에 작성하는 것이다. raw SQL을 사용하는데 더 세부적인 사항은 `using raw SQL` 을 참조하여라.

---

# Model inheritance

Django에서의 model inheritance는 Python에서 일반적인 class inheritance와 거의 동일하게 작동하지만, 이 페이지의 시작 부분의 기초 사항들은 따라야 한다. 즉, base class 역시 **`django.db.models.Model`**의 하위 클래스여야 한다.

너가 결정해야 할 유일한 것은 너의 부모 model이 그 고유의 model(자신의 database table을 갖고 있는)이 될 것인지, 아니면 부모 model이 단지 자식 class들을 통해서만 볼 수 있는 공통된 정보들을 담고 있을 것인지이다.

다음은 Django에서 가능한 세 가지 유형의 inheritance이다.

1. 종종 너는 부모 class를 사용하여 각 하위 model에 입력하고 싶지 않은 정보들을 보유하게 하기를 원할 것이다. 이 class는 따로 분리하여 사용하지 않으므로 `Abstract base classes` 가 다음에 볼 내용이다.
2. 만약 너가 이미 존재하는 model을 상속 받기 원하고(아마 완전히 다른 application으로 부터) 각각의 model이 그들의 database table을 갖고 있기를 원한다면, `Multi-table inheritnace`를 살펴야 한다.
3. 마지막으로 model field를 수정하지 않고 단지 model의 Python-level behavior를 수정하기 원한다면, 너는 `Proxy models`를 사용할 수 있다.

## Abstract base classes

abstract base classes는 다수의 다른 model들에게 공통된 정보들을 담고 싶을 때 유용하다. 너는 **base class**를 작성하고 `Meta` class에 abstract=True라고 설정하면 된다. 그러면 이 model은 database를 만드는데 사용되지 않는다. 대신에 이것이 다른 model들을 위한 base class로 사용될 때, 이것의 field들이 child class의 field들에 추가된다.

다음 예시를 보자 :

    from django.db import models
    
    class CommonInfo(models.Model):
    	name = models.CharField(max_length=100)
    	age = models.PositiveIntegerField()
    
    		class Meta:
    			abstract= True
    
    class Student(CommonInfo):
    	home_group = models.CharField(max_length=5)

**Student** model은 세 가지 field를 갖게 될 것이다: **name**, **age**, 그리고 **home_group**이다. **CommonInfo** model은 보통의 Django model처럼 사용될 수 없는데, 이것은 abstract base class이기 때문이다. 이것은 database table을 생성하지 않고 manager를 갖고 있지 않으며 instantiated 될 수 없고 직접적으로 저장 될 수 없다.

abstract base classes로부터 상속된 fields들은 또 다른 field나 값으로 override 할 수 있고 **None**으로 제거 할 수 있다.

많은 용도로 이런 종류의 model inheritance는 너가 정확히 원하는 것일 것이다. 이것은 Python level에서 공통 정보를 분석하는 방법을 제공하면서 동시에 database level에서는 자식 model 당 하나의 database table을 만든다. 

## Meta inheritance

abstract base class가 생성될 때, Django는 base class에서 선언 한 `Meta` inner class를 attribute로 사용할 수 있게 한다. 만약 자식 class가 그것 고유의 `Meta` class를 선언하지 않았을 경우, 그것은 부모의 `Meta`를 상속받는다. 만약 자식이 부모의 Meta class를 extend하고 싶다면, 상속받으면 된다. 예시를 보자:

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

Django는 abstract base class의 `Meta` class에 단 하나의 값을 조정 해준다: `Meta` attribute를 installing하기 전에, **abstract=False**로 설정한다. 이것은 abstract base classes의 자식들이 자동적으로 abstract class가 되지 않는다는 것을 의미한다. 물론, 너는 다른 abstract base class로부터 상속받는 또 다른 abstract base class를 만들 수 있다. 단지 매번 명시적으로 abstract=True라고 설정해야 한다는 것을 기억하면 된다.

몇몇 attribute들은 abstract base class의 `Meta` class 안에 있기에 부적합하다. 예를 들어 **db_table**을 포함한다는 것은 (그들 고유의 `Meta`를 명시하지 않은) 모든 자식 classes이 모두 같은 database table을 이용한다는 것인데, 이것은 너가 원하는 작업 아닐 것이다.

---

## Be careful with related_name and related_query_name

만약 너가 **ForeignKey** 또는 **ManyToManyField**에서 **`related_name`** 또는 **`related_query_name`**을 사용한다면, 너는 항상 반드시 field에 대한 *unique* reverse name과 query name을 명시해 주어야 한다. 이것은 보통 abstract base class에서 문제를 유발하는데, 이 class에 있는 field들이 매번 같은 값의 attribute(**`related_name`**과 **`related_query_name`**을 포함해서)로 각각의 자식 class에 포함되기 때문이다.

abstract base class에서 **`related_name`** 또는 **`related_query_name`**을 사용할 때 이 문제를 해결하기 위해서는, value의 일부분이 '**%(app_label)s**'와 '**%(class)s**'를 포함하고 있어야 한다. 

- '**%(class)s**'는 field가 사용되는 자식 class의 소문자 이름으로 대체된다.
- **'%(app_label)s'**는 자식 class가 포함되어 있는 app의 소문자 이름으로 대체된다. 각각의 installed applicatoin은 유일하고 app 안에 있는 model class 이름도 유일한 것이 틀림 없기에, 최종 이름은 서로 다르게 될 것이다.

예를 들어 **comon/models.py**에 다음과 같은 app이 있다:

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

**rare/models.py**에도 다음 app이 있다:

    from common.models import Base
    
    class ChildB(Base):
    	pass

**common.ChildA.m2m** field의 reverse name은 **common_childa_related**가 될 것이고 reverse query name은 **common_childas**가 될 것이다. **common.childB.m2m** field의 reverse name은 **common_childb_related**가 될 것이고 reverse query name은 **common_chlidbs**가 될 것이다. 마지막으로 **rare.ChildB.m2m** field의 reverse name은 **rare_childb_related**가 될 것이고 reverse query name은 **rare_childbs**가 될 것이다. 너의 related name이나 related query name을 구성하기 위해 '**%(class)s**'와 **'%(app_label)s'** 부분들을 어떻게 사용하는지는 너에게 달려있지만, 만약 너가 그것의 사용을 잊어버린다면, Django는 너가 system check(또는 **`migrate`**)를 사용할 때 error를 일으킬 것이다.

만약 너가 abstract base class의 field에 대한 `**related_name**` attribute를 명시하지 않았다면, default reverse name은 child class 이름 뒤에 **'_set'**이 붙은 것이 될 것이고, 너가 child class에 직접적으로 선언한 것처럼 정상적으로 작동할 것이다. 예를 들어 위의 코드에서, **`related_name`** attribute가 생략 되었다면, **m2m** field의 reverse name은 **ChildA**의 경우 **childa_set**이 될 것이고 **ChildB**의 경우 **childb_set**이 될 것이다.

---

## Multi-table inheritance

Django에서 지원하는 두 번째 타입의 model inheritance는 계층 구조의 각 model이 모두 하나의 model일 때이다. 각각의 model은 그들 고유의 database table에 대응되고 독립적으로 queried 되고 생성될 수 있다. inheritance relationship은 자식 model과 그들의 부모 사이의 links를 소개한다(자동적으로 생성되는 **`OneToOneField`**를 통해서). 예시를 보자:

 

    from django.db import models
    
    class Place(models.Model):
    	name = models.CharField(max_length=50)
    	address = models.CharField(max_length=80)
    
    class Restaurant(Place):
    	serves_hot_dogs = models.BooleanField(default=False)
    	serves_pizza = models.BooleanField(default=False)

data들은 서로 다른 database table에 있음에도 불구하고 **Place**의 모든 field들은 **Restaurant**에서도 모두 이용 가능하다. 따라서 다음 두 가지가 모두 가능하다:

    >>> Place.objects.filter(name="Bob's Cafe")
    >>> Restaurant.objects.filter(name="Bob's Cafe")

만약 너가 **Restaurant** 이기도 한 **Place**를 갖고 있다면, 너는 **Place** object로부터 소문자 model 이름을 이용함으로써 **Restaurant** object를 얻을 수 있다:

    >>> p = Place.objects.get(id=12)
    # If p is a Restaurant obejct, this will give the child class:
    >>> p.restaurant
    <Restaurant: ...>

그러나 만약 위 예시에서의 **p**가 **Restaurant**가 아니라면(그것이 **Place** object로 직접적으로 생성되었거나 다른 class의 부모였다면), **p.restaurant**를 호출하는 것은 **Restaurant.DoesNotExist** exception을 일으킬 것이다.

**Restaurant**에 자동적으로 생성된 **Place**로 link해주는 **OneToOneField**는 다음과 같이 생겼다:

    place_ptr = models.OneToOneField(
    	Place, on_delete=models.CASCADE,
    	parent_link=True,
    )

너는 해당 field를 **Restaurant**에 **`parent_link=True`** 와 함께 고유의 **`OneToOneField`**를 선언함으로써 override할 수 있다.

**Meta and multi-table inheritance**

multi-table inheritance의 경우에, 자식 class에서 부모의 `Meta` class를 상속받는 것은 말이 되지 않는다. 모든 `Meta` option들은 이미 부모 class에 적용되었고, 그들을 다시 적용하는 것은 보통 모순적인 행동으로 보일 것이다(이것은 abstract base class와는 반대인데, base class는 그들 스스로는 존재하지 않는다).

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
