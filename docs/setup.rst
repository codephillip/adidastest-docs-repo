Project setup
==============

This contains the local machine project setup instructions

Project structure
------------------
* The project has a master repo called k8s
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
* `Imagine.ai generator principles`_

.. _12 factor app: https://www.12factor.net/
.. _Node JS file structure: https://www.codementor.io/@evanbechtol/node-service-oriented-architecture-12vjt9zs9i
.. _Node JS file structure2: https://dev.to/santypk4/bulletproof-node-js-project-architecture-4epf
.. _Django structure: https://alexkrupp.typepad.com/sensemaking/2021/06/django-for-startup-founders-a-better-software-architecture-for-saas-startups-and-consumer-apps.html
.. _Imagine.ai generator principles: https://www.imagine.ai/docs/best-practices


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

* Or, apply the secret object ``kubectl apply -f foobarfolder/secrets.yaml`` if you happen to have a `secrets.yaml` file
* Clone main repo ``git clone git@github.com:codephillip/adidas-k8s.git``
* `Initialize submodules`_ ``git submodule update --init --recursive``
* We shall use sqlite in memory db for testing, otherwise, create Persistent_ volumes (PV)and Claims (PVC) yaml files for a more consistent database. Add these environment variables to `.prod.env` or `.dev.env` in express services. NOTE: Do this for all express services

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


CI/CD
--------

When a pull request is created, tests are run on the code and a coverage report is created on codecov. Consequently, when code is pushed to main branch, a github actions script runs and pushes the code to `Google Cloud Build`

* There are github actions in side .github/workflows directory for each service
* We shall use the `gke github action lib`_ to deploy the app to gke
* Restart the deployment with ``kubectl rollout restart deployment foobar-depl``


Setting up GCLOUD_AUTH variable 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Go to IAM , select service account
* Name it, give required premissions and click create key and select JSON. It will download a json file
* Run this : cat downloaded_file.json | base64 | tr -d '\n'
* Take the output from step 3, and save it in a secret (recommended). In example below I am saving it as GCLOUD_AUTH.
* Add the GCLOUD_AUTH to the github secrets

Add more secrets
~~~~~~~~~~~~~~~~~~

* Add these to github secrets. For more guidance, please visit_

.. code-block:: console
    
    # create codecov account and get token
    CODECOV_TOKEN
    # get the rest from the GKE project on GCP
    GCLOUD_AUTH
    GKE_CLUSTER=adidasttestcluster
    GKE_DEPLOYMENT=adidas-express-email-depl
    GKE_NAMESPACE=default
    GKE_PROJECT=sixth-loader-344609
    GKE_ZONE=europe-west2-c

* Add these to the top of the ``deploy.yaml`` inside ``.github/workflows`` folder. Use the appropriate container names for each service

.. code-block:: console

    IMAGE: adidas_express_email
    REGISTRY: us.gcr.io
    CONTAINER: adidas-express-email

* Since we can't save the production variables in version control, we have to dynamically_ create them

The end result of all these steps should result in the creation of deploy.yml and test.yml

.. note::
    Please edit these templates accordingly

*deploy.yml*

.. code-block:: console

    name: Deploy to GKE

    on:
      push:
        branches:
          - main

    env:
      IMAGE: adidas_express_email
      REGISTRY: us.gcr.io
      CONTAINER: adidas-express-email

    jobs:
      build:
        runs-on: ${{ matrix.os }}
        strategy:
          matrix:
            os: [ ubuntu-latest ]
        steps:
          - uses: actions/checkout@v2
          - name: Deploy
            uses: shashank0202/docker-build-push-gcr-update-gke-deployment-action@v1.0
            with:
              service_account: ${{ secrets.GCLOUD_AUTH }}
              zone: ${{ secrets.GKE_ZONE }}
              project_id: ${{ secrets.GKE_PROJECT }}
              registry: ${{ env.REGISTRY }}
              image_name: ${{ env.IMAGE }}
              cluster: ${{ secrets.GKE_CLUSTER }}
              namespace: ${{ secrets.GKE_NAMESPACE }}
              deployment: ${{ secrets.GKE_DEPLOYMENT }}
              # Container name can be difficult to find, let alone understand
              # https://stackoverflow.com/questions/58516617/kubectl-set-image-error-arguments-in-resource-name-form-may-not-have-more-than
              container: ${{ env.CONTAINER }}
          - run: |-
              gcloud --quiet auth configure-docker

          # Get the GKE credentials so we can deploy to the cluster
          - uses: google-github-actions/get-gke-credentials@fb08709ba27618c31c09e014e1d8364b02e5042e
            with:
              cluster_name: ${{ secrets.GKE_CLUSTER }}
              location: ${{ secrets.GKE_ZONE }}
              credentials: ${{ secrets.GCLOUD_AUTH }}
          # Deploy the Docker image to the GKE cluster
          - name: Restart deployment
            run: kubectl rollout restart deployment ${{ secrets.GKE_DEPLOYMENT }}


*test.yml*

.. code-block:: console

    name: Run Tests

    on:
      pull_request:
        paths:
          - 'src/**'

    jobs:
      build:
        runs-on: ${{ matrix.os }}
        strategy:
          matrix:
            os: [ ubuntu-latest ]
            node-version: [ 16.0 ]
        steps:
          - uses: actions/checkout@v2
          - name: Use Node.js ${{ matrix.node-version }}
            uses: actions/setup-node@v1
            with:
              node-version: ${{ matrix.node-version }}
          - name: Cache Node.js modules
            uses: actions/cache@v2
            with:
              # npm cache files are stored in `~/.npm` on Linux/macOS
              path: ~/.npm
              key: ${{ runner.OS }}-node-${{ hashFiles('**/package-lock.json') }}
              restore-keys: |
                ${{ runner.OS }}-node-
                ${{ runner.OS }}-
          - name: Install dependencies
            run: npm install
          - name: Run tests
            run: npm run test
          - name: Create coverage report
            run: yarn coverage
          - name: Upload coverage to Codecov
            uses: codecov/codecov-action@v1
            with:
              token: ${{ secrets.CODECOV_TOKEN }}
              directory: ./coverage



.. _gke github action lib: https://github.com/marketplace/actions/docker-build-push-gcr-update-gke-deployment-action?version=v1.0
.. _visit: https://docs.github.com/en/actions/deployment/deploying-to-your-cloud-provider/deploying-to-google-kubernetes-engine
.. _dynamically: https://github.com/marketplace/actions/create-env-file