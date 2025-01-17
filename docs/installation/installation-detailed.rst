..  This file is part of Invenio
    Copyright (C) 2014, 2015 CERN.

    Invenio is free software; you can redistribute it and/or
    modify it under the terms of the GNU General Public License as
    published by the Free Software Foundation; either version 2 of the
    License, or (at your option) any later version.

    Invenio is distributed in the hope that it will be useful, but
    WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
    General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with Invenio; if not, write to the Free Software Foundation, Inc.,
    59 Temple Place, Suite 330, Boston, MA 02111-1307, USA.

.. _installation_detailed:

Detailed installation guide
===========================

.. admonition:: CAVEAT LECTOR

   Invenio v3.0 alpha is a bleeding-edge developer preview version that is
   scheduled for public release in Q1/2016.

Introduction
------------

In this installation guide, we'll create an Invenio digital library instance
using a multi-machine setup where separate services (such as the database server
and the web server) run on separate dedicated machines. Such a multi-machine
setup emulates to what one would typically use in production. (However, it is
very well possible to follow this guide and install all the services onto the
same "localhost", if one wants to.)

We'll use six dedicated machines running the following services:

============= ============= ====================
node          IP            runs
============= ============= ====================
web           192.168.50.10 Invenio web application
postgresql    192.168.50.11 `PostgreSQL <http://www.postgresql.org/>`_ database server
redis         192.168.50.12 `Redis <http://redis.io/>`_ caching service
elasticsearch 192.168.50.13 `Elasticsearch <https://www.elastic.co/products/elasticsearch>`_ information retrieval service
rabbitmq      192.168.50.14 `RabbitMQ <http://www.rabbitmq.com/>`_ messaging service
worker        192.168.50.15 `Celery <http://www.celeryproject.org/>`_ worker node
============= ============= ====================

The instructions below are tested on Ubuntu 14.04 LTS (Trusty Tahr) and CentOS 7
operating systems. For other operating systems such as Mac OS X, you may want to
check out the "kickstart" set of scripts coming with the Invenio source code
that perform the below-quoted installation steps in an unattended automated way.

Environment variables
---------------------

Let's define some useful environment variables that will describe our Invenio
instance setup:

.. glossary::

   INVENIO_WEB_HOST
     The IP address of the Web server node.

   INVENIO_WEB_INSTANCE
     The name of your Invenio instance that will be created. Usually equal to
     the name of the Python virtual environment.

   INVENIO_WEB_VENV
     The name of the Python virtual environment where Invenio will be installed.
     Usually equal to the name of the Invenio instance.

   INVENIO_USER_EMAIL
     The email address of a user account that will be created on the Invenio
     instance.

   INVENIO_USER_PASS
     The password of this Invenio user.

   INVENIO_POSTGRESQL_HOST
     The IP address of the PostgreSQL database server.

   INVENIO_POSTGRESQL_DBNAME
     The database name that will hold persistent data of our Invenio instance.

   INVENIO_POSTGRESQL_DBUSER
     The database user name used to connect to the database server.

   INVENIO_POSTGRESQL_DBPASS
     The password of this database user.

   INVENIO_REDIS_HOST
     The IP address af the Redis server.

   INVENIO_ELASTICSEARCH_HOST
     The IP address of the Elasticsearch information retrieval server.

   INVENIO_RABBITMQ_HOST
     The IP address of the RabbitMQ messaging server.

   INVENIO_WORKER_HOST
     The IP address of the Celery worker node.

In our example setup, we shall use:

.. include:: ../../.inveniorc
   :start-after: # sphinxdoc-kickstart-configuration-variables-begin
   :end-before: # sphinxdoc-kickstart-configuration-variables-end
   :literal:

Let us save this configuration in a file called ``.inveniorc`` for future use.

Web
---

The web application node (192.168.50.10) is where the main Invenio application
will be running. We need to provision it with some system dependencies in order
to be able to install various underlying Python and JavaScript libraries.

