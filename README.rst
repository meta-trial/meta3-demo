|pypi|


META3 Demo Project
==================

This is a demo project of the EDC used by the META3 clinical trial.

This demo sets up the META3 EDC to run on a Django test server in DEBUG mode on your local machine.

These are NOT the installation steps for a production system.

See also:

https://www.lstmed.ac.uk/research/departments/international-public-health/respond-africa/meta

https://github.com/meta-trial/meta-edc

https://github.com/clinicedc

https://www.djangoproject.com

Installation
------------

You'll need setup `mysql` and `miniconda` on your local machine.

Get miniconda here
    https://docs.conda.io/en/latest/miniconda.html


Now that `mysql` and `miniconda` are install on your local machine, let's get started.

First, create the demo database

.. code-block:: bash

  mysql -Bse 'create database meta3_demo character set utf8;'


Create a project folder and clone the repo into it

.. code-block:: bash

  mkdir ~/clinicedc && \
  cd ~/clinicedc && \
  git clone https://github.com/meta-trial/meta3-demo.git


Create a `conda` environment named "meta3_demo" and activate it

.. code-block:: bash

  conda create -n meta3_demo python=3.10 && \
  conda activate meta3_demo


With the conda environment activated, install the `meta-edc` python package

.. code-block:: bash

  pip install meta-edc


Now that the application is installed, we need to make some changes to the configuration. 

Copy the sample environment file to a working copy

.. code-block:: bash

    cd ~/clinicedc/meta3-demo && cp .env-sample .env


Note
    Sensitive config values are stored in the environment by using an `.env` file and `environ`. (see also see https://12factor.net)

Next, edit your working copy of the environment file (.env). Look for ``DATABASE_URL`` at the top of the file. Change the value for ``DATABASE_URL`` to include your mysql user and password. The mysql account will need root or root-like permissions. Since this is a test server running locally, just use `root`.

.. code-block:: bash

  # find this line
  DATABASE_URL=mysql://<username>:<password>@127.0.0.1:3306/meta3_demo
  
  # and change with your details, for example
  DATABASE_URL=mysql://root:dumela@127.0.0.1:3306/meta3_demo

Next we need to create the keys used for data encryption. 

Run manage.py for the first time to create the encryption keys

.. code-block:: bash

  python manage.py check

Note
    The system encrypts sensitive data (personally identifiable information or PII) using `django-crypto-fields`. The first time you run
    `manage.py`, `django-crypto-fields` looks for the keys. If they do not exists, it creates them. 

Now go back and edit the environment file (.env). Change DJANGO_AUTO_CREATE_KEYS to False

.. code-block:: bash

    DJANGO_AUTO_CREATE_KEYS=False

Run manage.py `check` again. You should see a final message "System check identified 3 issues (1 silenced)". Since this is a test server, you may ignore these warnings.

.. code-block:: bash

  python manage.py check

Now you are ready to prepare the database that you created earlier.

To save time, we will just restore the demo data provided in this repo. The demo data is an empty database that is the exactly what you would get if you ran the 'migrate` command.

Restore the demo data

.. code-block:: bash

    cd ~/clinicedc/meta3-demo/demo_data && \
    tar xzf meta3_demo.sql.tar.gz && \
    mysql meta3_demo < meta3_demo.sql && \
    cd ~/clinicedc/meta3-demo/

Note
    There are a few caveats to migrating your own database instead of using the demo data. Running `migrate` on an empty database takes more than 30 min. Also, you will run into a few simple problems with the `data` migrations. See the note on`running migrations on an empty database` in **Troubleshooting** below.

Now that our database has the required data schema, we need to run the post-migrate signals to populate some static data. But this does not take long. 

To do this we just run the `migrate` command.

.. code-block:: bash

    # run migrate to trigger the post-migrate signals
    python manage.py migrate


Next, import the list of holidays that will be used when scheduling appointments.

.. code-block:: bash

    python manage.py import_holidays

Next, create a user. Do this from the command line using the `createsuperuser` command.

.. code-block:: bash

  python manage.py createsuperuser

Important
    The new user you just created is a "superuser". Once logged in you need to remove the superuser status for
    this account.

Now start up the test server using the `runserver` command

.. code-block:: bash

  python manage.py runserver


Open your browser and point it to

.. code-block:: bash

  localhost:8000

You should see the login screen.

Type in the credentials of the `superuser` account you just created.

Once logged in, go to your user account and edit the permissions on your account. You can use the link at the top right corner.

* Under the section **Personal Details**, fill in your name and email.
* Under the section **Permissions**, uncheck *Superuser status*.
* At the bottom of section **User Profile** you will see `Roles`. Add yourself to the following roles:

    * Account Manager
    * Staff
    * Clinician Super

Click `Home` on breadcrumbs to the left on the top bar.

Now you are ready to screen your first participant.


Troubleshooting
---------------

Running migrate on an empty database
++++++++++++++++++++++++++++++++++++

If you run `migrate` on an empty database, a few of the `data` migrations might fail.
Since these failed migrations are `data` migrations and not `schema` migrations, it is safe to run migrate until it fails,
fake the failed `data` migration, and continue.

.. code-block:: bash

    python manage.py migrate

    # fake the data migration
    python manage.py migrate meta_prn 0035 --fake

    # restart migrate
    python manage.py migrate

    # fake the data migration
    python manage.py migrate meta_subject 0107 --fake

    # restart migrate
    python manage.py migrate

    # fake the data migration
    python manage.py migrate meta_subject 0132 --fake

    # restart migrate
    python manage.py migrate

Removing the demo when you are done
-----------------------------------

drop the database::

  mysql -Bse "drop database meta3_demo;"

deactivate the conda environment::

  conda deactivate

remove the conda environment::

  conda env remove -n meta3_demo

Finally, delete the `clinicedc` folder.


.. |pypi| image:: https://img.shields.io/pypi/v/meta3-demo.svg
    :target: https://pypi.python.org/pypi/meta3-demo
