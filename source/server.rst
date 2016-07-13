Server deployment
=================

You can develop and test your app locally on your personal computer,
using the ordinary ``runserver`` command.

However, when you want to share your app with an audience,
you must deploy to a web server. oTree can be deployed to a cloud service like
Heroku, or to your own on-premises server.

Apache and MySQL on Windows
------

In general windows is not recommended as a platform for deployment but there
exists an easy solution based on WAMP Stack.

WAMP
~~~~~~~~~~~~~~~~~
`WAMP <http://www.wampserver.com/>`__ is web stack that includes Apache, MySQL and PHP.
For this walkthrough we will use 32-bits version of WampServer 3.
We will use default path for WAMP: ``C:\wamp``.
Keep in mind that both installation and Wamp manager files should be executed as Administrator.
If installation is successfull you should be able to launch Wamp manager and navigate to localhost.

Installing dependencies using a virtualenv
~~~~~~~~~~~~~~~~~

- Install virtualenv: ``pip install virtualenv``
- Navigate to folder with your apps ``cd C:\wamp\www\oTree``
- Create virtual environment: ``virtualenv venv``
- Activate virtual environment: ``venv\Scripts\activate.bat``
- Install modules required for oTree: ``pip install -r requirements_base.txt``
- Install compiled version of ``mod_wsgi`` module from `here <http://www.lfd.uci.edu/~gohlke/pythonlibs/#mod_wsgi>`__:
  ``pip``: ``pip install mod_wsgi-4.4.23+ap24vc9-cp27-cp27m-win32.whl``. Copy file ``venv\mod_wsgi.so`` into apache
  folder ``C:\wamp\bin\apache\apache2.4.17\modules``

.. note::

    You should install version of ``mod_wsgi`` that is compatible with version of your ``python``.
    Therefore choose correspondingly x86 or amd64 and Python2.* or Python 3.* 
    when you download precompiled version of ``mod_wsgi``.

- Install compiled version of ``mysql-python`` module from `here <http://www.lfd.uci.edu/~gohlke/pythonlibs/#mysql-python>`__:
  ``pip install MySQL_python-1.2.5-cp27-none-win32.whl``

Update Apache conf
~~~~~~~~~~~~~~~~~

- Add following line to ``C:\wamp\bin\apache\apache2.4.17\conf\httpd.conf`` after all ``LoadModule ...`` statements:

.. code-block:: apacheconf
  
    WSGIPythonHome C:\Python27
    WSGIPythonPath C:\wamp\www\oTree\venv\Lib\site-packages;C:\wamp\www\oTree
    LoadModule wsgi_module modules/mod_wsgi.so

- Add following lines to very end of file ``C:\wamp\bin\apache\apache2.4.17\conf\httpd.conf``:

.. code-block:: apacheconf

    <VirtualHost *>
        Alias /static/ C:/wamp/www/oTree/_static_root/
        
        <Directory C:/wamp/www/oTree/static>
          Order allow,deny
          Allow from all
          Require all granted
        </Directory>
        
        WSGIScriptAlias / C:/wamp/www/oTree/wsgi.py
        
        <Directory C:/wamp/www/oTree>
        <Files wsgi.py>
        Require all granted
        </Files>
        </Directory>
    </VirtualHost>

Setup oTree
~~~~~~~~~~~~~~~~~

- create entering point script for oTree apps ``C:\wamp\www\oTree\wsgi.py``:

.. code-block:: python

    import os
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'settings')

    from django.core.wsgi import get_wsgi_application
    from whitenoise.django import DjangoWhiteNoise

    application = get_wsgi_application()
    application = DjangoWhiteNoise(application)

- create database ``otree`` through `phpMyAdmin <http://localhost/phpmyadmin>`__ (login is root and password is blank)
- update database in ``settings.py``:

.. code-block:: python

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'otree',
            'USER': 'root',
            'PASSWORD': '',
            'HOST': 'localhost',
            'PORT': '3306',
        }
    }

- collect static files in one folder: ``python manage.py collectstatic``
- reset database: ``otree resetdb``

Set enviromental variables
~~~~~~~~~~~~~~~~~

- to prepare oTree for real session you need to update the script ``C:\wamp\www\oTree\wsgi.py``:

.. code-block:: python

    import os
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'settings')
    os.environ['OTREE_PRODUCTION'] = "1"
    os.environ['OTREE_AUTH_LEVEL'] = "STUDY"

    from django.core.wsgi import get_wsgi_application
    from whitenoise.django import DjangoWhiteNoise

    application = get_wsgi_application()
    application = DjangoWhiteNoise(application)

