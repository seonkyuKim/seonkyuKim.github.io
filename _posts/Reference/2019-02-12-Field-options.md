---
title: "Django 2.1 reference 번역: Field options"
categories: reference
tags:
- dango 2.1
- reference
toc: true
---
> 들어가며: 본 글은 장고 공식 문서 중 Field types의 Field options에 관한 번역입니다. 피드백은 언제나 감사합니다.

> 원본 링크: [https://docs.djangoproject.com/en/2.1/ref/models/fields/](https://docs.djangoproject.com/en/2.1/ref/models/fields/)

이 문서는 장고가 제공하는 모든 `field options`과 `field types`을 포함한 **`Field`**에 대해 설명하고 있습니다.


- **See also**

    만약 내장된 field가 작동하지 않으면 `django-localflavor`(`documentation`)를 시도해 보십시오. 이는 특별한 국가와 문화에 대해서 갖가지 유용한 code의 조각을 가지고 있습니다.

    또한 당신은 쉽게 당신만의 `custom model fields`를 만들 수 있습니다.

- **Note**

    기술적으로, 이런 모델들은 **django.db.models.fields**에 저장되어 있으나 편리함을 위해 그들은 **django.db.models**안에 imported 되어있습니다. 기본적인 명명 관습은 **from django.go import models**를 사용하는 것이며, fields는 **models.<Foo>Field**로 나타나는 것입니다.

# Field options

아래의 arguments는 모두 field의 타입들입니다. 모든 것은 선택적입니다.

> Field options는 Field type안에 포함되는 설정입니다. 예시: name = CharField(max_length=150, null=False). null은 field option이고, max_length는 필수 arguement입니다

## null

**Field.null**

만약 **True**라면, 장고는 데이터 베이스에 비어 있는 값으로 **NULL**을 저장할 것입니다. 기본적으로 **False**로 설정 되어있습니다.

**CharField** 와 **TextField**와 같은 문자열 기반 field의 경우는 **null**의 사용을 피하는 것이 좋습니다. 문자열을 저장하는 field에 **null=True** 라면, 데이터가 없을 경우 field에는 **NULL**과 빈 문자열 두 값이 모두 저장될 수 있습니다. 같은 값에 대해 두 가지 가능한 값을 갖는 것은 지양해야 합니다. 장고의 관습은 빈 문자열을 저장하는 것입니다.

한 가지 예외는 **CharField**가 **unique=True**와 **blank=True**를 전부 가지고 있는 때입니다. 이는 복수의 객체에서 blank value를 저장할 때 **unique** 조건에 위배되는 것을 피하기 위함입니다. (NULL은 여러 개 가질 수 있고, 빈 문자열은 하나의 값으로 인식되어 unique 때문에 하나만 가질 수 있는 것 같습니다.)

string-based 와 non-string-based field 에서 forms에 빈값을 넣는 것을 원한다면 **blank=True**로 설정해 주어야 합니다. **null** 값은 단지 database storage에 영향을 미치기 때문입니다(**blank**를 보십시오).

- **Note**

    Oracle database backend를 사용할 때, **NULL**은 이 attribute와 상관없이 빈 값을 나타내는 것으로 저장될 것입니다. 



## blank

**Field.blank**

만약 **True** 라면 field는 blank를 허용합니다. 기본적으로 **Flase**입니다.

`null`은 순수하게 database-related 인 반면에 `blank`는 validation-related 입니다. 만약 field가 **blank=True**를 가진다면 form validation은 empty value의 입력을 허용 해줄 것입니다. 만약 field가 **blank=False** 라면 field는 반드시 채워져야 합니다



## Choices

**Field.choices**

field에 choices로 사용하기 위해서는 두 items의 iterables 이거나 이를 포함하고 있는 iterable이어야 합니다(e.g. [(A, B), (A, B), ...]). 만약 choice가 주어진다면, 그들은 [model validation](https://docs.djangoproject.com/en/2.1/ref/models/instances/#validating-objects)에 의해 실행되고, text field가 아닌, select box에 선택지가 담긴 기본 위젯 form이 실행 될 것입니다.

각 튜플에서 첫번째 요소는 model에 저장되는 실질적 값이고, 두 번째 요소(element)는 사람이 읽게 되는 이름입니다. 예시:

    YEAR_IN_SCHOOL_CHOICES = (
        ('FR', 'Freshman'),
        ('SO', 'Sophomore'),
        ('JR', 'Junior'),
        ('SR', 'Senior'),
    )

일반적으로 model class 안에 choices를 각 값에 대해서 변함 없게 알맞은 이름을 정의하는 가장 좋은 방법은 다음과 같습니다.

    from django.db import models
    
    class Student(models.Model):
        FRESHMAN = 'FR'
        SOPHOMORE = 'SO'
        JUNIOR = 'JR'
        SENIOR = 'SR'
        YEAR_IN_SCHOOL_CHOICES = (
            (FRESHMAN, 'Freshman'),
            (SOPHOMORE, 'Sophomore'),
            (JUNIOR, 'Junior'),
            (SENIOR, 'Senior'),
        )
        year_in_school = models.CharField(
            max_length=2,
            choices=YEAR_IN_SCHOOL_CHOICES,
            default=FRESHMAN,
        )
    
        def is_upperclass(self):
            return self.year_in_school in (self.JUNIOR, self.SENIOR)

choices list를 model class의 외부에 정의하여 참조하게 할 수 있지만, model class 내부에 각 choice를 정의하는 것은 이것을 사용한 class에 모든 정보를 보관할 수 있다는 장점이 있습니다. 그리고 choices를 쉽게 참조 할 수 있게 합니다.(e.g. **Student** 모델이 import 된 어느 곳에서라도 **Student.SOPHOMORE**이 작동할 것입니다)

사용 가능한 choices를 체계화 하기 위한 목적으로 그룹에 이름을 지을 수 있습니다.

    MEDIA_CHOICES = (
        ('Audio', (
                ('vinyl', 'Vinyl'),
                ('cd', 'CD'),
            )
        ),
        ('Video', (
                ('vhs', 'VHS Tape'),
                ('dvd', 'DVD'),
            )
        ),
        ('unknown', 'Unknown'),
    )

각 tuple의 첫 번째 요소는 그룹을 나타낼 이름입니다. 두 번째 요소는 두 값의 iterable로, 저장되는 값과 사람이 읽을 수 있는 값으로 묶여진 tuple입니다. 하나의 list를 사용하여 그룹화 된 선택지들과 그룹화 되지 않은 선택지들을 합칠 수 있습니다.(예를 들면, 위에서의 unknown option들은 그룹화 되지 않은 선택지입니다)

choices 을 가진 model field 별로, field의 현재 값에 대응되는 사람에게 보여지는 이름을 받아오는 method를 추가합니다. 더 자세한 것은 database API documentation 안에 있는 `get_FOO_display()`를 참고하십시오.

choices는 그 어떤 iterable object로도 받을 수 있다는 것을 기억하십시오. 굳이 list나 tuple 로 구성할 필요가 없습니다. 이것은 choices를 dynamically하게 만들 수 있다는 의미입니다. 어쩌면 choices를 dynamic 하게 만들 방법을 찾아낼 수도 있지만, 아마 적절한 데이터베이스 테이블을 foreignkey와 함께 사용하는 것이 더 좋을 것입니다. choices는 어차피 크게 바꾸지 않을 정적 데이터이기 때문입니다.

**default**로 **blank=False**가 field에 설정되지 않는 이상 select box label에는 "——————————"가 담겨 제공될 것이다. 이를 override 하기 위해서는 **None**을 가진 choices tuple을 추가하십시오; e.g. **(None, 'Your String For Dispay').** 대안으로 예를 들어 CharField와 같이 적절한 곳에서는 empty string 대신 **None**을 자주 쓸 것입니다.



## db_colum

**Field.db_column**

field에 사용할 database column의 이름입니다. 만약 주어지지 않았다면, 장고는 field의 이름을 사용할 것이다.

만약 database column name 이 SQL reserved word, 또는 파이썬 변수이름에서는 사용할 수 없는 문자열을 포함하고 있어도 - 예를 들어 하이픈 - 장고에서는 사용할 수 있습니다. Django quotes column and table names behind the scenes.



## db_index

**Field.db_index**

만약 True 이면 이 field를 위해 database index가 생성될 것입니다.



## db_tablespaces

**Field.db_tablespace**

만약 field가 indexed 되어 있다면, 필드의 index에 대해 사용할 database tablespace의 이름입니다. 기본 값은 프로젝트의 **DEFAULT_INDEX_TABLESPACE** 설정, 또는 모델의 **db_tablespace**(설정된 경우)입니다. 만약 backend가 tablespace를 지원하지 않는다면 이 옵션은 무시될 것입니다.



## default

**Field.default**

필드에 대한 기본 값입니다. 이는 한 값이나 callable 객체가 될 수도 있습니다. 만약 callable 객체라면 새로운 객체가 생성될 때마다 호출될 것입니다.

default는 변경 가능한 객체(model instance, list, set 등)일 수 없으며, 이 객체의 동일한 인스턴스에 대한 참조는 모든 새로운 모델 인스턴스에서 기본 값으로 사용될 것입니다. 대신에, callable로 원하는 default를 wrap할 수 있습니다. 예를 들어, **JSONField**에 대해 default **dict**을 명시하고 싶다면, 다음 함수를 사용하십시오:

    def contact_default():
    	return {"email": "to1@example.com"}
    
    contact_info = FSONField("ContactInfo", default=contact_default)

**default**와 같은 field option으로 **lamda**는 사용할 수 없습니다. 왜냐하면 이들은 Migrations로 serialized되지 않기 때문입니다. 해당 설명서를 참고하십시오.

모델 인스턴스로 매핑하는 **ForeignKey**와 같은 필드에 대해서는, default는 모델 인스턴스 대신 그들이 참조하는 필드 값(**to_field**가 설정되지 않은 경우 **pk**)이어야 합니다.

> default 값을 모델을 설정하지 말고, 해당 모델의 field로 설정해야 한다는 뜻 같습니다.

default(기본 값)은 새로운 모델 인스턴스가 생기고, 해당 필드에 대해 값이 주어지지 않을 경우 사용됩니다. 필드가 primary key일 경우, 필드가 None으로 설정되어 있어도 default를 사용합니다.



## editable

**Field.editable**

만약 **False**라면, 이 field는 admin이나 **ModelForm**에서 표시되지 않을 것입니다. 또한 `model validation`을 생략할 것입니다. 기본적으로 **True**입니다.



## error_messages

**Fields.error_messages**

**error_messages** arguments는 이 field에서 에러가 발생할 때의 기본 메세지를 override합니다. override하고 싶은 에러에 대한 메세지를 dictionary로 넘겨주십시오.

에러 메세지의 key는 **null, blank, invalid, invalid_choice, unique,** 그리고 **unique_for_date**를 포함합니다. 추가적인 error message의 key들은 Field types 섹션에 각각의 필드에 명시돼 있습니다.

에러 메세지들은 종종 form으로 전달되지 않습니다. 모델의 error_messages에 대해 고려해야 할 사항에 대한 문서를 참조하십시오.



## help_text

**Field.help_text**

폼 위젯(form widget)에 표시될 추가적인 도움말입니다. 만약 폼에서 해당 필드를 사용하지 않는다 하더라도 설명하기에 유용합니다.

이 값은 자동으로 생성된 폼에서 HTML을 **help_text**에 원하는 대로 포함할 수 있게 해줍니다. 다음은 예시입니다:

    help_text="Please use the following format: <em>YYYY-MM-DD</em>."

일반 텍스트와 **django.utils.html.escape()**를 사용하여 HTML 특수 문자를 사용하지 않을 수 있습니다. cross-site scripting attack를 피하기 위해서 신뢰할 수 없는 사용자로부터 온 help text를 확실히 escape 해야 합니다.



## primary_key

**Field.primary_key**

**True**라면, 이 필드는 해당 모델에 primary key입니다.

어떤 모델에도 **primary_key=Ture**를 설정하지 않았다면, 장고는 자동으로 primary key 역할을 할 **AutoField**를 추가할 것입니다. 따라서 기본 primary-key behavior을 override하지 않는 이상, **primary_key=Ture**을 명시할 필요가 없습니다. 자세한 사항은 `Automatic primary key fields`를 보십시오.

**primary_key=True**는 **null=False**와 **unique=True**를 암시합니다. 한 객체에 대해서는 하나의 Primary key만 허용됩니다.

primary key 필드는 읽을 수만 있습니다. 만약 존재하는 객체의 primary key 값을 변경하고 저장한다면, 새로운 객체가 옛날 것 옆에 만들어질 것입니다.



## unique

**Field.unique**

**True**라면, 이 필드는 테이블 전체에 걸쳐 고유해야 합니다.

이는 데이터베이스 레벨에서, model validation에 의해 시행됩니다. 만약 unique 필드에 중복된 값의 객체를 저장하려 한다면, **django.db.IntegrityError**가 모델의 **save()** method에 의해 발생할 것입니다.

이 옵션은 **ManyToManyField**와 **OneToOneField**를 제외하고 모든 필드에서 사용할 수 있습니다.

**unique**가 **True**일 때, **db_index**를 명시할 필요가 없습니다. 왜냐하면 **unique**는 index의 생성을 의미합니다.



## unique_for_date

**Field.unique_for_date**

이 필드가 date field 값에 대해 unique하게 하기 위해서 **DateField** or **DateTimeField**의 이름으로 이 값을 설정하십시오. 

예를 들어, **unique_for_date="pub_date"**를 갖고 있는 **title** field를 갖고 있다고 가정하면, 장고는 **title**과 **pub_date**를 동시에 갖고 있는 레코드의 입력을 허락하지 않을 것입니다.

> pub_date = models.DateField( ... )

> title = models.CharField( ... , unique_for_date="pub_date")

이 설정을 **DateTimeField**를 가리키도록 설정하면, 오직 필드의 날짜 부분만 고려된다는 점을 주의하십시오. 게다가 **USE_TZ**가 **True**라면, 객체가 저장될 때 `현재 시간대`에서 점검(check)이 수행될 것입니다.

이는 **Model.validate_unique()**에 의해 수행되는데, model validation 동안 시행되지만 데이터베이스 레벨에서는 시행되지 않습니다. 만약 **unique_for_date** 제약 조건이 **ModelForm**의 일부가 아닌 필드(예: 필드 중 하나가 제외되거나 **editable=False**)라면, **Model.validate_unique()**는 해당 제약 조건에 대한 유효성(validation) 검사를 생략합니다.



## unique_for_month

**Field.unique_for_month**

**unique_for_date**와 비슷하지만, 달(month)에 대해서 고유한 필드를 요구합니다.



## unique_for_year

**Field.unique_for_year**

**unique_for_date**와 **unique_for_month**와 비슷합니다.



## verbose_name

**Field.verbose_name**

해당 필드에 대해 사람이 읽기 쉬운 이름입니다. 만약 verbose name이 주어지지 않는다면, 장고는 자동적으로 field의 attribute 이름에서 undersocre을 띄어쓰기로 바꾸어 사용할 것입니다. `Verbose field names`를 보십시오.



## validators

**Field.validators**

해당 필드에 대해서 실행할 validator의 list입니다. 자세한 사항은 validator document를 보십시오.

**Registering and fetching lookups**

**Field**는 `lookup registration API`를 구현했습니다. API를 사용하여 필드 클래스에 사용할 수 있는 lookup과 필드에서 lookup을 가져오는 방법을 customize할 수 있다.
