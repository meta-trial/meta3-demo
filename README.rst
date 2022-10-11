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

You'll need `mysql` and `conda`.

Create the database

.. code-block:: bash

  mysql -Bse 'create database meta_example character set utf8;'


Create a working folder and clone the repo

.. code-block:: bash

  mkdir ~/clinicedc && \
  cd ~/projects && \
  git clone https://github.com/meta-trial/meta3-sample.git


Create a conda environment named "meta3_sample" and activate it

.. code-block:: bash

  conda create -n meta3_sample python=3.10 && \
  conda activate meta3_sample


With the conda environment activated, install the meta-edc application

.. code-block:: bash

  pip install meta-edc


The application is now installed, but there is more to do.

We need to make some changes to the configuration. Sensitive config values are stored in the environment (see https://12factor.net) by using an `.env` and `environ`. A sample environment file has been provided. Copy the sample environment file to a working copy

.. code-block:: bash

    cd ~/clinicedc/meta3-sample && cp .env-sample .env


Edit the working copy of the environment file (.env). At the top of the file you will find ``DATABASE_URL``. Change the value for ``DATABASE_URL`` to include your mysql user and password. The mysql account will need root or root-like permissions. Since this is a test server running locally, just use `root`.

.. code-block:: bash

  DATABASE_URL=mysql://<username>:<password>@127.0.0.1:3306/meta3_sample


Next we need to create the keys used for data encryption. The system encrypts sensitive data (personally identifiable information or PII) using django-crypto-fields.

Run manage.py for the first time to create the encryption keys

.. code-block:: bash

  python manage.py check

Go back and edit the environment file (.env). Change DJANGO_AUTO_CREATE_KEYS to False

.. code-block:: bash

    DJANGO_AUTO_CREATE_KEYS=False

Run manage.py `check` again. You should see a final message "System check identified 3 issues (1 silenced)". For the test server, you may ignore these warnings.

.. code-block:: bash

  python manage.py check

Now you are ready to prepare the database.

For now we will just restore the sample data provided in this repo. (Of course, you may run `python manage.py migrate`
but see the note below before going that route.)

The sample database is the exactly
what you would get if you ran `python manage.py migrate`.

Restore the sample data instead migrating from a clean database

.. code-block:: bash

    cd ~/clinicedc/meta3-sample/sample_data && \
    tar xzf meta3_sample.sql.tar.gz && \
    mysql meta3_sample < meta3_sample.sql && \
    cd ~/clinicedc/meta3-sample/

Note
    Running `migrate` on an empty database takes more than 30 min. If you do try to run `migrate` instead of
    restoring the sample data, in addition to the time it takes, you will run into a few simple problems with
    the `data` migrations. See the note on `running migrations on an empty database` in **Troubleshooting** below.

Now that our database has the required data schema, we need to run the post-migrate signals. To do this we just
run the `migrate` command.

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

  mysql -Bse "drop database meta3-sample;"

deactivate the conda environment::
  
  conda deactivate

remove the conda environment::

  conda env remove -n meta3-sample

Finally, delete the `clinicedc` folder.


.. |pypi| image:: https://img.shields.io/pypi/v/meta3-sample.svg
    :target: https://pypi.python.org/pypi/meta3-sample
