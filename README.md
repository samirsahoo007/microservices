![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/SOAP-v-REST.png)

|	        	|          SOAP(Simple Object Access Protocol)    |              REST(REpresentational State Transfer)             |
------------------------|:-----------------------------------------------:|---------------------------------------------------------------:|
|Design         	|          protocol                     	  |              	          architectural style                    |
|Message format		|Only XML					  | Plain text, HTML, XML, JSON, YAML, and others
|Approach		|Function-driven (data available as services, e.g.: "getUser")	|Data-driven (data available as resources, e.g. "user")
|Statefulness		|Stateless by default, but it's possible to make a SOAP API stateful.	|Stateless (no server-side sessions).
|Caching		|API calls cannot be cached.	|API calls can be cached.
|Security		|WS-Security with SSL support. |Built-in ACID compliance.
|Transfer protocol(s)	|	HTTP, SMTP, UDP, and others.	|Only HTTP
|Performance    	|requires more bandwidth and resource than REST.  |              requires less bandwidth and resource than SOAP.   |
|Advantages		|High security, standardized, extensibility.	|Scalability, better performance, browser-friendliness, flexibility.
|Disadvantages		|Poorer performance, more complexity, less flexibility.	|Less security, not suitable for distributed environments.

SOAP defines its own security.                   
RESTful web services inherits security measures from the  underlying transport.

SOAP can't use REST because it is a protocol    
REST can use SOAP web services because it is a concept and can use any protocol like HTTP, SOAP.

# Building Microservices with Python

Microservices should have some basic features:
	Easy to start coding your logic, stop worrying about tools/patterns.

	Documentation, an essential feature to share how your service it is going to work. In this case, Swagger works pretty well.

	Serializing your input/output in a way shared among all applications. You need to chose a technology like Avro/Protobuf. This is mandatory to be sure all services are sharing the same entities.

## Stack and Patterns

In this basic setup we are going to include those packages:

	Flask (as a Framework)
	connexion (helpful tool to generate routes and Swagger docs)
	Flask-Injector (Dependency Injection package)
	Avro (or any data serialization package)

* Connexion adds a layer on top of Flask to help you building your RESTFul API in a simpler manner, with the great benefit to have at the end Swagger docs. Connexion gives you as well an elegant solution to protect your service behind oAuth2 and a way to versioning your API.
	OAuth 2.0 is the industry-standard protocol for authorization.

* Dependency Injection is a nice way to inject the dependencies your need in your methods/classes. For this purpose, I chose Flask-Injector, as you can see by the name it’s completely integrated on Flask. With this tool using the decorator @inject I can have the service I need, for instance, ElasticSearch or SQLAlchemy.
Did you have problems with services sending corrupted data or some payload with the wrong schema? Well, that can be solved using some Serialization tool like Avro or Protobuf. These tools help you to ensure whatever you are receiving or sending has the proper schema.

[a link](https://gist.githubusercontent.com/ssola/f678725ab5000497f34b636794edbd02/raw/9da81ea6c79ff6fd3d2d926f22451790e204ae64/app.py)

Create a Simple Serverless Microservice using Lambda and API Gateway
=========================================================
Ref: https://docs.aws.amazon.com/lambda/latest/dg/with-on-demand-https-example-configure-event-source_1.html

In serverless architecture, the server will not run all the time. The server will be active only when a request is there for processing, and the rest of the time, the server will be in sleep mode. Unlike server architecture, a monthly subscription plan need not be taken, as the user only has to pay for the server's active time, which helps to reduce the server cost.

You upload your code to Lambda, and it takes care of everything required to run and scale its execution and fulfill conditions and high availability requirements.

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/aws_lambda_microservices.png)


![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/sserverless_architecture.png)


We'll use the Lambda console to create a Lambda function, and an Amazon API Gateway endpoint to trigger that function. We'll will be able to call the endpoint with any method (GET, POST, PATCH, etc.) to trigger Lambda function.

DELETE: delete an item from a DynamoDB table

GET: scan table and return all items

POST: Create an item

PUT: Update an item

Create an API Using Amazon API Gateway
--------------------------------------

1. Sign in to the AWS Management Console and open the AWS Lambda console.

2. Choose Create Lambda function.

3. Choose Blueprint.

4. Enter microservice in the search bar. Choose the microservice-http-endpoint blueprint and then choose Configure.

5. Configure the following settings.

        Name – lambda-microservice.

        Role – Create a new role from one or more templates.

        Role name – lambda-apigateway-role.

        Policy templates – Simple microservice permissions.

        API – Create a new API.

        Security – Open.

    Choose Create function.

