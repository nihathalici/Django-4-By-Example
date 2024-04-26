01 / Implementing Social Authentication
========================================================

* Install Python Social Auth

```shell
pip install social-auth-app-django
```

* Edit settings, and add the Social Auth to the Installed Apps
```python

# bookmarks/bookmarks/settings.py

INSTALLED_APPS = [
    "account.apps.AccountConfig",
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    "social_django",  # new
]

```


* Sync the database
```shell
python manage.py migrate
```

* Update the urls.py file at project level
```python

# bookmarks/bookmarks/urls.py

from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path("account/", include("account.urls")),
    path("social-auth/",
         include("social_django.urls", namespace="social")),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL,
                          document_root=settings.MEDIA_ROOT)

```

* Open a Terminal window (for MacOS and Linux user)
```shell
sudo nano /etc/hosts
```

* Add a new line. Save it with Ctrl+O, and exit Ctrl+X   
```shell
127.0.0.1 mysite.com
```

* Check  http://mysite.com:8000/account/login/

* Update the settings
```python
# bookmarks/bookmarks/settings.py

ALLOWED_HOSTS = ["mysite.com", "localhost", "127.0.0.1"]
```

* Run the local server. Check http://mysite.com:8000/account/login/  again. The error should be gone.


Running the development server through HTTPS
========================================================

* Install Django Extensions
```shell
pip install django-extensions
```

* Install Werkzeug
```shell
pip install werkzeug==2.2.2
```

* Install pyOpenSSL
```shell
pip install pyOpenSSL==22.0.0
```

* Edit settings, and add the Social Auth to the Installed Apps
```python

# bookmarks/bookmarks/settings.py

INSTALLED_APPS = [
    "account.apps.AccountConfig",
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    "social_django",
    "django_extensions", # new
]

```

* Run local server with the runserver_plus command
```shell
python manage.py runserver_plus --cert-file cert.crt
```


Authentication using Facebook
========================================================

* Update the settings
```python
AUTHENTICATION_BACKENDS = [
    "django.contrib.auth.backends.ModelBackend",
    "account.authentication.EmailAuthBackend",
    "social_core.backends.facebook.FacebookOAuth2"  # new
]
```

* Go to https://developers.facebook.com/apps/ . Create a new Facebook app.