The web application node can be set up in an automated unattended way by running
the following script:

.. code-block:: shell

   source .inveniorc
   ./scripts/provision-web.sh

Let's see in detail what the web provisioning script does.

First, some useful system tools are installed:

* on Ubuntu 14.04 LTS (Trusty Tahr):

.. include:: ../../scripts/provision-web.sh
   :start-after: # sphinxdoc-install-useful-system-tools-trusty-begin
   :end-before: # sphinxdoc-install-useful-system-tools-trusty-end
   :literal:

* on CentOS 7:

.. include:: ../../scripts/provision-web.sh
   :start-after: # sphinxdoc-install-useful-system-tools-centos7-begin
   :end-before: # sphinxdoc-install-useful-system-tools-centos7-end
   :literal:

Second, an external Node.js package repository is enabled. We'll be needing to
install and run Npm on the web node later. The Node.js repository is enabled
as follows:

* on Ubuntu 14.04 LTS (Trusty Tahr):

.. include:: ../../scripts/provision-web.sh
   :start-after: # sphinxdoc-add-nodejs-external-repository-trusty-begin
   :end-before: # sphinxdoc-add-nodejs-external-repository-trusty-end
   :literal:

* on CentOS 7:

.. include:: ../../scripts/provision-web.sh
   :start-after: # sphinxdoc-add-nodejs-external-repository-centos7-begin
   :end-before: # sphinxdoc-add-nodejs-external-repository-centos7-end
   :literal:

Third, all the common prerequisite software libraries and packages that Invenio
needs are installed:

* on Ubuntu 14.04 LTS (Trusty Tahr):

.. include:: ../../scripts/provision-web.sh
   :start-after: # sphinxdoc-install-web-common-trusty-begin
   :end-before: # sphinxdoc-install-web-common-trusty-end
   :literal:

* on CentOS7:

.. include:: ../../scripts/provision-web.sh
   :start-after: # sphinxdoc-install-web-common-centos7-begin
   :end-before: # sphinxdoc-install-web-common-centos7-end
   :literal:

We want to use PostgreSQL database in this installation example, so we need to
install corresponding libraries too:

* on Ubuntu 14.04 LTS (Trusty Tahr):

.. include:: ../../scripts/provision-web.sh
   :start-after: # sphinxdoc-install-web-libpostgresql-trusty-begin
   :end-before: # sphinxdoc-install-web-libpostgresql-trusty-end
   :literal:

* on CentOS7:

.. include:: ../../scripts/provision-web.sh
   :start-after: # sphinxdoc-install-web-libpostgresql-centos7-begin
   :end-before: # sphinxdoc-install-web-libpostgresql-centos7-end
   :literal:

Fourth, now that Node.js is installed, we can proceed with installing Npm and
associated CSS/JS filter tools. Let's do it globally:

* on either of the operating systems:

.. include:: ../../scripts/provision-web.sh
   :start-after: # sphinxdoc-install-npm-and-css-js-filters-begin
   :end-before: # sphinxdoc-install-npm-and-css-js-filters-end
   :literal:

Finally, we'll install Python virtual environment wrapper tools and activate
them in the current user shell process:

* on either of the operating systems:

.. include:: ../../scripts/provision-web.sh
   :start-after: # sphinxdoc-install-virtualenvwrapper-begin
   :end-before: # sphinxdoc-install-virtualenvwrapper-end
   :literal:

Database
--------

The database server (192.168.50.11) will hold persistent data of our Invenio
installation, such as bibliographic records or user accounts. Invenio supports
MySQL, PostgreSQL, and SQLite databases. In this tutorial, we shall use
PostgreSQL that is the recommended database platform for Invenio.

The database server node can be set up in an automated unattended way by running
the following script:

.. code-block:: shell

   source .inveniorc
   ./scripts/provision-postgresql.sh

Let's see in detail what the database provisioning script does.

First, we install and configure the database software:

* on Ubuntu 14.04 LTS (Trusty Tahr):

