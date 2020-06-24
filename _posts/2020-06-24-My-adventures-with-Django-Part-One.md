---
layout: post
title: My adventures with Django Part One
description: My adventures with Django Part One
summary: My adventures with Django Part One
comments: false
---

On my quest to find the 'right' GUI framework (for me), I have decidedly chosen to partner up with Django and see where it takes me. 

Python has a huge number of GUI frameworks available, unfortunately I have felt dissatisfied with the offerings so far. This largely stems from feeling uncomfortable, knowingly crafting a native desktop application when deep down I knew I should move towards a Web App. 

Mainly because of the freedom it gives me (and others) to reach my application, and at the same time, doing away with any sort of necessary installation. This makes more sense as many of the things I typically interact with day to day, whether it be monitoring or dashboards, are hosted as Web Apps. **IMO there is nothing worse than having to RDP into a server just to view an application.**

Enter Django. 

In a nutshell, Django is Python-based FREE and open-source Web framework designed to make Web development easy. 

I will be following the official documentation on how to build my first web application, with the intention to host it with a Web Server such as Nginx, probably in AWS. The purpose of this write-up is not to plagiarize the excellent documentation provided by the djangoproject.com, but rather to simplify and also expand upon some things which I feel the tutorial eludes.

https://www.djangoproject.com/

---

### Install Django

```
(django-prod) lawrence@cacti:~/git/django$ pip3 install django
Collecting django
  Downloading Django-3.0.7-py3-none-any.whl (7.5 MB)
     |████████████████████████████████| 7.5 MB 1.7 MB/s 
Collecting sqlparse>=0.2.2
  Downloading sqlparse-0.3.1-py2.py3-none-any.whl (40 kB)
     |████████████████████████████████| 40 kB 7.8 MB/s 
Collecting pytz
  Downloading pytz-2020.1-py2.py3-none-any.whl (510 kB)
     |████████████████████████████████| 510 kB 23.1 MB/s 
Collecting asgiref~=3.2
  Downloading asgiref-3.2.9-py3-none-any.whl (19 kB)
Installing collected packages: sqlparse, pytz, asgiref, django
Successfully installed asgiref-3.2.9 django-3.0.7 pytz-2020.1 sqlparse-0.3.1
```

Check version of Django

```
(django-prod) lawrence@cacti:~/git/django$ python -m django --version
3.0.7
```

Running `django-admin` without an argument will display a list of sub-commands:

```
(django-prod) lawrence@cacti:~/git/django$ django-admin

Type 'django-admin help <subcommand>' for help on a specific subcommand.

Available subcommands:

[django]
    check
    compilemessages
    createcachetable
    dbshell
    diffsettings
    dumpdata
    flush
    inspectdb
    loaddata
    makemessages
    makemigrations
    migrate
    runserver
    sendtestemail
    shell
    showmigrations
    sqlflush
    sqlmigrate
    sqlsequencereset
    squashmigrations
    startapp
    startproject
    test
    testserver
Note that only Django core commands are listed as settings are not properly configured (error: Requested setting INSTALLED_APPS, but settings are not configured. You must either define the environment variable DJANGO_SETTINGS_MODULE or call settings.configure() before accessing settings.).
```

<br/>

### Creating a project

Documentation: https://docs.djangoproject.com/en/3.0/intro/tutorial01/

We're going to create a project by using the `startproject` command, giving a **name** as an argument. This will auto-generate some code that makes up the basis of our Django project. These files will include a **collection of settings for a Django instance**, including **database configuration**, Django-specific **options** and **application-specific settings**.

```
(django-prod) lawrence@cacti:~/git/django$ django-admin startproject first_project
```

Now I should have a directory named `first project` and within it, the files we just talked about:

```
(django-prod) lawrence@cacti:~/git/django$ tree first_project/
first_project/
├── first_project
│   ├── asgi.py
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py
```

---

**Heads Up!** Don't worry about storing your code in a place such as `/var/www`. With Django that isn't necessary, and it is generally considered a **bad idea** to keep any of your Python code within your Web server's document root as it runs the risk of exposing your code over the Web.

Therefore Django code should be stored in a directory **outside root** such as **/home/mycode**.

---

A brief overview of these files:

* `first_project/` within the root project directory is a **container for the project**. 
  * It can be renamed whenever and does not matter to Django.
* `manage.py` is a command line utilty for interacting with Django.
  * https://docs.djangoproject.com/en/3.0/ref/django-admin/ 
* `first_project/first_project/` is the actual Python package name for the project, and will be used to import anything inside it. 
  * For example `first_project.urls`.
* `first_project/asgi.py` is an **entry-point** for ASGI-compatible web servers to serve the project.
* `first_project/__init__.py` is an **empty file** that tells Python that this directory is considerd a Python package.
  * https://docs.python.org/3/tutorial/modules.html#tut-packages 
* `first_project/settings.py` is the settings/configuraton for the Django project.
  * https://docs.djangoproject.com/en/3.0/topics/settings/ 
* `first_project/urls.py` is the URL declarations for the project; this can be considered a **"table of contents"** for our Django-powered site.
  * https://docs.djangoproject.com/en/3.0/topics/http/urls/
