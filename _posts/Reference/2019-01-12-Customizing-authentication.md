---
title: "Django 2.1 reference 번역: Customizing Authentication in Django"
categories: reference
tags:
- django 2.1
- reference
toc: true
---

# Customizing authentication in Django

장고와 함께 제공되는 인증은 대부분의 일반적인 경우에 충분하지만, 당신은 커스터마이징 할 필요가 있을 것입니다.(The authentication that comes with Django is good enough for most common cases, but you may have needs not met by the out-of-the-box defaults.) 프로젝트에서 인증 시스템을 커스터마이징 하기 위해서는 제공된 시스템의 어떤 지점이 확장 가능하고 대체 가능한지 이해해야 합니다. 이 문서는 인증 시스템을 어떻게 커스터마이즈 하는지에 대한 세부사항을 제공합니다.

인증 백엔드는 사용자 모델과 함께 저장된 유저 이름과 암호를 장고의 디폴트 서비스가 아닌 다른 서비스에 대해 인증해야 할 때 확장 가능한 시스템을 제공합니다.

모델에 사용자 지정 권한을 부여할 수 있는데, 이는 장고의 권한부여 시스템을 통해 확인할 수 있습니다. 

디폴트 **User** 모델을 extend할 수 있고, 완전히 커스터마이즈된 모델로 대체할 수 있습니다.

# Other authentication sources

다른 인증 소스 즉, 다른 소스의 사용자 이름과 암호 혹은 인증 방법 등을 연결할 필요가 있을 수 있습니다. 

예를 들어, 당신의 회사는 이미 모든 직원의 사용자 이름과 비밀번호를 저장하는 LDAP 설정을 이미 가지고 있습니다. 사용자가 LDAP와 장고 기반 애플리케이션에서 별도의 계정을 갖고 있다면 네트워크 관리자와 사용자 자신 모두에게 번거로울 것입니다.

따라서 이러한 상황을 해결하기 위해, 장고 인증 시스템을 통해 다른 인증 소스로 연결(plug-in)할 수 있습니다. 장고 기본 데이터베이스 스키마를 오버라이드 하거나 다른 시스템과 함께 디폴트 시스템을 사용할 수 있습니다.

장고에 포함된 인증 백엔드에 대한 자세한 내용은 `authentication backend reference`를 참조하십시오.

## Specifying authentication backends

이면에서 장고는 인증을 확인하는 "인증 백엔드"의 리스트를 유지하고 있습니다. `How to log a user in`에 기술된 것 처럼 누군가가 **django.contrib.auth.authenticate()**를 호출할 때, 장고는 모든 인증 백엔드에 걸쳐 인증을 시도합니다. 만약 첫 번째 인증 메소드가 실패하면, 장고는 두 번째를 시도하고, 모든 백앤드가 시도될 때까지 계속합니다.

사용할 인증 백엔드의 리스트는 설정의 **AUTHENTICATION_BACKENDS**에 명시되어 있습니다. 이는 인증 방법을 알고 있는 파이썬 클래스를 가리키는 파이썬 경로 이름의 리스트여야 합니다. 이 클래스들은 당신의 파이썬 경로 어디에나 있을 수 있습니다.

기본 값으로 **AUTHENTICATION_BACKENDS**는 다음으로 설정되어 있습니다:

    ['django.contrib.auth.backends.ModelBackend']

이는 장고 유저 데이터베이스를 확인하고 내장 사용 권한을 쿼리하는 기본 인증 백엔드입니다. 이것은 어떤 속도 제한 메커니즘을 통한 강력한 공격에 대한 보호를 제공하지 않습니다. 당신은 커스텀 인증 백엔드에서 자신의 속도 제한 메커니즘을 구현하거나 대부분의 웹 서버에서 제공하는 메커니즘을 사용할 수 있습니다.

**AUTHENTICATION_BACKENDS**의 순서가 중요합니다. 따라서 같은 유저 이름과 비밀번호가 다양한 백엔드에서 유효할 경우, 장고는 첫 번째 매칭에서 처리를 중지합니다.

백엔드에서 **PermissionDenied** 예외가 일어날 경우, 인증은 즉시 실패하게 됩니다. 장고는 뒤의 백엔드를 확인하지 않을 것입니다.

**Note**

일단 사용자가 인증을 받으면, 장고는 사용자 세션에서 사용자 인증을 위해 사용되었던 백엔드를 저장하고, 현재 인증된 사용자에 대한 엑세스가 필요할 때마다 해당 세션 동안 동일한 백엔드를 다시 사용한다. 이는 인증 소스가 세션 단위로 캐싱된다는 것을 의미하므로 **AUTHENTICATION_BACKENDS**를 변경할 경우, 사용자가 다른 방법을 사용하여 재인증을 하도록 강제할 필요가 있는 경우 세션 데이터를 삭제해야 합니다. 그렇게 하는 가장 간단한 방법은 **Session.objects.all().delete()**를 실행하는 것입니다.

## Writing an authentication backend

인증 백엔드는 두 개의 필수 메소드인 **get_user(user_id)**와 **authenticate(request, **credentials)**, 그리고 권한부여 메소드와 관련된 몇 가지 선택적 권한들을 구현하고 있는 클래스입니다.

**get_user** 메소드는 (사용자 이름, 데이터베이스 ID가 될 수는 있지만) 유저 객체의 primary key여야 하는 **user_id**를 인자로 받아들이고 유저 객체를 반환합니다.

**authenticate** 메소드는 **request** argument와 keyword arguments로 credential을 받아들입니다. 대부분의 경우, 이는 다음과 같을 것입니다:

```python
class MyBackend:
    def authenticate(self, request, username=None, password=None):
        # Check the username/password and return a user.
        ...
```

또한 다음과 같이 토큰을 인증할 수 있습니다:

```python
class MyBackend:
    def authenticate(self, request, token=None):
        # Check the token and return a user.
        ...
```

