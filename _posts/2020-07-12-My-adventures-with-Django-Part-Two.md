---
layout: post
title: Django Part Two
description: Django Part Two
summary: Django Part Two
comments: false
---

As I said in my original **Part One** post, I will be following the official documentation on how to build my first web application, with the intention to host it with a Web Server such as Nginx, probably in AWS. 

The purpose of this write-up is not to plagiarize the excellent documentation provided by the djangoproject.com, but rather to simplify and also expand upon some things which I feel the tutorial eludes.

https://www.djangoproject.com/

<br/>

### Making our views actually 'do something'

Every view is responsible for doing one of two things.

1. Returning an **HttpResponse** ojbect containing the content for the requested page.

2. Or raising an exception such as **Http404**.

**All Django wants is that `HttpResponse`. The rest is up to us to decide.**

For example, a view can read records from a database. It can use a template system such as Django's, or a third-party Python template system. It can even generate a PDF file, output XML, create a ZIP file on the fly, and more.

The point is, **you can do anything you want, using whatever Python libraries you want.** So long as Django gets that `HttpResponse` (or `Http404`).

#### Building a new `index()`

For convenience sake, we're going to use Django's own database API which we covered earlier, and create a new `index()` view. 

Our new view is going to display the following:
* The latest 5 poll questions within our database.
* With each question separated by commas, according to publication date.

Naturally, we'll be amending our app's `views.py` file:

```python
from django.http import HttpResponse
from .models import Question

# Create your views here.

# Our OLD index view
#def index(request):
#    return HttpResponse("Hello, world. You're at the polls index.")

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# Leave the rest of the views (detail, results, vote) unchanged

def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

Let's look at what we've actually done here.

Here we've just added `from .models import Question` which imports our `Question` model from:

```python
(django-prod) lawrence@cacti:~/git/django/first_project$ cat polls/models.py 
...
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
...
```

Then, within our new `index()`, we create a `latest_question_list` list object from 5 `Question` objects ordered by `'-pub_date'`:

```python
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
```

Next we use the Python String `join()` method, joining all our items from our iterable `latest_question_list` object, into a **string**, using a comma character `, ` as a separator. 

```python
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
```

We refer to `[q.question_text for q in latest_question_list]` as **list comprehension**. 

#### List comprehension

For example:

```python
>>> foobar  = [ foo for foo in 'foobar' ]
>>> foobar
['f', 'o', 'o', 'b', 'a', 'r']
```

Rather than creating an empty list e.g. `foo = []` and adding each element to the end `'foobar'`, instead we **define the list and its contents at the same time**.

Each list comprehension in Python is made up of three elements:

```python
new_list = [expression for member in iterable]
```

These elements are:
1. `expression` is the member itself, a call to a method, or any other valid expression that returns a value. 
2. `member` is the object or value in the list or iterable. 
3. `iterable` is a `list`, `set`, `sequence`, `generator`, or any other object that can return its elements one at a time.

For example:

```python
>>> squares = [i * i for i in range(10)]
>>> squares
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

In the example above, the **expression** `i * i` is the square of the **member** value. The member value being `i`. And the `iterable` being `range(10)`.

Source: realpython.com

As per usual, realpython.com has an excellent post on using List Comprehension here:
* https://realpython.com/list-comprehension-python/

<br/>

### Continuing with our new `index()` view

In the previous step we added the following to our `polls/views.py` file:

```python
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
```

#### However, there is a problem here.

The page’s design is hard-coded in the **view**, therefore if we want to make any changes to the way the page looks, we'll have to edit our code. 

#### The solution.

We're going to workaround this by using Django’s template system to **separate the design from Python** by creating a template that the view can use.

We'll start by creating a directory called `templates` in your `polls/` directory. Django will look for templates in there:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ mkdir polls/templates
```

The project’s `TEMPLATES` setting describes how Django will **load and render** templates. The **default** `settings.py` configures a `DjangoTemplates` backend whose `APP_DIRS` option is set to `True`:

```python
# first_project/settings.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

By convention `DjangoTemplates` looks for a `templates/` subdirectory in each of the `INSTALLED_APPS`.

Within the `templates` directory you have just created, create another directory called `polls`, and within that create a file called `index.html`:

```
(django-prod) lawrence@cacti:~/git/django/first_project/polls/templates$ mkdir polls ; touch polls/index.html
```

So our directory structure that our **templates** reside in should look like `polls/templates/polls/index.html`.

Because of how the `app_directories` template loader works as described above, you can refer to this template within Django as `polls/index.html`.

#### Template namespacing (best practice)

You might be able to get away with putting our templates directly in `polls/templates/` rather than `polls/templates/polls/`, however this would be a bad idea.

**Django will choose the first template it finds whose name matches.** The flipside of this is if you had a template with the **same name** in a **different application**, Django would be unable to distinguish between them.

By **namespacing** our templates (putting our templates within another directory named for the app itself), we are able to point Django at the correct one. 

#### Populating our index.html template file

Let's begin by populating our `index.html` file with the following:

```html
(django-prod) lawrence@cacti:~/git/django/first_project$ cat polls/templates/polls/index.html

{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

For the sake of this tutorial, the **example HTML code** is incomplete. Individual HTML elements aren't very useful on their own, thus, in the real world these elements would be combined to form a **complete** HTML page.

Anatomy of a HTML document: https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/Getting_started#Anatomy_of_an_HTML_document.

#### Updating our index view to use the template

Next, is to update our `index` view within `polls/views.py` so that it can use our newly created template.

#### Original code:

```python
# polls/views.py
from django.http import HttpResponse
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
...    
```

#### New code pointing to template:

```python
# polls/views.py
from django.http import HttpResponse
from django.template import loader
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
...
```

#### django.template.loader

First we import the `loader` submodule from `django.template`:

```python
from django.template import loader
```

Then within our `index()` view, we create a `template` **object**, using the `django.template.loader.get_template` method to **load and return our template** `polls/index.html`: 

```python
    template = loader.get_template('polls/index.html')
```

* `django.template.loader` https://docs.djangoproject.com/id/2.2/_modules/django/template/loader/.
* https://docs.djangoproject.com/en/3.0/ref/templates/api/.

#### Context

Next we construct a `context` object. A `Context` is a **dictionary with variable names as the `key`** and **dictionary values as the `value`**. The **template variable names** are represented in double curly braces. The purpose of this is to pass our **dictionary** to the `django.shorcuts.render()` method, and populate all our **template variables** with the corresponding `key`, the `key` being a Python object:

```
    context = {
        'latest_question_list': latest_question_list,
    }
```

#### `Context` Example

I'll use an example from StackOverflow that I best describes this:
1. The `context` for the **template** looks like: `{myvar1: 101, myvar2: 102}`.
2. This is then rendered with the **template** `render()` method.
3. The **template variable names** (represented in double curly braces) then become the following: 
4. `{{ myvar1 }}` becomes `{{ 101 }}`.
5. `{{ myvar2 }}` becomes `{{ 102 }}`

Credit: https://stackoverflow.com/questions/20957388/what-is-a-context-in-django#:~:text=13,in%20each%20render()%20call.

#### Rendering a context with the `render()` shortcut

So once we've compiled our `Template` object, we can **render** our `Context` with it. This way, you **can use the same template to render it server times with different contexts**.

However, it very common to **load a template**, **fill a context** and **return an HttpResponse object with the result of the rendered template**. 

Therefore, to make things simpler, Django provides a shortcut called `django.shortcuts.render()`.

Here is the code we previously composed with the previous steps:

```python
# polls/views.py
from django.http import HttpResponse
from django.template import loader
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {'latest_question_list': latest_question_list}
    return HttpResponse(template.render(context, request))