* `first_project/wsgi.py` is an **entry-point** for WSGI-compatible web servers to server the project.


_`manage.py` is automatically created in each Django project. It does the same thing as django-admin but also sets the `DJANGO_SETTINGS_MODULE` environment variable so that it points to your project’s `settings.py` file._

<br/>

### Starting a development server

Django comes with a **lightweight Web server** written purely in Python, this is purely for development purposes, and mitigates having to work with a Web server such as Nginx or Apache until our project is ready for production.

#### Do not use this as a server in anything resembling a production environment. This is a Web framework, not a Web server.

Before we continue, let's confirm that our Django project works. By changing into the root directory `first_project/` I can then start a development server with `python manage.py runserver`:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 17 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.

June 18, 2020 - 19:30:39
Django version 3.0.7, using settings 'first_project.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

#### Django documentation states that the 'unapplied database migrations` can be ignored for now, and that we'll deal with later.

So now we should successfully have a development server running on `http://localhost:8000` or alternatively go to `http://127.0.0.1:8000/`.

#### Changing the port of your development server

By **default** the `runserver` command starts the development server on localhost port `8000`. 

You can change the default port by passing the desired port to the `runserver`, for example:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py runserver 8080
```

#### Letting others view your development server

A security feature of Django is that by default, hosts other than localhost are denied to access your development server. 

Attempting to reach the server from another host on the network will result int he following error:

```
Invalid HTTP_HOST header: '192.168.10.10:8000'. You may need to add '192.168.10.10' to ALLOWED_HOSTS.
```

This can be allowed by amending the `setting.py` file within the project:

```
lawrence@cacti:~/git/django/first_project$ vim first_project/settings.py
```

And changing:

```python
# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = []
```

To:

```python
DEBUG = True

ALLOWED_HOSTS = ['*']
```

Then run your development server but this time pass `0:8000` to the `runserver` command. 

`0` is a shortcut for `0.0.0.0`.

Your server should now be reachable via `localhost` and from another device on your network.

<br/>

### Creating our first app

Now that our environment is set up, we can begin building our app.

#### Projects vs. apps

What’s the difference between a project and an app? 
* An **app** is a Web application that does something – e.g. a Weblog system, a database of public records or a small poll app. 
* A **project** is a collection of configuration and apps for a particular website. 
  * A project can contain multiple apps. 
  * **An app can be in multiple projects.**

**Apps can live anywhere on our Python path**, however we will create this example app within the same directory as our `manage.py` file so it can be imported as its own top-level module, rather than a submodule of project `first_project`.

#### To create an app

We'll create our app within the root directory, the same directory as `manage.py`:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ tree
.
├── db.sqlite3
├── first_project
│   ├── asgi.py
│   ├── __init__.py
│   ├── __pycache__
│   │   ├── __init__.cpython-36.pyc
│   │   ├── settings.cpython-36.pyc
│   │   ├── urls.cpython-36.pyc
│   │   └── wsgi.cpython-36.pyc
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py
```

We'll create our app with the `startapp` parameter:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py startapp polls
```

This will create a directory named `polls` with the following file structure:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ tree polls/
polls/
├── admin.py
├── apps.py
├── __init__.py
├── migrations
│   └── __init__.py
├── models.py
├── tests.py
└── views.py

1 directory, 7 files
```

<br/>

### Writing our first **view**

__A **view** is a callable which takes a request and returns a response. This can be more than just a function, and Django provides an example of some classes which can be used as **views**. These allow you to structure your views and reuse code by harnessing inheritance and mixins.__

We'll begin by writing our first **view ** in `polls/views.py`. 

By **default** it will look like this:

```python
from django.shortcuts import render

# Create your views here.
```

We'll amend to the following:

```python
from django.http import HttpResponse

# Create your views here.
def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```    

**This is the simplest view possible in Django.** To call the **view**, we need to **map it to a URL** - and for this we need a **URLconf**.

To create a **URLconf** within our app (`polls`) directory, we need to create a file named `urls.py` within the root of the apps directory:

```
(django-prod) lawrence@cacti:~/git/django/first_project/polls$ touch urls.py
(django-prod) lawrence@cacti:~/git/django/first_project/polls$ tree
.
├── admin.py
├── apps.py
├── __init__.py
├── migrations
│   └── __init__.py
├── models.py
├── tests.py
├── urls.py
└── views.py
```

Then within the `polls/url.py` file we'll add the following code:

```python 
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

Next, is to **point the root URLconf at the `polls.urls` module.** 

In `mysite/urls.py`, add an **import** for `django.urls.include` and insert an `include()` in the urlpatterns list, I have detailed the before and after changes below:

Before:

```python
(django) lawrence@ulysses:~/git/django/first_project$ cat first_project/urls.py 
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path('admin/', admin.site.urls),
]
```

After:

```python
(django) lawrence@ulysses:~/git/django/first_project$ cat first_project/urls.py 
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    # add our 'polls' app:
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

Now, we've just **imported an additional module** from our `django.urls` package titled `include`.

