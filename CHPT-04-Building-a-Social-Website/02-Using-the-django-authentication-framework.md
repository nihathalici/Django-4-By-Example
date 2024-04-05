02 / Using the Django authentication framework
========================================================

* Create the forms.py file within the account folder

```python

# bookmarks/account/forms.py

from django import forms

class LoginForm(forms.Form):
    username = forms.CharField()
    password = forms.CharField(widget=forms.PasswordInput)

```

* Update the views
```python

# bookmarks/account/views.py

from django.http import HttpResponse
from django.shortcuts import render
from django.contrib.auth import authenticate, login
from .forms import LoginForm

def user_login(request):
    if request.method == 'POST':
        form = LoginForm(request.POST)
        if form.is_valid():
            cd = form.cleaned_data
            user = authenticate(request, 
                                username=cd["username"],
                                password=cd["password"])
            if user is not None:
                if user.is_active:
                    login(request, user)
                    return HttpResponse("Authenticated successfully")
                else:
                    return HttpResponse("Disabled account")
            else:
                return HttpResponse("Invalid login")
    else:
        form = LoginForm()
    return render(request, "account/login.html", {"form": form})

```

* Create urls.py within the account folder
```python
# bookmarks/account/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path("login/", views.user_login, name="login"),
]
```

* Update the urls.py file at project level
```python

# bookmarks/bookmarks/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path("account/", include("account.urls")),
]

```

* Create the templates folder within the account directory. 

* Create the base.html file within the templates folder.
```html
<!-- bookmarks/account/templates/base.html -->

{% load static %}
<!DOCTYPE html>
<html>
  <head>
    <title>{% block title %}{% endblock title %}</title>
    <link rel="stylesheet" href="{% static 'css/base.css' %}">
  </head>
  <body>
    <div id="header">
        <span class="logo">Bookmarks</span>
    </div>
    <div id="content">
      {% block content %}
      {% endblock %}        
    </div>  
  </body>    
</html>
```

* Create the account folder within the account/templates directory.

* Create the login.html file templates/account folder. 
```html
<!-- bookmarks/account/templates/account/login.html -->
{% extends 'base.html' %}

{% block title %}Log-in{% endblock title %}

{% block content %}
  <h1>Log-in</h1>
  <p>Please, use the following form to log-in:</p>
  <form method="post" action="">
    {{ form.as_p }}
    {% csrf_token %}
    <p> <input type="submit" value="Log in"></p>
  </form>    
{% endblock content %} 

```

* Create the superuser
```bash
python manage.py createsuperuser
```

* Run the development server
```bash
python manage.py runserver
```

* Go to http://127.0.0.1:8000/admin/ Create a test account

* Go to http://127.0.0.1:8000/account/login/ . Try with valid and invalid login data. Check the responses 
