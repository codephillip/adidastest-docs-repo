More
=======

These are instructions that may guide the setup, maintenance or creation of more services

.. note:: Somethings have been omitted for brevity


Resources
-----------

* Kubernetes cheetsheet_
* Kubectl `docs or book`_
* Microservices course_
* Kubernetes_ course
* Microservices article_ that covers; Django, k8s, Docker, Celery, Redis, AWS, Postgres, Prometheus, etc.
* Microservices patterns with Java by Richardson_
* Microservices: `Up and Running`_ by Ronnie Mitra
* `Building Microservices`_ by Sam Newman
* Architecting container and microservice-based applications_ microsoft
* .NET Microservices: Architecture for Containerized_ .NET Applications
* Sphinx_ reStructuredText docs
* Sourceforge_ reStructuredText docs
* `Nats demo`_
* `Nats JS docs`_
* `Nats python docs`_
* `Nats parameters and docs`_
* `k8s best practises`_
* `My blog post`_


.. _cheetsheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
.. _docs or book: https://kubectl.docs.kubernetes.io/guides/
.. _course: https://www.udemy.com/course/microservices-with-node-js-and-react/
.. _article: https://markgituma.medium.com/kubernetes-local-to-production-with-django-1-introduction-d73adc9ce4b4
.. _Richardson: https://www.amazon.com/Microservices-Patterns-examples-Chris-Richardson/dp/1617294543
.. _microservices: https://dzone.com/articles/design-patterns-for-microservices
.. _Kubernetes: https://www.udemy.com/course/kubernetes-made-easy
.. _Sphinx: https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html
.. _Sourceforge: https://docutils.sourceforge.io/docs/user/rst/quickref.html
.. _Nats demo: https://github.com/codephillip/nats-streaming-server-nodejs-demo
.. _Nats JS docs: https://github.com/nats-io/stan.js
.. _Nats python docs: https://github.com/nats-io/stan.py
.. _Nats parameters and docs: https://hub.docker.com/_/nats-streaming
.. _k8s best practises: https://www.youtube.com/playlist?list=PLIivdWyY5sqL3xfXz5xJvwzFW_tlQB_GB
.. _kubectx: https://github.com/ahmetb/kubectx
.. _Up and Running: https://www.oreilly.com/library/view/microservices-up-and/9781492075448/
.. _My blog post: https://medium.com/dev-scribbles
.. _Building Microservices: https://samnewman.io/books/building_microservices_2nd_edition/
.. _applications: https://docs.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/#container-design-principles
.. _Containerized: https://docs.microsoft.com/en-us/dotnet/architecture/microservices/


Common library management
--------------------------

This package manager(npm) is used by the `common library` to ease update of commonly used code ie eventtypes, constants that cut across services, etc

* Start by following these Instructions_
* Clone ``git clone git@github.com:codephillip/adidas-common-lib.git``
* Move into folder. ``cd adidas-common-lib``
* Setup `package.json` and other files if not done so already
* Install `npm` package version greater than *6.14.4* due to this error_ ``npm install npm@latest -g``. ``sudo`` may be required if error persists

.. warning:: Remember to update version number in package.json. Otherwise `npm publish` will fail

* Install node-typescript ``sudo apt install node-typescript``
* Initialize Typescript ``tsc --init``. This will create a `tsconfig.json` file which helps conversion of TS to JS

.. note:: Typescript in the common library is converted to Javascript before deployment to package manager

* Install packages. ``npm install del-cli typescript --save-dev``
* Make changes to the code
* Get access to the NPM package account
* Push to package registry. ``npm pub`` or ``npm publish``

.. _Instructions: https://docs.gitlab.com/ee/user/packages/npm_registry/#authenticate-to-the-package-registry
.. _error: https://github.com/npm/cli/issues/1185


Microservice setup
-------------------

Inbrief these are the steps;

