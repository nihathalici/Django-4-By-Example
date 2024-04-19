06 / User registration and user profiles
========================================================


* Update the forms.py within the account folder
```python
# bookmarks/account/forms.py

from django import forms
from django.contrib.auth.models import User

class LoginForm(forms.Form):
    username = forms.CharField()
    password = forms.CharField(widget=forms.PasswordInput)

class UserRegistrationForm(forms.ModelForm):
    password = forms.CharField(label="Password", widget=forms.PasswordInput)
    password2 = forms.CharField(label="Repeat password", widget=forms.PasswordInput)

    class Meta:
        model = User
        fields = ["username", "first_name", "email"]
    
    def clean_password2(self):
        cd = self.cleaned_data
        if cd["password"] != cd["password2"]:
            raise forms.ValidationError('Passwords don\'t match.')
        return cd["password2"]

```

* Update the views.py within the account folder
```python
# bookmarks/account/views.py

from django.http import HttpResponse
from django.shortcuts import render
from django.contrib.auth import authenticate, login
from django.contrib.auth.decorators import login_required
from .forms import LoginForm, UserRegistrationForm  # new

@login_required
def dashboard(request):
    return render(request,
                  "account/dashboard.html",
                  {"section": "dashboard"})

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


def register(request):  # new
    if request.method == 'POST':
        user_form = UserRegistrationForm(request.POST)
        if user_form.is_valid():
            new_user = user_form.save(commit=False)
            new_user.set_password(user_form.cleaned_data["password"])
            new_user.save()
            return render(request,
                          "account/register_done.html",
                          {"new_user": new_user})
    else:
        user_form = UserRegistrationForm()
    return render(request,
                  "account/register.html",
                  {"user_form": user_form})

```

* Update the urls.py file within the account folder

```python

# bookmarks/account/urls.py

from django.urls import path, include
from django.contrib.auth import views as auth_views
from . import views

urlpatterns = [    
    ...

    path("", include("django.contrib.auth.urls")),
    path("", views.dashboard, name="dashboard"),
    path("register/", views.register, name="register"), # new

]

```

* Create the register.html file within the templates/account folder
```html
<!-- bookmarks/account/templates/account/register.html -->
{% extends 'base.html' %}

{% block title %}Create an account{% endblock title %}

{% block content %}
  <h1>Create an account</h1>
  <p>Please, sign up using the following form:</p>
  <form method="post" action="">
    {{ user_form.as_p }}
    {% csrf_token %}
    <p> <input type="submit" value="Create my account"></p>
  </form>    
{% endblock content %}    
```

* Create the register_done.html file within the templates/account folder
```html
<!-- bookmarks/account/templates/account/register_done.html -->

{% extends 'base.html' %}

{% block title %}Welcome{% endblock title %}

{% block content %}
  <h1>Welcome {{ new_user.first_name }}!</h1>
  <p>
    Your account has been successfully created.
    Now you can <a href="{% url 'login' %}">log in</a>.
  </p>    
{% endblock content %}

```

* Check http://127.0.0.1:8000/account/register/ 

* Update the login.html within templates/registration folder
```html
<!-- bookmarks/account/templates/registration/login.html -->

...

  <p>Please, use the following form to log-in:
<!-- New -->If you don't have an account <a href="{% url 'register' %}">register here</a>.<!-- New --> 
  </p>

...

```

