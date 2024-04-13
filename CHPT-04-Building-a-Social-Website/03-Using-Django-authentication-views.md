03 / Using the Django authentication views
========================================================

* Update the urls.py within the account folder
```python
# bookmarks/account/urls.py

from django.urls import path
from django.contrib.auth import views as auth_views  # new
from . import views

urlpatterns = [
    path("login/", auth_views.LoginView.as_view(), name="login"),   # updated 
    path("logout/", auth_views.LogoutView.as_view(), name="logout"),  # new
]
```

* Create the registration folder within the account/templates directory 
* Create the login.html file within the templates/registration folder

```html
<!-- bookmarks/account/templates/registration/login.html -->
{% extends 'base.html' %}

{% block title %}Log-in{% endblock title %}

{% block content %}
<h1>Log-in</h1>

{% if form.errors %}
  <p>
    Your username and password didn't match.
    Please try again.  
  </p>
{% else %}
  <p>Please, use the following form to log-in:</p>    
{% endif %}
<div class="login-form">
    <form action="{% url 'login' %}" method="post">
        {{ form.as_p }}
        {% csrf_token %}
        <input type="hidden" name="next" value="{{ next }}" />
        <p><input type="submit" value="Log-in"></p>
    </form>
</div>
{% endblock content %}     
```

* Create the logged_out.html file within the templates/registration folder
```html
<!-- bookmarks/account/templates/registration/logged_out.html -->
{% extends 'base.html' %}

{% block title %}Logged out{% endblock title %}

{% block content %}
  <h1>Logged out</h1>
  <p>
    You have been successfully logged out.
    You can <a href="{% url 'login' %}">log-in again</a>.
  </p>
{% endblock content %}
```

* Update the views.py file within the account folder
```python

# bookmarks/account/views.py

from django.http import HttpResponse
from django.shortcuts import render
from django.contrib.auth import authenticate, login
from django.contrib.auth.decorators import login_required  # new
from .forms import LoginForm

@login_required  
def dashboard(request):  # new
    return render(request,
                  "account/dashboard.html",
                  {"section": "dashboard"})

```

* Create the dashboard.html file within the templates/account folder
```html
<!-- bookmarks/account/templates/account/dashboard.html -->

{% extends 'base.html' %}

{% block title %}Dashboard{% endblock title %}

{% block content %}
  <h1>Dashboard</h1>
  <p>Welcome to your dashboard.</p>    
{% endblock content %}
``` 

* Update the urls.py within the account folder
```python
# bookmarks/account/urls.py

from django.urls import path
from django.contrib.auth import views as auth_views
from . import views

urlpatterns = [
    path("login/", auth_views.LoginView.as_view(), name="login"),
    path("logout/", auth_views.LogoutView.as_view(), name="logout"),
    path("", views.dashboard, name="dashboard"),  # new
]
```

* Update the settings
```python
# bookmarks/bookmarks/settings.py

LOGIN_REDIRECT_URL = "dashboard"
LOGIN_URL = "login"
LOGOUT_URL = "logout"
```

* Update base.html
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
        {% if request.user.is_authenticated %}
          <ul class="menu">
            <li {% if section == "dashboard" %}class="selected"{% endif %}>
              <a href="{% url 'dashboard' %}">My dashboard</a>
            </li>
            <li {% if section == "images" %}class="selected"{% endif %}>
              <a href="#">Images</a>
            </li>
            <li {% if section == "people" %}class="selected" {% endif %}>
              <a href="#">People</a>
          </li>
          </ul>   
        {% endif %}
        <span class="user">
          
          {% if request.user.is_authenticated %}
            Hello {{ request.user.first_name|default:request.user.username }},
            <a href="{% url 'logout' %}">Logout</a>
          {% else %}
            <a href="{% url 'login' %}">Log-in</a>
          {% endif %}    
        </span>     
    </div>
    <div id="content">
      {% block content %}
      {% endblock %}        
    </div>  
  </body>    
</html>
```

* Check http://127.0.0.1:8000/account/login/