Then, using the `include()` module we add the line `path('polls/', include('polls.urls')),` to our `first_project/urls.py` file. This line of code points the root **URLconf file** we recently created at `polls/url.py`, to the `polls.urls` module.

So now we've just 'wired' an **index view** into the **URLconf**. We can verify it’s working by running our server and navigating to `http://localhost:8000/polls/`. 

Yes that's correct, we're going to http://localhost:8000/polls/ and not http://localhost:8000/.

<br/>

### Using `include()` to reference URLconfs

The `include()` function allows you to **reference** other URLconfs, therefore whenever Django encounters an `include()`: 
1. It **discards** whatever part of the URL **matched** up to that point.
2. It then **returns** the **remaining string**.
3. Sending it to the included URLconf for further processing.

The purpose of this is to simplify the ability to add **'plug-and-play' URLs**, therefore as long as the `path()` is correct, the app will continue to function.

It is advised to always use `include()` when you include other URL patterns, `admin.site.urls` being the only exception.

<br/>

### The `path()` function

The `path()` function can take up to **four args**, two being required; **route, view**, and two optional being: **kwargs, name**.

We can look at this in a little more detail:
* **`path()` arg: ROUTE**
  * Route is a string that contains a URL pattern. It starts at the first pattern in urlpatterns and continues down the list, comparing the requested URL against each pattern until it finds a match.
* **`path()` arg: VIEW**
  * When Django finds a matching pattern, it calls the specified view function with an HttpRequest object as the first argument and any "captured" values from the route as keyword arguments.  
* **`path()` arg: KWARGS**
  * Arbitrary keyword arguments can be passed in a dictionary to the target view.
* **`path()` arg: NAME**
  * Naming your URL lets you refer to it unambiguously from elsewhere in Django, especially from within templates. This powerful feature allows you to make global changes to the URL patterns of your project while only touching a single file.

---

**Heads Up!** **Patterns don’t search GET and POST parameters**, or the **domain name**. For example, in a request to https://www.example.com/myapp/, the URLconf will look for `myapp/`. In a request to https://www.example.com/myapp/?page=3, the URLconf will also look for `myapp/`.

---

### Database setup

For this part of the tutorial we'll be using https://docs.djangoproject.com/en/3.0/intro/tutorial02/.

Now that we are comfortable with the basic request and response flow. We can move on to building our database.

By default, the configuration uses **SQLite**, which is the easiest choice if you’re new to databases. **SQLite is included in Python**, so we won’t need to install anything extra to support our database. More experienced users might opt for something more scalable such as **PostgreSQL** to avoid database-switching issues later on.

#### Using an alernative database

To use a different database, make sure you **install the appropritate database bindings** as detailed below and **change the following keys** in the DATABASES 'default' item to match your database connection settings:

**Database bindings**- https://docs.djangoproject.com/en/3.0/topics/install/#database-installation.

**ENGINE** - Either 'django.db.backends.sqlite3', 'django.db.backends.postgresql', 'django.db.backends.mysql', or 'django.db.backends.oracle'. Other backends are also available.

**NAME** - The name of your database. If you’re using SQLite, the database will be a file on your computer; in that case, NAME should be the full absolute path, including filename, of that file. The default value, os.path.join(BASE_DIR, 'db.sqlite3'), will store the file in your project directory.

If you are not using SQLite as your database, additional settings such as USER, PASSWORD, and HOST must be added. For more details, see the reference documentation for DATABASES.

More information here: https://docs.djangoproject.com/en/3.0/ref/settings/#std:setting-DATABASES

<br/>

### Setting your time zone

While we're editing `first_project/settings.py`, set `TIME_ZONE` to your time zone:

```
(django) lawrence@ulysses:~/git/django/first_project$ cat first_project/settings.py | grep TIME_ZONE
TIME_ZONE = 'UTC'
```

In my case, my timezone will be `GB	+513030−0000731	Europe/London` according to: 
* https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
* https://kite.com/python/answers/how-to-change-the-django-time-zone-in-python#:~:text=Change%20the%20value%20of%20TIME_ZONE,be%20found%20by%20calling%20pytz.

```
(django) lawrence@ulysses:~/git/django/first_project$ cat first_project/settings.py | grep TIME_ZONE
TIME_ZONE = 'GB'
```

<br/>

### Installed apps

Also, note the `INSTALLED_APPS` setting at the top of the file. That holds the names of all Django **applications that are activated** in this Django instance. Apps can be used in multiple projects, and you can package and distribute them for use by others in their projects.

By default, `INSTALLED_APPS` contains the following apps, all of which come with Django:

* `django.contrib.admin` - The admin site. You’ll use it shortly.
* `django.contrib.auth` - An authentication system.
* `django.contrib.contenttypes` - A framework for content types.
* `django.contrib.sessions ` - A session framework.
* `django.contrib.messages` - A messaging framework.
* `django.contrib.staticfiles` - A framework for managing static files.

These applications are included by default as a convenience for the common case.

Some of these applications make use of at least one database table, though, so we need to create the tables in the database before we can use them. To do that, run `python manage.py migrate`:

