01 / Set-Up
========================================================

* Make a new directory

```shell
mkdir Chapter04
```

* Create a virtual environment for your project:

```shell
mkdir env
python -m venv env/bookmarks
```

* Activate the virtual environment
```shell
source env/bookmarks/bin/activate
```

* Install Django
```shell
pip install Django~=4.1.0
```

* Create a new Django project called bookmarks 
```shell
django-admin startproject bookmarks
```

* Get into your project directory
```shell
cd bookmarks/
```

* Create a new app named account
```shell
python manage.py startapp account
```

* Add the the new app to the project
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
]

```

* Sync the database
```shell
python manage.py migrate
```


* Run the Django development server
```shell
python manage.py runserver
```

* Quit the server with CONTROL-C.
