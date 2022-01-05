<div dir="rtl">

# Todo API

Over the course of the next two chapters we will build a Todo API back-end and then connect it with a React front-end. We have already made our first API and reviewed how HTTP and REST work in the abstract but it’s still likely you don’t “quite” see how it all fits together yet. By the end of these two chapters you will.

Since we are making a dedicated back-end and front-end we will divide our code into a similar structure. Within our existing code directory, we will create a todo directory containing our back-end Django Python code and our front-end React JavaScript code.
The eventual layout will look like this.

<div dir="ltr">

```
todo
|   ├──frontend
|       ├──React...
|   ├──backend
|       ├──Django...
```

</div>

This chapter focuses on the back-end and Chapter 4 on the front-end.

### Initial Set Up

The first step for any Django API is always to install Django and then later add Django REST
Framework on top of it. First create a dedicated todo directory within our code directory on the
Desktop.

Open a new command line console and enter the following commands:

<div dir="ltr">

```shell
$ cd ~/Desktop
$ cd code
$ mkdir todo && cd todo
```

</div>

Note: Make sure you have deactivated the virtual environment from the previous chapter. You
can do this by typing exit. Are there no more parentheses in front of your command line? Good.
Then you are not in an existing virtual environment.

Within this todo folder will be our backend and frontend directories. Let’s create the backend
folder, install Django, and activate a new virtual environment.

<div dir="ltr">

```shell
(backend) $ django-admin startproject config .
(backend) $ python manage.py startapp todos
(backend) $ python manage.py migrate
```

</div>

In Django we always need to add new apps to our INSTALLED_APPS setting so do that now. Open
up config/settings.py in your text editor. At the bottom of the file add todos.

<div dir="ltr">

```python
# config/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # Local
    'todos', # new
]
```

</div>

If you run python manage.py runserver on the command line now and navigate in your web
browser to http://127.0.0.1:8000/ you can see our project is successfully installed.

![Image 1](images/1.jpg)

We’re ready to go!

### Models

Next up is defining our Todo database model within the todos app. We will keep things basic and
have only two fields: title and body.

<div dir="ltr">

```python
# todos/models.py
from django.db import models

class Todo(models.Model):
    title = models.CharField(max_length=200)
    body = models.TextField()
    
    def __str__(self):
        return self.title
```

</div>

We import models at the top and then subclass it to create our own Todo model. We also add a
`__str__` method to provide a human-readable name for each future model instance.

Since we have updated our model it’s time for Django’s two-step dance of making a new migration
file and then syncing the database with the changes each time. On the command line type
Control+c to stop our local server. Then run these two commands:

<div dir="ltr">

```shell
(backend) $ python manage.py makemigrations todos
Migrations for 'todos':
    todos/migrations/0001_initial.py
        - Create model Todo
(backend) $ python manage.py migrate
Operations to perform:
    Apply all migrations: admin, auth, contenttypes, sessions, todos
Running migrations:
    Applying todos.0001_initial... OK
```

</div>

It is optional to add the specific app we want to create a migration file for–we could instead type
just python manage.py makemigrations–however it is a good best practice to adopt. Migration
files are a fantastic way to debug applications and you should strive to create a migration file for each small change. If we had updated the models in two different apps and then run python
manage.py makemigrations the resulting single migration file would contain data on both apps.
That just makes debugging harder. Try to keep your migrations as small as possible.

Now we can use the built-in Django admin app to interact with our database. If we went into
the admin straight away our Todos app would not appear. We need to explicitly add it via the
todos/admin.py file as follows.

<div dir="ltr">

```shell
# todos/admin.py
from django.contrib import admin
from .models import Todo

admin.site.register(Todo)
```

</div>

That’s it! Now we can create a superuser account to log in to the admin.

<div dir="ltr">

```shell
(backend) $ python manage.py createsuperuser
```

</div>

And then start up the local server again:

<div dir="ltr">

```shell
(backend) $ python manage.py runserver
```

</div>