Heroku
------

`Heroku <https://www.heroku.com/>`__ is a commercial cloud hosting provider.
If you are not experienced with web server administration, Heroku may be
the simplest option for you.

The Heroku free plan is sufficient for small-scale testing of your app,
but once you are ready to go live, you should upgrade to a paid server,
which can handle more traffic.

Here are the steps for deploying to Heroku.

Create an account
~~~~~~~~~~~~~~~~~

Create an account on `Heroku <https://www.heroku.com/>`__.
Select Python as your main language. However,
you can
skip the "Getting Started With Python" guide.

Install the Heroku Toolbelt
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Install the `Heroku Toolbelt <https://toolbelt.heroku.com/>`__.

This provides you access to the Heroku Command Line utility.

Once installed open PowerShell (Windows) or Terminal (Mac),
and go to your project folder.

Log in using the email address and password you used when
creating your Heroku account:

.. code-block:: bash

    $ heroku login

If the ``heroku`` command is not found,
close and reopen your command prompt.

Initialize your Git repo
~~~~~~~~~~~~~~~~~~~~~~~~

If you haven't already initialized a git repository
run this command from your project's root directory:

.. code-block:: bash

    git init

Create the Heroku app
~~~~~~~~~~~~~~~~~~~~~

Create an app on Heroku, which prepares Heroku to receive your source
code:

.. code-block:: bash

    $ heroku create
    Creating lit-bastion-5032 in organization heroku... done, stack is cedar-14
    http://lit-bastion-5032.herokuapp.com/ | https://git.heroku.com/lit-bastion-5032.git
    Git remote heroku added
    When you create an app, a git remote (called heroku) is also created and associated with your local git repository.

Heroku generates a random name (in this case lit-bastion-5032) for your
app. Or you can specify your own name; see ``heroku help create`` for more info.
(And see ``heroku help`` for general help.)


Upgrade oTree
~~~~~~~~~~~~~

We recommend you use the latest version of oTree, to get the latest bugfixes.
Run:

.. code-block:: bash

    $ pip install --upgrade otree-core

Deploy your code
~~~~~~~~~~~~~~~~

Use ``pip`` to write a list of all the Python modules you have installed
(including ``otree-core``),
to a file called ``requirements_base.txt``.

Heroku will read this file and install the same version of each library on your server.

If using Windows PowerShell, enter::

    pip freeze | out-file -enc ascii requirements_base.txt

Otherwise, enter::

    pip freeze > requirements_base.txt


Commit your changes (note the dot in ``git add .``):

.. code-block:: bash

    git add .
    git commit -am "your commit message"

Transfer (push) the local repository to Heroku:

.. code-block:: bash

    $ git push heroku master

.. note::

    If you get a message ``push rejected``
    and the error message says ``could not satisfy requirement``,
    open ``requirements_base.txt`` and delete every line except
    the ones for ``Django`` and ``otree-core``.

Reset the oTree database on Heroku.
You can get your app's name by typing ``heroku apps``.

.. code-block:: bash

    $ heroku run otree resetdb