```
(django) lawrence@ulysses:~/git/django/first_project$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying sessions.0001_initial... OK
```

The `migrate` command looks at the `INSTALLED_APPS` setting and **creates any necessary database tables according to the database settings** within `first_project/settings.py` file and the database migrations shipped with the app (we’ll cover them later). 

From observing the **above output** you'll receive a message for each migration it applies. If you’re interested, run the command-line client for your database and type `\dt` (PostgreSQL), `SHOW TABLES`; (MariaDB, MySQL), `.schema` (SQLite), or `SELECT TABLE_NAME FROM USER_TABLES`; (Oracle) to display the tables Django created.

The **default applications are included for the common case**, but not everybody needs them. If you don’t need any or all of them, feel free to comment-out or delete the appropriate line(s) from `INSTALLED_APPS` before running `migrate`. 

The `migrate` command will only run migrations for apps in `INSTALLED_APPS`.

<br/>

### Creating models

Now it's time to define our **models**, which is essentially our **database layout** with additional **metadata**.

"A **model** is the single, definitive source of truth about your data." It is comprised of the essential **fields** and **behaviors** of the data you're storing. The goal is to define your data model in one place and automatically derive things from it.

Django follows the **Don’t repeat yourself (DRY)** principle. "Every distinct concept and/or piece of data should live in one, and only one, place. **Redundancy is bad.** Normalization is good." 

#### Migrations

"This includes the migrations - unlike in Ruby On Rails, for example, migrations are entirely derived from your models file, and are essentially a history that Django can roll through to update your database schema to match your current models." - Source: djangoproject

<br/>

### Building the models for our 'polls' app

#### Model concepts

As part of our 'polls' app, we're going to create **two models:** 
* `Question` model.
  * The Question model with have `question_text` and `pub_date` field. 
* `Choice` model.
  * Each **Choice** will be associated with a **Question**.
    * `question = models.ForeignKey(Question, on_delete=models.CASCADE)`
  * The Choice model will have **two fields**:
    * Field 1: the text of the choice `choice_text`.
    * Field 2: a `votes` tally.

#### Represented in Python

These **models** are going to be represented by Python **classes**.

We will define our classes within the apps `polls/models.py` file:

```python
(django-prod) lawrence@cacti:~/git/django/first_project$ cat polls/models.py 
from django.db import models

# Create your models here.
class Question(models.Model):
    # The name of each fields instance represents each field's name e.g. 'question_text'
    # this is the value we will use in our Python code, and the db will use it as the 
    # column name.
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    # Each Choice is associated with a Question
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

With this code Django is able to do the following:
* Create a database schema (`CREATE TABLE` statements) for this app.
* Create a Python database-access API for accessing `Question` and `Choice` objects.

Each model is represented by a class that **subclasses** `django.db.models.Model`.

And within each class is a number of class **variables**, which each **represents a database field** in the model.

Each field is represented by an instance of a `Field` class, telling Django what type of data each field holds, for example:
* `CharField` for character fields.
* `DateTimeField` for datetimes.

#### Working with fields

Within our model/class, the name of the `Field` instance, e.g. `question_text` and `pub_date`, represents the field's name. 

This field name **variable** is the value we'll use within our Python code. And our database will use it as the **column name**.

#### Defining human-readable names

It is possible to use an **optional first positional** argument to a `Field` to designate a human-readable name. If this field isn't provided then it will use the machine-readable name, for example:

`Question.question_text` uses only a **machine-readable** name `question_text`:

```python
    question_text = models.CharField(max_length=200)
```

Whereas `Question.pub_date` has a **human-readable** name defined:

```python
    pub_date = models.DateTimeField('date published')
```

#### Required and optional arguments

Some `Field` **classes** have required arguments.

For example `CharField` requires that you give it a `max_length`, this is not only used within the database schema, but also in validation as we'll soon see:

```python
    question_text = models.CharField(max_length=200)
```

A `Field` can also have various optional arguments, for example, we have set the `default` value of `votes` to `0`:

```python
    votes = models.IntegerField(default=0)
```

Lastly, a **relationship is defined** using `ForeignKey`. This tells Django that each `Choice` is related to a single `Question`:

```python
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
```

<br/>

### Activating models

Now that we've created our two models within our apps `models.py`, we need to tell our `first_project` that the `polls` app has been installed.

To include our app within our project, we need to **reference its configuration class** within the `INSTALLED_APPS` setting.

Our `PollsConfig` class resides within `polls/apps.py`:

```python
(django-prod) lawrence@cacti:~/git/django/first_project$ cat polls/apps.py 
from django.apps import AppConfig


class PollsConfig(AppConfig):
    name = 'polls'
```    

Therefore its dotted path is `polls.apps.PollsConfig`, which is what we will add to our `INSTALLED_APPS` setting:

```python
(django-prod) lawrence@cacti:~/git/django/first_project$ vim first_project/settings.py 
...
# Application definition

INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
...
```

Now Django knows to include our new `polls` app within our project. 

Now we need to inform Django that we've made some changes to our models, in our case we've created **new models**, and that we want these changes to be stored as a `migration`. 

We do this by running the `makemigrations` command and passing it our `polls` app:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py makemigrations polls
Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice
```

