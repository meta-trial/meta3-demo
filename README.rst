|pypi|

Sample META3 project
====================

This is a demo project of the META3 clinical trial EDC.

This demo sets up the EDC to run a test server in DEBUG mode on your local machine.

These are not the installation steps for a production system.

See also

https://www.lstmed.ac.uk/research/departments/international-public-health/respond-africa/meta

https://github.com/meta-trial/meta-edc

https://github.com/clinicedc

Installation
------------

To setup and run a test server locally.

You'll need mysql and conda.

Create the database

.. code-block:: bash

  mysql -Bse 'create database meta_example character set utf8;'


Create a working folder and clone this sample repo

.. code-block:: bash

  mkdir -p ~/projects/
  cd ~/projects
  git clone https://github.com/meta-trial/meta3-sample.git


Create a conda env named "meta3_sample" and activate

.. code-block:: bash

  conda create -n meta3_sample python=3.10
  conda activate meta3_sample


Install meta-edc

.. code-block:: bash
  
  pip install meta-edc
    
Copy the .env file

.. code-block:: bash


    cd ~/projects && cp .env-sample .env


Edit the environment file (.env) to include your mysql password in the ``DATABASE_URL``.

.. code-block:: bash

  # look for at the top of the file
  DATABASE_URL=mysql://<username>:<password>@127.0.0.1:3306/meta3_sample


Run manage.py for the first time to create the encryption keys

.. code-block:: bash

  cd ~/projects && python manage.py check

Go back and edit the environment file (.env). Change DJANGO_AUTO_CREATE_KEYS

.. code-block:: bash

    DJANGO_AUTO_CREATE_KEYS=False

Run migrate

.. code-block:: bash

    python manage.py migrate

Import a dummy randomization list

.. code-block:: bash

    python manage.py import_randomization_list

Import a holidays for scheduling

.. code-block:: bash

    python manage.py import_holidays

Create a user

.. code-block:: bash

  python manage.py createsuperuser

Start up `runserver`

.. code-block:: bash

  python manage.py runserver


Open your browser and point it to

.. code-block:: bash

  localhost:8000

Go to your user account and edit the permissions on your account.
You should NOT operate the EDC as a superuser.

* Under the section **Personal Details**, fill in your name and email.
* Under the section **Permissions**, uncheck *Superuser status*.
* At the bottom of the **User Profile** section you will see `Roles`. Add yourself to the Roles:

    * Account Manager
    * Staff
    * Clinician Super


.. |pypi| image:: https://img.shields.io/pypi/v/meta3-sample.svg
    :target: https://pypi.python.org/pypi/meta3-sample
