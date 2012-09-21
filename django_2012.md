# How to start a Django project in 2012
Develop on Ubuntu; Source on Github; Deploy to Heroku

-----

## Install system packages

You may already have (some of) these installed.  It is best not to install Django system-wide, to avoid confusion with the Django instance that will be installed inside your virtual environment below.

``` bash
$ sudo apt-get install python python-virutalenv git
# Install Heroku (uses PPA)
$ wget -qO- https://toolbelt.heroku.com/install.sh | sh
```

## Create a new repository on Github

Create a new repo [the usual
way](https://help.github.com/articles/creating-a-new-repository).  Check the
"Initialize this repository with a README" box and choose a "Django" .gitignore
file from the list.

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

Declare your dependencies in a `requirements.base.txt` file:

```
django           # Web application framework
psycopg2         # PostgreSQL driver
south            # Database migration service
ipython          # Interactive Python shell
dj_database_url  # Get database URL from environment variables
pillow           # PIL-compatible imaging library in pure Python
gunicorn         # Production webserver
gevent           # High speed event handling library
py-bcrypt        # More secure password hashing
newrelic         # Cloud-based monitoring service
```

Install from requirements using `pip`:

``` bash
$ pip install -r requirements.base.txt
```

Now save the exact version into a `requirements.txt` file.  This file will be
used by Heroku to provision your app.  By using the exact version numbers
provided by `pip freeze`, we can guarantee dependencies in production will always be
the exact same version used in development.

``` bash
$ pip freeze > requirements.txt
$ head requirements.txt
Django==1.4.1
Pillow==1.7.7
South==0.7.6
argparse==1.2.1
boto==2.5.2
distribute==0.6.24
dj-database-url==0.2.1
gevent==0.13.8
greenlet==0.4.0
```

Start a Django project

```
$ django-admin.py startproject myproject .
```


Add `south` to your `INSTALLED_APPS` in `settings.py`.

``` python
INSTALLED_APPS += (
    'south',
)
```

Enable BCrypt as the preferred password hash.

``` python
from django.conf import global_settings

PASSWORD_HASHERS = (
    'django.contrib.auth.hashers.BCryptPasswordHasher',
) + global_settings.PASSWORD_HASHERS
```


Configure `settings.py` to use Heroku's Postgres database when deployed, and
SQLite locally.  Note that while this is convenient, it is [contrary to
12-Factor app design principles](http://www.12factor.net/dev-prod-parity).

``` python
import dj_database_url
import os
DATABASES = {'default': dj_database_url.config(default='sqlite:///' +
        os.path.join(os.path.dirname(os.path.abspath(__file__)), '..', 'db.sqlite') )}
```


Create a `Procfile` to to launch the web process using Gunicorn with Gevent-based workers.

``` bash
# Note the single quote marks so $PORT gets echoed literally
$ echo 'web: newrelic-admin run-program gunicorn myproject.wsgi -b 0.0.0.0:$PORT -k gevent ' > Procfile
```

Use [`foreman`](https://devcenter.heroku.com/articles/config-vars#local-setup)
to run your app locally in a production-like environment.  Foreman reads
environment variables from an `.env` file and sets them when starting the app.


Go make something awesome!


## Deploy to Heroku

``` bash
$ heroku create
$ heroku addons:add newrelic:standard
$ git push heroku master
$ heroku run ./manage.py syncdb
$ heroku open
```

Your app should now be running.  You can see all sorts of spiffy charts about
its performance etc from New Relic from your Heroku dashboard.

## Update your database with [South][south] and push the changes to Heroku

1. Make changes to your models.
2. Create a migration: `$ python ./manage.py schemamigration [appname] --auto`
3. Migrate your local DB: `$ python ./manage.py migrate [appname]`
4. [commit & push changes to heroku]
5. Migrate Heroku DB: `$ heroku run ./manage.py migrate [appname]`


## Working on your project later

Whenever you work on your project, you'll want to activate your virtualenv:

``` bash
$ source venv/bin/activate
```

Install new packages by adding them to `requirements.base.txt`, installing them, and when they have been 
confirmed to work, saving them to `requirements.txt` with `pip freeze`.

``` bash
$ vim requirements.base.txt
$ pip install -r requirements.base.txt
$ pip freeze > requirements.txt
$ git commit requirements.* -m "Added some_package to requirements.txt"
```

If `requirements.txt` was updated elsewhere, you can update your virtualenv:

``` bash
$ pip install -r requirements.txt
```

### Sources

- [Deploying Django applications on
  Heroku](http://offbytwo.com/2012/01/18/deploying-django-to-heroku.html)
- [Part 1: The Basics &mdash; South v0.7
  documentation](http://south.aeracode.org/docs/tutorial/part1.html)
- [Getting Started with Django on Heroku/Cedar | Heroku Dev
  Center](https://devcenter.heroku.com/articles/django)

[south]: http://south.aeracode.org/


# Copying

Originally based on https://gist.github.com/2596149 by Trey Piepmeier.

[ ![Creative Commons License](http://i.creativecommons.org/l/by-sa/3.0/88x31.png) ](http://creativecommons.org/licenses/by-sa/3.0/deed.en_US)
This work is licensed under a [Creative Commons Attribution-ShareAlike 3.0 
Unported License](http://creativecommons.org/licenses/by-sa/3.0/deed.en_US)