<br/>

### Migrations

**Migrations** are how Django stores changes made to our models, whether these are new models, or existing models with changes. Changes of which are then reflected within our **database schema**.

Migrations are stored within a file format and can be read if desired:

```python
(django-prod) lawrence@cacti:~/git/django/first_project$ cat polls/migrations/0001_initial.py
# Generated by Django 3.0.7 on 2020-06-23 16:31

from django.db import migrations, models
import django.db.models.deletion


class Migration(migrations.Migration):

    initial = True

    dependencies = [
    ]

    operations = [
        migrations.CreateModel(
            name='Question',
            fields=[
                ('id', models.AutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),
                ('question_text', models.CharField(max_length=200)),
                ('pub_date', models.DateTimeField(verbose_name='date published')),
            ],
        ),
        migrations.CreateModel(
            name='Choice',
            fields=[
                ('id', models.AutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),
                ('choice_text', models.CharField(max_length=200)),
                ('votes', models.IntegerField(default=0)),
                ('question', models.ForeignKey(on_delete=django.db.models.deletion.CASCADE, to='polls.Question')),
            ],
        ),
    ]
```    

**You are not expected to read a migration everytime it happens!** However they're designed to be human-editable in case you wish to tweak how Django handles these changes.

It is possible to `check` whether there are any problems in your project without making migrations or touchining the database with the `check` parameter:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py check
System check identified no issues (0 silenced).
```

#### Applying our migrations

Now we can run `migrate` again to create our **model** tables within our database:

```python
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying polls.0001_initial... OK
```

As we talked about earlier, the `migrate` command takes all the migrations that haven't yet been applied. Django tracks which ones that are applied using a special table in within our database called `django_migrations`, and runs them against our database - **synchronizing the changes made to our models with the schema in the database.**

<br/>

### Summarizing migrations

Let's do a quick recap on the migration process as a whole. 

Migrations are a **three-step** process. We did the following:

1. **Changed our models** within `<app>/models.py`.
2. **Created migrations** for those changes with `python manage.py makemigrations`.
3. **Applied changes to our database** with `python manage.py migrate`.

The reason for three steps is that it we'll commit migrations to our vcs and ship them with our app; making development easier, and also usable by other developers by other developers and in production.

Migrations are powerful and allow us to develop our models over time with our project. This prevents us from having to delete our database or tables and start again with new ones.

And they allow us to upgrade our database live, without losing data.

More information on `migrate` https://docs.djangoproject.com/en/3.0/ref/django-admin/#django-admin-migrate.

<br/>

### Playing with the API

Now we are ready to jump into our favourite interpreter and play with the Django API by invoking the `shell` command:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py shell
Python 3.6.9 (default, Apr 18 2020, 01:56:04) 
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> 
```

We use `manage.py` as it sets the `DJANGO_SETTINGS_MODULE` environment variables, which **imports the path to our project settings.py file: `first_project/settings.py`.

<br/>

### Making queries

Once we've created our data models, Django provides us with a database-abstraction API that lets us **create**, **retrieve**, **update** and **delete objects**.

Making queries within Django is detailed here https://docs.djangoproject.com/en/3.0/topics/db/queries/.

#### Let's test our newly created app:

First we'll import the two model classes we created within our `polls` app:

```python
>>> from polls.models import Choice, Question
```

Currently no questions exist yet:

```python
>>> Question.objects.all()
<QuerySet []>
```

Therefore we'll begin by creating a new question:

```python
>>> q = Question(question_text="Impressive work", pub_date=timezone.now())
Traceback (most recent call last):
  File "<console>", line 1, in <module>
NameError: name 'timezone' is not defined
```

We receive an error because the `pub_date` field expects a datetime. Support for time zones is enabled in the default `settings.py` file, use `timezone.now()` instead of `datetime.datetime.now()` and import the `timezone` submodule:

```python
>>> from django.utils import timezone
```

```python
>>> q = Question(question_text="Impressive work", pub_date=timezone.now())
```

Now we need to **save** our object into the database with the `save()` method:

```python
>>> q.save()
```

And now this time when we call our `Question` object, we see that our question has been successfully saved with an ID of `(1)`. You can use `objects.all()` to display all the questions within the database:

```python
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```

```
>>> q.id
1
```

Now we can access our model field values via their Python attributes:

```python
>>> q.question_text
'Impressive work'
```

We can even access the `pub_date` attribute:

```
>>> q.pub_date
datetime.datetime(2020, 6, 23, 17, 10, 44, 729363, tzinfo=<UTC>)
```

We can change the values of an attribute by assigning them a new value (be sure to save):

```python
>>> q.question_text = "Who says \"Impressive work, Ms. Vance\" ?"  
>>> q.save()
>>> q.question_text
'Who says "Impressive work, Ms. Vance" ?'
```

<br/>

### Making your code readable

It is safe to say, that:

```python
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```

Is not a human-readable representation of this object. Therefore we will add a `__str__()` method to both our `Question` and `Choice` models:

