---
layout: post
title: Working with QuerySets and datetime in Django
description: Working with QuerySets and datetime in Django
summary: Working with QuerySets and datetime in Django
comments: false
---

I've written this to help clarify a few things for a colleague of mine whom has been following the 'Writing your first Django app' on djangoproject.com, and wanted to better understand the interaction between QuerySets and using Django's utilities such as `timezone`.

### The code in question

This is the code I am going to be breaking down, in particular the `was_published_recently` method within my `Question` **model** inside `models.py`:

```python
# models.py
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
    def __str__(self):
        return self.question_text
    def was_published_recently(self):
        now = timezone.now()
        return now - datetime.timedelta(days=1) <= self.pub_date <= now  
```

<br/>

### Using `shell`

I will be using `shell` as working within an **interpreter** is one of the most effective ways to learn how a piece of code works.

So I'll begin by launching into my project with the `manage.py shell` command:

```
(django-prod) lawrence@cacti:~/git/django/first_project$ python manage.py shell
```

<br/>

### Using datetime

The first thing I'm going to do is use the `django.utils.timezone` method `.now()` to get the current time:

```python
>>> from django.utils import timezone
>>> timezone.now()
datetime.datetime(2020, 6, 30, 20, 0, 35, 849635, tzinfo=<UTC>)
```

The Django utility `timezone` uses the `datetime` module which provides **classes** for manipulating dates and times: https://docs.python.org/3/library/datetime.html

Now I'm going to `import datetime` so I can use the `datetime.timedelta()` method to represent the duration of 1 day (`days=1`), of which I'll then subtract **1 day** from `now`:

```python
>>> import datetime
>>> now
datetime.datetime(2020, 6, 30, 20, 1, 27, 53907, tzinfo=<UTC>)
>>> now - datetime.timedelta(days=1)
datetime.datetime(2020, 6, 29, 20, 1, 27, 53907, tzinfo=<UTC>)
```

You can see that the third value in `datetime.datetime()` changes from `30` to `29` successfully.

These can be evaluated down to `year`, `day`, `month` etc:

```python
>>> timezone.now()
datetime.datetime(2020, 6, 30, 20, 25, 2, 253662, tzinfo=<UTC>)
>>> timezone.now().year
2020
>>> timezone.now().day
30
```

<br/>

### QuerySets

**QuerySet API reference:** https://docs.djangoproject.com/en/1.11/ref/models/querysets/

A QuerySet is a collection of objects from my projects database, of which I can apply zero or as many filters to as I like. For now, I will start by querying my db for all objects pertaining to my `Question` **model**:

```python
>>> from polls.models import Choice, Question
>>> Question.objects.all()
<QuerySet [<Question: Who says "Impressive work, Ms. Vance" ?>]>
```

So it would seem that I only have one `Question` object, which is fine, that's plenty to work with.

Now I'll create a variable to refer to my `Question` object as `q`. I do this by calling the QuerySet `.filter()` for the `id`, which in this case is `1`:

```python
>>> Question.objects.values_list()
<QuerySet [(1, 'Who says "Impressive work, Ms. Vance" ?', datetime.datetime(2020, 6, 23, 17, 10, 44, tzinfo=<UTC>))]>
>>> q = Question.objects.filter(id=1)
```

I can now make a `get()` call for specific **field value** of my model instance `q`:

```python
>>> q.get(pub_date__year=timezone.now().year)
<Question: Who says "Impressive work, Ms. Vance" ?>
```

As the `Question` object was created within this year (`2020`), it was returned.

However if I was to do the following, and subtract a year from the current year:

```python
>>> subtract_year = lambda a : a - datetime.timedelta(days=365)
>>> subtract_year(now).year
2019
>>> one_year_ago = subtract_year(now).year
```

Now when I attempt to return my `Question`, it would expectedly fail:

```python
>>> q.get(pub_date__year=one_year_ago)
Traceback (most recent call last):
  File "<console>", line 1, in <module>
  File "/home/lawrence/.virtualenvs/django-prod/lib/python3.6/site-packages/django/db/models/query.py", line 417, in get
    self.model._meta.object_name
polls.models.Question.DoesNotExist: Question matching query does not exist.
```

https://www.digitalocean.com/community/tutorials/how-to-do-math-in-python-3-with-operators


<br/>

### Filtering a QuerySet

So we've established that we have our `q` object which has a set of values stored within a list and a dictionary:

```python
>>> q.values_list()
<QuerySet [(1, 'Who says "Impressive work, Ms. Vance" ?', datetime.datetime(2020, 6, 23, 17, 10, 44, tzinfo=<UTC>))]>
```

```python
>>> q.values()
<QuerySet [{'id': 1, 'question_text': 'Who says "Impressive work, Ms. Vance" ?', 'pub_date': datetime.datetime(2020, 6, 23, 17, 10, 44, tzinfo=<UTC>)}]>
```

Now I can call the `datetimes()` method of our `q` object and give it the following arguments:

```python
>>> q.datetimes('pub_date', 'date')
Traceback (most recent call last):
  File "<console>", line 1, in <module>
  File "/home/lawrence/.virtualenvs/django-prod/lib/python3.6/site-packages/django/db/models/query.py", line 866, in datetimes
    "'kind' must be one of 'year', 'month', 'week', 'day', 'hour', 'minute', or 'second'."
AssertionError: 'kind' must be one of 'year', 'month', 'week', 'day', 'hour', 'minute', or 'second'.
```

As you can see, the second argument must be one of `'year'`, `'month'`, `'week'`, `'day'`, `'hour'`, `'minute'`, or `'second'`.

Therefore we'll try `'day'`:

```python
>>> q.datetimes('pub_date', 'day')
<QuerySet [datetime.datetime(2020, 6, 23, 0, 0, tzinfo=<DstTzInfo 'GB' BST+1:00:00 DST>)]>
```

So now I'll create a `pub_date_day` object and store within it the `datetime` information I want to work with, in this case, the `'pub_date'` field of my `Question` **model**, specifically the `'day'` it was published:

```python
>>> pub_date_day = q.datetimes('pub_date', 'day')
>>> pub_date_day
<QuerySet [datetime.datetime(2020, 6, 23, 0, 0, tzinfo=<DstTzInfo 'GB' BST+1:00:00 DST>)]>
```

Now I am ready to start evaluating my date information.

<br/>

### Evaluating our data

I'll begin by evaluating the result of `now - datetime.timedelta(days=1)` with `pub_date_day`:

```python
>>> now - datetime.timedelta(days=1) <= pub_date_day
Traceback (most recent call last):
  File "<console>", line 1, in <module>
TypeError: '<=' not supported between instances of 'datetime.datetime' and 'QuerySet'
```

So we've received a `TypeError` which means there is a problem with our evaluation. 

The `TypeError` clearly states that `<=` is not supported between these two instances, that is that `now - datetime.timedelta(days=1)` is being evaluated to `pub_date_day`, and `pub_date_day` is a `<QuerySet>`:

```python
>>> type(pub_date_day)
<class 'django.db.models.query.QuerySet'>
```

To overcome this issue, we need to access the `datetime` **value** of our `QuerySet` object:

```python
>>> for item in pub_date_day:
...     item
... 
datetime.datetime(2020, 6, 23, 0, 0, tzinfo=<DstTzInfo 'GB' BST+1:00:00 DST>)
```

To access the `datetime` value I can perform a `.get()` call since our `pub_date_day` has already been filtered to provide the `datetime` information I require:

```python
>>> pub_date_day.get()
datetime.datetime(2020, 6, 23, 0, 0, tzinfo=<DstTzInfo 'GB' BST+1:00:00 DST>)
```

So now I have a `datetime` object I can work with and can begin evaulating against the rest of my data:

```python
>>> type(pub_date_day)
<class 'django.db.models.query.QuerySet'>
>>> type(pub_date_day.get())
<class 'datetime.datetime'>
```

<br/>

### Evaluating our data

So we've established that `pub_date_day.get()` is the date of which our question was published `2020, 6, 23`:

```python
>>> pub_date_day.get()
datetime.datetime(2020, 6, 23, 0, 0, tzinfo=<DstTzInfo 'GB' BST+1:00:00 DST>)
```

And `now` being the present day `2020, 7, 1`:

```python
>>> now
datetime.datetime(2020, 7, 1, 9, 4, 25, 119278, tzinfo=<UTC>)
```

I can now do my evaluation successfully:

```python
>>> now - datetime.timedelta(days=1) <= pub_date_day.get() <= now
False
```

#### Breaking down our evaluation

First I subtract `datetime.timedelta(days=1)` from the current time `now`:

```python
>>> now - datetime.timedelta(days=1)
datetime.datetime(2020, 6, 30, 9, 4, 25, 119278, tzinfo=<UTC>)
```

From the output we can see that a date of a day later (past) is returned.

Then we evaluate our returned `now` value with `pub_date_day.get()` using the less than or equal to operator `<=`:

```python
>>> now - datetime.timedelta(days=1) <= pub_date_day.get()
False
```

So `pub_date_day.get()` has the date `2020, 6, 23`. Our `now` value since, the subtraction looks like `2020, 6, 30` which is greater than `2020, 6, 23`. Therefore this evaluates to `False` as it was **NOT published recently**.

Remember that `now` is **present** day. So whilst we subtract `days=1` from `now`, our **published** date could never exceed `now` by more than one day, which is why `<=` is a valid operator for this case.

Lastly, we would evaluate `pub_date_day.get()` to `now` to confirm that the **published** date is less than or equal to the **present day**, in which case we already determined that our answer was `False` in the previous step so we know this is going to be `False`.

```python
>>> now - datetime.timedelta(days=1) <= pub_date_day.get() <= now
False
```

Easy!

<br/>

----

### Documentation 

Some reading on these topics, worth checking out:
* https://docs.djangoproject.com/en/3.0/topics/i18n/timezones/
* https://www.guru99.com/date-time-and-datetime-classes-in-python.html
* https://docs.python.org/3/library/datetime.html
* https://docs.djangoproject.com/en/3.0/ref/models/fields/
