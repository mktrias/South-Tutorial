South-Tutorial
==============

South is a migration tool that works with Django projects. As you know, Django alone is very limited in that it can only update your database with `syncdb` when you add a new model class. South takes care of changes within a model such as adding and removing fields, changing data types or attributes.  

Most of the info here has been distilled from the docs: http://south.readthedocs.org/en/latest/about.html

Properties of South:
- Database agnostic and supports five different database backends
- VCS-proof: automatically detects if someone else's migrations conflict with your own
- You can string together migrations in order to move forwards or backwards through your database schema

# General Tips

Do a single migration for each atomic code change (like a bug fix) instead of spreading the migrations out into a dozen files. 

# Migration Workflow

## Creating a new django app

**1) Create a virtual environment:**
```
$ curl -O https://raw.github.com/pypa/virtualenv/master/virtualenv.py
$ python virtualenv.py djangomeetup
$ . djangomeetup/bin/activate
```

**2) Install Django and start a project and an app**
```
$ pip install Django
$ django-admin.py startproject mysite
$ ./manage.py startapp main
```

**3) Edit `settings.py` so that Django recognizes your database and your app**
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

**4) Create `djtest` database**
```
$ ./manage.py dbshell
psql=# CREATE DATABASE djtest; 
```

**5) Create Django tables and answer 'yes' to creating a superuser**
```
$ ./manage.py syncdb
```

## Installing South

**1) Use pip to install**
```
$ pip install South
$ pip freeze > requirements.txt
```

**2) In `settings.py` under `INSTALLED_APPS`, add `'south'`.**

**3) South stores migration history in a table called `south_migrationhistory` so you'll need to run**
```
$ ./manage.py syncdb
```

## First Migration

**1) Create a model in `models.py`**
```
from django.db import models

class Person(models.Model):
	name = models.CharField(max_length=100)
	age = models.IntegerField()
	location_enabled = models.BooleanField()
```

**2) Now, instead of using `syncdb` as you normally would, create a migration file**
```
$ ./manage.py schemamigration main --initial
Creating migrations directory at '/Users/mktrias/Documents/DjangoMeetup/djangomeetup/mysite/main/migrations'...
Creating __init__.py in '/Users/mktrias/Documents/DjangoMeetup/djangomeetup/mysite/main/migrations'...
 + Added model main.Person
Created 0001_initial.py. You can now apply this migration with: ./manage.py migrate main
```

`0001_initial.py` is located in `mysite/main/migrations` and contains all the information about your current models, and functions to generate the SQL needed to migrate backwards and forwards.

Migration files must contain a `Migrations` class and the `forwards()` and `backwards()` methods. In theory, you could write your migrations by hand, but `schemamigration` does this for you. You may want to edit the `forwards()` function if you have dependencies or want to load data at a specific point in the migration chain. This can be extremely useful if you are using South with multiple apps that share relations in a database.

**3) Execute the migration**
```
$ ./manage.py migrate main
Running migrations for main:
 - Migrating forwards to 0001_initial.
 > main:0001_initial
 - Loading initial data for main.
Installed 0 object(s) from 0 fixture(s)
```

Now `main_person` is in your database (go and check).

At any point you can roll back to a particular migration by typing
```
$ ./manage.py migrate myapp 0008
```

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

`migrate` automatically runs each of the migrate files in ascii order, hence all the 0 prefixes. If you forsee having more than 9999 migrations, feel free to prepend another zero!

A useful option can show you which migrations exist and which have been committed (*)
```
$ ./manage.py migrate --list

 main
  (*) 0001_initial
  (*) 0002_auto__add_field_person_is_active
```
You can update the current migration using `--update`. Use this to refine your changes to an atomic code change (adding a single feature, or fixing a bug)
```
$ ./manage.py schemamigration main --auto --update
```

## Converting an existing app

- Add `south` to `INSTALLED_APPS` in `settings.py`
- `$ ./manage.py syncdb` to load `south_migrationhistory` table into your database
- `$ ./manage.py convert_to_south myapp` so that South can pretend to perform your first migration
 
If you have collaborators or are running your app on other servers:

- Commit your initial migration to the VCS
- All other users should install South as described above, make sure their models and schema are identical to the original, and then type `$ ./manage.py migrate main 0001 --fake` where `0001` should be replaced with the current migration number, and `main` should be replaced with your app name.

## Data migration

Data migration is used to change the data in your database to reflect a new schema or feature. It's also useful if you are working on a project with collaborators and want to synchronize your data.

As an example, let's populate the `main_person` table with some people. 

First, create a 'blank' migration file. Here, `main` is the name of my app, and `main_person` is the name I'd like to give the migration file.
```
$ ./manage.py datamigration main main_person
```
Edit the migration file to look like this:
```
class Migration(DataMigration):
	def forwards(self, orm):
		p1 = orm.Person('john',12,False,False)
		p2 = orm.Person('phil',24,True,False)
		p1.save()
		p2.save()
	
	def backwards(self, orm):
		raise RuntimeError('Cannot reverse this migration')
```
Now run the migration
```
$ ./manage.py migrate main
```
Go look at the database now and you should see two new rows of data in `main_person`! 

## Fields with `null=False`

If you add a column that is `NOT NULL` without setting a default value, south will ask you to enter a default value, so that the database backend doesn't complain. For example:
```
**models.py:**

class Person(model.Models):
	...
+	size = models.IntegerField(null=False)
```
Then when you migrate, south complains:
```
$ ./manage.py schemamigration main --auto
 ? The field 'Person.size' does not have a default specified, yet is NOT NULL.
 ? Since you are adding this field, you MUST specify a default
 ? value to use for existing rows. Would you like to:
 ?  1. Quit now, and add a default to the field in models.py
 ?  2. Specify a one-off value to use for existing columns now
 ? Please select a choice: 2
 ? Please enter Python code for your one-off default value.
 ? The datetime module is available, so you can do e.g. datetime.date.today()
 >>> 1
 + Added field size on main.Person
Created 0004_auto__add_field_person_size.py.
```

NOTE: South detects **some** changes to fields (such as adding `unique=True`) but not all changes. For instance, adding `null=False` does not get detected. A full list of autodetected changes is here: http://south.readthedocs.org/en/latest/autodetector.html#autodetector-supported-actions

NOTE 2: You cannot use any custom defined methods within your models through South migrations.

## Custom Fields

You can have them! See here: http://south.readthedocs.org/en/latest/tutorial/part4.html

## Working with teams and a VCS

If you notice any migration files when you do a pull, you should do a `./manage.py migrate myapp`. If you have made conflicting changes in the same timeslot as your co-worker, south will detect this and let you know:
```
Inconsistent migration history
The following options are available:
	--merge: will just attempt the migration ignoring any potential dependency conflicts.
```
If you've both been working on the same `models.py` file, you'll likely have to resolve conflicts by hand. And you probably should have communicated about this beforehand :) If you're working on separate apps, you can try:
```
$ ./manage.py migrate main --merge
```
South will then apply the missing migrations out of order, which should work because the migrations are independent.

## Dependencies