#### Original code:

```python
from django.db import models
  
# Create your models here.
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

#### New code: 

```diff
import datetime

from django.db import models
from django.utils import timezone

# Create your models here.
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

+   def __str__(self):
+       return self.question_text


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

+   def __str__(self):
+       return self.choice_text
```

**It is important that you add `__str__()` methods to your models!** This is not only for when dealing with the interpreter, but als because objects' representations are used throughout Django.

Therefore let's add a custom method to the `Question` model:

```diff
import datetime

from django.db import models
from django.utils import timezone

# Create your models here.
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def __str__(self):
        return self.question_text

+   def was_published_recently(self):
+       return self.pub_date >= timezone.now() - datetime.timedelta(days=1)    

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
        return self.choice_text
```

Now with the addition of the `import datetime` module and `from django.utils import timezone`, we can reference Python's standard `datetime` module and Django's time-zone-related utilities in `django.utils.timezone`.

More information about time zone handling within Python in:
* https://docs.djangoproject.com/en/3.0/topics/i18n/timezones/

Now with our new changes saved let's open our interactive prompt and try again:

```python
>>> from polls.models import Choice, Question
>>> Question.objects.all()
<QuerySet [<Question: Who says "Impressive work, Ms. Vance" ?>]>
>>> Question.objects.filter(id=1)
<QuerySet [<Question: Who says "Impressive work, Ms. Vance" ?>]>
>>> Question.objects.filter(question_text__startswith='Who')
<QuerySet [<Question: Who says "Impressive work, Ms. Vance" ?>]>
```

Much better **=)**.

Let's look at a few more examples:

```python
# Get the question that was published this year
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: Who says "Impressive work, Ms. Vance" ?>
```

Request an ID that does not exist, raising an **exception**:

```python
>>> Question.objects.get(id=2)
Traceback (most recent call last):
  File "<console>", line 1, in <module>
  File "/home/lawrence/.virtualenvs/django-prod/lib/python3.6/site-packages/django/db/models/manager.py", line 82, in manager_method
    return getattr(self.get_queryset(), name)(*args, **kwargs)
  File "/home/lawrence/.virtualenvs/django-prod/lib/python3.6/site-packages/django/db/models/query.py", line 417, in get
    self.model._meta.object_name