어느 쪽이든, **authenticate()**는 credentials를 확인하고 그것이 유효하다면 해당 credentials와 일치하는 유저 객체를 반환해야 합니다. 만약 이들이 유효하지 않다면, **None**을 반환해야 합니다.

**request**는 **HttpRequest**이고 (이것을 백엔드로 전달해주는) **authenticate()**에 전달되지 않았을 경우, 이것은 **None**이 될 수 없습니다.

장고 admin은 장고 유저 객체와 단단히 연결되어 있습니다. 이를 처리하는 가장 좋은 방법은 백엔드(예: LDAP 디렉토리, 외부 SQL 데이터베이스 등)를 위해 존재하는 각 사용자에 대한 장고 **User** 객체를 만드는 것입니다. 이 작업을 하기 위해 스크립트를 작성하거나, 사용자가 처음 로그인 할 때 **authenticate** 메소드를 사용할 수 있습니다.

**settings.py** 파일에 정의된 사용자 이름과 암호를 기준으로 인증하고, 사용자가 처음 인증할 때 장고 **User** 객체를 생성하는 백엔드 예제가 있습니다:

```python
from django.conf import settings
from django.contrib.auth.hashers import check_password
from django.contrib.auth.models import User
    
    class SettingsBackend:
    """
    Authenticate against the settings ADMIN_LOGIN and ADMIN_PASSWORD.
    
    Use the login name and a hash of the password. For example:
    
    ADMIN_LOGIN = 'admin'
    ADMIN_PASSWORD = 'pbkdf2_sha256$30000$Vo0VlMnkR4Bk$qEvtdyZRWTcOsCnI/oQ7fVOu1XAURIZYoOZ3iq8Dr4M='
    """
    
    def authenticate(self, request, username=None, password=None):
        login_valid = (settings.ADMIN_LOGIN == username)
        pwd_valid = check_password(password, settings.ADMIN_PASSWORD)
        if login_valid and pwd_valid:
            try:
                user = User.objects.get(username=username)
            except User.DoesNotExist:
                # Create a new user. There's no need to set a password
                # because only the password from settings.py is checked.
                user = User(username=username)
                user.is_staff = True
                user.is_superuser = True
                user.save()
            return user
        return None
    
    def get_user(self, user_id):
        try:
            return User.objects.get(pk=user_id)
        except User.DoesNotExist:
            return None
```

## Handling authorization in custom backends

커스텀 auth 백엔드는 그들 고유의 권한을 제공합니다.

유저 모델은 권한 조회 함수(**get_group_permissions(), get_all_permissions(), has_perm(), and has_module_perms()**)를 구현하고 있는 인증 백엔드로 이러한 함수들을 위임합니다.

사용자에게 부여된 권한은 모든 백엔드에서 반환되는 모든 권한의 상위 집합이 될 것입니다. 즉, 장고는 사용자에게 백엔드가 부여하는 권한을 부여합니다.

백엔드가 **has_perm()** 또는 **has_module_perms()**에서 **PermissionDenied** 예외를 일으킬 경우, 권한부여 시스템은 즉시 실패하며 장고는 다음 백엔드를 검사하지 않습니다.

위의 간단한 백엔드는 마법 같은 admin에 대한 권한을 상당히 간단하게 구현할 수 있습니다:

```python
class SettingsBackend:
    ...
    def has_perm(self, user_obj, perm, obj=None):
        return user_obj.username == settings.ADMIN_LOGIN
```

이는 위의 예에서 엑세스가 허가된 사용자에게 전체 권한을 부여합니다. 관련된 **django.contrib.auth.models.User** 함수에 전달된 같은 argument와 더불어, 백엔드 auth 함수는 모두 유저 객체(익명 사용자도 가능)를 받아들인다는 것을 유의하십시오.

전체 권한부여 구현은 django/contrib/auth/backends.py의 **ModelBackend**에서 찾아볼 수 있는데, 이는 디폴트 백엔드이고 대부분의 경우 **auth_permission** 테이블을 쿼리합니다. 백엔드 API의 일부에만 커스텀 동작을 제공하고자 하는 경우, 커스텀 백엔드에서 전체 API를 구현하는 대신 파이썬 상속 및 하위 클래스 **ModelBackend**를 이용할 수 있습니다.

### Authorization for anonymous users

익명 사용자는 인증되지 않은 사용자로, 유효한 인증 세부 정보를 제공하지 않았습니다. 그러나 그렇다고 해서 그들이 아무것도 할 수 있는 권한이 없는 것은 아닙니다. 가장 기본적인 수준에서, 대부분의 웹사이트는 익명 사용자에게 대부분의 사이트를 탐색할 수 있는 권한을 부여하며, 많은 웹사이트들은 익명의 댓글 등을 올릴 수 있도록 허용합니다.

장고의 권한 프레임워크는 익명 사용자에 대한 권한을 저장할 공간이 없습니다. 그러나 인증 백엔드로 전달되는 사용자 객체는 **django.contrib.auth.models.AnonymousUser** 객체가 될 수 있고, 백엔드에서 익명 사용자 객체에 대한 커스텀 권한부여 동작을 지정할 수 있습니다. 이는 익명 엑세스를 제어하는 경우와 같이 어떤 설정이 필요할 때보다는, 권한부여와 관련된 모든 질문을 auth 백엔드에 위임하는 재사용 가능한 앱의 작성자에게 특히 유용합니다.

### Authorization for inactive users

비활성 사용자는 **is_active** 필드가 **False**로 설정된 사용자입니다. **ModelBackend**와 **RemoteUserBackend** 인증 백엔드는 이러한 사용자가 인증하는 것을 금지합니다. 커스텀 유저 모델이 **is_active** 필드가 없을 경우, 모든 유저는 인증이 허가됩니다.

비활성 사용자를 인증하고 싶은 경우 **AllowAllUsersModelBackend** 또는 **AllowAllUsersRemoteUserBackend**를 사용할 수 있습니다.