If you navigate to http://127.0.0.1:8000/admin/ you can now log in. Click on “+ Add” next to
Todos and create 3 new todo items, making sure to add a title and body for both. Here’s what
mine looks like:

![Image 2](images/2.jpg)

We’re actually done with the traditional Django part of our Todo API at this point. Since we are
not bothering to build out webpages for this project, there is no need for website URLs, views,
or templates. All we need is a model and Django REST Framework will take care of the rest.

### Django REST Framework

Stop the local server Control+c and install Django REST Framework via pipenv.

<div dir="ltr">

```shell
(backend) $ pipenv install djangorestframework~=3.11.0
```

</div>

Then add rest_framework to our INSTALLED_APPS setting just like any other third-party application. We also want to start configuring Django REST Framework specific settings which all exist
under REST_FRAMEWORK. For starters, let’s explicitly set permissions to AllowAny35. This line goes
at the bottom of the file.

<div dir="ltr">

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # 3rd party
    'rest_framework', # new
    
    # Local
    'todos',
]

# new
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.AllowAny',
    ]
}
```

</div>

Django REST Framework has a lengthy list of implicitly set default settings. You can see the
complete list here36
. AllowAny is one of them which means that when we set it explicitly, as
we did above, the effect is exactly the same as if we had no DEFAULT_PERMISSION_CLASSES config
set.

Learning the default settings is something that takes time. We will become familiar with a number
of them over the course of the book. The main takeaway to remember is that the implicit
default settings are designed so that developers can jump in and start working quickly in a local
development environment. The default settings are not appropriate for production though. So
typically we will make a number of changes to them over the course of a project.

Ok, so Django REST Framework is installed. What next?

Unlike the Library project in the previous chapter where we built both a webpage and an API, here
we are just building an API. Therefore we do not need to create any template files or traditional
Django views.

Instead we will update three files that are Django REST Framework specific to transform our
database model into a web API: urls.py, views.py, and serializers.py.

### URLs

I like to start with the URLs first since they are the entry-point for our API endpoints. Just as in
a traditional Django project, the urls.py file lets us configure the routing.

Start at the Django project-level file which is config/urls.py. We import include on the second
line and add a route for our todos app at api/.

<div dir="ltr">

```python
# config/urls.py
from django.contrib import admin
from django.urls import include, path # new

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('todos.urls')), # new
]
```

</div>

Next create our app-level todos/urls.py file.

<div dir="ltr">

```shell
(backend) $ touch todos/urls.py
```

</div>

And update it with the code below.

<div dir="ltr">

```python
# todos/urls.py
from django.urls import path
from .views import ListTodo, DetailTodo

urlpatterns = [
    path('<int:pk>/', DetailTodo.as_view()),
    path('', ListTodo.as_view()),
]
```

</div>

Note that we are referencing two views—ListTodo and DetailTodo—that we have yet to create.
But the routing is now complete. There will be a list of all todos at the empty string '', in other
words at api/. And each individual todo will be available at its primary key, which is a value
Django sets automatically in every database table. The first entry is 1, the second is 2, and so on.
Therefore our first todo will eventually be located at the API endpoint api/1/.

### Serializers

Let’s review where we are so far. We started with a traditional Django project and app where
we made a database model and added data. Then we installed Django REST Framework and
configured our URLs. Now we need to transform our data, from the models, into JSON that will
be outputted at the URLs. Therefore we need a serializer.

Django REST Framework ships with a powerful built-in serializers class that we can quickly
extend with a small amount of code. That’s what we’ll do here.

First create a new serializers.py file in the todos app.

<div dir="ltr">

```shell
(backend) $ touch todos/serializers.py
```

</div>

Then update it with the following code.

<div dir="ltr">

```python
# todos/serializers.py
from rest_framework import serializers
from .models import Todo

class TodoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Todo
        fields = ('id', 'title', 'body',)