.. include:: ../../scripts/provision-postgresql.sh
   :start-after: # sphinxdoc-install-postgresql-trusty-begin
   :end-before: # sphinxdoc-install-postgresql-trusty-end
   :literal:

* on CentOS 7:

.. include:: ../../scripts/provision-postgresql.sh
   :start-after: # sphinxdoc-install-postgresql-centos7-begin
   :end-before: # sphinxdoc-install-postgresql-centos7-end
   :literal:

We can now create a new database user with the necessary access permissions on
the new database:

* on either of the operating systems:

.. include:: ../../scripts/provision-postgresql.sh
   :start-after: # sphinxdoc-setup-postgresql-access-begin
   :end-before: # sphinxdoc-setup-postgresql-access-end
   :literal:

Redis
-----

The Redis server (192.168.50.12) is used for various caching needs.

The Redis server can be set up in an automated unattended way by running the
following script:

.. code-block:: shell

   source .inveniorc
   ./scripts/provision-redis.sh

Let's see in detail what the Redis provisioning script does.

* on Ubuntu 14.04 LTS (Trusty Tahr):

.. include:: ../../scripts/provision-redis.sh
   :start-after: # sphinxdoc-install-redis-trusty-begin
   :end-before: # sphinxdoc-install-redis-trusty-end
   :literal:

* on CentOS 7:

.. include:: ../../scripts/provision-redis.sh
   :start-after: # sphinxdoc-install-redis-centos7-begin
   :end-before: # sphinxdoc-install-redis-centos7-end
   :literal:

Elasticsearch
-------------

The Elasticsearch server (192.168.50.13) is used to index and search
bibliographic records, fulltext documents, and other various interesting
information managed by our Invenio digital library instance.

The Elasticsearch server can be set up in an automated unattended way by running
the following script:

.. code-block:: shell

   source .inveniorc
   ./scripts/provision-elasticsearch.sh

Let's see in detail what the Elasticsearch provisioning script does.

* on Ubuntu 14.04 LTS (Trusty Tahr):

.. include:: ../../scripts/provision-elasticsearch.sh
   :start-after: # sphinxdoc-install-elasticsearch-trusty-begin
   :end-before: # sphinxdoc-install-elasticsearch-trusty-end
   :literal:

* on CentOS 7:

.. include:: ../../scripts/provision-elasticsearch.sh
   :start-after: # sphinxdoc-install-elasticsearch-centos7-begin
   :end-before: # sphinxdoc-install-elasticsearch-centos7-end
   :literal:

RabbitMQ
--------

The RabbitMQ server (192.168.50.14) is used as a messaging middleware broker.

The RabbitMQ server can be set up in an automated unattended way by running the
following script:

.. code-block:: shell

   source .inveniorc
   ./scripts/provision-rabbitmq.sh

Let's see in detail what the RabbitMQ provisioning script does.

* on Ubuntu 14.04 LTS (Trusty Tahr):

.. include:: ../../scripts/provision-rabbitmq.sh
   :start-after: # sphinxdoc-install-rabbitmq-trusty-begin
   :end-before: # sphinxdoc-install-rabbitmq-trusty-end
   :literal:

* on CentOS 7:

.. include:: ../../scripts/provision-rabbitmq.sh
   :start-after: # sphinxdoc-install-rabbitmq-centos7-begin
   :end-before: # sphinxdoc-install-rabbitmq-centos7-end
   :literal:

Worker
------

The Celery worker node (192.168.50.15) is used to execute potentially long tasks
in asynchronous manner.

The worker node can be set up in an automated unattended way by running the
following script:

.. code-block:: shell

   source .inveniorc
   ./scripts/provision-worker.sh

Let's see in detail what the worker provisioning script does.

* on Ubuntu 14.04 LTS (Trusty Tahr):

.. include:: ../../scripts/provision-worker.sh
   :start-after: # sphinxdoc-install-worker-trusty-begin
   :end-before: # sphinxdoc-install-worker-trusty-end
   :literal:

* on CentOS 7:

.. include:: ../../scripts/provision-worker.sh
   :start-after: # sphinxdoc-install-worker-centos7-begin
   :end-before: # sphinxdoc-install-worker-centos7-end
   :literal:

Invenio
-------

Now that all the prerequisites have been set up, we can proceed with the
installation of the Invenio itself. The installation is happening on the web
node (192.168.50.10).

The installation of Invenio can be done in an automated unattended way by
running the following script:

.. code-block:: shell

   source .inveniorc
   ./scripts/install.sh

Let's see in detail what the Invenio installation script does.

We start by creating a fresh new Python virtual environment that will hold our
Invenio v3.0 instance:

.. include:: ../../scripts/install.sh
   :start-after: # sphinxdoc-create-virtual-environment-begin
   :end-before: # sphinxdoc-create-virtual-environment-end
   :literal:

We continue by installing Invenio v3.0 base package and most of its available
modules (by using option ``full`` as opposed to using option ``minimal``) from
PyPI:

.. include:: ../../scripts/install.sh
   :start-after: # sphinxdoc-install-invenio-full-begin
   :end-before: # sphinxdoc-install-invenio-full-end
   :literal:

We can now create a new Invenio instance:

.. include:: ../../scripts/install.sh
   :start-after: # sphinxdoc-create-instance-begin
   :end-before: # sphinxdoc-create-instance-end
   :literal:

Let's briefly customise our instance with respect to the location of the
database server, the Redis server, and all the other dependent services in our
multi-server environment:

.. include:: ../../scripts/install.sh
   :start-after: # sphinxdoc-customise-instance-begin
   :end-before: # sphinxdoc-customise-instance-end
   :literal:

In the instance folder, we run Npm to install any JavaScript libraries that
Invenio needs:

.. include:: ../../scripts/install.sh
   :start-after: # sphinxdoc-run-npm-begin
   :end-before: # sphinxdoc-run-npm-end
   :literal:

We can now collect and build CSS/JS assets of our Invenio instance:

.. include:: ../../scripts/install.sh
   :start-after: # sphinxdoc-collect-and-build-assets-begin
   :end-before: # sphinxdoc-collect-and-build-assets-end
   :literal:

We proceed by creating a dedicated database that will hold persistent data of
our installation, such as bibliographic records or user accounts. The database
tables can be created as follows:

.. include:: ../../scripts/install.sh
   :start-after: # sphinxdoc-create-database-begin
   :end-before: # sphinxdoc-create-database-end
   :literal:

We continue by creating a user account:

.. include:: ../../scripts/install.sh
   :start-after: # sphinxdoc-create-user-account-begin
   :end-before: # sphinxdoc-create-user-account-end
   :literal:

Let's now start Celery worker that will execute instance tasks:

.. include:: ../../scripts/install.sh
   :start-after: # sphinxdoc-start-celery-worker-begin
   :end-before: # sphinxdoc-start-celery-worker-end
   :literal:

Now that Celery is running, we can populate our Invenio instance with demo
records:

.. include:: ../../scripts/install.sh
   :start-after: # sphinxdoc-populate-with-demo-records-begin
   :end-before: # sphinxdoc-populate-with-demo-records-end
   :literal:

Let's register persistent identifiers for the uploaded demo records:

.. include:: ../../scripts/install.sh
   :start-after: # sphinxdoc-register-pid-begin
   :end-before: # sphinxdoc-register-pid-end
   :literal:

Finally, let's start the web application (in debugging mode):

.. include:: ../../scripts/install.sh
   :start-after: # sphinxdoc-start-application-begin
   :end-before: # sphinxdoc-start-application-end
   :literal:

We should now see our demo record on the web:

.. code-block:: shell

   firefox http://localhost:5000/records/1

and we can access it via REST API:

.. code-block:: shell

   curl -i -H "Accept: application/json" \
        http://localhost:5000/api/records/1

We are done! Our first Invenio v3.0 instance is fully up and running.
