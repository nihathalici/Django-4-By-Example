05 / Reset password views
========================================================


* Update the urls.py within the account folder
```python
# bookmarks/account/urls.py

from django.urls import path
from django.contrib.auth import views as auth_views
from . import views

urlpatterns = [
    # login / logout
    path("login/", auth_views.LoginView.as_view(), name="login"),
    path("logout/", auth_views.LogoutView.as_view(), name="logout"),
    # change password
    path("password-change/", 
         auth_views.PasswordChangeView.as_view(),
         name="password_change"),
    path("password-change/done/", 
         auth_views.PasswordChangeDoneView.as_view(),
         name="password_change_done"),
    # reset password
    path("password-reset/",  # new 
         auth_views.PasswordResetView.as_view(),
         name="password_reset"),
    path("password-reset/done/",   # new
         auth_views.PasswordResetDoneView.as_view(),
         name="password_reset_done"),
    path("password-reset/<uid64>/<token>",  # new 
         auth_views.PasswordResetConfirmView.as_view(),
         name="password_reset_confirm"),
    path("password-reset/complete/",   # new
         auth_views.PasswordResetCompleteView.as_view(),
         name="password_reset_complete"),
    path("", views.dashboard, name="dashboard"),
]
```

* Create the password_reset_form.html file within the templates/registration folder
```html
<!-- bookmarks/account/templates/registration/password_reset_form.html -->

{% extends 'base.html' %}

{% block title %}Reset your password{% endblock title %}

{% block content %}
  <h1>Forgotten your password?</h1>
  <p>Enter your e-mail address to obtain a new password.</p>
  <form action="" method="post">
    {{ form.as_p }}
    <p><input type="submit" value="Send e-mail"></p>
    {% csrf_token %}
  </form>
{% endblock content %}
```

* Create the password_reset_email.html file within the templates/registration folder
```html
<!-- bookmarks/account/templates/registration/password_reset_email.html -->

Someone asked for password reset for email {{ email }}. Follow the link below:
{{ protocol }}://{{ domain }}{% url "password_reset_confirm" uidb64=uid token=token %}
Your username, in case you've forgotten: {{ user.get_username }}
```

* Create the password_reset_done.html file within the templates/registration folder
```html
<!-- bookmarks/account/templates/registration/password_reset_done.html -->

{% extends 'base.html' %}

{% block title %}Reset your password{% endblock title %}

{% block content %}
  <h1>Reset your password</h1>
  <p>We've emailed you instructions for setting your password.</p>
  <p>If you don't receive an email, please make sure you've entered the address
    you registered with.</p>
  
{% endblock content %}
        
```

* Create the password_reset_confirm.html file within the templates/registration folder
```html
<!-- bookmarks/account/templates/registration/password_reset_form.html -->

{% extends 'base.html' %}

{% block title %}Reset your password{% endblock title %}

{% block content %}
  <h1>Reset your password</h1>
  
  {% if validlink %}
  <p>Please enter your new password twice:</p>
  <form action="" method="post">
    {{ form.as_p }}
    {% csrf_token %}
    <p><input type="submit" value="Change my password"></p>
  </form>
  {% endif %}
{% endblock content %}
        
```

* Create the password_reset_complete.html file within the templates/registration folder
```html
<!-- bookmarks/account/templates/registration/password_reset_complete.html -->

{% extends 'base.html' %}

{% block title %}Password reset{% endblock title %}

{% block content %}
  <h1>Password set</h1>
  <p>We've emailed you instructions for setting your password.</p>
  <p>Your password has been set. You can <a href="{% url 'login' %}">log in now</a></p>
  
{% endblock content %}
        
```

* Update the login.html
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
    <p>
      <a href="{% url 'password_reset' %}">
        Forgotten your password?        
      </a>
    </p>
</div>
{% endblock content %}  
```

* Edit the settings and add this new line

```python
# bookmarks/bookmarks/settings.py

EMAIL_BACKEND = "django.core.mail.backends.console.EmailBackend"
```

* Check http://127.0.0.1:8000/account/login/ 