...
```

Now here is the full `index()` **view**, but rewritten with the `render()` **shortcut**:

```python
# polls/views.py
from django.http import HttpResponse
from django.shortcuts import render
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
...
```

Notice that we no-longer require `loader.get_template`  as well as returning a `HttpResponse`.

The `render()` function takes the `request` object as its **first argument**, the template name `polls/index.html` as the **second argument**, and lastly our `context` dictionary as an **optional third argument**.

It then **returns an `HttpResponse` object** of the given template rendered with the given context. Neat huh.

<br/>

### Raising a 404 error

You should remember earlier, that we said each view is rsponsible for doing one of two things: returning an `HttpResponse` object containing the **content** for the requested page, or raising an **exception** such as `Http404`. 

So now that we've written our `index()` **view**, capable of generating a `HttpResponse` along with our **content**, we now require our **view** to be able to raise an **exception**.

To do this, we can use a good ol' `try` block and `raise` an `Http404` exception if a question with the requested ID doesn't exist:

```python
# polls/views.py
from django.http import HttpResponse
from django.template import loader
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)

# Raise a 404 error
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```

Within our exception, we reference a new `template` named `detail.html`. We'll talk about this soon, so now and for the sake of functionality, we will keep it simple:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ cat >polls/templates/polls/detail.html
{{ question }}
```

#### Using a shortcut to raise Http404

Yep, you guessed it, just like we were able to use a **shortcut** for our `HttpResponse` object, Django has a shortcut for raising an exception if the object doesn't exist.

Here is our `detail()` **view**  rewritten with the `get_object_or_404()` **shortcut**:

```python
# polls/views.py
from django.http import HttpResponse
from django.template import loader
from .models import Question
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
...
```

The `get_object_or_404()` function takes a Django **model** as its **first argument**, in this case our `Question` model.

It can then take an arbitrary number of keyword arguments, which it passes to the `get()` function of the **model’s manager**, in our case, the `question_id`. 

Lastly it raise an `Http404` response if the object doesn’t exist.

#### Django's 'coupling' philosophy

Why do we use a helper function `get_object_or_404()` instead of automatically catching the `ObjectDoesNotExist` exceptions at a higher level, or having the model API raise `Http404` instead of `ObjectDoesNotExist`?

**Because that would couple the model layer to the view layer.** One of the foremost design goals of Django is to maintain **loose coupling**. Some controlled coupling is introduced in the `django.shortcuts` module.

There’s also a `get_list_or_404()` function, which works just as `get_object_or_404()` – except using `filter()` instead of `get()`. It raises `Http404` if the **list is empty**.

<br/>

### Using the template system

In a previous step, we mentioned that our new `detail()` **view** references a new `template` named `detail.html`. Right now it only looks like this:

```html
# polls/templates/polls/detail.html
{{ question }}
```

The `{{ question }}` variable returns our `question` **object** as a result of the code we recently added to our `detail()` **view**:

```python
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

As a result of our very basic `detail.html` **template**, navigating to http://localhost:8000/polls/1/ returns the `question` object and nothing else:

```html
<html><head></head><body>Who says "Impressive work, Ms. Vance" ?
</body></html>
```

#### Revamping templates/polls/detail.html

Let's spruce our template up a bit, after all it makes sense that our `detail()` **view** should return some detail about our `question`.

Considering the context variable `question`, ideally we would want to see what choices are associated with the question, therefore update the template with the following:

```html
# polls/templates/polls/detail.html
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

Just like our `index()` view `template`, the template system uses **dot-lookup syntax** to access variable attributes. 

In the example of `{{ question.question_text }}`, first Django does a **dictionary lookup** on the object `question`:

```html
<h1>{{ question.question_text }}</h1>
```

The **dictionary lookup** fails, of which it then tries an **attribute lookup** – which works. If the attribute lookup had failed, it would’ve tried a **list-index lookup**. I'll explain...

<br/>

### Dot lookup process in Django templates

Dots have a special meaning in **template rendering**. A dot in a variable name signifies a lookup. 

When the Django template system encounters a dot in a variable name such as `{{ foo.bar }}`, it attempts a successful lookup in the following order:

#### 1. Dictionary Lookup

In dictionary lookup, it will try to perform lookup assuming `foo` as a **dictionary** and `bar` as a **key** in that dictionary:

```
foo["bar"] # perform dictionary lookup   
```

#### 2. Attribute Lookup

When the dictionary lookup fails, it performs an attribute lookup, attempting to access the `bar` **attribute** in `foo`:

```
foo.bar # perform attribute lookup  
```

#### 3. List-index Lookup

Lastly, it will attempt a list-index lookup, attempting to access the `bar` **index** in `foo`:

```
foo[bar] # perform index lookup   
```

#### Example from djangoproject.com

```python
>>> from django.template import Context, Template
>>> t = Template("My name is {{ person.first_name }}.")

# Dictionary lookup
>>> d = {"person": {"first_name": "Joe", "last_name": "Johnson"}}
>>> t.render(Context(d))
"My name is Joe."

# Attribute lookup
>>> class PersonClass: pass
>>> p = PersonClass()
>>> p.first_name = "Ron"
>>> p.last_name = "Nasty"
>>> t.render(Context({"person": p}))
"My name is Ron."

# List-Index lookup
>>> t = Template("The first stooge in the list is {{ stooges.0 }}.")
>>> c = Context({"stooges": ["Larry", "Curly", "Moe"]})
>>> t.render(c)
"The first stooge in the list is Larry."
```

Credit goes to the kind people on StackOverflow for demystifying this for me:
* https://stackoverflow.com/questions/32767629/django-dot-lookup-syntax-to-access-variable-attributes
* https://docs.djangoproject.com/en/3.0/ref/templates/api/#variables-and-lookups

<br/>

### Back to using the template system

Going back to our `detail.html` template, as mentioned, our attribute lookup is successful for:

```html
# polls/templates/polls/detail.html
<h1>{{ question.question_text }}</h1>
```

Next, method-calling happens in the **for** loop; `question.choice_set.all` is interpreted as the Python code `question.choice_set.all()`, returning an interable of `Choice` objects:

```html
#<ul>
#{% for choice in question.choice_set.all %}
#    <li>{{ choice.choice_text }}</li>
#{% endfor %}
#</ul>
```

Now when we receive our `HttpResponse` content, we should see the following in our browser:

```html
<html><head></head><body><h1>Who says "Impressive work, Ms. Vance" ?</h1>
<ul>
    <li>Dr. Breen</li>
    <li>G-Man</li>
    <li>Vortigaunt</li>
</ul></body></html>
```

More information on templates here https://docs.djangoproject.com/en/3.0/topics/templates/.

<br/>

### Removing hardcoded URLs in templates

If we revisit our `templates/polls/index.html` template:

```html
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

The URL link to our `question.id` was partially hardcoded.  

The issue with hardcoding, is that it goes against Django's loose-coupling approach, meaning we would need to amend the URL depending on the project. This becomes challenging trying to change URLs on projects with lots of templates.

* https://docs.djangoproject.com/en/3.0/misc/design-philosophies/#loose-coupling

However, since we defined the **name argument** in the `path()` functions in the `polls.urls` module:





We can therefore remove a reliance on specific URL paths defined in our URL configurations by using the **URL template tag**.

Changing this:

```html
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```  

To this:

```html
        <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

This way, Django looks up the **URL definition** as specified in the `polls.urls` module. You can clearly see where the **URL name** of `detail` is defined here:

```python
# polls.urls
...
    # the 'name' value as called by the url template tag
    path('<int:question_id>/', views.detail, name='detail'),
...
```

#### Changing the URL of the polls detail view

If we wanted to change the URL of the polls detail view to something else, such as `polls/specifics/12/`, **we no-longer have to amend the template** since it was decoupled. Instead we would change it in `polls/urls.py`.

Example `polls/urls.py`:

```python
...
    #path('<int:question_id>/', views.detail, name='detail'),
    path('specifics/<int:question_id>/', views.detail, name='detail'),
...
```

Now the URL http://localhost:8000/polls/1/ would no-longer work. 

In which case the new URL would be http://localhost:8000/polls/specifics/1/.

<br/>

### Namespacing URL names

Currently our project `first_project` only has one app named `polls`. In reality, a Django project might contain anything from five, ten, to twenty apps (or more).

**Django is able to differentiate the URL names between them by us adding namespaces to our URLconf.**

For example, without namespacing, another app within our project such as a blog, might have a `detail` **view** such as the one within our `polls` app, therefore Django would not know which **app view** to create for the URL when using the URL template tag.

#### Setting the application namespace

We can fix this problem by adding namespaces to our **URLconf** in the `polls/urls.py` file, adding an `app_name` to **set the application namespace**:

```python
from django.urls import path
from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

This code sets the application namespace, so now Django knows which app view we are referring to:

```python
app_name = 'polls'
```

Lastly, to make these changes live, we need to amend the `polls/index.html` **template** from:

```html
        <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

And use `'polls:detail'` so that it points at the **namespaced** `detail` **view** within `polls`:

```html
        <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

So now the complete `template/polls/index.html` **template** should look like this:

```html
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

Easy. That covers the basics of writing views.

<br/>

## Form processing and generic views

In part 4 of the djangoproject.com tutorial, we'll focus on **form processing** and trimming down our code https://docs.djangoproject.com/en/3.0/intro/tutorial04/.

<br/>

### Writing a minimal form

We'll begin by updating our `templates/polls/detail.html` **template** so that it contains a HTML `<form>` element.

Old `index.html` code:

```html
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

New `index.html` code with `<form>` element:

```html
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>
```

So what have we done here?

For a start, it is good to recognise that there are different HTML form elements available.

The `<input>` element can be displayed in several ways depending on the `type` attribute:

```html
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
```    

`<input>` elements of `type="radio"` are generally used in radio groups, **collections of radio buttons describing a set of related options**. A radio button or option button is a graphical control element that allows the user to choose only one of a predefined set of mutually exclusive options.

We use a `forloop.counter` indicating how many times the `for` **tag** has gone through its loop.

The HTML `<label>` element represents a caption for an item within our graphical interface.

So `for` each `choice` we have a `radio` button, and a `label` denoting its name `choice_text`.

We have another `<input>` element, in this case `type="submit"` named `"vote"`, which defines a submit button which **submits all form values to a form-handler**, the **form-handler** being specified in our forms `form action` attribute:

```html
<form action="{% url 'polls:vote' question.id %}" method="post">
...
<input type="submit" value="Vote">
</form>
```

The `method="post"` method attribute specifies how to send our form-data, in this case to the URL specified within the `action` attribute `"url 'polls:vote' question.id"`. The **form-data** can be sent as either:
* **URL variables** `method="get"`.
* Or as a **HTTP post transaction** with `method="post"`.

Whenever creating a form that alters data server-side, use `method="post"` (as opposed to `method="get"`), this is a general rule for Web development.

As we're creating a POST form (which can have the effect of modifying data), we need to consider **security** and **Cross Site Request Forgeries**. Thankfully Django has a system for protecting against such things, therefore all POST forms that are targeted at internal URLs should  use `csrf_token` **template tag**.

_To summarise a **Cross-site request forgery**, also known as **one-click attack** or **session riding** and abbreviated as CSRF or XSRF, is a type of malicious exploit of a website where unauthorized commands are transmitted from a user that the web application trusts._

#### Concluding our <form>

So now http://localhost:8000/polls/1/ should return the following in our browser:

```html
<html><head></head><body><h1>Who says "Impressive work, Ms. Vance" ?</h1>

<form action="/polls/1/vote/" method="post">
<input type="hidden" name="csrfmiddlewaretoken" value="MHfr8AoNDdrwAw7E0VIPQueX2kmtZna3JlCz9N6HHBFSauOiOWPBWNPO9xgX6nBu">

    <input type="radio" name="choice" id="choice1" value="1">
    <label for="choice1">Dr. Breen</label><br>

    <input type="radio" name="choice" id="choice2" value="2">
    <label for="choice2">G-Man</label><br>

    <input type="radio" name="choice" id="choice3" value="6">
    <label for="choice3">Vortigaunt</label><br>

<input type="submit" value="Vote">
</form></body></html>
```

So now we have a working HTML form which follows the basic concept of:
* A template that displays a radio button for each question choice.
* A `value` for each radio button being associated with the **question choice's ID**.
* The `name` of each radio button is `"choice"`, so when any one of the radio buttons is selected and the form is submitted, it'll send a POST data `choice=#` where `#` is the ID of the selected choice.

<br/>

### Handling our submitted data

So now that we've create a `<form>` capable of submitting data, now we need to tell Django how to handle that data.

If you remember, we create a `URLconf` for our polls application: 

```python
# polls/urls.py
from django.urls import path
from . import views

# sets applications namespace to 'polls'
app_name = 'polls'
urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    #path('specifics/<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),    
]
```

This line being of interest to us:

```python
    # polls/urls.py
    path('<int:question_id>/vote/', views.vote, name='vote'),
```

We also created a **dummy** implementation of the `vote()` function within our `polls/views.py` file:

```python
# polls/views.py
...
def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

So now let's create a working version of `vote()` that can actually handle our data.

Original code:

```python
from django.http import HttpResponse
from django.shortcuts import get_object_or_404, render
from .models import Question

# Create views here
def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

We'll add the following code to our `vote()` function:

```python
# polls/views.py
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question

def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

So now our `polls/views.py` file should look like the following:

```python
# polls/views.py
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from .models import Choice, Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

#### Dissecting our new 'vote()' function

Now that we've realised our new `vote()` function, **how does it actually work?**

Firstly `request.POST` is a **dict-like object** that lets you access submitted data by `key` name. In this case, `request.POST['choice']` returns the **ID of the selected choice**, as a **string**. 

`request.POST` values are always strings. 

Django also provides `request.GET` for accessing GET data in the same way – but we’re explicitly using `request.POST` in our code, to ensure that data is only altered via a **POST call**.

```python
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
```

`request.POST['choice']` will raise `KeyError` if `choice` wasn’t provided in **POST data.** 

```python
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
```

The above code checks for `KeyError` and redisplays the question form with an error message if `choice` isn’t given:

```python
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        }
```

After incrementing the choice count:

