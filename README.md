# meta3-sample
Sample META3 project

Installation
------------

To setup and run a test server locally.

You'll need mysql and conda.

Create the database

.. code-block:: bash

  mysql -Bse 'create database meta_example character set utf8;'


Create a working folder

.. code-block:: bash

  mkdir -p ~/projects/meta3-sample/settings
  touch ~/projects/meta3-sample/settings/__init__.py
  cd ~/projects


Create a conda env named "meta3_sample" and activate

.. code-block:: bash

  conda create -n meta3_sample python=3.10
  conda activate meta3_sample

Install meta-edc

.. code-block:: bash
  
  pip install meta-edc
    
Copy the .env file

.. code-block:: bash

  cp .env-sample .env


Edit the environment file (.env) to include your mysql password in the ``DATABASE_URL``.

.. code-block:: bash

  # look for and update this line
  DATABASE_URL=mysql://user:password@127.0.0.1:3306/meta_sample


Continue with the installation, From meta-edc project folder

.. code-block:: bash

  python manage.py migrate
  python manage.py import_randomization_list
  python manage.py import_holidays

Create a user and start up `runserver`

.. code-block:: bash

  cd ~/projects/meta-edc
  git checkout master
  python manage.py createsuperuser
  python manage.py runserver


Login::

  localhost:8000
