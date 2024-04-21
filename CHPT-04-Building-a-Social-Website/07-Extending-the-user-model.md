07 / Extending the user model
========================================================


* Update the models.py within the account folder
```python
# bookmarks/account/models.py

from django.db import models
from django.conf import settings

class Profile(models.Model):
    user = models.OneToOneField(settings.AUTH_USER_MODEL,
                                on_delete=models.CASCADE)
    date_of_birth = models.DateField(blank=True, null=True)
    photo = models.ImageField(upload_to='users/%Y/%m/%d/',
                              blank=True)
    def __str__(self):
        return f'Profile of {self.user.username}'

```

07 / Installing Pillow and serving media files
========================================================

* Install pillow
```shell
pip install Pillow==9.2.0
```

* Update the settings
```python

# bookmarks/bookmarks/settings.py

MEDIA_URL = "media/"
MEDIA_ROOT = BASE_DIR / "media"

```

* Update the urls.py file at the project level
```python

# bookmarks/bookmarks/urls.py

from django.contrib import admin
from django.urls import path, include
from django.conf import settings  # new
from django.conf.urls.static import static  # new

urlpatterns = [
    path('admin/', admin.site.urls),
    path("account/", include("account.urls")),
]

if settings.DEBUG:  # new
    urlpatterns += static(settings.MEDIA_URL,
                          document_root=settings.MEDIA_ROOT)

```


07 / Creating migrations for the profile model
========================================================

* Sync the database

```shell
python manage.py makemigrations
python manage.py migrate
```

* Update the admin.py file within the account folder, and register the Profile model
```python
from django.contrib import admin
from .models import Profile

@admin.register(Profile)
class ProfileAdmin(admin.ModelAdmin):
    list_display = ["user", "date_of_birth", "photo"]
    raw_id_fields = ["user"]
```

* Run the local server. Check http://127.0.0.1:8000/admin/


* Update the forms.py file within the account folder
```python

# bookmarks/account/forms.py

from django import forms
from django.contrib.auth.models import User
from .models import Profile  # new

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

class UserEditForm(forms.ModelForm):  # new
    class Meta:
        model = User
        fields = ["first_name", "last_name", "email"]

class ProfileEditForm(forms.ModelForm):  # new
    class Meta:
        model = Profile
        fields = ["date_of_birth", "photo"]
```

* Update the views.py within the account folder
```python
# bookmarks/account/views.py

from django.http import HttpResponse
from django.shortcuts import render
from django.contrib.auth import authenticate, login
from django.contrib.auth.decorators import login_required
from .forms import LoginForm, UserRegistrationForm, UserEditForm, ProfileEditForm  # new
from .models import Profile  # new

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


def register(request):
    if request.method == 'POST':
        user_form = UserRegistrationForm(request.POST)
        if user_form.is_valid():
            new_user = user_form.save(commit=False)
            new_user.set_password(user_form.cleaned_data["password"])
            new_user.save()
            Profile.objects.create(user=new_user)  # new  
            return render(request,
                          "account/register_done.html",
                          {"new_user": new_user})
    else:
        user_form = UserRegistrationForm()
    return render(request,
                  "account/register.html",
                  {"user_form": user_form})

@login_required
def edit(request):  # new
    if request.method == "POST":
        user_form = UserEditForm(instance=request.user,
                                 data=request.POST)
        profile_form = ProfileEditForm(
            instance=request.user.profile,
            data=request.POST,
            files=request.FILES)
        if user_form.is_valid() and profile_form.is_valid():
            user_form.save()
            profile_form.save()
    else:
        user_form = UserEditForm(instance=request.user)
        profile_form = ProfileEditForm(instance=request.user.profile)
    return render(request,
                  "account/edit.html",
                  {"user_form": user_form,
                   "profile_form": profile_form})  

```

* Update the urls.py file within the account folder
```python

# bookmarks/account/urls.py

from django.urls import path, include
from django.contrib.auth import views as auth_views
from . import views

urlpatterns = [    


    path("", include("django.contrib.auth.urls")),
    path("", views.dashboard, name="dashboard"),
    path("register/", views.register, name="register"),
    path("edit/", views.edit, name="edit"),  # new
]

```

* Create the edit.html file within the templates/account folder
```html
<!-- bookmarks/account/templates/account/edit.html -->
{% extends 'base.html' %}

{% block title %}Edit your account{% endblock title %}

{% block content %}
  <h1>Edit your account</h1>
  <p>You can edit your account using the following form:</p>
  <form method="post" enctype="multipart/form-data">
    {{ user_form.as_p }}
    {{ profile_form.as_p }}
    {% csrf_token %}
    <p> <input type="submit" value="Save changes"></p>
  </form>    
{% endblock content %}    

```