```python
selected_choice.votes += 1
```

The code returns an `HttpResponseRedirect` rather than a normal `HttpResponse`. `HttpResponseRedirect` takes a single argument: the URL to which the user will be redirected (see the following point for how we construct the URL in this case):

```python
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```        

As the Python comment above points out, you hould **always return** an `HttpResponseRedirect` after successfully dealing with POST data. **This tip isn’t specific to Django**; it’s good Web development practice in general.

We are using the `reverse()` function in the `HttpResponseRedirect` constructor in this example:

```python
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

This function helps prevent having to hardcode a URL into the `view()` function. It is given the name of the view that we want to pass control to and the variable portion of the URL pattern that points to that view. In this case, using the URLconf we set up in Tutorial 3, this `reverse()` call will return a string like:

```
'/polls/3/results/'
```

Where the `3` is the value of `question.id`. This redirected URL will then call the `'results'` view to display the final page.

<br/>

### Fixing up results()

As we discussed earlier, `request` is an `HttpRequest` **object**, more information on HttpRequest objects here:
* https://docs.djangoproject.com/en/3.0/ref/request-response/#django.http.HttpRequest

Now that we've got a working `vote()` function view, after somebody votes, they are redirected to the `results` page via the `HttpResponseRedirect`, for the question:

```python
# polls/views.py
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

Submitting via the `vote()` function on our page, currently returns the following HTML:

```html
<html><head></head><body>You're looking at the results of question 1.</body></html>
```

As you can see, it's not very helpful, with no results displayed. Therefore we now need to write a view for results, which does exactly that:

Old `results()` function:

```python
def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)
```

New `results()` function:

```python
from django.shortcuts import get_object_or_404, render

def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```

Now we're referencing a `templates/polls/results.html` file that doesn't yet exist, so let's fix that:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ touch polls/templates/polls/results.html
```

```html
# polls/results.html
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

Now when we go to `polls/1/` in our browser, and vote in our question, we should see the following in our browser:

```html
<html><head></head><body><h1>Who says "Impressive work, Ms. Vance" ?</h1>

<ul>
    <li>Dr. Breen -- 2 votes</li>

    <li>G-Man -- 1 vote</li>

    <li>Vortigaunt -- 0 votes</li>
</ul>

<a href="/polls/1/">Vote again?</a></body></html>
```

Much better!

And if we submit the form without selecting a choice, we should receive an error message:

```html
<html><head></head><body><h1>Who says "Impressive work, Ms. Vance" ?</h1>

<p><strong>You didn't select a choice.</strong></p>

<form action="/polls/1/vote/" method="post">
<input type="hidden" name="csrfmiddlewaretoken" value="9igZAU2CHqacszNumQMdrGn9kDiqWl9G6WD7B7KwLOoy2xu8aRTZxZY0rQcU3lA7">

    <input type="radio" name="choice" id="choice1" value="1">
    <label for="choice1">Dr. Breen</label><br>

    <input type="radio" name="choice" id="choice2" value="2">
    <label for="choice2">G-Man</label><br>

    <input type="radio" name="choice" id="choice3" value="6">
    <label for="choice3">Vortigaunt</label><br>

<input type="submit" value="Vote">
</form></body></html>
```

Looks good so far.

#### One more thing

The code for our `vote()` **view** has a small problem.

Currently it works in the following way:
1. It first gets the `selected_choice` object from the database.
2. Then it computes the new value of `votes`. 
3. Lastly it then saves it back to the database.

**This is a potential problem** if two users of our website were to try to vote at exactly the same time.

For example, the same value `42` will be retrieved for `votes`, then for both users the new value of `43` is computed and saved, but `44` would be the expected value as two people voted, not one.

This is known as a **race condition**, and can be avoided using `F()`. To solve this issue, read how to avoid race conditions using F():
* https://docs.djangoproject.com/en/3.0/ref/models/expressions/#avoiding-race-conditions-using-f

<br/>

### Use generic views: less code is better

https://docs.djangoproject.com/en/3.0/intro/tutorial04/

If we take a look back at our `polls/views.py` file, the `index()`, `detail()` and `results()` **views** are very short, and also feature redundant code.

```python
def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})

def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```    

These views represent a common case of basic web development: 
1. Getting data from the database according to a parameter passed in the URL.
2. Loading a template.
3. Returning a rendered template. 

As this is so common, Django provides a (you guessed it) **shortcut**, called the **"generic views"** system.

#### Generic views

Generic views abstract common patterns to the point where you don't even need to write Python code to write an app. 

Up until now we have been writing our views "the hard way", this was so we could better learn the core concepts of Django. Normally when writing an app in Django, we would evaluate whether generic views are a good fit for our problem, and if so, implement them at the start rather than refactoring our code midway.

Let's convert the polls app to use the generic views system, then we can remove some of our previously written code. We will do this in the following order:

1. Convert the URLconf in `polls/urls.py`.
2. Delete some of our old, unneeded views within `views.py`.
3. Introduce new views based on Django's generic views to `views.py`.

#### Amend URLconf

To begin, we'll amend our `polls/urls.py` file from:

```python
from django.urls import path
from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),    
]
```

To the following:

```python
from django.urls import path
from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

For a start, we've amended the first **path string**:

```python
    # old code
    path('', views.index, name='index'),
    # new code
    path('', views.IndexView.as_view(), name='index'),
```    

Here we're referring to our new (and not yet created) `IndexView` **view**.

The same goes for the other **views**, of which we've updated to `DetailView` and `ResultsView`. We'll create these soon when we replace our old views.

Notice that the matched pattern in the **path strings** of the second and third patterns has changed from `<question_id>` to `<pk>`:

```python
    # Old code
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    # New code
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
```

One of our generic views, `generic.DetailView`, expects the primary key value from the URL to be called `"pk"`, therefore we've changed `question_id` to `pk` for the sake of our generic views.

#### Primary key fields

`pk` is short for **primary key**, which is a unique identifier for each record in a database. 

**Every Django model has a field which serves as its primary key**, and whatever other name it has, it can also be referred to as `pk`.

An `id` field is added automatically, but this behavior can be overridden:

By default, Django gives each **model** the following field:

```python
id = models.AutoField(primary_key=True)
```

This is what is referred to as an **auto-incrementing primary key**.

If we want to specify a **custom primary key,** we need to add `primary_key=True` to one of our fields, then Django sees we've explicitly set `Field.primary_key`, and as a result it won’t add the automatic `id` column.

Each model requires exactly one field to have `primary_key=True` (either explicitly declared or automatically added).

Automatic keys aside, this means we no-longer have to refer to our `Question` **model** and its `id` by name, rather we can use its **primary key** instead:

Source: 
* https://tutorial.djangogirls.org/en/extend_your_application/
* https://docs.djangoproject.com/en/3.0/topics/db/models/

### Transforming our views into 'generic views'

Now that we've amended our `polls/urls.py` URLconf, we're going to remove our old `index`, `detail`, and `results` **views** and replace them with **generic views**.

Our old (non generic) views:

```python
def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})

def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```  

Our new **generic views**:

```python
from django.views import generic

class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]

class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'

