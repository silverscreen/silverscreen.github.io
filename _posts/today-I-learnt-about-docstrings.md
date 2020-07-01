"""
high level support for doing this and that.
"""

from django.http import HttpResponse # imports things
from .models import Question

# Create your views here.

# Our OLD index view
#def index(request):
#    return HttpResponse("Hello, world. You're at the polls index.")
def index(request):
    """
    high level support for doing this and that.
    """
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


    https://www.python.org/dev/peps/pep-0257/
    https://stackoverflow.com/questions/7877522/how-do-i-disable-missing-docstring-warnings-at-a-file-level-in-pylint
    https://www.datacamp.com/community/tutorials/docstrings-python



To disable the annoying 'docstrings' pylint error, add the following `"--disable=C0111"` to 

Preferences>language settings>Python>

```
{
    "python.linting.pylintArgs": ["--load-plugins", "pylint_django", "--disable=C0111"],
    "workbench.colorTheme": "Solarized Light",
    "workbench.iconTheme": "vscode-icons",
    "editor.renderControlCharacters": false,
    "editor.detectIndentation": false,
    "window.zoomLevel": 0,
    "editor.minimap.enabled": false,
    "[python]": {
    }
}
```

It's useful, though `"python.linting.pylintArgs": ["--disable=C0111"],` is probably moreso as it just quiets docstring warnings. However setting addresses the OP's question of how to disable these warnings only at a module level. 

https://code.visualstudio.com/docs/python/linting