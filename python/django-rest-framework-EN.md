
  
  
# Django Rest Framework Tutorial
<img src="https://static.djangoproject.com/img/logos/django-logo-negative.png"    
alt="Markdown Monster icon"    
style="float: left; margin-right: 10px;" />  
  
    
*Disclaimer: this tutorial is aimed to help you quickly start your Django project with Pycharm. Pretty much everything is transferable to the IDE of your choice, except a little section on the server configuration that might be different  depending on the tool you are using.*  

## Installation  
  
Let's step right in! We are going to start an empty Pycharm project and install **Django** and **rest-framework** in our environment. Usually Pycharm is automagically setting you up a virtual environment. It is not an absolute necessity but as for any Python project, using a venv is highly recommended as it will greatly help to encapsulate the project dependencies.
  
```python  
$ pip install django  
$ pip install djangorestframework  
```  
  
 Next, we will make sure that those dependencies are saved in a `requirements.txt` file.
  
```python  
$ pip freeze > requirements.txt  
```  
  
  
## Setup  
  
#### Create your Django project 
  
  Let's now ask Django to help us setup a project (don't forget the " . " at the end).
```python  
$ django-admin startproject <PROJECT_NAME> .  
```  
  
  
#### Create an app
  
You can now enter the project directory that Django created for you and then ask Django to create an app. 
```python  
$ cd <PROJECT_NAME>  
$ django-admin startapp <APP_NAME>  
```  
  
#### Point to Django where to look at...  
  
  In the file *tutorial/settings.py*, let's add the "rest_framework" and the newly created app in the list of `INSTALLED_APPS`.
  
Example:  
```python  
# tutorial/settings.py  
  
INSTALLED_APPS = [  
	'django.contrib.admin',
	'django.contrib.auth',
	'django.contrib.contenttypes',
	'django.contrib.sessions',
	'django.contrib.messages',
	'django.contrib.staticfiles',
 -> 'rest_framework',
 -> '<PROJECT_NAME>.<APP_NAME>
 ]  
  
```  
  
#### First migration  
  
```python  
$ python manage.py migrate  
```  
  
#### Create your super user 
  
```python  
$ python manage.py createsuperuser  
```  
  
  
## To use a PostgreSQL database
  
  You first need to install the psycopg2 library.
```  
$ pip install psycopg2  
```  
  
If it doesn't work, you can always install the binary version with:  
```  
$ pip3 install psycopg2-binary  
```  
  
Lastly, you need to tell Django which database you want to use in the *tutorial/settings.py* file.   
  
```python  
# tutorial/settings.py  
  
DATABASES = {    
	'default': {  
		'ENGINE': 	'django.db.backends.postgresql_psycopg2',
		'NAME': 	'<DATABASE_NAME>',
		'USER': 	'<USER>',
		'PASSWORD': '<PASSWORD>',
		'HOST':		'<SERVER>',
		'PORT': 	'<PORT>'
	}
}   
```  
---  
##  Pycharm  Configuration
In order to run the Django server with Pycharm: <br>  
1) Edit configurations <br>  
2) (+) Add new configuration <br>  
3) Django server <br>  
4) In environment variables: `PYTHONUNBUFFERED=1;DJANGO_SETTINGS_MODULE=<PROJECT_NAME>.settings`
5) Apply  
  
When you are going to run it for the first time, Pycharm is going to ask us if we want to "Enable Django Support". We toggle yes and we specify where the project root is (the folder containing the */manage.py* file) and then specify where the *<PROJECT_NAME>/settings.py* file can be found.
  
We can also set this up with the preferences tab:  
`Preferences \> Languages & Frameworks \> Django`  
  
  
---  
---  
---  
  
# A quick example of a working API 
  
<img src="https://i.pinimg.com/originals/69/37/cd/6937cd0be7744386e9c5ac84ce0e1d9a.jpg"    
alt="Markdown Monster icon"    
style="float: left; margin-right: 10px;" />  
  