When you complete the wizard and create your function, Lambda creates a proxy resource named lambda-microservice under the API name you selected. For more information about proxy resources, see Configure Proxy Integration for a Proxy Resource.(https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-set-up-simple-proxy.html)

A proxy resource has an AWS_PROXY integration type and a catch-all method ANY. The AWS_PROXY integration type applies a default mapping template to pass through the entire request to the Lambda function and transforms the output from the Lambda function to HTTP responses. The ANY method defines the same integration setup for all the supported methods, including GET, POST, PATCH, DELETE and others.

Test Sending an HTTPS Request
-----------------------------
In this step, you will use the console to test the Lambda function. In addition, you can run a curl command to test the end-to-end experience. That is, send an HTTPS request to your API method and have Amazon API Gateway invoke your Lambda function. In order to complete the steps, make sure you have created a DynamoDB table and named it "MyTable". For more information, see Create a DynamoDB Table with a Stream Enabled

To test the API

With your MyLambdaMicroService function still open in the console, choose the Actions tab and then choose Configure test event.

Replace the existing text with the following:

{
	"httpMethod": "GET",
	"queryStringParameters": {
	"TableName": "MyTable"
    }
}
After entering the text above choose Save and test.

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/1.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/2.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/3.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/4.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/5.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/6.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/7.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/8.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/9.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/10.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/11.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/12.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/13.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/14.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/15.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/16.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/17.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/18.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/19.png)

