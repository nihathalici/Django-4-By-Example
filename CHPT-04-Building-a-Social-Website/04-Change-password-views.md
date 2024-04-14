04 / Change password views
========================================================


* Update the urls.py within the account folder
```python
# bookmarks/account/urls.py

from django.urls import path
from django.contrib.auth import views as auth_views  # new
from . import views

urlpatterns = [
    path("login/", auth_views.LoginView.as_view(), name="login"),
    path("logout/", auth_views.LogoutView.as_view(), name="logout"),
    path("password-change/",   # new
         auth_views.PasswordChangeView.as_view(),
         name="password_change"),
    path("password-change/done/",   # new
         auth_views.PasswordChangeDoneView.as_view(),
         name="password_change_done"),
    path("", views.dashboard, name="dashboard"),
]
```

* Create the password_change_form.html within the templates/registration folder
```html
<!-- bookmarks/account/templates/registration/password_change_form.html -->

{% extends 'base.html' %}

{% block title %}Changeyour password{% endblock title %}

{% block content %}
  <h1>Change your password</h1>
  <p>Use the form below to change your password.</p>
  <form action="" method="post">
    {{ form.as_p }}
    <p><input type="submit" value="Change"></p>
    {% csrf_token %}
  </form>
{% endblock content %}

```

* Create the password_change_done.html within the templates/registration folder
```html
<!-- bookmarks/account/templates/registration/password_change_done.html -->

{% extends 'base.html' %}

{% block title %}Password changed{% endblock title %}

{% block content %}
  <h1>Password changed</h1>
  <p>Your password has been successfully changed.</p>
{% endblock content %}
        
```

* Check http://127.0.0.1:8000/account/password-change/