```

</div>

At the top we have imported serializers from Django REST Framework as well as our models.py
file. Next we create a class TodoSerializer. The format here is very similar to how we create
model classes or forms in Django itself. We’re specifying which model to use and the specific
fields on it we want to expose. Remember that id is created automatically by Django so we didn’t
have to define it in our Todo model but we will use it in our detail view.

And that’s it. Django REST Framework will now magically transform our data into JSON exposing
the fields for id, title, and body from our Todo model.

The last thing we need to do is configure our views.py file

### Views

In traditional Django views are used to customize what data to send to the templates. In Django
REST Framework views do the same thing but for our serialized data.

The syntax of Django REST Framework views is intentionally quite similar to regular Django views
and just like regular Django, Django REST Framework ships with generic views for common use
cases. That’s what we’ll use here.

Update the todos/views.py file to look as follows:

<div dir="ltr">

```python
# todos/views.py
from rest_framework import generics
from .models import Todo
from .serializers import TodoSerializer

class ListTodo(generics.ListAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer

class DetailTodo(generics.RetrieveAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
```

</div>

At the top we import Django REST Framework’s generics views and both our models.py and
serializers.py files.

Recall from our todos/urls.py file that we have two routes and therefore two distinct views.
We will use ListAPIView37 to display all todos and RetrieveAPIView38 to display a single model
instance.

Astute readers will notice that there is a bit of redundancy in the code here. We essentially repeat
the queryset and serializer_class for each view, even though the generic view extended is
different. Later on in the book we will learn about viewsets and routers which address this issue
and allow us to create the same API views and URLs with much less code.

But for now we’re done! Our API is ready to consume. As you can see, the only real difference
between Django REST Framework and Django is that with Django REST Framework we need
to add a serializers.py file and we do not need a templates file. Otherwise the urls.py and
views.py files act in a similar manner.

### Consuming the API

Traditionally consuming an API was a challenge. There simply weren’t good visualizations for all
the information contained in the body and header of a given HTTP response or request.

Instead most developers used a command line HTTP client like cURL39, which we saw in the
previous chapter, or HTTPie40.

In 2012, the third-party software product Postman41 was launched and it is now used by millions
of developers worldwide who want a visual, feature-rich way to interact with APIs.

But one of the most amazing things about Django REST Framework is that it ships with a powerful
browsable API that we can use right away. If you find yourself needing more customization
around consuming an API, then tools like Postman are available. But often the built-in browsable
API is more than enough.

### Browsable API

Let’s use the browsable API now to interact with our data. Make sure the local server is running.

<div dir="ltr">

```shell
(backend) $ python manage.py runserver
```

</div>

Then navigate to http://127.0.0.1:8000/api/ to see our working API list views endpoint.

![Image 3](images/3.jpg)

This page shows the three todos we created earlier in the database model. The API endpoint is
known as a collection because it shows multiple items.

There is a lot that we can do with our browsable API. For starters, let’s see the raw JSON view—
what will actually be transmitted over the internet. Click on the “GET” button in the upper right
corner and select JSON.

![Image 4](images/4.jpg)

If you go back to our list view page at http://127.0.0.1:8000/api/ we can see there is additional
information. Recall that the HTTP verb GET is used to read data while POST is used to update or create data.

Under “List Todo” it says GET /api/ which tells us that we performed a GET on this endpoint.
Below that it says HTTP 200 OK which is our status code, everything is working. Crucially below
that it shows ALLOW: GET, HEAD, OPTIONS. Note that it does not include POST since this is a
read-only endpoint, we can only perform GET’s.

We also made a DetailTodo view for each individual model. This is known as an instance and is
visible at http://127.0.0.1:8000/api/1/.

![Image 5](images/5.jpg)

You can also navigate to the endpoints for:

- http://127.0.0.1:8000/api/2
- http://127.0.0.1:8000/api/3

### CORS

There’s one last step we need to do and that’s deal with Cross-Origin Resource Sharing
(CORS)42. Whenever a client interacts with an API hosted on a different domain (mysite.com vs yoursite.com) or port (localhost:3000 vs localhost:8000) there are potential security issues.

Specifically, CORS requires the server to include specific HTTP headers that allow for the client
to determine if and when cross-domain requests should be allowed.

Our Django API back-end will communicate with a dedicated front-end that is located on a
different port for local development and on a different domain once deployed.

The easiest way to handle this–-and the one recommended by Django REST Framework43–-is
to use middleware that will automatically include the appropriate HTTP headers based on our
settings.

The package we will use is django-cors-headers44, which can be easily added to our existing
project.

First quit our server with Control+c and then install django-cors-headers with Pipenv.

<div dir="ltr">

```shell
(backend) $ pipenv install django-cors-headers==3.4.0
```

</div>

Next update our config/settings.py file in three places:

- add corsheaders to the INSTALLED_APPS
- add CorsMiddleware above CommonMiddleWare in MIDDLEWARE
- create a CORS_ORIGIN_WHITELIST

<div dir="ltr">

```python
# config/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # 3rd party
    'rest_framework',
    'corsheaders', # new
    
    # Local
    'todos',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'corsheaders.middleware.CorsMiddleware', # new
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# new
CORS_ORIGIN_WHITELIST = (
    'http://localhost:3000',
    'http://localhost:8000',
)
```

</div>

It’s very important that corsheaders.middleware.CorsMiddleware appears in the proper location. That is above django.middleware.common.CommonMiddleware in the MIDDLEWARE setting
since middlewares are loaded top-to-bottom. Also note that we’ve whitelisted two domains:
localhost:3000 and localhost:8000. The former is the default port for React, which we will
use for our front-end in the next chapter. The latter is the default Django port.

### Tests

You should always write tests for your Django projects. A small amount of time spent upfront will
save you an enormous amount of time and effort later on debugging errors. Let’s add two basic
tests to confirm that the title and body content behave as expected.
Open up the todos/tests.py file and fill it with the following:

<div dir="ltr">

```python
# todos/tests.py
from django.test import TestCase
from .models import Todo

class TodoModelTest(TestCase):
    @classmethod
    def setUpTestData(cls):
        Todo.objects.create(title='first todo', body='a body here')
    
    def test_title_content(self):
        todo = Todo.objects.get(id=1)
        expected_object_name = f'{todo.title}'
        self.assertEqual(expected_object_name, 'first todo')

    def test_body_content(self):
        todo = Todo.objects.get(id=1)
        expected_object_name = f'{todo.body}'
        self.assertEqual(expected_object_name, 'a body here')
```

</div>

This uses Django’s built-in TestCase45 class. First we set up our data in setUpTestData and then
write two new tests. Then run the tests with the python manage.py test command.

<div dir="ltr">

```shell
(backend) $ python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..
----------------------------------------------------------------------
Ran 2 tests in 0.002s
OK
Destroying test database for alias 'default'...
```

</div>

And that’s it! Our back-end is now complete. Make sure the server is running as we’ll be using it
in the next chapter.

<div dir="ltr">

```shell
(backend) $ python manage.py runserver
```

</div>

### Conclusion

With a minimal amount of code Django REST Framework has allowed us to create a Django API
from scratch. The only pieces we needed from traditional Django was a models.py file and our
urls.py routes. The views.py and serializers.py files were entirely Django REST Framework
specific.

Unlike our example in the previous chapter, we did not build out any web pages for this project
since our goal was just to create an API. However at any point in the future, we easily could!
It would just require adding a new view, URL, and a template to expose our existing database
model.

An important point in this example is that we added CORS headers and explicitly set only the
domains localhost:3000 and localhost:8000 to have access to our API. Correctly setting CORS
headers is an easy thing to be confused about when you first start building APIs.
There’s much more configuration we can and will do later on but at the end of the day creating
Django APIs is about making a model, writing some URL routes, and then adding a little bit of
magic provided by Django REST Framework’s serializers and views In the next chapter we will build a React front-end and connect it to our Todo API backend.

</div>