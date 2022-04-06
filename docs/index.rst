============================================
Adidas Test Documentation - Phillip Kigenyi
============================================

Introduction
============

This is the adidas backend challenge.

All in all, the three services; public, email and subscription were created successfully. Evenmore, kubernetes configurations files were created ``infra/k8s`` and CI/CD was also implemented ``.github/workflows``


Setup pages
============

#. :doc:`Architecture diagram </arch>`
#. :doc:`Production test </productiontest>`
#. :doc:`Setup instructions </setup>`
#. :doc:`CICD </cicd>`
#. :doc:`More </other>`

Tools
-------

#. Typescript: Used to enforce data types mostly in asynchronous communication
#. Swagger: Used for documentation
#. Postman: Used to test various endpoints and create documentation
#. Expressjs: Used to run microservices using Javascript and Node version 16(16-slim)
#. Kubernetes: Used to manage microservice deployment, scaling and monitoring
#. Docker: Used to encapsulate each service for easy deployment
#. NATS: Used as an event bus in asynchronous communication
#. Skaffold: Used for quick testing and running k8s commands
#. Minikube: Used to run k8s on dev environment
#. kubectl: Used to manage k8s resources
#. kubectx: Used to change between k8s contexts ie minikube, gke_foobar
#. PostgresDB: Used for storage of more permanent and consistent data
#. NPM CommonLibrary: Used to share commonly reused components between services
#. Ingress-Nginx: Routes incoming traffic to the services/pods
#. reStructuredText(rst): Used to write the docs
#. Markdown: Used to write short docs in the README.md
#. Django: Used to create web microservice(only authentication service)
#. Django Rest Framework: Used to create REST api endpoints in Django
#. Python3: Used to implement the programming logic in Django microservices
#. GCP: Used to build and run the k8s production app
#. Github actions. Used these to create the CI/CD pipeline which tests, builds and deploys to production
#. Travis. Used it run tests
#. Codecov. Used it to check the coverage and provide a report


Github repos
-------------
- https://github.com/codephillip/adidas-k8s
- https://github.com/codephillip/adidas-express-public
- https://github.com/codephillip/adidas-express-sub
- https://github.com/codephillip/adidas-express-email
- https://github.com/codephillip/adidas-django-auth
- https://github.com/codephillip/adidas-common-lib (NPM library)
- https://github.com/codephillip/adidastest-docs-repo


Microservices
---------------

- adidas-express-public. A backend for frontend service that acts a proxy server. Handles authentication, DDOS attacks
- adidas-express-sub. Allows users to subscribe to a newsletter or campaign(one public url, the rest are admin access only)
- adidas-express-email. Allows admins to send emails to the users(admin access only)
- adidas-django-auth. An authentication service written in django and generates JWT tokens

Contacts
==============

Incase of any inquiries, please reach me on any of these channels

Email: codephillip@gmail.com
Phone: +256756878460
LinkedIn: https://www.linkedin.com/in/kigenyi-phillip-461778101/


