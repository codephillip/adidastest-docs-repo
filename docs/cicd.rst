CI/CD
=======

When a pull request is created, tests are run on the code and a coverage report is created on codecov. Consequently, when code is pushed to main branch, a github actions script runs and pushes the code to `Google Cloud Build`



Summary
----------

* There are github actions in side .github/workflows directory for each service
* We shall use the `gke github action lib`_ to deploy the app to gke
* Restart the deployment with ``kubectl rollout restart deployment foobar-depl``



Instructions
--------------

Setting up GCLOUD_AUTH variable 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Go to IAM, select service account
* Name it, give required premissions and click create key and select JSON. It will download a json file.
* Run ``cat downloaded_file.json | base64 | tr -d '\n'``
* Take the output from step 3, and save it in a secret (recommended).
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
    POSTGRES_DB=adidas-test-foobar
    POSTGRES_USER=adidas-testdb-instance
    POSTGRES_PASSWORD=s2e7gvCdG3eGxvCJ
    POSTGRES_HOST=35.189.219.141

* Add these to the top of the ``deploy.yaml`` inside ``.github/workflows`` folder. Use the appropriate container names for each service

.. code-block:: console

    IMAGE: adidas_express_email
    REGISTRY: us.gcr.io
    CONTAINER: adidas-express-email

* Since we can't save the production variables in version control, we have to dynamically_ create them


Final result
-------------

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
          - name: Make .env file
            uses: SpicyPizza/create-envfile@v1.3.0
            with:
              envkey_POSTGRES_DB: ${{ secrets.POSTGRES_DB }}
              envkey_POSTGRES_USER: ${{ secrets.POSTGRES_USER }}
              envkey_POSTGRES_PASSWORD: ${{ secrets.POSTGRES_PASSWORD }}
              envkey_POSTGRES_HOST: ${{ secrets.POSTGRES_HOST }}
              file_name: .env
              fail_on_empty: true
          - name: Make .prod.env file
            uses: SpicyPizza/create-envfile@v1.3.0
            with:
              envkey_TYPEORM_CONNECTION: postgres
              envkey_TYPEORM_DATABASE: ${{ secrets.POSTGRES_DB }}
              envkey_TYPEORM_USERNAME: ${{ secrets.POSTGRES_USER }}
              envkey_TYPEORM_PASSWORD: ${{ secrets.POSTGRES_PASSWORD }}
              envkey_TYPEORM_HOST: ${{ secrets.POSTGRES_HOST }}
              envkey_TYPEORM_ENTITIES: dist/**/*.model.js
              envkey_TYPEORM_MIGRATIONS: dist/migrations/*.js
              envkey_TYPEORM_MIGRATIONS_DIR: src/migrations
              envkey_TYPEORM_MIGRATIONS_RUN: true
              envkey_TYPEORM_SYNCHRONIZE: true
              envkey_TYPEORM_LOGGING: true
              file_name: .prod.env
              fail_on_empty: true
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