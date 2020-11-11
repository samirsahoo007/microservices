# What is REST(REpresentational State Transfer)?
The characteristics of a REST system are defined by six design rules:

	* Client-Server: There should be a separation between the server that offers a service, and the client that consumes it.

	* Stateless: Each request from a client must contain all the information required by the server to carry out the request. In other words, the server cannot store information provided by the client in one request and use it in another request.

	* Cacheable: The server must indicate to the client if requests can be cached or not.

	* Layered System: Communication between a client and a server should be standardized in such a way that allows intermediaries to respond to requests instead of the end server, without the client having to do anything different.

	* Uniform Interface: The method of communication between a client and a server must be uniform.

	* Code on demand: Servers can provide executable code or scripts for clients to execute in their context. This constraint is the only one that is optional.


# Designing a simple web service

GET	http://[hostname]/todo/api/v1.0/tasks			Retrieve list of tasks

GET	http://[hostname]/todo/api/v1.0/tasks/[task_id]		Retrieve a task

POST	http://[hostname]/todo/api/v1.0/tasks			Insert/Create a new task

PUT	http://[hostname]/todo/api/v1.0/tasks/[task_id]		Update an existing task

DELETE	http://[hostname]/todo/api/v1.0/tasks/[task_id]		Delete a task




We can define a task as having the following fields:

id: unique identifier for tasks. Numeric type.

title: short task description. String type.

description: long task description. Text type.

done: task completion state. Boolean type.


1. Setup Flask in a virtual envoronment https://github.com/samirsahoo007/technologies/blob/master/setting_up_of_virtualenv.md

```
	mkdir todo-api;cd todo-api
	virtualenv flask
	source flask/bin/activate
	pip install flask
```

2. Create app.py file with the following

```
#!flask/bin/python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return "Hello, World!"

if __name__ == '__main__':
    app.run(debug=True)
```

3. Run app.py

```
$ chmod a+x app.py
$ ./app.py
 * Running on http://127.0.0.1:5000/
```

** Done. Now your webservice is running on http://localhost:5000

# Implementing RESTful services in Python and Flask

Let's keep it simple...
In place of a database we will store our task list in a memory structure. This will only work when the web server that runs our application is single process and single threaded. This is okay for Flask's own development web server. It is not okay to use this technique on a production web server, for that a proper database setup must be used.

1. GET: Modify app.py as below to retrive list of tasks

```
#!flask/bin/python
from flask import Flask, jsonify

app = Flask(__name__)

tasks = [
    {
        'id': 1,
        'title': u'Buy groceries',
        'description': u'Milk, Cheese, Pizza, Fruit, Tylenol', 
        'done': False
    },
    {
        'id': 2,
        'title': u'Learn Python',
        'description': u'Need to find a good Python tutorial on the web', 
        'done': False
    }
]

@app.route('/todo/api/v1.0/tasks', methods=['GET'])
def get_tasks():
    return jsonify({'tasks': tasks})

if __name__ == '__main__':
    app.run(debug=True)
```

2. Test it: 

```
$ curl -i http://localhost:5000/todo/api/v1.0/tasks
```

3. GET: Add the following to retrieve a single task

```
from flask import abort

@app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['GET'])
def get_task(task_id):
    task = [task for task in tasks if task['id'] == task_id]
    if len(task) == 0:
        abort(404)
    return jsonify({'task': task[0]})
```
 
4. Test it:

```
$ curl -i http://localhost:5000/todo/api/v1.0/tasks/2
$ curl -i http://localhost:5000/todo/api/v1.0/tasks/3
```

5. POST: Let's insert a new item in our task database

```
from flask import request

@app.route('/todo/api/v1.0/tasks', methods=['POST'])
def create_task():
    if not request.json or not 'title' in request.json:
        abort(400)
    task = {
        'id': tasks[-1]['id'] + 1,
        'title': request.json['title'],
        'description': request.json.get('description', ""),
        'done': False
    }
    tasks.append(task)
    return jsonify({'task': task}), 201
```

6. Test it

```
$ curl -i -H "Content-Type: application/json" -X POST -d '{"title":"Read a book"}' http://localhost:5000/todo/api/v1.0/tasks

Verify the new entry
$ curl -i http://localhost:5000/todo/api/v1.0/tasks
```

7. PUT and DELETE

```
@app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['PUT'])
def update_task(task_id):
    task = [task for task in tasks if task['id'] == task_id]
    if len(task) == 0:
        abort(404)
    if not request.json:
        abort(400)
    if 'title' in request.json and type(request.json['title']) != unicode:
        abort(400)
    if 'description' in request.json and type(request.json['description']) is not unicode:
        abort(400)
    if 'done' in request.json and type(request.json['done']) is not bool:
        abort(400)
    task[0]['title'] = request.json.get('title', task[0]['title'])
    task[0]['description'] = request.json.get('description', task[0]['description'])
    task[0]['done'] = request.json.get('done', task[0]['done'])
    return jsonify({'task': task[0]})

@app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['DELETE'])
def delete_task(task_id):
    task = [task for task in tasks if task['id'] == task_id]
    if len(task) == 0:
        abort(404)
    tasks.remove(task[0])
    return jsonify({'result': True})
```

8. Test it