.. note::

    In older versions of oTree (before March 2016), you need to instead run
    ``otree-heroku resetdb your-heroku-app``
    (but it's probably better to update oTree anyway.)

Open the site in your browser:

.. code-block:: bash

    $ heroku open

(This command must be executed from the directory that contains your project.)


Turn on worker Dyno
~~~~~~~~~~~~~~~~~~~

To enable full functionality, you should go to the `Heroku Dashboard <https://dashboard.heroku.com/apps>`__,
click on your app, click to edit the dynos, and turn on the "worker"
dyno.

.. image:: _static/heroku-worker-dyno.JPG
    :align: center
    :scale: 100 %

You may need to upgrade from Heroku's "free" to "hobby" tier to turn on the
worker dyno.

If you are just testing your app, oTree will still function without the "worker" dyno,
but if you are running a study with real participants, we recommend turning it on.
This will ensure that the page timeouts defined by ``timeout_seconds``
still work even if a user closes their browser.

.. note::

    If you do not see a "worker" entry, make sure your ``Procfile``
    looks like `this <https://github.com/oTree-org/oTree/blob/master/Procfile>`__.


To add an existing remote:
~~~~~~~~~~~~~~~~~~~~~~~~~~

If you previously created a Heroku app and want to link your local oTree git repository
to that app, use this command:

.. code-block:: bash

    $ heroku git:remote -a [myherokuapp]

Making updates and modifications
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you make modifications to your app and want to push the updates
to Heroku, enter::

    git add .
    git commit -am "my commit message"
    git push heroku master
    heroku run otree resetdb

.. note::

    In older versions of oTree (before March 2016), you need to instead run
    ``otree-heroku resetdb your-heroku-app``


Scaling up the server
~~~~~~~~~~~~~~~~~~~~~

The Heroku free plan is sufficient for small-scale testing of your app, but once you are ready to go live,
you need to upgrade to a paid plan.

After you finish your experiment,
you can scale your dynos and database back down,
so then you don't have to pay the full monthly cost.

Postgres
++++++++

we recommend you upgrade your Postgres database to a paid tier
(at least the cheapest paid plan).

To provision the $50/month "Standard 0" database::

    $ heroku addons:create heroku-postgresql:standard-0
    Adding heroku-postgresql:standard-0 to sushi... done, v69
    Attached as HEROKU_POSTGRESQL_RED
    Database has been created and is available

This command will give you the name of your new DB (in the above example, ``HEROKU_POSTGRESQL_RED``).
Then you need to promote (i.e. "activate") this new database::

    $ heroku pg:promote HEROKU_POSTGRESQL_RED
    Promoting HEROKU_POSTGRESQL_RED_URL to DATABASE_URL... done

More info on the database plans `here <https://elements.heroku.com/addons/heroku-postgresql>`__,
and more technical documentation `here <https://devcenter.heroku.com/articles/heroku-postgresql>`__.

Upgrade dynos
+++++++++++++

In the Heroku dashboard, click on your app's "Resources" tab,
and in the "dynos" section, select "Upgrade to Hobby".
Then select either "Hobby" or "Professional".

Setting environment variables (optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you would like to turn off debug mode, you should set the ``OTREE_PRODUCTION``
environment variable, like this:

.. code-block:: bash

    $ heroku config:set OTREE_PRODUCTION=1

However, this will hide error pages, so you should set up :ref:`sentry`.

To password protect parts of the admin interface,
you should set ``OTREE_AUTH_LEVEL``):

.. code-block:: bash

    $ heroku config:set OTREE_AUTH_LEVEL=DEMO

More info at :ref:`AUTH_LEVEL`.

Logging with Sentry & Papertrail
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before launching a study, you should set up :ref:`sentry`.

In addition to Sentry, we recommend installing the free "Papertrail" logging add-on::

    heroku addons:create papertrail:choklad

Deploying to an on-premises server
----------------------------------

.. note::

    If you are just testing your app locally, you can use the ``resetdb`` and
    ``runserver`` commands, which are simpler than the below steps.

Although Heroku deployment may be the easiest option,
you may prefer to run oTree on your own server. Reasons may include:

-  You do not want your server to be accessed from the internet
-  You will be launching your experiment in a setting where internet
   access is unavailable
-  You want full control over how your server is configured

oTree runs on top of Django, so oTree setup is the same as Django setup.
Django runs on a wide variety of servers, except getting it to run on
a Windows server like IIS may require extra work; you can find info about
Django + IIS online. Below, instructions are given for using Unix and Gunicorn.

Database
~~~~~~~~

oTree's default database is SQLite, which is fine for local development,
but insufficient for production.
We recommend PostgreSQL, although you can also use MySQL, MariaDB, or any other database
supported by Django.

To use Postgres, first install Postgres, create a user (called ``postgres`` below),
and start your Postgres server. The instructions for doing the above depend on your OS.

Once that is done, you can create your database::

    $ psql -c 'create database django_db;' -U postgres

Now you should tell oTree to use Postgres instead of SQLite.
The default database configuration in ``settings.py`` is::

    DATABASES = {
        'default': dj_database_url.config(
            default='sqlite:///' + os.path.join(BASE_DIR, 'db.sqlite3')
        )
    }

However, instead of modifying the above line directly,
it's better to set the ``DATABASE_URL`` environment variable on your server::

    DATABASE_URL=postgres://postgres@localhost/django_db

(To learn what an "environment variable" is, see `here <http://superuser.com/a/284351>`__.)

Once ``DATABASE_URL`` is defined, oTree will use it instead of the default SQLite.
(This is done via `dj_database_url <https://pypi.python.org/pypi/dj-database-url>`__.)
Setting the database through an environment variable
allows you to continue to use SQLite locally (which is easier and more convenient).

Then, instead of installing ``requirements_base.txt``, install ``requirements.txt``.
This will install ``psycopg2``, which is necessary for using Postgres.

You may get an error when you try installing ``psycopg2``, as described
`here <http://initd.org/psycopg/docs/faq.html#problems-compiling-and-deploying-psycopg2>`__.

The fix is to install the ``libpq-dev`` and ``python-dev`` packages.
On Ubuntu/Debian, do:

.. code-block:: bash

    sudo apt-get install libpq-dev python-dev

Deploy your code
~~~~~~~~~~~~~~~~

If you are using a remote webserver, you need to push your code there,
typically using Git.

Open your shell, and make sure you have committed any changes as follows:

.. code-block:: bash

    pip freeze > requirements_base.txt
    git add .
    git commit -am '[commit message]'

(If you get the message
``fatal: Not a git repository (or any of the parent directories): .git``
then you first need to initialize the git repo.)

Then do:

.. code-block:: bash

    $ git push [remote name] master

Where [remote name] is the name of your server's git remote.


Running the server
~~~~~~~~~~~~~~~~~~

If you are just testing your app locally, you can use the usual ``runserver``
command.

However, when you want to use oTree in production, you need to run the
production server, which can handle more traffic. You should use a process
control system like Supervisord, and have it launch otree with the command
``otree runprodserver``.

This will run the ``collectstatic`` command, and then
launch the server as specified in the ``Procfile`` in your project's root
directory. The default ``Procfile`` launches the Gunicorn server.
If you want to use another server like Nginx, you need to modify the
``Procfile``. (If you instead want to use Apache, consult the Django docs.)

.. warning::

    Gunicorn doesn't work on Windows, so if you are trying to run oTree on a
    Windows server or use ``runprodserver`` locally on your Windows PC, you
    will need to specify a different server in your ``Procfile``.


.. _sentry:

Sentry
------

We recommend you use our free Sentry service (sign up `here <https://docs.google.com/forms/d/1aro9cL4smi1jbyFM--CqsJpr2oRHjNCE-UVHZEYHQcE/viewform>`__),
which can log all errors on your server and send you email notifications.
(`General info on Sentry <https://getsentry.com/welcome/>`__.)

A service like Sentry is necessary because once you have set the ``OTREE_PRODUCTION`` `environment variable <http://superuser.com/a/284351>`__.),
you will no longer see Django's yellow error pages; you or your users will just see generic "500 server error" pages.
Sentry can send you the details of each error by email.

