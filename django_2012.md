# How to start a Django project in 2012
Develop on Ubuntu; Source on Github; Deploy to Heroku

-----

## Install system packages

You may already have (some of) these installed.  It is best not to install Django system-wide, to avoid confusion with the Django instance that will be installed inside your virtual environment below.

``` bash
$ sudo apt-get install python python-pip python-virutalenv git
# Install Heroku via PPA
$ wget -qO- https://toolbelt.heroku.com/install.sh | sh
```

## Create a new repository on Github

Create a new repo [the usual way](https://help.github.com/articles/creating-a-new-repository).  Check the "Initialize this repository with a README" box and choose a "Django" .gitignore file from the list.

Clone the repo to your local computer.

``` bash
$ git clone git@github.com:jmcvetta/myproject.git
```

## Create the project 

Setup and activate virtualenv.

``` bash
$ cd myproject
$ virtualenv --no-site-packages venv
$ source venv/bin/activate
```

(`venv` for virtualenv. That is the conventional name, but you can choose whatever you like.)

Install some things into your virtualenv.

``` bash
$ pip install Django psycopg2 south pillow dj_database_url ipython \
gunicorn gevent django-bcrypt
```
* [psycopg2](https://initd.org/psycopg/) is a PostgreSQL adapter for Python.
* [South][south] handles database migrations.
* [Pillow](https://pypi.python.org/pypi/Pillow) is a virtualenv-safe
  replacement for PIL.
* [dj_database_url](https://github.com/kennethreitz/dj-database-url) makes DB
  settings for Heroku easy.
* [IPython](https://ipython.org) provides an enhanced Python shell for your
  development pleasure.
* [Gunicorn](https://gunicorn.org) 'Green Unicorn' is a Python WSGI HTTP Server
  for UNIX.
* [django-bcrypt](https://github.com/dwaiter/django-bcrypt) provides more
  secure password encryption using bcrypt.


Now save the exact version into a `requirements.txt` file.

``` bash
$ pip freeze > requirements.txt
```

Start a Django project

```
$ django-admin.py startproject myproject .
```


Add `south` and `gunicorn` to your `INSTALLED_APPS` in `settings.py`.

``` python
INSTALLED_APPS = (
    ...
    'south',
    'gunicorn',
    'django_bcrypt',
)
```

Configure settings to use Heroku's Postgres database when deployed, and SQLite locally.

``` python
import dj_database_url
import os
DATABASES = {'default': dj_database_url.config(default='sqlite:///' +
        os.path.join(os.path.dirname(os.path.abspath(__file__)), '..', 'db.sqlite') )}
```

Create a `Procfile` to to launch the web process using Gunicorn.

``` bash
# Note the single quote marks so $PORT gets echoed literally
$ echo 'web: gunicorn myproject.wsgi -b 0.0.0.0:$PORT' > Procfile
```

Go make something awesome!

## Deploy to Heroku

``` bash
$ heroku create
$ git push heroku master
$ heroku run ./manage.py syncdb
$ heroku open
```

## Update your database with [South][south] and push the changes to Heroku

1. Make changes to your models
2. `$ python ./manage.py schemamigration [appname] --auto`
3. `$ python ./manage.py migrate [appname]`
4. [commit & push changes to heroku]
5. `$ heroku run ./manage.py migrate [appname]`

## Working on your project later

Whenever you work on your project, you'll want to activate your virtualenv:

``` bash
$ source venv/bin/activate
```

Install new packages with Pip, then update `requirements.txt` file.

``` bash
$ pip install django-nose
$ pip freeze > requirements.txt
$ git commit requirements.txt -m "Added django-nose to requirements.txt"
```

If `requirements.txt` was updated elsewhere, you can update your virtualenv:

``` bash
$ pip install -r requirements.txt
```

Sync and/or migrate your database:

``` bash
$ python ./manage.py syncdb
$ python ./manage.py migrate [appname]
```

Finally, fire up your server:

``` bash
$ python ./manage.py runserver
```

### Sources

- [Deploying Django applications on
  Heroku](http://offbytwo.com/2012/01/18/deploying-django-to-heroku.html)
- [Part 1: The Basics &mdash; South v0.7
  documentation](http://south.aeracode.org/docs/tutorial/part1.html)
- [Getting Started with Django on Heroku/Cedar | Heroku Dev
  Center](https://devcenter.heroku.com/articles/django)

[south]: http://south.aeracode.org/
