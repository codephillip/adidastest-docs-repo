Project setup
==============

This contains the local machine project setup instructions

Project structure
------------------
* The project has a master repo called ``adidas-k8s``
* Each microservice is in its own repo and connected to the master repo through `git submodule`
* Folder `infra` contains k8s configuration files
* Share code is stored inside adidas-common-lib


File structure architecture
----------------------------

Every microservice follows these guidelines

* `12 factor app`_
* `Node JS file structure`_
* `Node JS file structure2`_
* `Django structure`_

.. _12 factor app: https://www.12factor.net/
.. _Node JS file structure: https://www.codementor.io/@evanbechtol/node-service-oriented-architecture-12vjt9zs9i
.. _Node JS file structure2: https://dev.to/santypk4/bulletproof-node-js-project-architecture-4epf
.. _Django structure: https://alexkrupp.typepad.com/sensemaking/2021/06/django-for-startup-founders-a-better-software-architecture-for-saas-startups-and-consumer-apps.html


Localhost development setup
----------------------------

* Install essential dependencies

    * Skaffold_
    * Minikube
    * Docker
    * npm
    * pip3

* Install minikube
* Install kubectl ``sudo snap install kubectl --classic``
* Run ``minikube start``
* Run ``minikube ip`` then add the ip address to `/etc/hosts` with a custom url. Run ``sudo vim /etc/hosts``. Then add ``x.x.x.x www.testadidas.io``
* Create the postgres credentials_ on your local machine

.. code-block::

    # create postgres user
    sudo -u postgres createuser adidasuser
    # create dbs
    sudo -u postgres createdb adidasemail
    sudo -u postgres createdb adidasauth
    sudo -u postgres createdb adidassub
    # change user password
    sudo -u postgres psql
    alter user adidasuser with encrypted password 'superP4WORD$';
    GRANT ALL PRIVILEGES ON DATABASE  adidasauth TO adidasuser;
    GRANT ALL PRIVILEGES ON DATABASE  adidassub TO adidasuser;
    GRANT ALL PRIVILEGES ON DATABASE  adidasemail TO adidasuser;

* Add environment variables

.. code-block::

    kubectl create secret generic postgres-user --from-literal=POSTGRES_USER='adidasuser'
    kubectl create secret generic postgres-password --from-literal=POSTGRES_PASSWORD='superP4WORD$'
    kubectl create secret generic postgres-host --from-literal=POSTGRES_HOST='127.0.0.1'
    kubectl create secret generic database-type --from-literal=DATABASE_TYPE='test'
    kubectl create secret generic postgres-sub-db --from-literal=POSTGRES_DB='mydatabase'
    kubectl create secret generic postgres-email-db --from-literal=POSTGRES_DB='mydatabase'
    kubectl create secret generic postgres-auth-db --from-literal=POSTGRES_DB='mydatabase'
    kubectl create secret generic api-version --from-literal=API_VERSION='v1'
    kubectl create secret generic secret-key --from-literal=SECRET_KEY='mn871rqc=2v$omiosampfodasmfdbyp62c)4794#y@s4123214'
    kubectl create secret generic debug --from-literal=DEBUG='true'
    kubectl create secret generic node-env --from-literal=NODE_ENV='test'

* Or, apply the secret object ``kubectl apply -f foobarfolder/secrets.yaml`` if you happen to have a `secrets.yaml` file
* Clone main repo ``git clone git@github.com:codephillip/adidas-k8s.git``
* `Initialize submodules`_ ``git submodule update --init --recursive``
* We shall use sqlite in memory db for testing, otherwise, create Persistent_ volumes (PV)and Claims (PVC) yaml files for a more consistent database. Add these environment variables to or `.dev.env` in express services. NOTE: Do this for all express services