class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'
```

Here we are using two generic views:

The first being `generic.ListView` - "display a list of objects".

The second being `generic.DetailView` - "display a detail page for a particular type of object". This generic view expects the primary key value from the URL to be called `"pk"`, therefore we've changed `question_id` to `pk` for the sake of our generic views.

Each generic view needs to know what model it will be acting upon, therefore we specify this with the `model =` attribute.

#### generic.DetailView

The `generic.DetailView` uses a **default template** called `<app name>/<model name>_detail.html`:

```python
    model = Question
    template_name = 'polls/detail.html'
```

Within our example, it would use the template `"polls/question_detail.html"`. The `template_name` attribute is used to instruct Django to use a specific template name instead of the autogenerated template name.

#### generic.ListView

Similarly, the `generic.ListView` uses a **default template** called `<app name>/<model name>_list.html`. We use `template_name` to tell `generic.ListView` to use our existing `polls/index.html` **template**.

We also specify the `template_name` for `generic.ListView` to ensure the **detail** and **results** views have a different appearance when rendered, even though, behind the scenes they're both a `generic.DetailView`.

<br/>

### Telling Django to use the variables YOU want

Earlier on in the tutorial, the **templates** have been provided with a **context** that contains the `question` and `latest_question_list` **context** variables. For example:

```html
# templates/polls/detail.html
<h1>{{ question.question_text }}</h1>
...
```

```html
# templates/polls/index.html
{% if latest_question_list %}
...
```

```html
# templates/polls/results.html
...
{% for choice in question.choice_set.all %}
...
```

Remember that the `Context` **class** lives at `django.template.Context`, and the constructor takes two **(optional)** arguments:
* A dictionary mapping variable names to variable values.
* The name of the current application. This application name is used to help resolve namespaced URLs. 
  * If you’re not using namespaced URLs, you can ignore this argument.

So **a context is a dictionary mapping variable names to variable values**, and that we use `render()` to populate the template variables.

#### How to inform Django what variable it should use

Now, going back to our **generic views**, the `generic.DetailView` `question` **variable** is provided automatically as we're using `model = Question` so Django can determine an appropriate name for the **context variable** (in this case, `question`):

```python
class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'

class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'
```

However, for `generic.ListView`, the automatically generated context variable is `question_list`, therefore we override this by providing a `context_object_name`, in this case `latest_question_list`:

```python
class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'
```

As an **alternative approach**, you could change the templates to match the new default context variables - but it's a lot easier to do as we just did, and just **tell Django to use the variable you want**.

Now we can run the server, and use our new polling app based on our generic views =).

For full details on generic views, see the generic views documentation:
* https://docs.djangoproject.com/en/3.0/topics/class-based-views/

<br/>

## Introducing automated testing

We’ve built a Web-poll application, and we’ll now create some automated tests for it.

https://docs.djangoproject.com/en/3.0/intro/tutorial05/

#### What are automated tests?

Tests are routines that check the operation of your code and can operate at different levels:

Some tests might apply to a tiny detail, such as "does a particular model method return values as expected?". Whilst others examine the overall operation of the software, such as "does a sequence of user inputs on the site produce the desired result?".

We can debug our code **manually**, such earlier when we used the `shell` to examine the behavior of a method, or by just **running the application** and entering data to see how it behaves.

The difference between these methods and **automated tests** is that the testing work is done for us by the system. The idea is that we create a set of tests once, and then as we make changes to our app, we can check that the code still works as originally intended, without having to perform those manual tests again.

#### Tests will save you time

Sometimes, "check that it seems to work" will be a satisfactory test. However, in a more sophisticated application, there might be dozens of complex interactions between components. Any changes in those components could have unforseen consequences on our apps behavior, therefore a more thorough test is necessary.

The good news is that automated tests can perform these checks in just a matter of seconds. It can flag whether something is wrong immediately, and help us identify the code that's causing the unexpected behavior.

Whilst tearing yourself away from being productive, just so you can test your code to know that it's working properly, isn't all that enticing - writing tests are at least more fulfilling than spending hours testing your application manually.

#### Tests don't just identify problems, they help prevent them

One of the great things about testing, is that it can highlight things that you hadn't yet realised weren't working correctly. 

#### Tests make your code more attractive

Without tests, you may find that other developers will refuse to look at it because they don't trust it. 

#### Tests help teams work together

Complex applications will be maintained by teams. Tests guarantee that colleagues don't inadvertently break your code (and vice versa).

<br/>

### Basic testing strategies

When it comes to testing, there are multiple approaches to writing them.

Some prefer an approach known as **"test-driven development"**, whereby you write tests before you write code. This may seem like a strange way of doing things, but it is actually similar to how most people often solve something; 

* 1. Describe a problem
* 2. Create some code to solve it

**Test-drive development** formalises the problem in a Python test case.

Oftentimes someone new to testing will write some code and choose to add tests later on. Thankfully, this doesn't matter too much, so long as tests are written.

The **tricky part** can sometimes be where to start with writing your tests. For example, if you have thousands of lines of code, focusing on a specific thing to test might not be an easy job. Therefore, consider writting your first test the next time you make changes to your code.

<br/>

### Writing our first test

For the sake of our example, we left a bug in our `polls` app so we can get started right away.

#### We identify our first bug

The **bug** in question lies within our `Question.was_published_recently()` custom method in `polls/models.py`:

```python
# polls/models.py
import datetime
from django.db import models
from django.utils import timezone

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

The method returns `True` if the `Question` was published within the last day (which is correct) but also if the Question's `pub_date` field is in the future (which it definitely isn't):

```python
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)   
```

First we're going to confirm the bug using `shell` to check the method.

```
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py shell
Python 3.6.9 (default, Apr 18 2020, 01:56:04) 
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> 
```

Import the relevant packages:

```
>>> import datetime
>>> from django.utils import timezone
>>> from polls.models import Question
```

Now we need to create a `Question` instance with a `pub_date` in the future, in this **30 days in the future**:

```
>>> future_question = Question(pub_date=timezone.now() + datetime.timedelta(days=30))
```

Now let's test our `was_published_recently(self)` method using our newly created `Question` object:

```
>>> future_question.was_published_recently()
True
```

We got `True`, which confirms that we have a **bug**. As something published in the future is not considered to be **recent**, therefore this is undesirable behaviour and we should look to fix it.

<br/>

### Creating a test to expose our bug

Now that we've confirmed we have a bug, we can begin writing our automated test.

A common place for an application's tests is within the app's `tests.py` file; the **testing system** will automatically find tests in any file whose name begins with `test`.

We're going to populate our `polls/tests.py` file with the following:

```python
import datetime

from django.test import TestCase
from django.utils import timezone

from .models import Question


class QuestionModelTests(TestCase):

    def test_was_published_recently_with_future_question(self):
        """
        was_published_recently() returns False for questions whose pub_date
        is in the future.
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertIs(future_question.was_published_recently(), False)
```

#### Breaking down our automated test

Here we're importing the **test client** `django.test`, in this case its **subclass** `TestCase`. 

Source: https://docs.djangoproject.com/en/3.0/topics/testing/tools/#django.test.TestCase

Then with `TestCase`we create a **method** which creates a `Question` instance with a `pub_date` in the future: 

```python
    def test_was_published_recently_with_future_question(self):
        # set our time in the future
        time = timezone.now() + datetime.timedelta(days=30)
        # create a question instance with future time set
        future_question = Question(pub_date=time)        
```

Afterwards we check the output of `was_published_recently()` which **should** be `False`:

```python
        self.assertIs(future_question.was_published_recently(), False)
```        

