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
$ ./manage.py startapp main
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
$ ./manage.py dbshell
psql=# CREATE DATABASE djtest; 
```

5) Create Django tables and answer 'yes' to creating a superuser
```
$ ./manage.py syncdb
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
$ ./manage.py syncdb
```

## First Migration

1) Create a model in `models.py`
```
from django.db import models

class Person(models.Model):
  name = models.CharField(max_length=100)
	age = models.IntegerField()
	location_enabled = models.BooleanField()
```

2) Now, instead of using `syncdb` as you normally would, create a migration file
```
$ ./manage.py schemamigration main --initial
Creating migrations directory at '/Users/mktrias/Documents/DjangoMeetup/djangomeetup/mysite/main/migrations'...
Creating __init__.py in '/Users/mktrias/Documents/DjangoMeetup/djangomeetup/mysite/main/migrations'...
 + Added model main.Person
Created 0001_initial.py. You can now apply this migration with: ./manage.py migrate main
```

`0001_initial.py` is located in `mysite/main/migrations` and contains all the information about your current models, and the SQL needed to migrate backwards and forwards.

3) Execute the migration
```
$ ./manage.py migrate main
Running migrations for main:
 - Migrating forwards to 0001_initial.
 > main:0001_initial
 - Loading initial data for main.
Installed 0 object(s) from 0 fixture(s)
```

Now `main_person` is in your database (go and check). So far you haven't done anything earth-shattering, but now you are set up for auto-migrations!

## Future migrations

Next time you edit `models.py` schemamigration can autodetect what has changed since your last migration, so use the `--auto` option.
```
$ ./manage.py schemamigration main --auto
 + Added field is_active on main.Person
Created 0002_auto__add_field_person_is_active.py. You can now apply this migration with: ./manage.py migrate main
$
$ ./manage.py migrate main
Running migrations for main:
 - Migrating forwards to 0002_auto__add_field_person_is_active.
 > main:0002_auto__add_field_person_is_active
```

## Converting an existing app

- Add `south` to `INSTALLED_APPS` in `settings.py`
- `$ ./manage.py syncdb` to load `south_migrationhistory` table into your database
- `$ ./manage.py convert_to_south myapp` so that South can pretend to perform your first migration
 
If you have collaborators or are running your app on other servers:

- Commit your initial migration to the VCS
- All other users should install South as described above, make sure their models and schema are identical to the original, and then type `$ ./manage.py migrate main 0001 --fake` where `0001` should be replaced with the current migration number, and `main` should be replaced with your app name.



