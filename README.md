
**Django Project with Python3 and MySQL**
---

#### Index 
- [1. Overview](#1-overview)   
- [2. Virtualenv and Django Installation](#2-virtualenv-and-django-installation)  
- [3. Create Django Application](#4-create-django-application)  
- [4. Setup MySQL Database](#3-setup-mysql-database)  
- [5. Running Django Application](#5-running-django-application)  
- [6. Setting Up Database Case Study](#7-setting-up-database-case-study)  
- [7. Create an Admin Superuser](#6-create-an-admin-superuser)  
- [8. Accessing Database API from Command Line](#8-accessing-database-api-from-command-line)  
- [9. Views Creation](#9-views-creation)  
- [10. Urls Creation](#10-urls-creation)  
- [11. Deploy with Apache](#11-deploy-with-apache)

#### 1. Overview


#### 2. Virtualenv and Django Installation

```
virtualenv -p python3.8 venv3.8
. venv3.8/bin/activate
# if locale issues appear, run:
## sudo locale-gen es_CL es_CL.UTF-8
## sudo dpkg-reconfigure locales
pip install --upgrade pip
pip install Django
sudo apt-get install python3-dev libmysqlclient-dev
pip install -U wheel
pip install mysqlclient
```
The created and activated virtualenv will call `python3.8` with the alias `python`.

#### 3. Create Django Application
```
django-admin startproject mysite .
```


#### 4. Setup MySQL Database

In `mysite/settings.py` add the following:
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': '<database_name>',
        'USER': '<user>',
        'PASSWORD': '<password>',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

#### 5. Running Django Application
```
python manage.py runserver
```
Access the system with `localhost:8000`  


#### 6. Setting Up Database Case Study
At the same directory level than the file `manage.py`, create a parallel app to be plugged into the main `mysiste` app.  
```
python manage.py startapp polls
```
Install the app in `mysite/settings.py` with: 
```
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    ...
```
The following apps come as default with Django:  
```
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
```

In order to initialize the database resources for these apps run the following:  
```
python manage.py migrate
```
The following database tables will be created:  
```
+----------------------------+
| Tables_in_web_mg           |
+----------------------------+
| auth_group                 |
| auth_group_permissions     |
| auth_permission            |
| auth_user                  |
| auth_user_groups           |
| auth_user_user_permissions |
| django_admin_log           |
| django_content_type        |
| django_migrations          |
| django_session             |
+----------------------------+
```
After this, the admin url will be active and can be checked at: `localhost:8000/admin`. However, to login a superuser will be necessary to be created:  
```
python manage.py createsuperuser
```


**Create Models for Application:**  
In our custom app `polls`, include (As an example) the following in the file `polls/models.py`:  
```
import datetime
from django.db import models

from django.utils import timezone

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
    def __str__(self):
        return self.question_text
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
    def __str__(self):
        return self.choice_text
```

**Create Migration for Last Database Changes:**  
```
python manage.py makemigrations polls
```
Which will provide a result like:  
```
Migrations for 'polls':
  polls/migrations/0001_initial.py:
    - Create model Choice
    - Create model Question
    - Add field question to choice
```
To see the corresponding `SQL` code for this migration:  
```
python manage.py sqlmigrate polls 0001
```

Finally, to really reflect the models' changes into the database, run:  
```
python manage.py migrate
```
**IMPORTANT:** "Migrations are very powerful and let you change your models over time, as you develop your project, without the need to delete your database or tables and make new ones - it specializes in upgrading your database live, without losing data."

#### 7. Create an Admin Superuser
**Only if not created in previous section**  
```
python manage.py createsuperuser
```
To log in as this user, go to `localhost:8000/admin`.  

#### 8. Accessing Database API from Command Line
```
python manage.py shell
```

#### 9. Views Creation  
In `polls/views.py` include the following:  
```
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

#### 10. Urls Creation
Create custom app url in `polls/urls.py` with:  
```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```
Then create the url in the main app `mysite` including the following in the 'mysite/urls.py' file:  
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

Then, running `python manage.py runserver` the content will be available at `localhost:8000/polls`.  

#### 11. Deploy with Apache  

Ref: https://www.thecodeship.com/deployment/deploy-django-apache-virtualenv-and-mod_wsgi/  