* Create package.json and install dependencies
* Write Dockerfile
* Create src/server/index.ts to run project
* Build image and push to container registry
* Write k8s files for deployment and service
* Update skaffold.yaml to do file sync for the new service
* Write k8s file for database(mongo/postgres) deployment and service PVC(optional)
* Add sql credentials to ``.prod.env`` or ``.dev.env``  in the root folder and set typeorm_ `TYPEORM_MIGRATIONS_RUN`, `TYPEORM_SYNCHRONIZE` and `TYPEORM_LOGGING` to true

    .. code-block:: console

        TYPEORM_CONNECTION=postgres
        TYPEORM_DATABASE=foobar-db
        TYPEORM_USERNAME=foobar
        TYPEORM_PASSWORD=foobar
        TYPEORM_HOST=foobar
        TYPEORM_ENTITIES=dist/**/*.model.js
        TYPEORM_MIGRATIONS=dist/migrations/*.js
        TYPEORM_MIGRATIONS_DIR=src/migrations
        TYPEORM_MIGRATIONS_RUN=true
        TYPEORM_SYNCHRONIZE=true
        TYPEORM_LOGGING=true


    .. warning:: TYPEORM_SYNCHRONIZE=true should only be run once, any changes afterwards should use TYPEORM_MIGRATIONS_RUN=true. Otherwise ALL DATABASE DATA WILL BE DELETED

* Add sql credentials to ``.env`` in the root folder

    .. code-block:: console

        POSTGRES_DB=foobar-db
        POSTGRES_USER=foobar
        POSTGRES_PASSWORD=foobar
        POSTGRES_HOST=foobar

* Add secret/env to k8s. ``kubectl create secret generic foobar-foo --from-literal=FOOBAR=asdf``

Remove CORS and CSRF warnings(not in production)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For express
* Add CORS exception. Add ``"cors": "^2.8.5",`` to `package.json`
* Add CORS middleware to ``src/server/app.ts``

  .. code-block:: javascript

      import cors from 'cors';

      function setUpAPIRoutes() {
        app.use(cors())
        ...

Deleting a service
~~~~~~~~~~~~~~~~~~~

In rare cases, this may be required

* Follow these instructions to delete the folder and remove `submodule entry`_
* Delete `yaml` files
* Delete from `ingress.yaml`
* Delete skaffold entry from `skaffold.yaml`


.. _submodule entry: https://gist.github.com/myusuf3/7f645819ded92bda6677
.. _`python images`: https://pythonspeed.com/articles/alpine-docker-python/



GKE testing deployment or environment
======================================

Setting up GKE cluster
------------------------

- Create GKE Standard cluster and nodes of count three
- Change `Node security` settings to `Allow access to all Cloud APIs`
- Add docker/k8s context by clicking `connect` button and copying the command ``gcloud container clusters get-credentials adidas-cluster1 --zone europe-west6-c --project adidas-317008``

    .. note:: To switch back to minikube or another context, run ``kubectx minikube``

- Setup `nginx ingress`_
- Add ingress to GKE cluster ``kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/cloud/deploy.yaml``
- Update skaffold.yaml with GKE configs

    .. code-block::

        build:
          googleCloudBuild:
            projectId: foobarId

        # as well as all the images to start with us.gcr.io
        image: us.gcr.io/foobarId/foobarservice

- Login to gcloud using ``gcloud auth application-default login``
- Enable Google Cloud Build API
- If this error is thrown ``could not create build: googleapi: Error 400: could not resolve source: googleapi: Error 403: 491169042809@cloudbuild.gserviceaccount.com does not have storage.objects.get access to the Google Cloud Storage object., forbidden``, add cluster user to bucket permissions_
- Give the user permissions of ``Storage Legacy Build Owner``
- Run ``skaffold run``
- Incase of error ``no endpoints available for service "ingress-nginx-controller-admission"`` or ``Error from server (InternalError): error when creating "STDIN": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io"`` or ``for: "STDIN": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1/ingresses?timeout=10s": x509: certificate signed by unknown authority``, delete the `ValidatingWebhookConfiguration` by running ``kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission``
- Incase of error ``Deploy Failed. Could not connect to cluster due to "https://34.136.245.244/version?timeout=32s": error executing access token command "/snap/google-cloud-sdk/199/bin/gcloud config config-helper --format=json":`` connect to cluster afresh ``gcloud container clusters get-credentials foobar-cluster4 --zone us-central1-c --project foobar-111111``
- Set zone if necessary ``gcloud config set compute/zone us-central1-c``
- Incase of error ``Unable to connect to the server: error executing access token command "/snap/google-cloud-sdk/196/bin/gcloud config config-helper --format=json": err=fork/exec /snap/google-cloud-sdk/196/bin/gcloud: no such file or directory output= stderr=``, reconnect to the cluster by clicking the `connect` button then copying the code ``gcloud container clusters get-credentials adidas-foobar --zone us-central1-c --project adidas-foobar``


Port forwarding to access NATS
-------------------------------------------------------

- Get running deployments ``kubectl get deployments``
- Open NATS port ``kubectl expose deployment foobar-nats-depl --type=LoadBalancer --port 8222 --target-port 8222``


Event-bus(NATS) setup
=======================

Initial setup for both listener and publisher services
---------------------------------------------------------

Common lib/repo
~~~~~~~~~~~~~~~~~

.. warning:: Only edit files under ``events/core/dynamic`` otherwise seek clarity before making edits to the shared library(adidas-common-lib)

* Make sure you are inside the adidas-common-lib repo
* Add subject to ``subjects.ts``

    .. code-block:: javascript

        export enum Subjects {
            FooBarCreated = 'foobar:created',
        }

* Add queue group to ``queueGroupNames.ts``

    .. code-block:: javascript

        export enum QueueGroupNames {
            FooBarService = 'foobar-service',
        }

* Create an interface for the event type definition

    .. code-block:: javascript

        import {Subjects} from './subjects';

        export interface FooBarCreatedEvent {
            subject: Subjects.FooBarCreated;

            //todo replace accordingly
            data: {
                foo: int;
                bar: string;
            };
        }

* Export the interface in the index.ts

    .. code-block:: javascript

        export * from './events/dynamic/fooBarCreatedEvent';
        export * from './events/dynamic/foobarEvent';
        export * from './events/dynamic/queueGroupNames';
        export * from './utils/db-utils';

* Run ``npm run pub`` to save and publish changes


Service folder/repo
~~~~~~~~~~~~~~~~~~~~

* Create ``events`` folder under ``src``
* Create ``listeners`` and ``publishers`` folders under ``events``
* Initialize NATS under ``index.ts`` and add the listener

    .. code-block:: console

        await natsWrapper.connectNatsListener(
            process.env.NATS_CLUSTER_ID,
            process.env.NATS_CLIENT_ID,
            process.env.NATS_URL
        );


Listener services
---------------------

* Create ``foobar-listener.ts`` under listener if the event is to listen for `foobar`

.. code-block:: javascript

    export class EmailNotificationCreatedListener extends Listener<EmailNotificationCreatedEvent> {
        readonly subject = Subjects.FooBar;
        readonly queueGroupName = QueueGroupNames.FooBarService;

        async onMessage(data: FooBarEvent['data'], msg: Message) {

            const {foo, bar} = data;

            //todo perform action here

            // required to tell the eventbus that the message has been received
            msg.ack();
        }
    }

* Initialize NATS under ``index.ts`` inside `start()` and add the listener. ``new EmailNotificationCreatedListener(natsWrapper.client).listen();``


Publisher services
--------------------

* Create ``FooBarPublisher`` that extends ``Publisher`` inside ``foobar-publisher.ts``

    .. code-block:: javascript

        export class FooBarPublisher extends Publisher<FooBarEvent> {
            readonly subject = Subjects.FooBarCreated;
        }

* Use this anywhere to publish an event

    .. code-block:: javascript

        const publisher = new FoobarPublisher(natsWrapper.client);
        //todo replace accordingly
        await publisher.publish({
            foo: 1,
            bar: 'fizzbuzz'
        })

NATS monitoring(event bus)
----------------------------

* Port forward port 8222 ``kubectl port-forward <podid> 8222:8222``
* Open browser on http://localhost:8222/streaming
* Navigate into the channels subscribers http://localhost:8222/streaming/channelsz?subs=1
* Add json chrome extension_ to view the json better
* Analyze the entities_

.. _entities: https://docs.nats.io/nats-streaming-concepts/monitoring/endpoints
.. _extension: https://chrome.google.com/webstore/detail/json-formatter/bcjindcccaagfpapjjmafapmmgkkhgoa/related?hl=en


Common commands
=================

Kubernetes
-----------

* Most common `k8s commands`_
* Display all pds ``kubectl get pods``
* Display all deployments ``kubectl get deployments``
* Reset minikube ``minikube delete --all --purge``


Clearing space on local dev env
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Delete all docker containers incase system runs out of space ``docker system prune``
* Run ``minikube stop``
* Run ``minikube start``

.. _`k8s commands`: https://kubernetes.io/docs/reference/kubectl/cheatsheet/

Express
--------

* Deploy `common library` used by express microservices ``npm run pub``


kubectl
--------

* ssh into minikube ``minikube ssh`` for inspection of resources
* ssh into pod ``kubectl exec -it <podid> sh``
* List all persistentvolumes ``kubectl get pv``
* List all pods ``kubectl get pods``
* Port forwarding``kubectl port-forward <podid> port:port``