```
$ curl -i -H "Content-Type: application/json" -X PUT -d '{"done":true}' http://localhost:5000/todo/api/v1.0/tasks/2		# Update task 2

Verify it
$  curl -i http://localhost:5000/todo/api/v1.0/tasks

$ curl -i -X DELETE http://localhost:5000/todo/api/v1.0/tasks/2		# Delete task 2
Verify it
$  curl -i http://localhost:5000/todo/api/v1.0/tasks

```

# Securing a RESTful web service

1. pip install flask-httpauth

2. Add the following to app.py

```
from flask_httpauth import HTTPBasicAuth
auth = HTTPBasicAuth()

@auth.get_password
def get_password(username):
    if username == 'miguel':
        return 'python'
    return None

@auth.error_handler
def unauthorized():
    return make_response(jsonify({'error': 'Unauthorized access'}), 401)
```

3. Add "@auth.login_required" to the function you want to make PROTECTED

e.g.
```
@app.route('/todo/api/v1.0/tasks', methods=['GET'])
@auth.login_required
def get_tasks():
    return jsonify({'tasks': tasks})
```

4. Test it

```
$ curl -u miguel:python -i http://localhost:5000/todo/api/v1.0/tasks
```

See details here:
[Designing a RESTful API with Python and Flask](https://blog.miguelgrinberg.com/post/designing-a-restful-api-with-python-and-flask/page/5)
[Designing a RESTful API using Flask-RESTful](https://blog.miguelgrinberg.com/post/designing-a-restful-api-using-flask-restful)
[RESTful Authentication with Flask](https://blog.miguelgrinberg.com/post/restful-authentication-with-flask)

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
	fastavro (Avro or any data serialization package)

* Connexion adds a layer on top of Flask to help you building your RESTFul API in a simpler manner, with the great benefit to have at the end Swagger docs. Connexion gives you as well an elegant solution to protect your service behind oAuth2 and a way to versioning your API.
	OAuth 2.0 is the industry-standard protocol for authorization.

* Dependency Injection is a nice way to inject the dependencies your need in your methods/classes. For this purpose, I chose Flask-Injector, as you can see by the name it’s completely integrated on Flask. With this tool using the decorator @inject I can have the service I need, for instance, ElasticSearch or SQLAlchemy.
Did you have problems with services sending corrupted data or some payload with the wrong schema? Well, that can be solved using some Serialization tool like Avro or Protobuf. These tools help you to ensure whatever you are receiving or sending has the proper schema.

### Serializing your payloads

A common pitfall building services are sharing wrong payloads. For instance, you expect an Item object with a name and an id. But working with plain JSON is not possible to enforce it.
We can do it right now with your connexion yaml, it will throw exceptions if something is wrong. But in this case, we need to ensure the data between different services using for instance RabbitMQ.
We can use JSON as I said, but then for each application, you will need to have all the required validations. If your platform works with several programming languages that can be a hassle.
With some tools like Avro or Protobuf, we can have a common serialization objects among different programming languages. A good practice would be to have a repo where you have all your Avro/Protobuf serializers. Those can be used almost for any programming language.

In this case, Avro gives us:
	Rich data structures.
	A compact, fast, binary data format.
	A container file, to store persistent data.
	Remote procedure call (RPC).

Simple integration with dynamic languages. Code generation is not required to read or write data files nor to use or implement RPC protocols. Code generation as an optional optimization, only worth implementing for statically typed languages.

##### Libraries:

	import connexion

	from injector import Binder
	from flask_injector import FlaskInjector
	from connexion.resolver import RestyResolver
	from services.provider import ItemsProvider

Ref: https://github.com/samirsahoo007/microservices/blob/master/python-flask-microservice/app.py


See details:

[Part 1](https://medium.com/@ssola/building-microservices-with-python-part-i-5240a8dcc2fb)

[Part 2](https://medium.com/@ssola/building-microservices-with-python-part-2-9f951199094a)

[Part 3](https://medium.com/@ssola/building-microservices-with-python-part-3-a556a4c4bc00)

# How is Kafka used with microservice architecture?

If you consider a set of micro services that collectively make up a product, not all of will be mission critical.

But there are couple of mission critical components where in if a network call is missed the loss can be unrecoverable.

Let me tell you a real world use case from my day to day life and at the end I would expect you to say yes, this is the reason why I choose kafka.

Let's say you are booking a hotel with Goibibo where we say the maximum number of times a promoCode can be used is 2.

So after a successfull payment with a promo code the payments microservice has to notify the discounting microservice that a user had used this promo code. So that next time when user comes to the booking page to book the hotel, we check his usage on the promocode and let the UI microservice know if a promo code is applicable for the user or not.

Now consider this scenario where in once the user made the payment and by the time payments service is about to call discounting service , the discounting service went down for a toss.

As a normal Rest Service the payments microservice retries for 3 or 5 times and if the discounting service is still not up what should it do ?. Should it halt the transaction just because of a tech issue where one microservice is down . Well that's a worst user experience.

Here comes the hero (Kafka ) to our aid.

The payments service pushes a message to Kafka with the event details and Kafka stores the message with a new offset. The discounting service keeps polling for newer messages and read the new message and does some stuff and then commits to Kafka saying boss, i received an event and I am done with my application logic. Now commit the offset saying I have read the message .

So in this way even if the consumer service is down for sometime as the message is pushed to Kafka , the payment microservice simply updates the transaction as success . It's up to discounting service how it picks from Kafka and does logic on top of the event.

With the inclusion of Kafka we can make the transactions reliable.

Any sort of events which cannot be missed capturing had to be through a message broker and not through a network call.

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