Once you have signed up, we will send you a registration link you need to click.
You will also be provided with a special URL called a "Sentry DSN".

In your ``settings.py``, you should set ``SENTRY_DSN`` to your DSN URL,
which makes your server send crash info to our Sentry server.
Once that is done, you will automatically get notified with any exceptions when debug mode is turned off.
You can also view the errors through the `web interface <http://sentry.otree.org/auth/login/sentry/>`__.

If you later want other collaborators on your team to receive emails as well, or if you need to manage multiple projects,
send an email to chris@otree.org.

Database backups
----------------

When running studies, it is your responsibility to back up your database.

In Heroku, you can set backups for your Postgres database. Go to your `Heroku Dashboard <https://dashboard.heroku.com/apps/>`__,
click on the "Heroku Postgres" tab, and then click "PG Backups".
More information is available `here <https://devcenter.heroku.com/articles/heroku-postgres-backups>`__.

Modifying an existing database
------------------------------

If your database already contains data and you want to update the structure
without running ``resetdb`` (which will delete existing data), you can use Django's migrations feature.
Below is a quick summary; for full info see the Django docs `here <https://docs.djangoproject.com/en/1.9/topics/migrations/#workflow>`__.

The first step is to run ``python manage.py makemigrations my_app_name`` (substituting your app's name),
for each app you are working on. This will create a ``migrations`` directory in your app,
which you should add to your git repo, commit, and push to your server.

Instead of using ``otree resetdb`` on the server, run ``python manage.py migrate`` (or ``otree migrate``).
If using Heroku, you would do ``heroku run otree migrate``.
This will update your database tables.

If you get an error ``NameError: name 'Currency' is not defined``,
you need to find the offending file in your app's ``migrations`` folder,
and add ``from otree.common import Currency`` at the top of the file.

If you make further modifications to your apps, you can run
``python manage.py makemigrations``. You don't need to specify the app names in this command;
migrations will be updated for every app that has a ``migrations`` directory.
Then commit, push, and run ``python manage.py migrate`` again as described above.

More info `here <https://docs.djangoproject.com/en/1.9/topics/migrations/#workflow>`__