If you're not familiar with the basic concepts of testing, then you probably haven't come across `assertIs()`, in which case:

* `assertIs(first, second, msg=None)`
  * This tests that first and second args **evaluate** to the same object.
* `assertIsNot(first, second, msg=None)`
  * This tests that first and second args **don’t evaluate** to the same object. 

See **unittest**: https://docs.python.org/3/library/unittest.html

<br/>

### Running our tests

Now that we have our automated test ready, we can begin running it.

```shell
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py test polls
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_was_published_recently_with_future_question (polls.tests.QuestionModelTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/home/lawrence/git/django/first_project/polls/tests.py", line 19, in test_was_published_recently_with_future_question
    self.assertIs(future_question.was_published_recently(), False)
AssertionError: True is not False

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
Destroying test database for alias 'default'...
```

As you can see, our test `FAILED`, which is what we expected given the result of our `was_published_recently` method returned `True` and what we were testing for was `False`:

```python
# models.py
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)   
```        

```python
# tests.py
        self.assertIs(future_question.was_published_recently(), False)
```

#### Breaking it down

1. By running `manage.py test polls` Django looked for tests within the `polls` app.
2. It found a **subclass** of the `djago.test.TestCase` **class**.
3. It then created a special database for the purpose of testing.
4. It looked for test methods - methods with names that begin with `test` i.e. `def test_was_published_recently_with_future_question()`. If this method was did not start with `test...` then it would not be picked up by the testing system.
5. Then within our `test_was_published_recently_with_future_question()` method it created a `Question` instance whose `pub_date` field is `days=30` in the future.
6. Using the `assertIs()` method, it discovered that `was_published_recently()` returns `True` (not `False` - which is what we ideally want).
7. Lastly the test informs us which **test** failed and flags the offending line which the failure occurred.

<br/>

### Fixing the bug (test-drive development)

Obviously, in this case, we already knew what the 'bug' was and thus the `FAILED` result of our automated test was as we expected. Now we can amend the `Question.was_published_recently()` method within our **models** `polls/models.py` file so that it will only return `True` if the date was in the past (not as well as the future).

Old buggy code:

```python
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)    
```

New code:

```python
    def was_published_recently(self):
        # set current timezone in 'now'
        now = timezone.now()
        # <= less than or equal to 'pub_date'
        return now - datetime.timedelta(days=1) <= self.pub_date <= now    
```        

Source: `timedelta` https://docs.python.org/3/library/datetime.html
* https://www.guru99.com/date-time-and-datetime-classes-in-python.html
* https://docs.djangoproject.com/en/3.0/topics/i18n/timezones/

Now that we've fixed our buggy code, we can run our test again:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py test polls
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
```

And this time our test passes with `no issues`. 

It is good to bare in mind that many other things might go wrong with our app in the future, however, we can be sure that in the event this bug gets reintroduced, we now have a **test** within our `tests.py` file that will warn us immediately.

This is why unit tests are an important and staple part of developing. 

Testing code and administering changes in this fashion is what is known as **test-driven development**.

<br/>

###  More comprehensive tests

Since we are looking at the `was_published_recently()` method, it would make sense for us to perform a more comprehensive test so to outline any further issues.

We'll do this by adding two more test methods to the **same class**, this way we can test the behaviour of the method more comprehensively:

```python
# polls/tests.py
def test_was_published_recently_with_old_question(self):
    """
    was_published_recently() returns False for questions whose pub_date
    is older than 1 day.
    """
    time = timezone.now() - datetime.timedelta(days=1, seconds=1)
    old_question = Question(pub_date=time)
    self.assertIs(old_question.was_published_recently(), False)

def test_was_published_recently_with_recent_question(self):
    """
    was_published_recently() returns True for questions whose pub_date
    is within the last day.
    """
    time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
    recent_question = Question(pub_date=time)
    self.assertIs(recent_question.was_published_recently(), True)
```

Now within the `tests.py` file, we should have three tests to help us confirm whether `Question.was_published_recently()` returns correct values for **past**, **recent**, and **future** questions.

```
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py test polls
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
...
----------------------------------------------------------------------
Ran 3 tests in 0.002s

OK
Destroying test database for alias 'default'...
```

As you can see from running our automated test again, that everything looks to be working correctly.

<br/>

### Test a view

In its current state, the polls app will publish any question, **including questions whose `pub_date` field resides in the future**. We'll tighten this up by ensuring that any question with a `pub_date` in the future, results in the Question being published at that moment and in the meantime, is invisible until then.

Now in our first test, we looked at the internal behaviour of our code, however for our next test we will check its behaviour through the web browser, as would be experienced by a user.

#### The Django `test` client

Django provides a **test** `Client` to simulate a user interfacing with the code at the view level. 

We can use the **test** `Client` in our `tests.py` file or even within the `shell`.

I'll begin again with the `shell`, here I'll need to do a couple things that won't be necessary in `tests.py`.

#### Setting up our test environment with testing tools

* https://docs.djangoproject.com/en/3.0/topics/testing/tools/
* https://docs.djangoproject.com/en/3.0/topics/testing/advanced/#django.test.utils.setup_test_environment

```
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py shell
```

```python
>>> from django.test.utils import setup_test_environment
>>> setup_test_environment()
```

The `setup_test_environment()` sub-module installs a **template renderer** which allows us to see some additional attributes on responses that we would not normall be able to see such as `response.context`.

It is important to note that `setup_test_environment()` **does not setup a test database**. Therefore the following examples are going to be run against the existing database and therefore the output may differ slightly depending on the questions we've created so far.

Next is to **import the test `Client()` class** (later on within `tests.py` we'll be using the `django.test.TestCase` class which comes with its own client so this won't be required).

```python
>>> from django.test import Client
>>> # create an instance of the client for our use
>>> client = Client()
```

Now we ready to begin testing our responses.

<br/>

### Testing responses

More information here: https://docs.djangoproject.com/en/3.0/topics/testing/tools/

1. Get a response from `'/'`:
   
```python
>>> response = client.get('/')
Not Found: /
```

2. We should have received a `404` (not found) from this address so we'll confirm with the `status_code` method:

```python
>>> response.status_code
404
```

3. We should expect to find something at `/polls/` so we'll use `reverse()` rather than a hardcoded URL:

```python
>>> from django.urls import reverse
>>> response = client.get(reverse('polls:index'))
>>> response.status_code
200
```

The HTTP `200` OK success status response code indicates that the request has succeeded.

The `django.urls` **utility** `reverse()` is useful when you want to use something similar to the `url` template tag in your code, it takes the following arguments:

```python
reverse(viewname, urlconf=None, args=None, kwargs=None, current_app=None)¶
```

The `viewname` can be a URL pattern name or in our case, the callable view object `polls:index`.

More information here: https://docs.djangoproject.com/en/3.0/ref/urlresolvers/

4. I can then call the `.content` of that request:

```python
>>> response.content
b'\n    <ul>\n    \n        <li><a href="/polls/1/">Who says &quot;Impressive work, Ms. Vance&quot; ?</a></li>\n    \n    </ul>\n'
>>> response.context['latest_question_list']
<QuerySet [<Question: Who says "Impressive work, Ms. Vance" ?>]>
```

The `.content` being the **body of the response**, as a bytestring. This is the final page content as rendered by the (index) view, or any error message.

And that sums up our test environment.

<br/>

### Improving our view

As mentioned earlier, our polls app currently shows questions that have not yet been published (i.e questions with a `pub_date` in the **future**), so we're going to fix that now.

Previously we introduced a class-based view, based on `generic.ListView`:

```python
class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]
```        

We're going to amend the `get_queryset()` method we created, to additionally check the date by comparing the `pub_date` with `timezone.now()`. 

I'll use `Question.objects.filter(pub_date__lte=timezone.now())` to return a `<QuerySet>` containing questions whose `pub_date` is **less than or equal to** (i.e earlier or equal to) `timezone.now`.

First we need to add an `import` for the Django utility `django.utils.timezone` in `views.py`:

```python
# views.py
from django.utils import timezone
```

Next we'll amend the `query_set` method within the `IndexView()` view as discussed:

```python
# views.py
# class IndexView(generic.ListView)
    def get_queryset(self):    
        # Return the last five published questions (not including 
        # those set to be published in the future)
        return Question.objects.filter(
            pub_date__lte=timezone.now()).order_by('-pub_date')[:5]