polls.models.Question.DoesNotExist: Question matching query does not exist.
```

Perform a **primary-key lookup**, Django provides a shortcut for 'pk' exact lookups:

```python
>>> Question.objects.get(pk=1)
<Question: Who says "Impressive work, Ms. Vance" ?>
```

Check that the custom `was_published_recently` method we added, works:

```python
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)   
```

```python
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True
```

Looks good!

#### Assigning our `Question` `Choice`(s)

To give our Question some Choices, I can use the `create()` call to construct a new `Choice` object. 

This does the following:
* Creates a new `Choice` object.
* Does the `INSERT` statement (inserting our data into our database).
* Adds the `Choice` to the set of available choices.
* Returns the new `Choice` object.

```python
>>> q = Question.objects.get(pk=1)
>>> q.
q.DoesNotExist(              q.delete(                    q.objects                    q.save_base(
q.MultipleObjectsReturned(   q.from_db(                   q.pk                         q.serializable_value(
q.check(                     q.full_clean(                q.prepare_database_save(     q.unique_error_message(
q.choice_set(                q.get_deferred_fields(       q.pub_date                   q.validate_unique(
q.clean(                     q.get_next_by_pub_date(      q.question_text              q.was_published_recently(
q.clean_fields(              q.get_previous_by_pub_date(  q.refresh_from_db(           
q.date_error_message(        q.id                         q.save(    
```

We will use the `choice_set` method to create our Choices.

Woah woah woaaah, where did `choice_set` come from? OK let's take a breather and look at how this happened in the next section...

<br/>

### Creating choices with `foo_set`

So how the heck did we end up with `choice_set()` as a method, where did it come from?

Well, Django creates a `set` to hold the "other side" of a `ForeignKey` relation (e.g. a question's choice) which can be accessed via the API.

So let's recap and take a look at our **original code** within `polls/models.py`:

```python
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def __str__(self):
        return self.question_text
    # our custom method
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)    

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
        return self.choice_text
```        

#### Breaking it down

We created a `ForeignKey` on `Choice`. 

So now each `Choice` we create will have **relationship** with `Question`.

Therefore each `Choice` explicitly has a `question` field that we declared within our `Choice` model.

Django's ORM (an implementation of the **object-relational mapping (ORM)** concept), see's the relationship between `Question` and `Choice`, and generates a field within each instance called `foo_set` or in our case `choice_set`. 

(If you're not familiar with what an instance is, read up on Classes.)

Where `foo` is the **model** (in our case `Choice`), with a `ForeignKey` field pertaining to that **model**. Therefore `foo_set` becomes `choice_set`.

Now, `choice_set` is a `RelatedManager` which can create QuerySets of `Choice`. A **QuerySet** represents a collection of objects from within a database. 

Source: Related objects reference https://docs.djangoproject.com/en/3.0/ref/models/relations/.

Therefore, these **QuerySets** of `Choice` objects, relate to the `Question` instance, i.e:

```python
q.choice_set.all()
```

#### Phew... 

Now, it is possible to change the `ForeignKey.relate_name`, in turn changing `foo_set`. This might be useful if you wanted to distinguish between a conflicting ForeignKey with the same model.

Source: https://docs.djangoproject.com/en/dev/ref/models/fields/#django.db.models.ForeignKey.related_name

<br/>

### Taking our newly `foo_set` knowledge to `shell` 

So now that we have a better idea of how we ended up with `choice_set`, let's begin creating some choices for our `Question`.

Currently we have no `Choice`(s) from the related object `set`:

```python
>>> q.choice_set.all()
<QuerySet []>
```

Therefore we're going to create **three choices**:

```python
>>> q.choice_set.create(choice_text='Dr. Breen', votes=0)
<Choice: Dr. Breen>
>>> q.choice_set.create(choice_text='G-Man', votes=0)
<Choice: G-Man>
>>>c = q.choice_set.create(choice_text='Vortigaunt', votes=0)
<Choice: Vortigaunt>
```

As we recently went over, these `Choice` objects have API access to their related `Question` objects, and vice versa:

```python
>>> c.question
<Question: Who says "Impressive work, Ms. Vance" ?>
```

```python
>>> q.choice_set.all()
<QuerySet [<Choice: Dr. Breen>, <Choice: G-Man>, <Choice: Vortigaunt>]>
```

```python
>>> q.choice_set.count()
3
```

Lastly we can delete our choices using the `delete()` method:

```python
>>> c = q.choice_set.filter(choice_text__startswith='Vortigaunt')
>>> c.delete()
(1, {'polls.Choice': 1})
```

Django's API will automatically follow relationships as far as you need.

Use double underscores `foo__bar` to separate relationships. This works as many levels deep as you want; there's no limit e.g. `foo__bar__fubar`.

We can then find all `Choices` for any `question` whose `pub_date` is in this `year` (`current_year` variable we created above):

```python
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Dr. Breen>, <Choice: G-Man>, <Choice: Vortigaunt>]>
```

More information here:
* Related objects reference: https://docs.djangoproject.com/en/3.0/ref/models/relations/
* Field lookups: https://docs.djangoproject.com/en/3.0/topics/db/queries/#field-lookups-intro
* Database API reference: https://docs.djangoproject.com/en/3.0/topics/db/queries/

<br/>

### Django Admin

If you didn't already know, Django was developed in a "fast-paced newsroom environment", designed to make common web-development tasks fast and easy, providing us with the tools we need to be able to build and administer a database-driven web app. 

Creating admin sites to allow site managers to **add**, **change**, and **delete** content, is tedious work and leaves little to the imagination. **Therefore Django automates the creation of admin interfaces for our models.**

#### Creating an admin user

First we'll begin by creating a user that can login to our admin site with the `createsuperuser` command:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py createsuperuser
Username (leave blank to use 'lawrence'): 
Email address: example@gmail.com
Password: 
Password (again): 
Superuser created successfully.
```

#### Entering the admin site

The admin site is activated by default, therefore I can now start my **development server** and login via the admin login screen at https://127.0.0.1:8000/admin/.

Upon logging in with the superuser credentials we created in the previous step, you'll arrive at the **admin index page**. 

Under 'Authentication and authorization' are a few types of editable content: **Groups** and **Users**. These are provided by `django.contrib.auth`, which is the **authentication framework** shipped by Django.

However our 'polls' app is absent from the admin index page, this is because we need to tell the admin that `Question` **objects** have an **admin interface**.

<br/>

### Making your app modifiable within admin

To tell our admin site that `Question` **objects** should an **admin interface**, we need to navigate to the `poll/admin.py` file and add the following:

```python
(django-prod) lawrence@cacti:~/git/django/first_project$ cat polls/admin.py 
from django.contrib import admin

# Register your models here.
from .models import Question 

admin.site.register(Question)
```

Now that we've **registered** `Question`, re-running the development server should now show the 'polls' app `Question` objects within our **admin index page**.

Clicking on 'Questions' polls our server for our `Question` objects, returning all the questions within our database:

```
[24/Jun/2020 14:47:56] "GET /admin/polls/question/ HTTP/1.1" 200 4394
[24/Jun/2020 14:47:56] "GET /admin/jsi18n/ HTTP/1.1" 200 3223
```

Here we can **create**, **edit** and **delete** our `Question` objects.

```
QUESTION
	Who says "Impressive work, Ms. Vance" ?
```

If we were to comment out the following code within our `Question` model in `polls/models.py`:

```python
# polls/models.py
    def __str__(self):
        return self.question_text
```        

Then our `Question` **object** would now look like:

```
	Question object (1)
```

Attempting to **Change** the question will return an amendable **Question text** field, as well as **Date published** field.

#### Things to note

This **'Change question'** form is automatically generated from the following `Question` model code:

```python
# polls/models.py
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
```

The different model field types (`DateTimeField`, `CharField`) correspond to the appropriate HTML input widget. Each type of field knows how to display itself in the Django admin.

Each `DateTimeField` receives JavaScript shortcuts, with **date** receiving a "Today" shortcut and calendar popup, and **times** receiving a "Now" shortcut as well as popup for commonly entered times.

<br/>

### An Overview (of views)

A **view is a "type" of web page** within a Django application that generally serves a specific function and has a specific template. In Django, web pages and other content are delivered by **views**.

Each **view** is represented by a **Python function** (or **method**, as is the case of class-based views).

Django will choose a view by examining the URL request. Django also allows much cleaner more "elegant" URL patterns such as: `/newsarchive/<year>/<month>/`.

For example, within in a **blog app**, we might have the following **views**:

* **Blog homepage** – displays the latest few entries.
* **Entry “detail” page** – permalink page for a single entry.
* **Year-based archive page** – displays all months with entries in the given year.
* **Month-based archive page** – displays all days with entries in the given month.
* **Day-based archive page** – displays all entries in the given day.
* **Comment action** – handles posting comments to a given entry.

In our **poll app**, we’ll have the following **four views**:

* **Question "index" page** – displays the latest few questions.
* **Question "detail" page** – displays a question text, with no results but with a form to vote.
* **Question "results" page** – displays results for a particular question.
* **Vote action** – handles voting for a particular choice in a particular question.

Source: https://docs.djangoproject.com/en/3.0/intro/tutorial03/

URL to view mappings are known as **URLconfs**, the URLconf mapping the URL pattern to a view. Currently we have **one view** which is viewable via http://localhost:8000/polls.

#### `polls/views.py`

```python
lawrence@cacti:~/git/django/first_project$ cat polls/views.py 
from django.http import HttpResponse

# Create your views here.
def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

#### `polls/urls.py`

```python
lawrence@cacti:~/git/django/first_project$ cat polls/urls.py 
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
```

#### `first_project/urls.py`

Any changes made to `polls.urls` will be included because the `path()` exists within our projects `urls.py`:

```python
lawrence@cacti:~/git/django/first_project$ cat first_project/urls.py 
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    # add our 'polls' app:
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

For more information on URLconfs, see here: https://docs.djangoproject.com/en/3.0/topics/http/urls/

<br/>

### Writing more views

Currently we only have one view. Now we're going to add some more views, however these views will be slightly different because they will take an **argument** - `question_id`.

#### Create additional views

```diff
lawrence@cacti:~/git/django/first_project$ cat polls/views.py 
  from django.http import HttpResponse
  
  # Create your views here.
  def index(request):
      return HttpResponse("Hello, world. You're at the polls index.")
  
+ def detail(request, question_id):
+     return HttpResponse("You're looking at question %s." % question_id)

+ def results(request, question_id):
+     response = "You're looking at the results of question %s."
+     return HttpResponse(response % question_id)

+ def vote(request, question_id):
+     return HttpResponse("You're voting on question %s." % question_id)
```

#### Wire views into URLconf

Now we need to **wire** these newly created views into the `polls.urls` module by adding the following `path()` calls:

```diff
lawrence@cacti:~/git/django/first_project$ cat polls/urls.py 
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
+   # ex: /polls/5/
+   path('<int:question_id>/', views.detail, name='detail'),
+   # ex: /polls/5/results/
+   path('<int:question_id>/results/', views.results, name='results'),
+   # ex: /polls/5/vote/
+   path('<int:question_id>/vote/', views.vote, name='vote'),    
]
```

So what does this code do? Well, ensure that your dev server is running and go to http://localhost:8000/polls/34/

As a result of the this code, you should see `'You're looking at question 34.'` in your browser:

```python
    path('<int:question_id>/', views.detail, name='detail'),
```

Likewise, the lines that proceed it should also work, displaying the placeholder results and voting pages: http://localhost:8000/polls/34/results and http://localhost:8000/polls/34/vote.

#### The request

1. A request is made, such as http://localhost:8000/polls/34/.
2. Django will load the `mysite.urls` **Python module** because it is pointed to by the `ROOT_URLCONF` setting.
3. It then finds the variable named `urlpatterns` and **searches the patterns** in order.
4. After finding a match at `'polls/'`, is **strips off** the matching text (in this case `'polls/'`).
5. It then sends the remaining text `'34/'`, to the `polls.urls` URLconf for further processing.
6. Lastly it matches `'<int:question_id>/'`, resulting in a call to the `detail()` view like so:

```
detail(request=<HttpRequest object>, question_id=34)
```

The `question_id=34` part comes from `<int:question_id>`. Using **angle brackets** captures part of the URL and sends it as a keyword argument to the view function.

The `:question_id>` part of the string defines the name that will be used to identify the matched pattern, and the `<int:` part is a converter that determines what patterns should match this part of the URL path.

Source: https://docs.djangoproject.com/en/3.0/intro/tutorial03/

Therefore there is no need to add URL 'cruft' such as `.hmtl`, unless you wanted to, in which case it would look like this:

```python
path('polls/latest.html', views.index),
```

But this is unnecessary and advised against.

---

And that concludes **part one** of my adventures with Django. 

More to come `=)`
