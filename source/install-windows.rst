:orphan:

.. _install-windows:

Installing oTree on Windows
===========================

Important note
--------------

If you publish research conducted using oTree,
you are required by the oTree license to cite
`this paper <http://dx.doi.org/10.1016/j.jbef.2015.12.001>`__.
(Citation: Chen, D.L., Schonger, M., Wickens, C., 2016. oTree - An open-source
platform for laboratory, online and field experiments.
Journal of Behavioral and Experimental Finance, vol 9: 88-97)

If the below steps don't work for you, please email chris@otree.org with details.

Step 1: Install Python
----------------------

Download and install `Python 3.6 <https://www.python.org/ftp/python/3.6.4/python-3.6.4-amd64.exe>`__.
Check the box to add Python to PATH:

.. figure:: _static/setup/py-win-installer.png

Step 2: Install oTree
---------------------

In your Start Menu, search for the program "cmd",
open Command Prompt, and enter:

.. code-block:: bash

    pip3 install -U otree

.. note::

    If you get an error like this::

        error: Microsoft Visual C++ 9.0 is required (Unable to find vcvarsall.bat).

    To fix this, install the `Visual C++ Build Tools <http://go.microsoft.com/fwlink/?LinkId=691126>`__.


Step 3: Run oTree
-----------------

Create your project folder::

    otree startproject oTree

When it asks you "Include sample games?" choose yes.

Move into the folder you just created::

    cd oTree

Run the server::

    otree devserver

Open your browser to `http://localhost:8000/ <http://localhost:8000/>`__.
You should see the oTree demo site.

To stop the server, press ``Control + C`` at your command line.

.. _pycharm:

Step 4: Install PyCharm
-----------------------

Install `PyCharm <https://www.jetbrains.com/pycharm/download/>`__,
which you will use for editing your Python files.

After installing, open PyCharm, go to "File -> Open..." and select your project folder
(It's usually ``C:\Users\<your_username>\oTree``).

Then click on ``File –> Settings``, go to "Project -> Project interpreter",
and set it to the location of your Python executable,
maybe ``C:\Program Files\Python36\python.exe`` or
``C:\Users\<your_username>\AppData\Local\Programs\Python\Python36-32\python.exe``.

If PyCharm displays this warning, select "Ignore requirements":

.. figure:: _static/setup/pycharm-psycopg2-warning.png

Note: Even if you normally use another text editor,
we recommend at least trying PyCharm, because PyCharm's autocompletion
makes learning oTree much easier:

.. figure:: _static/setup/pycharm-autocomplete.gif


.. _upgrade:
.. _upgrade-otree-core:

Upgrading/reinstalling oTree
----------------------------

We recommend you upgrade on a weekly basis,
so that you can get the latest bug fixes and features.
This will also ensure that you are using a version that is consistent with the current documentation.

The command to upgrade is the same as the command to install:

.. code-block:: bash

    pip3 install -U otree