```

<br/>

### Testing our new index view

Now we've mitigated any 'future' problems, we can confirm everything is working as it should by running our server and creating `Questions` with dates in the past and future. If all is right, then we'll only see those that have been published recently are listed.

I'll also add another test to `polls/tests.py` so as to ensure that this gets always tested going forwards.

First I'll need to add `reverse()` which is a urlresolver we mentioned earlier.

```python
# polls/tests.py
from django.urls import reverse
```

Then **create a function** that creates questions:

```python
# polls/tests.py
def create_question(question_text, days):
    """
    Create a question with the given `question_text` and published the
    given number of `days` offset to now (negative for questions published
    in the past, positive for questions that have yet to be published).
    """
    time = timezone.now() + datetime.timedelta(days=days)
    return Question.objects.create(question_text=question_text, pub_date=time)
```

As the docstring suggests, this `create_question()` function, creates a question with `question_text` and sets a publish date with the given number of `days` offset to now.

Then I'll **create a new class** named `QuestionIndexViewTests()`:

```python
# polls/tests.py
class QuestionIndexViewTests(TestCase):
    def test_no_questions(self):
        """
        If no questions exist, an appropriate message is displayed.
        """
        response = self.client.get(reverse('polls:index'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "No polls are available.")
        self.assertQuerysetEqual(response.context['latest_question_list'], [])

    def test_past_question(self):
        """
        Questions with a pub_date in the past are displayed on the
        index page.
        """
        create_question(question_text="Past question.", days=-30)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question.>']
        )

    def test_future_question(self):
        """
        Questions with a pub_date in the future aren't displayed on
        the index page.
        """
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse('polls:index'))
        self.assertContains(response, "No polls are available.")
        self.assertQuerysetEqual(response.context['latest_question_list'], [])

    def test_future_question_and_past_question(self):
        """
        Even if both past and future questions exist, only past questions
        are displayed.
        """
        create_question(question_text="Past question.", days=-30)
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question.>']
        )

    def test_two_past_questions(self):
        """
        The questions index page may display multiple questions.
        """
        create_question(question_text="Past question 1.", days=-30)
        create_question(question_text="Past question 2.", days=-5)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question 2.>', '<Question: Past question 1.>']
        )
```

#### Breakdown of our new tests

Now, it's fair to say that's a lot of code we just added to `tests.py`, so we'll quickly run through each function before we move on.

Here we use the `TestCase` class for writing our tests, which provides us with some additional assertion methods such as in our case; `assertContains()` and `assertQuerysetEqual()`.

https://docs.djangoproject.com/en/3.0/topics/testing/tools/#django.test.TestCase

The `test_no_questions()` method confirms whether there are any questions available by using `assertQuerysetEqual()` to assert whether or not a queryset `latest_question_list` returns a particular list of values, in this case an empty list `[]`:

```python
    def test_no_questions(self):
        response = self.client.get(reverse('polls:index'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "No polls are available.")
        self.assertQuerysetEqual(response.context['latest_question_list'], [])
```

The `test_past_question()` method creates a question in the **past** by `days=-30` and tests whether it's displayed on the index page:

```python
    def test_past_question(self):
        create_question(question_text="Past question.", days=-30)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question.>']
        )
```

**NOTE that the database is reset for each test method, so the first question we created in `test_past_question()` is no longer there.**

The `test_future_question()` method creates a question with a `pub_date` in the **future** and checks whether the **index page** has any questions in it:

```python
    def test_future_question(self):
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse('polls:index'))
        self.assertContains(response, "No polls are available.")
        self.assertQuerysetEqual(response.context['latest_question_list'], [])
```

The `test_future_question_and_past_question()` method **creates two questions**, one with a **past** `pub_date` and the other being in the **future**. It then queries the index page to confirm that only the question in the **past** is displayed:

```python
    def test_future_question_and_past_question(self):
        create_question(question_text="Past question.", days=-30)
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question.>']
        )
```

Lastly, the `test_two_past_questions()` method **creates two questions** just like the previous function, only this time both questions are set in the **past**. It then confirms whether both are displayed on the index page:

```python
    def test_two_past_questions(self):
        create_question(question_text="Past question 1.", days=-30)
        create_question(question_text="Past question 2.", days=-5)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question 2.>', '<Question: Past question 1.>']
        )
```

So whilst it seems like a lot of code, in reality the functions are quite simple, and share a lot of similarities between one another and thus didn't require much effort to test.

These tests allow us to check at every state and for every new change in the state of the system, and help us confirm the overall **user experience** on the front-end side of things.

<br/>

### Testing the DetailView

Now that we've written the tests for our `generic.ListView` **index** view, now we need to amend our `DetailView` **view**, as whilst future questions don't appear in the **index page**, users could still reach them if they knew (or guessed) the URL.

#### Amending DetailView

Therefore we'll need to add a similar constraint to `DetailView()`:

```python
# polls/views.py
class DetailView(generic.DetailView):
    ...
    def get_queryset(self):
        """
        Excludes any questions that aren't published yet.
        """
        return Question.objects.filter(pub_date__lte=timezone.now())
```

So now our `DetailView` **view** should look like:

```python
# polls/views.py

class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'

    def get_queryset(self):
        return Question.objects.filter(pub_date__lte=timezone.now())
```

#### Testing DetailView

Now, just as we did for our **index page**, we also should add some additional tests to check that only our **questions** with a `pub_date` in the **past** can be displayed, and that those with a `pub_date` in the future are not:

```python
# polls/tests.py

class QuestionDetailViewTests(TestCase):
    def test_future_question(self):
        """
        The detail view of a question with a pub_date in the future
        returns a 404 not found.
        """
        future_question = create_question(question_text='Future question.', days=5)
        url = reverse('polls:detail', args=(future_question.id,))
        response = self.client.get(url)
        self.assertEqual(response.status_code, 404)

    def test_past_question(self):
        """
        The detail view of a question with a pub_date in the past
        displays the question's text.
        """
        past_question = create_question(question_text='Past Question.', days=-5)
        url = reverse('polls:detail', args=(past_question.id,))
        response = self.client.get(url)
        self.assertContains(response, past_question.question_text)
```

<br/>

### Running our finalised polls tests

Now that we've finished writing our tests and made our amendments where necessary, let's run them again:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py test polls
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..........
----------------------------------------------------------------------
Ran 10 tests in 0.023s

OK
Destroying test database for alias 'default'...
```