* Go to http://127.0.0.1:8000/account/register/ . Register a new user.

* Go to http://127.0.0.1:8000/account/edit/ , and edit the profile details.

* Update the dashboard.html file within the templates/account folder
```python
<!-- bookmarks/account/templates/account/dashboard.html -->

{% extends 'base.html' %}

{% block title %}Dashboard{% endblock title %}

{% block content %}
  <h1>Dashboard</h1>
  <p>Welcome to your dashboard. You can <a href="{% url 'edit' %}">edit your profile</a> or <a href="{% url 'password_change' %}">change your password</a>.
  </p>    
{% endblock content %}

```

* Check http://127.0.0.1:8000/account/


Using the messages framework
========================================================

* Update the base.html within the templates folder

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

<!-- New -->
    {% if messages %}
      <ul class="messages">
        
        {% for message in messages %}
          <li class="{{ message.tags }}">
            {{ message|safe }}
            <a href="#" class="close">x</a>
          </li>          
        {% endfor %}          
      </ul>  
    {% endif %}

<!-- New - End -->

    <div id="content">
      {% block content %}
      {% endblock %}        
    </div>  
  </body>    
</html>

```

* Update the views.py file within the account folder

```python
# bookmarks/account/views.py

from django.http import HttpResponse
from django.shortcuts import render
from django.contrib.auth import authenticate, login
from django.contrib.auth.decorators import login_required
from django.contrib import messages  # new

from .forms import LoginForm, UserRegistrationForm, UserEditForm, ProfileEditForm
from .models import Profile

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


def register(request):
    if request.method == 'POST':
        user_form = UserRegistrationForm(request.POST)
        if user_form.is_valid():
            new_user = user_form.save(commit=False)
            new_user.set_password(user_form.cleaned_data["password"])
            new_user.save()
            Profile.objects.create(user=new_user)  
            return render(request,
                          "account/register_done.html",
                          {"new_user": new_user})
    else:
        user_form = UserRegistrationForm()
    return render(request,
                  "account/register.html",
                  {"user_form": user_form})

@login_required
def edit(request):  # new
    if request.method == "POST":
        user_form = UserEditForm(instance=request.user,
                                 data=request.POST)
        profile_form = ProfileEditForm(
            instance=request.user.profile,
            data=request.POST,
            files=request.FILES)
        if user_form.is_valid() and profile_form.is_valid():
            user_form.save()
            profile_form.save()
            messages.success(request, "Profile updated successfully")  # new
        else:
            messages.error(request, "Error updating your profile")  # end of new lines
    else:
        user_form = UserEditForm(instance=request.user)
        profile_form = ProfileEditForm(instance=request.user.profile)
    return render(request,
                  "account/edit.html",
                  {"user_form": user_form,
                   "profile_form": profile_form})  

```

* Check http://127.0.0.1:8000/account/edit/


Building a custom authentication backend
========================================================

* Create the authentication.py file within the account folder

```python
# bookmarks/account/authentication.py

from django.contrib.auth.models import User

class EmailAuthBackend:
    """
    Authenticate using an e-mail address.
    """
    def authenticate(self, request, username=None, password=None):
        try:
            user = User.objects.get(email=username)
            if user.check_password(password):
                return user
            return None
        except (User.DoesNotExist, User.MultipleObjectsReturned):
            return None
    
    def get_user(self, user_id):
        try:
            return User.objects.get(pk=user_id)
        except User.DoesNotExist:
            return None

```

* Update the settings, and the foolowing lines

```python

AUTHENTICATION_BACKENDS = [
    "django.contrib.auth.backends.ModelBackend",
    "account.authentication.EmailAuthBackend",
]

```

* Check http://127.0.0.1:8000/account/login/


Preventing users from using an existing email
========================================================

* Update the forms.py file within the account folder

```python

# bookmarks/account/forms.py

from django import forms
from django.contrib.auth.models import User
from .models import Profile

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
    
    def clean_email(self):  # new
        data = self.cleaned_data["email"]
        if User.objects.filter(email=data).exists():
            raise forms.ValidationError("Email already in use.")
        return data


class UserEditForm(forms.ModelForm):
    class Meta:
        model = User
        fields = ["first_name", "last_name", "email"]
    
    def clean_email(self):  # new
        data = self.cleaned_data["email"]
        qs = User.objects.exclude(id=self.instance.id).filter(email=data)
        if qs.exists():
            raise forms.ValidationError(" Email already in use.")
        return data


class ProfileEditForm(forms.ModelForm):
    class Meta:
        model = Profile
        fields = ["date_of_birth", "photo"]

```


