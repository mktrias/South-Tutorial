South-Tutorial
==============

South is a migration tool that works with Django projects. As you know, Django alone is very limited in that it can only update your database with `syncdb` when you add a new model class. South takes care of changes within a model such as adding and removing fields, changing data types or attributes.  

Docs: http://south.readthedocs.org/en/latest/about.html

Properties of South:
- Database agnostic and supports five different database backends
- VCS-proof: automatically detects if someone else's migrations conflict with your own
- You can string together migrations in order to move forwards or backwards through your database schema

# Migration Workflow

## Creating a new django app

1) Create a virtual environment:
```
$ curl -O https://raw.github.com/pypa/virtualenv/master/virtualenv.py
$ python virtualenv.py djangomeetup
$ . djangomeetup/bin/activate
```

2) Install Django and start a project and an app
```
$ pip install Django
$ django-admin.py startproject mysite
$ python manage.py startapp main
```

3) Edit `settings.py` so that Django recognizes your database and your app
```
DATABASES = {
  'default': {
    'ENGINE': 'django.db.backends.postgresql_psycopg2',
    'NAME': 'djtest',
    'USER': 'mktrias',
    'PASSWORD': '',
    'HOST': 'localhost',
    'PORT': '',
  }
}
INSTALLED_APPS = (
  ...
  'main',
)
```

4) Create `djtest` database
```
$ python manage.py dbshell
psql=# CREATE DATABASE djtest; 
```

5) Create Django tables and answer 'yes' to creating a superuser
```
$ python manage.py syncdb
```

## Installing South

1) Use pip to install
```
$ pip install South
$ pip freeze > requirements.txt
```

2) In `settings.py` under `INSTALLED_APPS`, add `'south'`.

3) South stores migration history in a table called `south_migrationhistory` so you'll need to run 
```
$ python manage.py syncdb
```

Sample output:
```
(djangomeetup) mysite $ python manage.py syncdb
Syncing...
Creating tables ...
Creating table south_migrationhistory
Installing custom SQL ...
Installing indexes ...
Installed 0 object(s) from 0 fixture(s)

Synced:
 > django.contrib.auth
 > django.contrib.contenttypes
 > django.contrib.sessions
 > django.contrib.sites
 > django.contrib.messages
 > django.contrib.staticfiles
 > main
 > south

Not synced (use migrations):
 - 
(use ./manage.py migrate to migrate these)
```

4) Create a model in `models.py`
```
from django.db import models

class Person(models.Model):
  name = models.CharField(max_length=100)
	age = models.IntegerField()
	location_enabled = models.BooleanField()
```

5) Now, instead of using `syncdb` as you normally would, do your first migration
```
$ python manage.py schemamigration main --initial
Creating migrations directory at '/Users/mktrias/Documents/DjangoMeetup/djangomeetup/mysite/main/migrations'...
Creating __init__.py in '/Users/mktrias/Documents/DjangoMeetup/djangomeetup/mysite/main/migrations'...
 + Added model main.Person
Created 0001_initial.py. You can now apply this migration with: ./manage.py migrate main
```