Everything was successful, looks good, and now we can begin moving on. But before we do, what other tests could we have added?

#### Ideas for more tests

We haven't yet written any tests for our `ResultsView()` view, so it would make sense to add a similar `get_queryset` method and **create a new test class** for this specific view as we have done the others.

```python
# polls/views.py
...
class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'
```

The application could also be improved in other ways, such as currently it is possible to publish questions that have no choices, so we could have our views check for this, and exclude such questions from being show. We would then reflect this in our tests by letting them create a `Question` without a `Choice` and testing whether or not it is published to the **view**, as well as creating a similar `Question` with multiple `Choice`, confirming it **is** published. 

You might even want **logged-in admin** users to be able to see unpublished questions, just not ordinary users of the site.

Whatever needs to be added to the software to accomplish the task, should always be accompanied by a test, in which order this happens it does not matter, so long as there is one to complement the other.

#### More is better

It's natural when writing tests, to repeat code, and whilst not aesthetically pleasing to look at, tests exist exactly to do just one thing and that's **test**. 

You'll find that your tests will continue to grow, and it is not uncommon to end up with 'test bloat'. This doesn't matter. For mosts tests, they can be written once and then forgetten about it whilst you continue to develop your program.

Sometimes you'll need to update a test, for example if we were to make some amendments to our views, we'd probably need to update our tests otherwise they'd begin failing. In this sense, tests help look after themselves, as when they fail, you know exactly what you need to focus on. 

You may even find over time, that some of the tests you write become redundant, in which case, testing redundancy is a good thing.

#### Arranging your tests

Sensibly arranging tests will help them from becoming unmanageable, therefore it is considered a general rule-of-thumb to including having:
* A separate `TestClass` for each model or view.
* A separate test **method** for each set of conditions you want to test.
* Test **method names** that describe their function.

That concludes the basics of testing. There if far more you can do within the world of testing, but for now that should get you on your way.

<br/>

---- 

<br/>

### Customising the look and feel of our app with CSS

https://docs.djangoproject.com/en/3.0/intro/tutorial06/

So far we've built, debugged, and confirmed our app works, now we are ready to do a little front-end web development to spruce things up.

We'll begin by adding a **style sheet** and an image. When a browser reads a style sheet, it will format the HTML document according to the information in the style sheet.

There are three types of Style Sheets:

* **Embedded:** The style rules are included within the HTML at the top of the Web page - in the head.
* **Inline:** The style rules appear throughout the HTML of the Web page - i.e. in the body.
* **Linked:** The style rules are stored in a separate file external to all the Web pages.

#### Static files

It is common for a web application to usually serve additional files other than just HTML, such as **images**, **JavaScript**, or **CSS**. These are often necessary to render a complete web page and are referred to as **"static files"** within Django.

Things however, can start to get complicated when working on larger projects comprised of multiple apps, especially when each app requires its own set of static files.

Thankfully there is a solution, `django.contrib.staticfiles` collects static files from each of our applications (and any additional places that we specify), and stores them in a single location ready to serve in production.

More information: https://docs.djangoproject.com/en/3.0/ref/contrib/staticfiles/

<br/>

### Static files and STATICFILES_FINDERS

Django's `STATICFILES_FINDERS` setting contains a list of finders that know how to discover static files from various sources:

```python
# default
[
    'django.contrib.staticfiles.finders.FileSystemFinder',
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
]
```

One of the default finders is `AppDirectoriesFinder`, which looks for a `"static"` subdirectory in each of `INSTALLED_APPS`: 

```python
# first_project/settings.py
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

The **admin site** also uses the same directory structure for static files.

https://docs.djangoproject.com/en/3.0/ref/settings/#std:setting-STATICFILES_FINDERS

<br/>

### Customizing our app's look and feel

So to begin, we'll create a folder named `static/` within our `poll/` directory. And as mentioned, Django will look for **static files** within `static/`, similarly to how it finds templates inside `polls/templates/`:

```
lawrence@cacti:~/git/django/first_project/polls$ mkdir static
```

Within our `static/` directory, we'll namespace our file structure by creating another directory named `polls/`, and within that create a file `style.css`:

```
lawrence@cacti:~/git/django/first_project/polls$ cd static/
lawrence@cacti:~/git/django/first_project/polls/static$ mkdir polls
lawrence@cacti:~/git/django/first_project/polls/static$ touch polls/style.css
```

---

**Static file namespacing** - If you remember what we did with `templates`, we _could_ get away with putting our files in `polls/static`, however we would run the risk of Django choosing the wrong files. **Django chooses the first static file it finds whose name matches**, therefore if we had a static file with the same name in a different application, it would be unable to distinguish between them. By namespacing, we are able to point Django to the right one.

---

Now we'll add the following CSS code to `polls/static/polls/style.css`:

```
lawrence@cacti:~/git/django/first_project/polls/static$ cat >polls/style.css 
li a {
    color: green;
}
```

Then we'll add the following HTML to `polls/templates/polls/index.html`:

```html
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">
```

So now our `style.css` file should look like the following:

```css
li a {
    color: orange;
}
```

And `index.html` should look like this:

```html
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">

{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

The `{% static %}` **template tag** generates the absolute URL of **static files**.

Now we can start our server again:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
July 12, 2020 - 17:06:19
Django version 3.0.7, using settings 'first_project.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

And reload `http://localhost:8000/polls/` and we should see that our question links are `orange` which indicates our **style sheet** was correctly loaded.

<br/>

### Adding a background-image

Now let's add a background image. We'll start by creating a subdirectory for images.

Create an `images/` directory within `polls/static/polls/`:

```
(django-prod) lawrence@cacti:~/git/django/first_project/polls/static/polls$ mkdir images
```

Then add an image to our `images/` directory and call it `background.png`:

```
(django-prod) lawrence@cacti:~/git/django/first_project/polls/static/polls$ cd images/(django-prod) lawrence@cacti:~/git/django/first_project/polls/static/polls/images$ wget -O background.png https://i.redd.it/1b7cpnxxcw631.png
--2020-07-12 17:18:46--  https://i.redd.it/1b7cpnxxcw631.png
Resolving i.redd.it (i.redd.it)... 199.232.57.140
Connecting to i.redd.it (i.redd.it)|199.232.57.140|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7149109 (6.8M) [image/png]
Saving to: background.png’

background.png                      100%[===========================================>]   6.82M  11.8MB/s    in 0.6s    

2020-07-12 17:18:47 (11.8 MB/s) - background.png’ saved [7149109/71491
```

Now add the image to our style sheet within `polls/static/polls/style.css`:

```css
body {
    background: white url("images/background.png") no-repeat;
}
```

So now `style.css` should look like the following:

```css
li a {
    color: green;
}
body {
    background: white url("images/background.png") no-repeat;
}
```

Now let's reload `http://localhost:8000/polls/` and you should see the background loaded in the top left of the screen.

And that concludes this basic tutorial of Django and front-end web development. I'll leave some links below on how to work with static files and hopefully I can go into this a little deeper in another post:

How-to: https://docs.djangoproject.com/en/3.0/howto/static-files/.

Reference: https://docs.djangoproject.com/en/3.0/ref/contrib/staticfiles/.

Deployment: https://docs.djangoproject.com/en/3.0/howto/static-files/deployment/.