So here we're going to presume we've already created a project named *tutorial* and an app named *country*.
  
## URLs  
  
### Let's define our first endpoint!

We are going to add our very first endpoint to the list of url patterns found in the *tutorial/urls.py* file. Here, we will simply take the root of the endpoint.

```python  
# tutorial/urls.py  
  
from django.contrib import admin
from django.urls import path, include    

urlpatterns = [
	path('admin/', admin.site.urls),  
	path('api/country', include('tutorial.country.urls'))
]  
  
```  
  
### Let's create the url file for our endpoint

We can now create a new file named *tutorial/country/urls.py* and add a *urlpatterns* array that we will fill later on. 

```python  
# tutorial/country/urls.py  
  
from django.urls import path, include    
 urlpatterns = [    
 ]  

```

## Models

> *"A model is the single, definitive source of information about your data. It contains the essential fields and behaviors of the data you’re storing. Generally, each model maps to a single database table."*

### Create your model

You can now create the model you want. A model is where you define the property (attributes) of the class (SQL table). Let's define the attributes of our Country table.

```python
# tutorial/country/models.py

from django.db import models    

class Country(models.Model):    
    name = models.CharField(max_length=60)    
    continent = models.CharField(max_length=50)    
    population = models.IntegerField()
    
```  
  
### Migrations  
  
Now that our models are done, let's apply the migrations to update all those tables definitions in our database. 
   
```  
$ python manage.py makemigrations  
$ python manage.py migrate  
```  
  
### Deploy  

At this point, we can run the server with Pycharm or simply use the terminal to do so.

```python  
$ python manage.py runserver  
```  

Open a browser, enter the URI with the slug `/admin` and enter your super user credentials to connect to the admin page. If you want to see the model you just created, in *tutorial/country/admin.py* just add:

```python  
from django.contrib import admin
from .models import Country

admin.site.register(Country)  
  
```  
  

## Serializers

> *"Serializers allow complex data such as querysets and model instances to be converted to native Python datatypes that can then be easily rendered into `JSON`, `XML` or other content types. Serializers also provide deserialization, allowing parsed data to be converted back into complex types, after first validating the incoming data."*  

We will create the rules that we want to apply to the data of our model. Let's create the file *tutorial/country/serializers.py* so we call tell to Django how to serialize and deserialize our Country class. The heritage of ModelSerializer is useful if we don't want to define the whole serializer ourselves.

```python  
# tutorial/country/serializers.py  
  
from rest_framework import serializers
from country.models import Country    

class CountrySerializer(serializers.ModelSerializer):    
	class Meta:    
        model = Country    
        fields = ("name", "continent", "population")

```  
  
  
## Views  

We can create our view by simply declaring it's related query set and serializer class.

```python  
# tutorial/country/views.py  
  
from rest_framework import viewsets
from country.models import Country
from country.serializers import CountrySerializer    
    
class CountryView(viewsets.ModelViewSet):    
	queryset = Country.objects.all()    
    serializer_class = CountrySerializer

```  
  
---  
  
  
## Routers  
  
> *"Resource routing allows you to quickly declare all of the common routes for a given resourceful controller. Instead of declaring separate routes for your index... a resourceful route declares them in a single line of code."*  
  
Now we have to associate the *Country* view to the endpoint's router.

```python  
# tutorial/country/urls.py

from django.urls import path, include
from tutorial.country import views
from rest_framework import routers
  
router = routers.DefaultRouter() router.register("country", views.CountryView)

urlpatterns = [
	path("", include(router.urls)
]

```  
  
  
---  
  
## References  
  
Most of the informations contained in this tutorial are coming from this really good video:
  
<a href="http://www.youtube.com/watch?feature=player_embedded&v=263xt_4mBNc" target="_blank">  
<img src="https://files.realpython.com/media/Screenshot_2018-12-09_at_17.58.16.20be0c5d3f1e.png"   
alt="IMAGE ALT TEXT HERE" width="240" height="180" border="10" /></a></div>