.. code-block:: console

    TYPEORM_CONNECTION=sqlite
    TYPEORM_DATABASE=mydatabase
    TYPEORM_ENTITIES=dist/**/*.model.js
    TYPEORM_MIGRATIONS=dist/migrations/*.js
    TYPEORM_MIGRATIONS_DIR=src/migrations
    TYPEORM_MIGRATIONS_RUN=true
    TYPEORM_SYNCHRONIZE=true
    TYPEORM_LOGGING=true


* Also add database credentials to the `.env` file. This is required by Typeform. NOTE: Do this for all express services

.. code-block:: console

    DB_NAME=mydatabase.sqlite
    DB_STORAGE=mydatabase.sqlite

* Enable ingress. ``minikube addons enable ingress``
* Replace ``us.gcr.io/sixth-loader-344609`` with ``codephillip`` in the microservices deployment files ie inside infra/k8s/adidas-express-email-depl.yaml
* Run ``skaffold dev -f skaffold_dev.yaml`` to start local minikube(k8s) tools

.. _Persistent: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
.. _credentials: https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e

More setup instructions(optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Update submodules incase of updates ``git pull --recurse-submodules && git submodule update --recursive`` and ``git submodule foreach git pull origin master``
* Incase postgres fails to start with error ``Is the server running on host "auth-postgres-service" (x.x.x.x) and accepting TCP/IP connections on port 5432?`` run this command to check active ports ``sudo lsof -i -P -n | grep LISTEN``.
* Then `Deactivate postgres`_ ``systemctl stop postgresql`` and disable autostart ``systemctl disable postgresql``
* Run ``skaffold dev -f skaffold_dev.yaml``
* Incase databases(PV and PVC) are getting deleted on local development, run ``skaffold dev -f skaffold_dev.yaml --cleanup=false``
* Incase of limited internet, use ``skaffold dev -f skaffold_dev.yaml --cache-artifacts=true``
* Optional: Set your IDE autosave threshold to 15 seconds to prevent skaffold from auto building
* Incase you get errors in skaffold while installing npm packages ``unable to stream build output: Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io``. Run `this command`_ ``minikube stop && minikube start`` or ``minikube delete && minikube start``
* Incase you run out of space. Run list images ``minikube ssh -- docker images -f dangling=true`` then delete using ``minikube ssh -- docker image prune``
* Install npm packages to stop the IDE false errors

    .. code-block:: console

        npm install
        npm install @adidastest-phillip/common

.. _Skaffold: https://skaffold.dev/docs/install/
.. _Deactivate postgres: https://stackoverflow.com/a/49828382/4991437
.. _this command: https://stackoverflow.com/a/65753467/4991437
.. _Initialize submodules: https://stackoverflow.com/questions/1030169/easy-way-to-pull-latest-of-all-git-submodules

.. note:: Sometimes the `skaffold dev` tools may malfunction and stop accepting requests to and from the pods. Such an error may appear ``Error: getaddrinfo *EAI_AGAIN* xyz``. This may occur during npm package installation or when the pod has fully deployed. Quick solution is to run ``minikube stop`` then ``minikube start``. If all else fails run ``docker system prune`` and ``minikube ssh -- docker system prune``

Local dev machine setup to push directly to production with skaffold
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Make code changes and push to gitlab
- Ask for permission to the GCP project from the lead developer
- Install ``gcloud`` on your local machine
- Login to gcloud using ``gcloud auth application-default login``
- Add docker/k8s context by clicking `connect` button and copying the command ``gcloud container clusters get-credentials adidasttestcluster --zone europe-west2-c --project sixth-loader-344609``
- Set zone if necessary ``gcloud config set compute/zone europe-west2-c``
- Add environment variables if not done so already

.. code-block::

    kubectl create secret generic postgres-user --from-literal=POSTGRES_USER='adidas-testdb-instance'
    kubectl create secret generic postgres-password --from-literal=POSTGRES_PASSWORD='s2e7gvCdG3eGxvCJ'
    kubectl create secret generic postgres-host --from-literal=POSTGRES_HOST='35.189.219.141'
    kubectl create secret generic database-type --from-literal=DATABASE_TYPE='production'
    kubectl create secret generic postgres-sub-db --from-literal=POSTGRES_DB='adidas-test-sub'
    kubectl create secret generic postgres-email-db --from-literal=POSTGRES_DB='adidas-test-email'
    kubectl create secret generic postgres-auth-db --from-literal=POSTGRES_DB='adidas-test-auth'
    kubectl create secret generic api-version --from-literal=API_VERSION='v1'
    kubectl create secret generic secret-key --from-literal=SECRET_KEY='mn871rqc=2v$e-z9$rvl1m3njf+0byp62c)4794#y@s4y8d3@^*y'
    kubectl create secret generic debug --from-literal=DEBUG='false'
    kubectl create secret generic node-env --from-literal=NODE_ENV='test'


* Add these environment variables to `.prod.env` in express services. NOTE: Do this for all services

.. code-block:: console

    TYPEORM_CONNECTION=postgres
    TYPEORM_DATABASE=adidas-test-email
    TYPEORM_USERNAME=adidas-testdb-instance
    TYPEORM_PASSWORD=s2e7gvCdG3eGxvCJ
    TYPEORM_HOST=35.189.219.141
    TYPEORM_ENTITIES=dist/**/*.model.js
    TYPEORM_MIGRATIONS=dist/migrations/*.js
    TYPEORM_MIGRATIONS_DIR=src/migrations
    TYPEORM_MIGRATIONS_RUN=true
    TYPEORM_SYNCHRONIZE=true
    TYPEORM_LOGGING=true

* Also add database credentials to the `.env` file. This is required by Typeform. NOTE: Do this for all services

.. code-block:: console

    # adidas test use one of the POSTGRES_DB shown above
    POSTGRES_DB=adidas-test-foobar
    POSTGRES_USER=adidas-testdb-instance
    POSTGRES_PASSWORD=s2e7gvCdG3eGxvCJ
    POSTGRES_HOST=35.189.219.141

- Run ``skaffold dev`` if you want to monitor directly in your terminal. Otherwise ``skaffold run`` works best


.. _`nginx ingress`: https://kubernetes.github.io/ingress-nginx/deploy/#gce-gke
.. _owner: https://console.cloud.google.com/iam-admin/iam?authuser=1&project=adidas-317008
.. _permissions: https://console.cloud.google.com/storage/browser/adidas-317008_cloudbuild;tab=permissions?forceOnBucketsSortingFiltering=false&authuser=1&project=adidas-317008&prefix=&forceOnObjectsSortingFiltering=false

Code Update
~~~~~~~~~~~~~~

* Run ``skaffold dev -f skaffold_dev.yaml`` if not done so already
* Edit source files in repo
* Since we are using skaffold, autoreload is enabled therefore no need to restart minikube(k8s)
* Run tests ``npm run test``
* Monitor k8s resources with ``minikube dashboard``
* Check for common library updates ``npm update @adidastest-philip/common``
* Save changes by running ``git add .`` , ``git commit -m "my message"`` and then ``git push``
