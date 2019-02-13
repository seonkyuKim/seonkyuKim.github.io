---
title: "Django 2.1 reference 번역: Introduction to models 1"
categories: reference
tags:
- django 2.1
- reference
toc: true
---
>들어가며: 본 글의 장고 공식 문서 중 Introduction to models의 앞의 절반, 즉 모델 상속 전까지의 번역본 입니다.

>원본 링크: [https://docs.djangoproject.com/en/2.1/topics/db/models/](https://docs.djangoproject.com/en/2.1/topics/db/models/)

# Models

model은 데이터에 대한 하나의, 결정적인 정보입니다. 모델은 필수적인 field들과 저장하고 있는 데이터들의 행동을 결정합니다. 일반적으로, 각각의 model은 하나의 database table로 매핑됩니다.

기초 사항:

- 각각의 model은 **`django.db.models.Model`**의 하위 python 클래스입니다.
- model 클래스의 각각의 attribute는 database field를 나타냅니다.
- 이것들과 함께, Django는 기본적인 database-access API를 제공합니다; `Making queries`참고하십시오.

# Quick example

다음 예시는 **first_name**과 **last_name**을 갖고 있는 **Person** model입니다:

    from django.db import models
    
    class Person(models.Model):
    	first_name = models.CharField(max_length=30)
    	last_name = models.CharField(max_length=30)

**first_name**과 **last_name**은 model의 `field`입니다. 각각의 field는 class attribute로 구체화되며, 각각의 attribute는 하나의 database column으로 매핑됩니다.

위의 **Person** model은 다음과 같은 데이터베이스 테이블을 생성합니다:

    CREATE TABLE myapp_person (
    	"id" serial NOT NULL PRIMARY KEY,
    	"first_name" varchar(30) NOT NULL,
    	"last_name" varchar(30) NOT NULL
    );

Some technical notes:
    

- 테이블의 이름인 **myapp_person**은 model metadata로부터 자동적으로 생성되지만, override할 수 있습니다. 자세한 사항은 `Table names`를 참조하십시오.
- **id** field는 자동적으로 추가되지만 이 역시 override할 수 있습니다. `Automatic primary key fields`를 참조하십시오.
- 위 예시의 **CREATE TABLE**은 PostgreSQL 문법으로 쓰여 있는데, Django가  `settings file`에 명시된 database backend 맞춤 SQL 을 사용한다는 것을 기억하십시오.

---

# Using models

models를 정의한 후, Django의 settings file을 수정하여 앞으로 그 models를 사용할 것이라고 알려주어야 합니다. 이를 위해 **INSTALLED_APPS**파일에 **models.py**를 포함하고 있는 module의 이름을 추가하십시오.

예를 들어, 만약 당신의 application의 models가 **myapp.models** module 안에 있다면 (**manage.py startapp** 명령어를 사용하여 만든 application package structure 안에 있다면), **INSTALLED_APPS**는 다음과 같습니다:

    INSTALLED_APPS = [
    	#...
    	'myapp',
    	#...
    ]

새로운 app들을 **INSTALLED_APPS**에 추가한 뒤, **manage.py makemigrations**를 이용하여 부분적 migrations를 만든 후, **manage.py migrate**를 꼭 실행하도록 하십시오.

---

# Fields

기존에 정의된 database field들의 리스트를 아는 것은 model에서 가장 중요한 부분입니다 (그리고 model에서 신경써야 할 유일한 부분입니다). Fields는 class attirbute에 의해 명시됩니다. **clean, save** 또는 **delete**와 같은 `models API`와 field의 이름이 충돌하지 않도록 주의하십시오.

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

model 안의 각각의 field는 적당한 **Field** class안의 instance여야 합니다. Django는 다음 몇 가지를 결정하기 위해 field class types를 사용합니다:

- Column type은 어떤 종류의 database를 저장할지 알려 줍니다.(e.g.**INTEGER, VARCHAR, TEXT**)
- Default HTML `widget`은 form field들을 rendering할 때 사용됩니다.(e.g. **`<input type="text"><select>`**)
- Minimal validation requirements는 Django의 admin과 자동으로 생성되는 form에서 사용됩니다.

Django는 수십 개의 내장 field-type들을 갖고 있습니다; 그 전체 목록을 `model field reference`에서 확인할 수 있다. 만약 Django의 내장 field가 제 기능을 못 할 경우, 당신만의 field를 쉽게 작성할 수 있다; `Writing custom model fields`를 참조하십시오.



## Field options

각각의 field는 해당 field에 필수적인 각각의 arguments를 갖고 있습니다(`model field reference`에 표시되어 있습니다). 예를 들어, **CharField**(그리고 그것의 subclasses)는 데이터를 저장하는데 사용되는 **VARCHAR** database field의 사이즈를 명시해주는 **max_length** 인자를 필요로 합니다.

또한 모든 field type에 적용되는 공통 argument들이 있습니다. 모두 optional 입니다. 이들은 `reference`에서 전부 설명되어 있지만, 여기에서는 가장 많이 사용하는 몇 가지만 요약을 하겠습니다.

>제 블로그 reference 에 Field options 문서를 번역해놓았으니 참고하시면 좋을 것 같습니다.

**null**

만약 **True**라면, Django는 database에 빈 값인 **NULL**값을 저장할 수 있습니다. 초기 값은 **False**입니다.

**blank**

만약 **True**라면, field는 빈 칸이 있을 수 있습니다. 초기 값은 False입니다.

blank는 null과 다릅니다. null은 database-related인 반면, blank는 validation-related입니다. 만약 field에서 **blank=True**라면, form validation은 entry의 비어있는 값을 허락합니다. 만약 field에서 **blank=False**라면, 그 field의 값은 항상 요구됩니다.

**choices**

2-tuples로 구성된 iteralbe(e.g. list or tuple)을 이 field의 choices로 사용할 수 있습니다. 만약 choices가 주어지면, default form widget은 기본적인 text field가 아니라 select box가 될 것이고, 주어진 choices로 선택권이 제한될 것입니다.

choices list의 예시는 다음과 같습니다:

    YEAR_IN_SCHOOL_CHOICES = (
    	('FR', 'Freshman'),
    	('SO', 'Sophomore'),
    	('JR', 'Junior'),
    	('SR', 'Senior'),
    	('GR', 'Graduate'),
    )

각각의 tuple의 첫 번째 요소는 database에 저장되는 값입니다. 두 번째 요소는 field의 form widget에 나타나는 값(display value)입니다.

model instance가 주어졌을 때, **choices**를 갖고 있는 field에서 표시되는 값은 **get_F00_display()** method를 이용하여 알 수 있습니다. 예시를 보십시오:

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

**default**

field의 초기 값 입니다. 어떠한 한 값이나 callable object가 될 수 있습니다. 만약 callable하다면 이 객체는 새로운 object가 생성될 때마다 실행될 것입니다.

**help_text**

form widget에 표시될 추가적인 도움말 입니다. 만약 폼에서 해당 필드를 사용하지 않더라도 설명을 표기하는데 유용합니다

**primary_key**

만약 **True**라면, 이 field는 model의 primary key가 됩니다.

만약 model에서 어떤 field에도 **primary_key=True**로 설정해 두지 않았다면, Django는 자동적으로 primary key로 사용할 **IntegerField**를 추가하기 때문에 이러한 기본 primary-key를 override하고 싶지 않은 이상 어떤 field에 **primary_key=True**를 설정할 필요 없습니다. `Automatic primary key fields`에 더 자세히 기술되어 있습니다.

primary key field는 읽을 수만 있습니다. 만약 기존 object의 primary key 값을 변경한다면, 기존 object 옆에 새로운 object가 생성될 것입니다. 예시를 보십시오:

    from django.db import models
    
    class Fruit(models.Model):
    	name = models.CharField(max_length=100, primary_key=True)

    >>> fruit = Fruit.objects.create(name='Apple')
    >>> fruit.name = 'Pear'
    >>> fruit.save()
    >>> Fruit.objects.values_list('name', flat=True)
    <QuerySet ['Apple', 'Pear']>

**unique**

만약 True라면, 이 field는 table 전체에서 유일해야 합니다.

다시 한 번 말하지만, 이들은 모두 공통된 field option들에 대한 짧은 설명일 뿐입니다. 자세한 내용은 `common model field option reference`에 기술되어 있습니다.



## Automatic primary key fields

초기 값으로, Django는 각각의 model에 다음과 같은 field를 생성합니다:

    id = models.AutoField(primary_key=True)

이는 자동적으로 증가하는 (auto-incrementing) primary key입니다.

만약 custom primary key를 사용하고 싶다면, 모델의 field들 중 하나에 **primary_key=True**를 명시하십시오. 만약 명시적으로 **Field.primary_key**를 설정 했다면, Django는 위의 자동적으로 생성된 **id** column을 추가하지 않을 것입니다.

각각의 model은 (명시적으로 추가 되었든지 아니면 자동적으로 추가 되었든지간에) 오직 하나의 field만이 **primary_key=True**를 갖고 있어야 합니다.



## Verbose field names

>Verbose name을 직역하면 장황한 이름입니다.
**ForeignKey**, **ManyToManyField**와 **OneToOneField**를 제외하고, 각각의 field type들은 선택적으로(optional) 첫 번째 위치의 argument로 verbose name을 갖습니다. 만약 verbose name이 주어지지 않았다면, Django는 자동적으로 field의 attribute 이름에 있는 underscores를 space로 바꾸어 verbose name을 생성합니다.

이 예시에서 verbose name은 "**person's first name**"입니다:

    first_name = models.CharField("person's first name", max_length=30)

이 예시에서 verbose name은 "**first name**"입니다:

    first_name = models.CharField(max_length=30)

**ForeignKey**, **ManyToManyField**와 **OneToOneField**는 첫 번째 argument로 model class가 필요하기 때문에 **verbose_name** keyword argument를 사용합니다.

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

관습적으로 **verbose_name**의 첫 글자는 대문자를 쓰지 않습니다. Django가 자동적으로 첫 번째 글자를 대문자로 바꾸어주기 때문입니다.



## Relationships

명확하게, relational database의 강점은 테이블간에 relating이 되어있다는 점입니다. Django는 가장 많이 사용되는 세 가지 database relationship을 정의해 줍니다: many-to-one, many-to-many 그리고 one-to-one.

**Many-to-one relationships**

many-to-one relationship을 정의하기 위해, **django.db.models.ForeignKey**를 사용합니다. 이것을 다른 **Field** 타입처럼 사용하면 됩니다: model에 class attribute로 추가하십시오.

**ForeignKey**는 정해진 위치의 필수 argument를 요구합니다: 모델과 relate된 class를 지정합니다.

예를 들어, **Car** model이 **Manufacturer**을 갖고 있다면 - 즉, **Manufacturer**은 다양한 cars를 만들지만 각각의 **Car**는 단 하나의 **Manufacturer**를 갖고 있다 - 다음과 같이 할 수 있습니다:

    from django.db import models
    
    class Manufacturer(models.Model):
    	# ...
    	pass
    
    class Car(models.Model):
    	manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
    	# ...

또한 `recursive relationship`(자기 자신에게 many-to-one relationship을 갖고 있는 object)을 만들 수 있고 `아직 정의되지 않은 model에게 relationship`을 만들 수 있습니다; 자세한 사항은 `model field reference`를 참고하십시오.

필수적인 것은 아니지만, **ForeignKey field**의 이름(위의 예에서는 **manufacturer**)은 소문자의 model 이름을 사용하는 것이 좋습니다. 물론, 원하는 대로 field 이름을 정해도 됩니다. 예시를 보십시오:

    class Car(models.Model):
    	company_that_makes_it = models.ForeignKey(
    		Manufacturer,
    		on_delete=models.CASCADE,
    	)

- **See also**

    **ForeignKey** field는 `model field references`에 설명되어 있는 다른 arguements도 사용할 수 있습니다. 이 옵션들은 relationship이 어떻게 작동할지를 규정해 줍니다; 모두 optional이다. 

    backward-related objects에 접근하는 방법에 대해서는 `Following relationships backward example`을 참고하십시오.

    sample code를 보고 싶다면 `Many-to-one relationship model example`을 참고하십시오.

---

**Many-to-many relationships**

Many-to-many relationships를 사용하기 위해서는, **ManyToManyField**를 사용하십시오. 이것을 다른 **Field** type처럼 사용하시면 됩니다: model에 class attribute로 추가하십시오.

**ManyToManyField**는 지정된 위치의 필수 argument를 요구합니다: relate할 class를 지정합니다.

예를 들어, **Pizza** 가 다양한 **Topping** objects를 갖고 있다면 - 즉, 한 **Topping**이 다양한 pizzas에 있을 수 있고 각각의 **Pizza**도 다양한 toppings를 가질 수 있다면 - 다음과 같이 이를 나타낼 수 있습니다:

    from django.db import models
    
    class Topping(models.Model):
    	# ...
    	pass
    
    class Pizza(models.Model):
    	# ...
    	toppings = models.ManyToManyField(Topping)

**ForeignKey**와 마찬가지로, recursive relationship(자기 자신에게 many-to-many relationship을 갖고 있는 객체)를 만들 수 있고 `아직 정의되지 않은 model에게 relationship`을 만들 수 있습니다.

필수적인 것은 아니지만, related 모델을 나타내는 **ManyToManyField**의 이름은 복수형을 하는 것이 좋습니다(위 예에서는 **toppings**).

어떤 model이 **ManyToManyField**를 갖고 있는지는 상관 없지만, 두 model 모두가 아니라 한 쪽에만 설정해야 합니다.

위의 예시에서는 폼에서 Pizza의 toppings를 선택할 수 있게 해줍니다(**Topping**이 많은 **pizzas**를 갖고 있는 것이 아닙니다). 이는 pizza가 많은 topping들이 갖고 있다고 생각하는 것이 topping이 많은 pizza들 위에 있다고 생각하는 것보다 자연스럽기 때문입니다. 이와 같이 일반적으로 **ManyToManyField** instance는 폼에서 수정되어야 하는 객체 안에 있어야 합니다. 

- **See also**

    전체 예시를 보기 위해서는 `Many-to-many relationship model example`을 참조하십시오.

**ManyToManyField** field는 `model field references`에 설명되어 있는 다른 arguements도 사용할 수 있습니다. 이 옵션들은 relationship이 어떻게 작동할지 규정합니다; 모두 optional입니다.

---

**Extra fields on many-to-many relationships**

pizza와 topping과 같은 단순한 many-to-many relationship을 다루고 있다면, **ManyToManyField**만 사용하시면 됩니다. 하지만, 때때로 relationship이 있는 두 models간의 데이터들을 연결지어야 할 것입니다.

예를 들어 musicians이 속해있는 musical group이 있는 경우를 생각해보십시오. 여기에는 person과 그들이 멤버로서 속해있는 group들 간의 many-to-many relationship이 있고, 이 관계를 나타내기 위해 **ManyToManyField**를 사용할 수 있습니다. 그러나, membership과 관련된 세부 사항들, 예를 들어 person들이 group에 들어간 날짜 등이 많이 있다고 가정합시다. 

이러한 상황에서, Django에서는 many-to-many relationship을 관리하는 model을 명시할 수 있습니다. 또한 이 intermidate model에 추가적인 field들을 넣을 수 있습니다. 중간 테이블 역할을 할 model을 알려주는  **through** argument를 이용하여 중간 model은 **ManyToManyField**와 연결될 수 있습니다. 우리의 musician 예시는 다음 코드와 같이 쓸 수 있습니다:

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

중간 model을 만들 때는 foreign key를 이용하여 many-to-many relationship에 포함될 model들을 명시적으로 지정해주어야 합니다. 이 명시적 선언은 두 model들이 어떻게 relate되는지 정의합니다.

다음은 중간 model에서 몇 가지 제약 사항들입니다:

- intermediate model은 source model(우리의 예에서는 **Group**)로 향하는 foreign key는 단 하나만 갖고 있어야 합니다. 또는 **ManyToManyField.through_fields**를 사용하여 Django가 relationship을 위해 사용해야 하는 foreign key를 명시적으로 지정해 주어야 합니다. 만약 한 개 이상의 foreign key를 갖고 있거나 through_fields가 명시되어 있지 않으면 validation error가 발생할 것입니다. 비슷한 제약은 target model(우리의 예에서는 **Person**)에서도 적용됩니다.
    >**through_fields**의 내용은 model field reference를 참고해주세요.
- 중간 model을 이용하여 자기 자신에게 many-to-many relationship을 갖는 model에 대해서는, 같은 model을 향한 두 개의 foreign key가 있을 수 있습니다. 하지만 그들은 many-to-many relationship에서 서로 다른 측면에 있는 것으로 간주됩니다. 만약 두 개 이상의 foreign key가 있다면, 역시 **through_fields**를 위와 같이 명시해 주어야 validation error가 발생하지 않을 것입니다.
- 중간 model을 이용하여 자기 자신에게 **many-to-many relationship**을 정의할 때, **symmetrical=False** 를 사용해야 합니다(model field reference를 참조하십시오).

이제 중간 model(예시의 경우 **Membership**)을 이용하기 위한 **ManyToManyField** 준비 작업을 마쳤고, many-to-many relationship을 사용할 준비가 되었습니다. intermediate model의 instances를 만들어 사용할 수 있습니다:

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

보통의 many-to-many field과는 다르게, relationship을 만들기 위해서 **add()**, **create()**, 또는 **set()** 을  사용할 수 없습니다.

    >>> # The following statements will not work
    >>> beatles.members.add(john)
    >>> beatles.members.create(name="George Harrison")
    >>> beatles.members.set([john, paul, ringo, george])

왜 그럴까요? 단순히 **Person**과 **Group**사이의 relationship을 만들 수 없습니다 - 왜냐하면 **Membership** model이 필요로 하는 모든 세부 사항들을 명시해 주어야 하기 때문입니다. 단순히 **add**, **create** 그리고 assignment call으로는 이러한 추가적인 세부 사항들을 알려줄 방법이 없습니다. 결과적으로, 이들 함수는 중간 model을 이용하는 many-to-many relationship에서 사용 불가능합니다. 이런 종류의 relationship을 만드는 유일한 방법은 intermeidate model의 instance들을 생성하는 것입니다.

**remove()** method도 비슷한 이유로 사용 불가능합니다. 예를 들어, 만약 중간 model의  `(model1, model2)`의 유일성이 보장되지 않는다면, **remove()** 호출은 중간 model의 어떤 instance를 삭제해야 하는지 충분한 정보를 제공하지 않기 때문입니다:

    >>> Membership.objects.create(person=ringo, group=beatles,
    ...			date_joined=date(1968, 9, 4),
    ...			invite_reason="You've been gone for a month and we miss you.")
    >>> beatles.members.all()
    <QuerySet [<Person: Ringo Starr>, <Person: Paul McCartney>, 
    <Person: Ringo Starr>]>
    >>> # This will not work because it cannot tell which membership to remove
    >>> beatles.members.remove(ringo)

그러나 **clear()** method는 모든 many-to-many relationships의 instance들을 삭제하기 위해 사용할 수 있습니다:

    >>> # Beatles have broken up
    >>> beatles.members.clear()
    >>> # Note that this deletes the intermediate model instances
    >>> Membership.objects.all()
    <QuerySet []>

일단 중간 model의 instance를 생성하여 many-to-many relationship을 형성했다면, queries를 통해 볼 수 있습니다. 평범한 many-to-many relationship과 마찬가지로, many-to-many로 연결된 model들의 attribute들을 이용하여 query할 수 있습니다.

    # Find all the groups with a member whose name starts with 'Paul'
    >>> Group.objects.filter(members__name__startswith='Paul')
    <QuerySet [<Group: The Beatles>]>

또한 중간 model들의 attributes들을 이용하여 query 할 수 있다.

    # Find all the members of the Beatles that joined after 1 Jan 1961
    >>> Person.objects.filter(
    ...			group__name='The Beatles',
    ...			membership__date_joined__gt=date(1961,1,1))
    <QuerySet [<Person: Ringo Starr]>

만약 membership의 정보들에 접근해야 한다면 **Membership** model을 직접적으로 query 할 수 있습니다:

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

one-to-one relationship을 정의하기 위해, **OneToOneField**를 사용하십시오. 어느 다른 **Field** type처럼 이용하면 됩니다: model의 class attribute로 추가하십시오.

이것은 한 객체가 다른 객체를 "extends" 하고 있을 때 객체의 primary key에서 사용하면 가장 유용합니다.

**OneToOneField**는 지정된 위치의 필수 argument를 요구합니다: 모델과 relate된 class를 지정합니다.

예를 들어, 만약 "places"라는 데이터베이스를 만들고 있다면, address, phone number 등과 같이 평범한 내용들을 데이터 베이스에 포함할 것이다. 그리고 나서, 그 place들 위에 restaurant들의 데이터 베이스를 세우고 싶다면, **Restaurant** model에 그 fields들을 붙여넣는 것 대신에, **Restaurant**가 **Place**로 **OneToOneField**를 갖게 만들어 주면 됩니다(왜냐하면 reataurant "is a" place이기 때문이다; 사실, 이을 위해 전형적으로 상속을 사용하는데, 이것은 암묵적으로 one-to-one relation을 갖습니다).

**ForeignKey** 와 함께 `recursive relationship`은 정의될 수 있고 `references to as-yet undefined model`도 만들어 질 수 있습니다.

- **See also**

    전체 예시를 보기 위해 `One-to-one relationship model example`을 참고하십시오.

**OneToOneField**는 또한 optional **parent_link** argument를 사용할 수 있습니다.

**OneToOneField** class들은 자동적으로 model의 primary key에 연결되곤 했습니다. 하지만 더 이상 그렇지 않습니다(직접 **primary_key** argument들을 넘겨주어도 말입니다). 그러므로, 이제 하나의 model에서 OneToOneField type의 다양한 field를 갖는 것이 가능합니다.



## Models across files

한 model에서 다른 app에 있는 model로 relate를 할 수 있습니다. 이를 위해 당신의 모델이 정의되어 있는 파일 가장 위에 related model을 import해야 합니다. 이후, 필요한 곳 어디든지 그 다른 model class를 참조하면 됩니다. 예를 보십시오:

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



## Field name restrictions

Django는 model field 이름에 단 두 가지 제약만이 있습니다. 

1. field name은 Python reserved word가 될 수 없습니다. 왜냐하면 이는 Python syntax error를 발생시킬 것입니다. 예시를 보십시오:

        class Example(models.Model):
        	pass = models.IntegerField() # 'pass' is a reserved word!

2. field name은 두 개 이상의 underscore을 가질 수 없는데, 이는 Django의 query lookup syntax words이기 때문입니다. 예시를 보십시오:

        class Example(models.Model):
        	foo__bar = models.IntegerField() # 'foo__bar' has two underscores!

이러한 제약 사항들은 피해갈 수 있습니다. 왜냐하면 field name은 database column 이름과 꼭 같을 필요가 없기 때문입니다. `db_column` option을 참조하십시오.

SQL reserved words, 예를 들어 **join**, **where** 또는 **select**는 model field name으로 사용할 수 있는데, 왜냐하면 Django는 기본 SQL query에 있는 모든 database table 이름과 column 이름들을 직접적으로 사용하지 않기 때문입니다. 그것은 너의 특정 database 엔진의 quoting syntax를 사용합니다.



## Custom field types

만약 기존의 model fields들을 목적에 맞게 사용할 수 없다면, 또는 일반적으로 사용되지 않는 database column type을 이용하려면 자체적인 field class를 만들 수 있다. 고유의 field를 만드는 전체 방법은 `Writing custom model fields`에 나와 있습니다.

---

# Meta options

inner **class Meta**를 다음과 같이 사용하여 model에게 metadata를 줄 수 있습니다:

    from django.db import models
    
    class Ox(models.Model):
    	horn_length = models.IntegerField()
    
    	class Meta:
    		ordering = ["horn_length"]
    		verbose_name_plural = "oxen"

Model의 metadata는 "field가 아닌 모든 것"입니다. 예를 들어 ordering options(**ordering**), database table name(**db_table**), 또는 사람이 읽을 수 있는 복수형 또는 단수형 이름들(**verbose_name** 그리고 **verbose_name_plural**)이 있습니다. 모두 필수는 아니고 **class Meta**를 model에 추가하는 것은 전적으로 optional입니다.

**Meta** option에 대한 전체 목록은 model option reference에서 확인 할 수 있습니다.

---

# Model attributes

**objects**

model의 가장 중요한 attribute는 **Manager**입니다. 이는 interface인데 database query operation들이 이를 통해 Django model로 제공되고 database로부터 `retrieve the instances` 하는데 사용됩니다. 만약 custom **Manager**가 정의되지 않는다면, 초기 이름은 **objects**입니다. Managers은 model instance가 아니라, 단지 model class를 통해서만 접근 할 수 있습니다.

---

# Model methods

객체를 위한 custom "row-level" 기능을 model에 추가하기 위해 custom method를 정의하십시오. **Manager** mehods들은 "table-wide" 관련 작업들을 하기 위해 만들어진 반면, model methods는 특정 model instance에 대해 작동합니다.

이는 business logic을 한 곳(model)에 유지하기 위한 값진 기술입니다.

예를 들어, 다음 model은 몇 개의 custom method를 갖고 있습니다:

    from django.db import models
    
    class Person(models.Model):
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

이 예시의 마지막 method는 `property`입니다.

model `instance reference`에는 `각각의 model에게 자동적으로 주어진 mothod`들의 전체 리스트가 있습니다. 이들을 override할 수 있습니다 - `overriding predefined model methods`를 참고하십시오 - 그러나 거의 항상 정의해야 할 몇 가지 method들이 있습니다:

**__str__()**

모든 object의 문자 표현을 반환하는 Python의 "magic method"이다. model instance가 순수 string으로 표현되어야 할 때마다 Python과 Django이 이를 호출할 것입니다. 제일 알아 두어야 할 것은, 이는 대화형 console이나 admin에 obejct를 표시해야 할 때 일어납니다.

항상 이 mothod를 정의하고 싶어할 것입니다; 초기 값이 전혀 도움이 되지 않기 때문입니다.

**get_absolute_url()**

이는 Django에게 한 object에 대해 URL을 어떻게 계산할지 알려줍니다. Django는 이를 그것의 admin interface 안에서 사용하거나 한 객체의 URL을 알아내야 할 때 필요합니다.

object를 고유하게 식별하는 URL을 가진 object는 이 method를 정의해야 합니다.

## Overriding predefined model methods

데이터 베이스 동작을 encapsulate하고 있는 또 다른 종류의 `model method` 집합이 있고, 이를 customize하고 싶을 것입니다. 특히 **save()**와 **delete()**가 동작하는 방법을 종종 바꾸고 싶을 것입니다.

자유롭게 이 methods(그리고 다른 model methods) 역시 override할 수 있고 동작을 바꿀 수 있습니다.

내장 method들을 override하는 전형적인 경우는 object를 저장할 때마다 어떤 일이 일어나게 하고 싶을 때 입니다. 예시를 보십시오( **save()**에서 받아들이는 `parameter의 문서`를 참조하십시오):

    from django.db import models
    
    class Blog(models.Model):
    	name = models.CharField(max_length=100)
    	tagline = models.TextFeild()
    
    	def save(self, *args, **kwargs):
    		do_something()
    		super().save(*args, **kwargs)
    		do_something_else()

또한 잘못된 저장을 방지할 수 있습니다:

    from django.db import models
    
    class Blog(models.Model):
    	name = models.CharField(max_length=100)
    	tagline = models.TextField()
    
    	def save(self, *args, **kwargs):
    		if self.name == "Yoko One's blog":
    			return # Yoko shall never have her own blog!
    		else:
    			super().save(*args, **kwargs) # Call the "real" save() method

database에 object가 저장되는 것을 확실히 하기 위해 superclass의 method를 호출해야 하는 것을 기억하십시오 - 바로 **super.save(*args, **kwargs)** 입니다. 만약 superclass의 method를 호출하는 것을 잊는다면, default behavior은 발생하지 않을 것이고 database는 변하지 않을 것입니다.

또한 model method 에 사용되는 parameter를 전달하는 것이 중요합니다 - 바로 ***args, **kwargs**들이 그렇게 해줍니다. Django는 수시로 새로운 argument를 추가하면서 내장된 model method 기능들을 확장합니다. 만약 method를 정의할 때 ***args**, ****kwargs**를 사용한다면, 당신의 코드는 argument들이 추가될 때 자동적으로 이들을 지원할 것입니다.

- **Overriden model methods are not called on bulk operations**

    `deleting objects in bulk using a QuerySet`을 하거나 **cascading delete**의 결과로 삭제 될 때 객체의 **delete()** method는 항상 호출되지는 않습니다. customized delete logic이 실행 되었는지 확인하고 싶다면, **pre_delete** 또는 **post_delete**를 사용할 수 있습니다.

    불행히도 bulk로 객체를 생성하거나 업데이트 할 때는 해결할 수 있는 방법이 없습니다. 왜냐하면 **save()**, **pre_save**, 그리고 **post_save** 그 어느 것도 호출되지 않기 때문입니다.



## Executing custom SQL

다른 일반적인 패턴은 custom SQL statements를 model method와 module-level method에 작성하는 것입니다. raw SQL을 사용하는데 더 세부적인 사항은 `using raw SQL` 을 참조하십시오.