![alt text](https://github.com/samirsahoo007/microservices/blob/master/images/20.png)




Writing a MicroService with Python (Django)
===========================================

Typical Project Layout
-----------------------

::

    api
        migrations
            __init__.py
        __init__.py
        admin.py
        api.py
        models.py
        tests.py
        views.py
    projectservice
        __init__.py
        settings.py
        urls.py
        wsgi.py
    data
        initial.json
    .gitignore
    db.sqlite3
    manage.py
    README.md
    requirements.txt


Toolset
--------

* Documentation: Sphinx + `ReadTheDocs <https://readthedocs.org/>`_.
* Django Rest Framework    

Project Setup
-------------

**Instructions**

1. Setup and activate your virtual environment

  ::

    virtualenv env
    source env/bin/activate

2. Create a requirements.txt file with the following and run it

  ::

    django==1.7
    gunicorn
    requests
    djangorestframework==3
    django-rest-swagger
    django-filter

    ## dev requirements
    sphinx
    sphinx_rtd_theme
    mock
    responses
    ipdb
    ipython

    ## Test and quality analysis

    pylint
    coverage
    django-jenkins
    django-extensions
    django-cors-headers

    ## custom libs:
    -e git://github.com/TangentMicroServices/PythonAuthenticationLib.git#egg=tokenauth

Run the requirements file using::

    pip install -r requirements.txt

3. Create the python project

  ::

    django-admin.py startproject projectservice .

.. note::

    * projectservice all lowercase 
    * note that . at the end: so it creates it in the current directory
  

4. Check that your structure is as follows::

    LICENSE     
    README.md   
    manage.py   
    requirements.txt
    projectservice    
      __init__.py 
      settings.py 
      urls.py   
      wsgi.py

5. Create an API app::

    python manage.py startapp api

6. Create api.py in the api app::

    touch api/api.py

7. Add the following to settings.py::

    # CUSTOM AUTH
    AUTHENTICATION_BACKENDS = (
        'django.contrib.auth.backends.ModelBackend',
        'tokenauth.authbackends.TokenAuthBackend'
    )

    ## REST
    REST_FRAMEWORK = {
        'DEFAULT_PERMISSION_CLASSES': (
            'rest_framework.permissions.IsAuthenticated',
        ),
        'DEFAULT_AUTHENTICATION_CLASSES': (
            ## we need this for the browsable API to work
            'rest_framework.authentication.SessionAuthentication',
            'tokenauth.authbackends.RESTTokenAuthBackend',        
        )
    }

    # Services:

    ## Service base urls without a trailing slash:
    USER_SERVICE_BASE_URL = 'http://staging.userservice.tangentme.com'

    JENKINS_TASKS = (
        'django_jenkins.tasks.run_pylint',
        'django_jenkins.tasks.with_coverage',
        # 'django_jenkins.tasks.run_sloccount',
        # 'django_jenkins.tasks.run_graphmodels'
    )

    PROJECT_APPS = (
        'api',
    )

8. Update INSTALLED_APPS in settings.py::

    INSTALLED_APPS = (

        ...

        ## 3rd party
        'rest_framework',
        'rest_framework_swagger',

        ## custom
        'tokenauth',
        'api',

        # testing etc:
        'django_jenkins',
        'django_extensions',
        'corsheaders',
    )

9. Update MIDDLEWARE_CLASSES in setttings.py::

    MIDDLEWARE_CLASSES = (

        ## add this:
        'tokenauth.middleware.TokenAuthMiddleware',
        'corsheaders.middleware.CorsMiddleware',
        'django.middleware.common.CommonMiddleware',
    )

.. note::

    Note that CorsMiddleware needs to come before Django's CommonMiddleware if you are using Django's USE_ETAGS = True setting, otherwise the CORS headers will be lost from the 304 not-modified responses, causing errors in some browsers.

10. Update settings.py with the following setting at the bottom

    ::

        CORS_ORIGIN_ALLOW_ALL = True


Build the Database
------------------

1. Sync the database::

    python manage.py syncdb

.. note::
    
    Make the username admin and password a by default

2. Perform any migrations if necessary::

    python manage.py makemigrations
    python manage.py migrate

Initial Data
------------

1. Login to the admin panel and create some test data

2. Dump the data::

    python manage.py dumpdata > data/initial.json

3. Run the data to test that it works::

    python manage.py loaddata data/initial.json


Writing some Code
--------------------

Create some end points using - `Django REST Framework <http://www.django-rest-framework.org/>`_.

.. note::

    To include a Swagger API explorer for your API. Add::

        url(r'^api-explorer/', include('rest_framework_swagger.urls')), 

    to `urls.py`. for more info on using Swagger with Django Rest Framework, see: 

.. warning::

    The following code is for the hours service using entry. Rename accordingly.

1. In models.py add the following::

    from django.contrib.auth.models import User
    ...
    
    class Entry(models.Model):

        user = models.ForeignKey(User)
        title = models.CharField(max_length=200)

2. In api.py add the following::

    from rest_framework import viewsets, routers, serializers
    from rest_framework.decorators import detail_route
    from rest_framework.response import Response

    ...
    class EntryViewSet(viewsets.ModelViewSet):
        model = Entry
        serializer_class=EntrySerializer

    hours_router = routers.DefaultRouter()
    hours_router.register('entry', EntryViewSet)

3. In urls.py add the following::

    from api.api import hours_router
    ...

    urlpatterns = patterns('',
        url(r'^', include(hours_router.urls)), 
    )

4. python manage.py runserver


Authentication
--------------

Documenting
------------

1. Build the documentation in Sphinx

  ::

    sphinx-quickstart

This will create a folder called /docs and the structure should like this this::

    Makefile  
    make.bat
    build/    
    source/
      _static   
      _templates  
      conf.py   
      index.rst

2. Add /docs/build/ to .gitignore file


3. Write your own documentation as you go - `RST Docs <http://docutils.sourceforge.net/docs/user/rst/quickref.html>`_.

4. Update the readme file with instructions on how to setup the project

.. warning::

    The following code is for the hours service. Rename accordingly.
::

    # HoursService

    [![Build Status](http://jenkins.tangentme.com/buildStatus/icon?job=Build HoursService)](http://jenkins.tangentme.com/view/MicroServices/job/Build%20HoursService/)

    A Service for time tracking

    ## Setting Up

    1. Start and activate environment
        
            Virtualenv env
            source env/bin/activate

    1. Run the requirements

            pip install -r requirements.txt
        
    1. Install the database

            python manage.py syncdb
            
    1. Run the initial data (if required - this is test data only)

            python manage.py loaddata data/initial.json

    1. Run the tests to ensure the project is up and running correctly

            python manage.py test



Testing
------------

Unit Tests
___________

Uint tests can be run with::

    python manage.py test

Integration Tests
__________________

Integration tests should be stored in files matching the pattern `*_ITCase.py`. They can be run with:: 

    python manage.py test --pattern="*_ITCase.py"


Continious Integration with Jenkins
----------------------------------------

**Requirements**

* pip install pylint
* pip install coverage
* pip install django-jenkins
* pip install django-extensions

**Instructions**

1. Install requirements::

    pip install -r requirements.txt
    pip install pylint
    pip install coverage
    pip install django-jenkins
    pip install django-extensions

2. Configure settings.py::

    JENKINS_TASKS = (
      'django_jenkins.tasks.run_pylint',
      'django_jenkins.tasks.with_coverage',
      # 'django_jenkins.tasks.run_sloccount',
      # 'django_jenkins.tasks.run_graphmodels'
    )

    ## Apps to run analysis over:
    PROJECT_APPS = (
        'api',
    )

3. Run:: 

    `./manage.py jenkins`

This will:

* Run tests (build junit report)
* Generate coverage report (cobertura)
* Run pylint (generate checkstyle report)

All files are generated in the `reports` directory