권한 시스템에서 익명 사용자에 대한 지원은 비활성 인증 사용자는 할 수 없는 반면 익명 사용자는 무언가를 할 수 있는 권한을 갖는 시나리오에서 사용됩니다.

당신 고유의 백엔드 권한 메소드에서 **is_active** attribute의 사용자에 대한 테스트를 진행하는 것을 잊지 마십시오.

### Handling object permissions

장고의 권한 프레임워크는 객체 권한의 핵심적인 구현은 없지만 객체 권한에 대한 기반을 갖고 있습니다. 즉, 객체 권한을 확인하는 것은 항상 **False**나 빈 리스트를 반환합니다(실행된 검사에 따라 다름). 인증 백엔드는 **obj**와 권한부여 메소드와 관련된 각각의 객체에 대한 **user_obj**를 keyword parameter로 받아들이고 필요에 따라 객체 레벨 권한을 반환할 수 있습니다.

# Custom permissions

주어진 모델 객체에 대한 커스텀 권한을 생성하기 위해서 **permissions** model Meta attribute를 사용하십시오.

이 예시는 **Task** 모델이 두 개의 커스텀 권한을 생성합니다. 즉, 특정한 당신의 앱에서 사용자가 **Task** 인스턴스와 무언가를 할 수 있는, 또는 없는 액션을 생성합니다:

```
class Task(models.Model):
    ...
    class Meta:
        permissions = (
            ("change_task_status", "Can change the status of tasks"),
            ("close_task", "Can remove a task by setting its status as closed"),
        )
```

이것이 하는 유일한 행동은 **manage.py migrate**를 실행할 때 추가적인 권한을 생성합니다 (권한을 생성하는 함수는 **post_migrate** 신호와 연결됩니다). 사용자가 앱이 제공하는 기능에 접근하려고 할 때 (작업의 상태 변경 또는 작업 종료), 당신의 코드는 이러한 권한의 값을 확인하는 역할을 합니다. 위의 예를 계속 진행한다면, 사용자가 작업을 닫을 수 있는지 다음과 같이 점거합니다:

```python
user.has_perm('app.close_task')
```

# Extending **the existing User model**

자신의 모델을 대체하지 않고 디폴트 **User** 모델을 extend하는 두 가지 방법이 있습니다. 필요한 변경사항이 순전히 행동적이어서 데이터베이스에 저장된 내용을 변경할 필요가 없는 경우, **User**를 기반으로 프록시 모델을 작성할 수 있습니다. 이것은 디폴트 정렬, 커스텀 managers, 혹은 커스텀 모델 메소드를 포함하여 프록시 모델에 의해 제공되는 모든 기능을 허용합니다.

만약 **User**와 관련된 정보를 저장하고자 하는 경우, 추가 정보를 위한 필드가 포함된 모델에 **OneToOneField**를 사용할 수 있습니다. 이 일대일 모델은 사이트 사용자에 대한 non-auth와 관련된 정보들을 저장할 수 있기 때문에 종종 프로필 모델이라 불립니다. 예를 들어 Employee 모델을 생성할 수 있습니다:

```python
from django.contrib.auth.models import User
    
class Employee(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    department = models.CharField(max_length=100)
```

User 및 Employee 모델을 모두 갖고 있는 기존 Employee Fred Smith를 가정하자. 당신은 장고 표준 관계 모델 규약(Django's standard related model convention)을 사용하여 관련된 정보에 접근할 수 있습니다:

    >>> u = User.objects.get(username='fsmith')
    >>> freds_department = u.employee.department

프로필 모델의 필드를 admin의 사용자 페이지에 추가하려면 앱의 **admin.py** 안에 **InlineModelAdmin**(우리의 예에서는 **StackedInline**)을 정의하고, 그것을 **User** 클래스와 함께 등록된 **UserAdmin** 클래스에 추가하십시오:

```
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from django.contrib.auth.models import User
    
from my_user_profile_app.models import Employee
    
# Define an inline admin descriptor for Employee model
# which acts a bit like a singleton
class EmployeeInline(admin.StackedInline):
    model = Employee
    can_delete = False
    verbose_name_plural = 'employee'
    
# Define a new User admin
class UserAdmin(BaseUserAdmin):
    inlines = (EmployeeInline,)
    
# Re-register UserAdmin
admin.site.unregister(User)
admin.site.register(User, UserAdmin)
```

이러한 프로필 모델은 어떤 면에서도 특별하지 않습니다 - 단지 사용자 모델과 일대일 링크를 가진 장고 모델 일 뿐입니다. 따라서 그들은 유저가 생성될 때 자동으로 생성되는 것이 아니라 **django.db.models.signals.post_save**를 사용하여 관련 모델을 적절하게 생성하거나 업데이트 할 수 있습니다.

관계된 모델을 사용하는 것은 관련된 데이터를 받기 위해 추가적인 쿼리나 조인을 발생시킵니다. 필요에 따라 관련 필드를 포함하는 커스텀 유저 모델이 더 나은 선택이 될 수 있지만, 프로젝트 앱 안의 디폴트 유저 모델과의 기존 관계는 추가적인 데이터베이스 로드의 원인이 됩니다.

# Substituting a custom User model

어떤 종류의 프로젝트에는 장고의 내장 **User** 모델이 항상 적절한 것은 아닙니다. 예를 들어, 어떤 사이트에서는 이메일 주소를 사용자 이름 대신 식별 토큰으로 사용하는 것이 더 합당합니다.

장고는 커스텀 모델을 참조하는 **AUTH_USER_MODEL**의 값을 조정하여 디폴트 유저 모델을 오버라이드 할 수 있게 해줍니다:

    AUTH_USER_MODEL = 'myapp.MyUser'

이 쌍은 장고 앱의 이름(**INSTALLED_APPS** 안에 있어야 합니다)과 유저 모델로 사용하고 싶은 장고 모델의 이름을 나타냅니다.

## Using a custom user model when starting a project

만약 새로운 프로젝트를 시작하고 있다면 디폴트 **User** 모델이 충분할지라도 커스텀 유저 모델을 작성하는 것을 매우 추천합니다. 이 모델은 디폴트 유저 모델과 동일하게 작동하지만, 필요가 생긴다면 미래에 그것을 커스터마이징 할 수 있습니다:

```python
from django.contrib.auth.models import AbstractUser
    
class User(AbstractUser):
    pass
```

**AUTH_USER_MODEL**을 지정하는 것을 잊지 마십시오. migrations를 만들거나 첫 번째 **manage.py migrate**를 실행하기 전에 이를 하십시오.

또한 앱의 admin.py에 모델을 등록하십시오:

```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import User
    
admin.site.register(User, UserAdmin)
```

## Changing to a custom user model mid-project

데이터베이스 테이블을 만든 후 **AUTH_USER_MODEL**를 변경하는 것은 foreign key와 다대다 관계 등에 영향을 미치기 때문에 훨씬 어렵습니다.

이러한 변경은 자동적으로 완료되지 않으며 스키마를 수동적으로 수정하고 이전 사용자 테이블에서 데이터를 이동하며 일부 migration을 수동으로 다시 적용해야 합니다. 이 단계의 개요는 [#24313](https://code.djangoproject.com/ticket/25313)를 참조하십시오.

스왑 가능한 모델에 대한 장고의 동적 종속성 특징의 한계(Django's dynamic dependency feature) 때문에, **AUTH_USER_MODEL**에 의해 참조된 모델은 앱의 첫 번째 migration(보통 **0001_initial**)에서 생성되어야 합니다. 그렇지 않으면 종속성 문제가 발생할 것입니다.

또한 동적 종속성으로 인해 장고가 종속성 루프를 자동적으로 끊을 수 없기 때문에 migration을 실행할 때 **CircularDependencyError**에 부딪힐 수 있습니다. 이 오류를 보게 된다면, 유저 모델에 종속적인 모델을 두 번째 migration으로 이동시켜 루프를 끊어야 합니다. (이것이 일반적으로 어떻게 해결 되는지 보고 싶다면, foreignkey가 서로를 향하는 두 개의 일반 모델을 만들어보고 makemigrations가 순환 종속성을 어떻게 해결하는지 볼 수 있습니다.)

## Reusable apps and AUTH_USER_MODEL

재사용 가능한 앱은 커스텀 모델을 구현해서는 안됩니다. 프로젝트는 많은 앱을 사용할 수 있으며, 커스텀 유저 모델을 구현한 두 개의 재사용 가능한 앱을 함께 사용할 수 없습니다. 앱에 사용자별 정보를 저장해야 할 경우, 아래 기술된 바와 같이 **ForeginKey** 또는 **OneToOneField**를 **settings.AUTH_USER_MODEL**에 사용하십시오.

## Referencing the User model

**User**를 직접 참조하는 경우 (예를 들어 foreign key로 참조하는 경우), **AUTH_USER_MODEL** 설정이 다른 유저 모델로 변경된 프로젝트에서는 코드가 작동하지 않습니다.

**get_user_model()**

**User**를 직접 참조하는 대신 **django.contrib.auth.get_user_model()**를 사용하여 유저 모델을 참조해야 합니다. 이 메소드는 현재 활성화된 사용자 모델 즉, 지정된 경우 커스텀 유저 모델이, 아니면 **User**가 반환됩니다.

foreign key 혹은 다대다 관계를 유저 모델에 정의할 때는 **AUTH_USER_MODEL** 설정을 사용하여 커스텀 모델을 지정해야 합니다. 예를 보십시오:

```python
from django.conf import settings
from django.db import models
    
class Article(models.Model):
    author = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
    )
```

유저 모델에서 보내지는 신호에 연결할 때는 **AUTH_USER_MODEL** 설정을 사용해 커스텀 모델을 지정해야 합니다. 예를 보십시오:

```python
from django.conf import settings
from django.db.models.signals import post_save
    
def post_save_receiver(sender, instance, created, **kwargs):
    pass
    
post_save.connect(post_save_receiver, sender=settings.AUTH_USER_MODEL)
```

일반적으로 말해서, import 시에 실행되는 코드에서는 AUTH_USER_MODEL 설정을 사용해 유저 모델을 참조하는 것이 가장 쉽지만, 장고가 모델을 import하는 동안에 **get_user_model()**를 호출하는 것이 가능하므로 **models.ForeignKey(get_user_model(), ...)**와 같이 사용할 수 있습니다.

예를 들어, **@override_settings(AUTH_USER_MODEL=...)**를 사용하여 다양한 유저 모델로 앱을 테스트하고 **get_user_model()**의 결과를 모듈 레벨 변수에 캐시하는 경우, 캐시를 지우기 위해서는 **setting_changed** 신호를 listen해야 합니다. 예를 들면 다음과 같습니다:

```python
from django.apps import apps
from django.contrib.auth import get_user_model
from django.core.signals import setting_changed
from django.dispatch import receiver
    
@receiver(setting_changed)
def user_model_swapped(**kwargs):
    if kwargs['setting'] == 'AUTH_USER_MODEL':
        apps.clear_cache()
        from myapp import some_module
        some_module.UserModel = get_user_model()
```

## Specifying a custom user model

- **Model design considerations**

    커스텀 유저 모델에서 인증과 직접적 관련이 없는 정보를 취급하기 전에 신중하게 생각해 보십시오.

    유저 모델과 관계가 있는 모델에 앱별 사용자 정보를 저장하는 것이 나을 수 있습니다. 이를 통해 각 앱은 다른 앱과의 충돌 위험을 감수하지 않고 자체 사용자 데이터 요구사항을 지정할 수 있습니다. 한편 이와 관련된 정보의 쿼리는 데이터베이스 조인이 수반되어 성능에 영향을 미칠 수 있습니다.

장고는 당신의 커스텀 모델이 몇 가지 최소 요건을 충족할 것을 기대합니다.

기본 인증 백엔드를 사용하는 경우 모델에는 식별 목적으로 사용할 수 있는 단일 고유 필드가 있어야 합니다. 이것은 사용자 이름, 이메일 주소 또는 다른 고유한 특성이 될 수 있습니다. 비고유 사용자 이름 필드는 이를 지원할 수 있는 사용자 지정 인증 백엔드를 사용할 경우 허용됩니다.

규격에 맞는 유저 모델을 구성하는 가장 쉬운 방법은 **AbstractBaseUser**로부터 상속하는 것입니다. **AbstractBaseUser**는 해시된 암호와 토큰화된 암호 재설정을 포함한 사용자 모델의 핵심 구현을 제공합니다. 그런 다음 몇 가지 주요 구현 세부 정보를 구현해야 합니다.

***class* models.CustomUser**

**USERNAME_FIELD**

고유 식별자로 사용되는 사용자 모델의 필드 이름을 기술하는 문자열입니다. 이는 보통 어떤 종류의 사용자 이름이 될 수 있지만, 이메일 주소나 다른 독특한 식별자가 될 수도 있습니다. 고유하지 않은 사용자 이름을 지원할 수 있는 커스텀 인증 백엔드를 사용하지 않는 한, 필드는 고유해야 합니다(즉, **unique=True**를 갖고 있어야 합니다)

다음 예시에서 필드 **identifier**는 식별자 필드로 사용됩니다:

```python
class MyUser(AbstractBaseUser):
    identifier = models.CharField(max_length=40, unique=True)
    ...
    USERNAME_FIELD = 'identifier'
```

**EMIAL_FIELD**

**User** 모델의 이메일 필드 이름을 기술하는 이름입니다. 이 값은 **get_email_field_name()**에 의해 반환됩니다.

**REQUIRED_FIELDS**

**createsuperuser** management 명령을 통해 사용자를 생성할 때 프롬포트 될 필드 이름의 목록입니다. 사용자는 각 필드에 값을 입력하라는 메세지가 표시될 것입니다. 여기에는 **blank**가 **False**이거나 정의되지 않은 필드, 또는 유저가 대화식으로(interactively) 생성될 때 당신이 원하는 추가 필드가 포함될 수 있습니다. **REQUIRED_FIELDS**는 admin에서 유저를 생성하는 것과 같이 장고의 다른 부분에 영향을 미치지 않습니다.

예를 들어, 여기에는 생년월일과 키라는 두 가지 필수 필드를 정의하는 유저 모델에 대한 부분적 정의가 있습니다:

```python
   class MyUser(AbstractBaseUser):
    ...
    date_of_birth = models.DateField()
    height = models.FloatField()
    ...
    REQUIRED_FIELDS = ['date_of_birth', 'height']
```

- **Note**

    **REQUIRED_FIELDS**는 유저 모델의 필수 필드를 포함해야 하지만, **USERNAME_FIELD** 또는 **password**는 항상 프롬포트되므로 이들을 포함하면 안됩니다.

**is_active**

사용자를 "활성"으로 간주하는지 여부를 나타내는 boolean attribute입니다. 이 attribute는 디폴트로 **True**인 **AbstractBaseUser**에 있는 attribute로써 제공된다. 당신이 그것을 어떻게 구현할지 선택하는 것은 당신이 선택한 auth 백엔드의 세부 사항에 달려있습니다. 자세한 사항에 대해서는 `is_active attribute on the built-in user model` 문서를 보십시오.

**get_full_name()**

선택적입니다. 사용자의 전체 이름과 같은 더 긴 공식 식별자입니다. 구현될 경우, 이것은 **django.contrib.admin**의 객체 히스토리의 이름을 나타냅니다.

**get_short_name()**

선택적입니다. 사용자 이름(first name)과 같은 간략하고 비공식적인 식별자입니다. 구현될 경우, 이것은 **django.contrib.admin**의 헤더에 있는 사용자 인사말의 유저 이름을 대체합니다.

- **Changed in Django 2.0**

    이전 버전에서, 상속은 **get_short_name()**과 **get_full_name()**을 구현해야 했습니다. 왜냐하면 **AbstractBaseUser**가 **NotImplementedError**를 일으키기 때문입니다.

- Importing AbstractBaseUser

    **AbstractBaseUser**과 **BaseUserManager**는 **django.contrib.auth.base_user**에서 import할 수 있으므로 **INSTALLED_APPS**에 **django.contrib.auth**를 포함하지 않아도 이들을 import할 수 있습니다.

다음 attributes와 methods는 **AbstractBaseUser**의 하위 클래스에서 사용 가능 합니다:

***class* models.AbstractBaseUser**

**get_username()**

**USERNAME_FIELD**에 지명된 필드 값을 반환합니다.

**clean()**

**normalize_username()**을 호출하여 사용자 이름을 표준화합니다. 이 메소드를 오버라이드할 경우, 표준화를 하기 위해 **super()**을 호출하는 것을 잊지 마십시오.

***classmethod* get_email_field_name()**

**EMAIL_FIELD** attribute에 명시된 이메일 필드의 이름을 반환합니다. **EMAIL_FIELD**가 없을 경우 디폴트 값은 '**email**'입니다.

***classmethod* normalize_username(*username*)**

사용자 이름에 NFKC Unicode 표준화를 적용하여 다른 Unicode지만 겉보기에 같은 문자들을 동일한 것으로 간주하게 만듭니다.

**is_authenticated**

읽기 전용 attribute로 항상 **True**입니다. (**AnonymousUser.is_authenticated**는 항상 False로 반대입니다.) 이것은 사용자가 인증 되었는지 확인하는 방법입니다. 이는 어떠한 권한도 의미하지 않으며 사용자가 활성화돼 있거나 유효한 세션이 있는지 확인하지 않습니다. 유저가 (현재 로그인 된 유저를 나타내는) **AuthenticationMiddleware**에 의해 생겼는지를 알기 위해 **request.user**에서 이 attribute를 종종 찾을 것이지만, 이 attribute는 모든 **User** 인스턴스에 대해 **True**라는 것을 알아야 합니다.

**is_anonymous**

읽기 전용 attribute로 항상 **False**입니다. 이는 User와 AnonymousUser 객체를 구분하기 위함입니다. 일반적으로 이 attribute에 애해 **is_authenticated**를 선호할 것입니다.

**set_password(*raw_password*)**

패스워드 해싱을 처리하면서 사용자의 암호를 주어진 raw 문자열로 설정합니다. **AbstractBaseUser** 객체는 저장하지 않습니다.

raw_password가 **None**일 경우, 비밀번호는 **set_unusable_password()**를 사용한 것처럼 사용 불가능한 암호로 지정됩니다.

**check_password(*raw_password*)**

주어진 raw 문자열이 유저에 대해 올바른 경우 **True**를 반환합니다. (비교를 할 때 패스워드 해싱이 사용됩니다.)

**set_unusable_password()**

비밀번호가 설정되지 않은 사용자로 표시합니다. 이는 비밀번호로 빈 문자열을 갖는 것과 같지 않습니다. 이 사용자에 대해 **check_password()**는 **True**를 절대 반환하지 않습니다. **AbstractBaseUser** 객체를 저장하지 않습니다.

**has_usable_password()**

set_unusable_password()가 유저에 대해 호출되었을 경우 **False**를 반환합니다.

**has_session_auth_hash()**

패스워드 필드의 HMAC를 반환합니다. `Session invalidation on password change`에서 사용됩니다.

AbstractUser는 AbstractBaseUser를 상속받습니다:

***class* models.AbstractUser**

**clean()**

**BaseUserManager.normalize_email()**을 호출하여 이메일을 표준화합니다. 이 메소드를 오버라이드 할 경우, 표준화를 하기 위해 **super()**의 호출을 잊지 마십시오.

## Writing a manager for a custom user model

또한 사용자 모델에 대한 커스텀 매니저를 정의해야 합니다. 당신의 유저 모델이 장고 디폴트 유저 모델과 같이 **username, email, is_staff, is_active, is_superuser, last_login,** 그리고 **date_joined** 필드를 갖고 있을 경우, 장고 **UserManager**를 쓸 수 있습니다; 하지만 유저 모델이 다른 필드를 갖고 있을 경우, 다음 두 가지 추가 메소드를 제공하는 **BaseUserManager**를 extend하는 커스텀 매니저를 정의해야 합니다. 

***class* models.CustomUserManager**

**create_user(**username_field*, password=None, **other_fields*)**

create_user()의 프로토타입은 사용자 이름 필드와 모든 필수 필드를 arguments로 받습니다. 예를 들어, 당신의 유저 모델이 사용자 이름 필드로 **email**을 사용하고 **date_of_birth**를 필수 필드로 갖고 있을 경우**, create_user**는 다음과 같이 정의해야 합니다:

```python
def create_user(self, email, date_of_birth, password=None):
    # create user here
    ...
```

**create_superuser(**username_field*, password, **other_fields*)**

create_superuser()의 프로토타입은 사용자 이름 필드와 모든 필수 필드를 argument로 받습니다. 예를 들어, 당신의 유저 모델이 사용자 이름 필드로 **email**을 사용하고 **date_of_birth**를 필수 필드로 갖고 있을 경우**, create_user**는 다음과 같이 정의해야 합니다:

```python
def create_superuser(self, email, date_of_birth, password):
    # create superuser here
    ...
```

**create_user()**과 달리, **create_superuser()**은 반드시 호출자에게 암호를 제공해야 합니다.

**USERNAME_FIELD** 또는 **REQUIRED_FIELDS**의 **ForeignKey**에 대해, 이러한 메소드는 기존 인스턴스의 **to_field**(기본적으로 **primary_key**) 값을 수신합니다.

BaseUserManager는 다음과 같은 유틸리티 메소드를 제공합니다:

***class* models.BaseUserManager**

***classmethod* normalize_email(*email*)**

이메일 주소의 도메인 부분을 소문자로 바꿈으로써 전자 메일 주소를 표준화합니다.

**get_by_natural_key(*username*)**

USERNAME_FIELD에서 지정한 필드의 값을 사용하여 유저 인스턴스를 검색합니다.

**make_random_password(*length=10, allowed_chars='abcdefghjkmnpqrstuvwxyzABCDEFGHJKLMNPQRSTUVWXYZ23456789'*)**

주어진 길이와 주어진 허용 문자열을 가진 임의의 비밀번호를 반환합니다. allowed_chars의 기본값은 다음을 다음과 같이 유저에게 혼란을 야기할 수 있는 문자들은 포함하지 않습니다:

- i, l, I, and 1 (소문자 i, 소문자 L, 대문자 i, 그리고 숫자 일)
- o, O, and 0 (소문자 o, 대문자 o, and 숫자 영)

## Extending Django's default User

만약 당신이 장고 유저 모델에 전적으로 만족하고, 단지 약간의 프로필 정보를 추가하기 원한다면, 당신은 단순히 **django.contrib.auth.models.AbstractUser**를 상속하여 커스텀 프로필 필드를 추가할 수 있습니다. (저희는 `Specifying a custom user model`의 "모델 디자인 고려사항"에 기술된 것과 같이 커스텀 유저 모델을 분리하는 것을 추천함에도 불구하고 말입니다.) AbstractUser는 추상적인 모델로서 디폴트 모델에 대한 완전한 구현을 제공합니다.

## Custom users and the built-in auth forms

장고의 내장 폼과 뷰는 그들이 작업하고 있는 유저 모델에 특정 가정을 합니다.

다음 폼은 **AbstractBaseUser**의 어떠한 하위 클래스에서도 사용 가능합니다:

- **AuthenticationForm**: **USERNAME_FIELD**로 명시된 사용자 이름 필드를 사용합니다.
- **SetPasswordForm**
- **PasswordChangeForm**
- **AdminPasswordChangeForm**

다음과 같은 폼은 유저 모델에 대한 가정을 하고, 그런 가정이 충족될 경우 다음과 같이 사용할 수 있습니다:

- **PasswordResetForm**: 유저 모델에는, 유저 이메일 주소를 저장하는, **get_email_field_name()**에 의해 반환되는 이름(기본값으로 **email**)의 필드가 있고, 이 필드는 사용자 식별에 사용됩니다. 또한 비활성 사용자들이 비밀번호를 재설정하는 것을 막기 위해 is_active라는 boolean 필드를 갖고 있습니다.

마지막으로, 다음과 같은 폼들이 **User**와 연관되어 있으며, 커스텀 유저 모델과의 작업을 위해 다시 쓰거나 extend해야 합니다:

- UserCreationForm
- UserChangeForm

커스텀 모델이 AbstractUser의 단순한 하위 클래스인 경우, 다음과 같은 방법으로 이러한 폼을 extend할 수 있습니다:

```python
from django.contrib.auth.forms import UserCreationForm
from myapp.models import CustomUser
    
class CustomUserCreationForm(UserCreationForm):
    
    class Meta(UserCreationForm.Meta):
        model = CustomUser
        fields = UserCreationForm.Meta.fields + ('custom_field',)
```

## Custom users and django.contrib.admin

커스텀 유저 모델을 admin과 함께 작동하려면 유저 모델은 몇 가지 추가 attributes와 메소드를 정의해야 합니다. 이러한 메소드를 통해 admin은 다음과 같은 방법으로 사용자의 admin 내용에 대한 액세스를 제어할 수 있습니다.

***class* models.CustomUser**

**is_staff**

사용자가 admin 사이트에 접근이 허용될 경우 **True**를 반환합니다.

**is_active**

사용자 계정이 현재 활성화된 경우 **True**를 반환합니다.

**has_perm(perm, obj=None):**

사용자가 해당 권한이 있을 경우 **True**를 반환합니다. **obj**가 주어진 경우, 권한은 해당 객체 인스턴스에 대해 확인됩니다.

has_module_perms(app_label):

사용자가 주어진 앱의 모델에 접근할 권한이 있는 경우 **True**를 반환합니다.

당신은 또한 커스텀 유저 모델을 admin에 등록해야 합니다. 커스텀 유저 모델이 **django.contrib.auth.models**를 extend하는 경우, 당신은 장고의 기본 **django.contrib.auth.admin.UserAdmin** 클래스를 사용할 수 있습니다. 그러나 당신의 유저 모델이 **AbstractBaseUser**를 extend하는 경우, 당신은 커스텀 **ModelAdmin** 클래스를 정의해야 합니다. 이는 디폴트 **django.contrib.auth.admin.UserAdmin**을 상속받아 할 수 있습니다; 그러나 당신은 **django.contrib.auth.models.AbstractUser**의 필드 중 당신의 커스텀 유저 클래스에 없는 필드를 참조하는 모든 정의된 부분을 오버라이드 해야합니다.

## Custom users and permissions

장고의 권한 프레임워크를 당신의 유저 클래스에 쉽게 포함하기 위해, 장고는 PermissionsMixin을 제공합니다. 이것은 당신의 유저 모델 클래스 계층에 포함할 수 있는 추상 모델이며, 장고의 권한 모델을 지원하는데 필요한 모든 메소드와 데이터베이스 필드를 제공합니다. 

**PermissionsMixin**은 다음과 같은 메소드와 attributes를 제공합니다:

***class* models.PermissionsMixin**

**is_superuser**

Boolean. 명시적으로 할당할 필요 없이 이 사용자가 모든 권한을 갖고 있음을 명시합니다.

**get_group_permissions(*obj=None*)**

자신의 그룹을 통해 사용자가 갖고 있는 일련의 권한들의 문자열을 반환합니다.

**obj**가 전달된 경우, 이 특정 객체에 대한 사용 권한만을 반환합니다. 

**get_all_permissions(*obj=None*)**

그룹 및 사용자 권한을 통해 사용자가 갖고 있는 일련의 사용 권한 문자열을 반환합니다.

**obj**가 전달된 경우, 이 특정 객체에 대한 사용 권한만을 반환합니다.

**has_perm(*perm, obj=None*)**

사용자가 지정한 권한들을 갖고 있는 경우 **True**를 반환합니다. 여기서 각각의 권한은 "**<app label>.<permission codename>**"의 형태로 주어집니다. 만약 **User.is_active**와 **is_superuser**가 모두 **True**일 경우, 이 메소드는 항상 **True**를 반환합니다.

**obj**가 전달될 경우, 이 메소드는 모델에 대한 사용 권한이 아닌, 특정 객체에 대한 사용 권한을 검사합니다.

**has_module_perms(*package_name*)**

사용자가 지정된 패키지(Django app label)에 사용 권한을 갖고 있는 경우 **True**를 반환합니다. **User.is_active**와 **is_superuser**가 모두 **True**이면 이 메소드는 항상 **True**를 반환합니다.

- **PermissionsMixin and ModelBackend**

    **PermissionMixin**을 포함하지 않으면 **ModelBackend**에서 권한과 관련된 메소드를 호출하지 않도록 해야합니다. **ModelBackend**는 유저 모델에서 특정 필드를 사용할 수 있다고 가정합니다. 유저 모델이 이러한 필드를 제공하지 않는 경우, 사용 권한을 검사할 때 데이터베이스 오류가 발생할 것입니다. 

## Custom users and proxy models

커스텀 모델의 한 가지 제약은 커스텀 모델을 설치할 경우 **User**를 extend하는 프록시 모델이 파괴될 것이라는 점입니다. 프록시 모델은 concrete base 클래스에 기반해야 합니다; 커스텀 유저 모델을 정의함으로써 당신은 기본 클래스를 신뢰성 있게 식별할 수 있는 장고 기능을 제거합니다.

프로젝트에서 프록시 모델을 사용하는 경우, 프로젝트에서 사용중인 사용자 모델을 extend하기 위해 프록시를 수정하거나 프록시 동작을 **User** 하위 클래스로 합쳐야 합니다.

## A full example

여기에 admin-compliant 커스텀 유저 앱의 예가 있습니다. 이 유저 모델은 이메일 주소를 사용자 이름으로 사용하며, 필수적으로 생년월일을 갖고 있습니다; 사용자 계정의 간단한 **admin** flag를 넘어서서 어떠한 권한도 확인하지 않습니다.(it provides no permission checking, beyond a simple admin flag on the user account.) 이 모델은 사용자 생성 폼을 제외하고 모든 내장 auth 폼 및 뷰와 호환됩니다. 이 예는 대부분의 요소들이 어떻게 연동되는지 설명하지만, 생산용 프로젝트로 직접 복사하기 위한 용도는 아닙니다.

이 코드는 커스텀 인증 시스템의 **models.py**에 있습니다:

```python
from django.db import models
from django.contrib.auth.models import (
    BaseUserManager, AbstractBaseUser
)
    
    
class MyUserManager(BaseUserManager):
    def create_user(self, email, date_of_birth, password=None):
        """
        Creates and saves a User with the given email, date of
        birth and password.
        """
        if not email:
            raise ValueError('Users must have an email address')
    
        user = self.model(
            email=self.normalize_email(email),
            date_of_birth=date_of_birth,
        )
    
        user.set_password(password)
        user.save(using=self._db)
        return user
    
    def create_superuser(self, email, date_of_birth, password):
        """
        Creates and saves a superuser with the given email, date of
        birth and password.
        """
        user = self.create_user(
            email,
            password=password,
            date_of_birth=date_of_birth,
        )
        user.is_admin = True
        user.save(using=self._db)
        return user
    
    
class MyUser(AbstractBaseUser):
    email = models.EmailField(
        verbose_name='email address',
        max_length=255,
        unique=True,
    )
    date_of_birth = models.DateField()
    is_active = models.BooleanField(default=True)
    is_admin = models.BooleanField(default=False)
    
    objects = MyUserManager()
    
    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['date_of_birth']
    
    def __str__(self):
        return self.email
    
    def has_perm(self, perm, obj=None):
        "Does the user have a specific permission?"
        # Simplest possible answer: Yes, always
        return True
    
    def has_module_perms(self, app_label):
        "Does the user have permissions to view the app `app_label`?"
        # Simplest possible answer: Yes, always
        return True
    
    @property
    def is_staff(self):
        "Is the user a member of staff?"
        # Simplest possible answer: All admins are staff
        return self.is_admin
```

그리고 커스텀 유저 모델은 장고 admin에 등록하기 위해, 다음과 같은 코드가 앱의 **admin.py** 파일에 있어야 합니다:

```python
from django import forms
from django.contrib import admin
from django.contrib.auth.models import Group
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from django.contrib.auth.forms import ReadOnlyPasswordHashField
    
from customauth.models import MyUser
    
    
class UserCreationForm(forms.ModelForm):
    """A form for creating new users. Includes all the required
    fields, plus a repeated password."""
    password1 = forms.CharField(label='Password', widget=forms.PasswordInput)
    password2 = forms.CharField(label='Password confirmation', widget=forms.PasswordInput)
    
    class Meta:
        model = MyUser
        fields = ('email', 'date_of_birth')
    
    def clean_password2(self):
        # Check that the two password entries match
        password1 = self.cleaned_data.get("password1")
        password2 = self.cleaned_data.get("password2")
        if password1 and password2 and password1 != password2:
            raise forms.ValidationError("Passwords don't match")
        return password2
    
    def save(self, commit=True):
        # Save the provided password in hashed format
        user = super().save(commit=False)
        user.set_password(self.cleaned_data["password1"])
        if commit:
            user.save()
        return user
    
    
class UserChangeForm(forms.ModelForm):
    """A form for updating users. Includes all the fields on
    the user, but replaces the password field with admin's
    password hash display field.
    """
    password = ReadOnlyPasswordHashField()
    
    class Meta:
        model = MyUser
        fields = ('email', 'password', 'date_of_birth', 'is_active', 'is_admin')

    def clean_password(self):
        # Regardless of what the user provides, return the initial value.
        # This is done here, rather than on the field, because the
        # field does not have access to the initial value
        return self.initial["password"]
    
    
class UserAdmin(BaseUserAdmin):
    # The forms to add and change user instances
    form = UserChangeForm
    add_form = UserCreationForm
    
    # The fields to be used in displaying the User model.
    # These override the definitions on the base UserAdmin
    # that reference specific fields on auth.User.
    list_display = ('email', 'date_of_birth', 'is_admin')
    list_filter = ('is_admin',)
    fieldsets = (
        (None, {'fields': ('email', 'password')}),
        ('Personal info', {'fields': ('date_of_birth',)}),
        ('Permissions', {'fields': ('is_admin',)}),
    )
    # add_fieldsets is not a standard ModelAdmin attribute. UserAdmin
    # overrides get_fieldsets to use this attribute when creating a user.
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': ('email', 'date_of_birth', 'password1', 'password2')}
        ),
    )
    search_fields = ('email',)
    ordering = ('email',)
    filter_horizontal = ()
    
# Now register the new UserAdmin...
admin.site.register(MyUser, UserAdmin)
# ... and, since we're not using Django's built-in permissions,
# unregister the Group model from admin.
admin.site.unregister(Group)
```
최종적으로 **settings.py**의 **AUTH_USER_MODEL** 설정을 이용하여 프로젝트의 디폴트 유저 모델을 커스텀 모델로 명시하십시오:

    AUTH_USER_MODEL = 'customauth.MyUser'
