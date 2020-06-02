

# Tutorial Django
<img src="https://static.djangoproject.com/img/logos/django-logo-negative.png"  
alt="Markdown Monster icon"  
style="float: left; margin-right: 10px;" />

## Installation

Pour commencer, on doit partir un projet Pycharm vide et s'assurer
que l'on a **Django** et **rest_framework** sur la machine.

```python
$ pip install django
$ pip install djangorestframework
```

Ensuite, on s'assure que les dépendances sont persistées
dans le fichier `requirements.txt`.

```python
$ pip freeze > requirements.txt
```


## Setup

#### Créer un projet

On demande à Django de nous setupper un projet (Ne pas oublier le ' . ' à la fin) .
```python
$ django-admin startproject <NOM_PROJET> .
```


#### Créer une app

On entre dans le projet et on peut créer une app.
```python
$ cd <NOM_PROJET>
$ django-admin startapp <NOM_APP>
```

#### Dire à Django quoi regarder...

Dans le fichier *tutorial/settings.py*, on ajoute 'rest_framework' et le nom de l'app créée dans la liste de `INSTALLED_APPS`. 

Exemple:
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
 -> '<NOM_PROJET>.<NOM_APP>'
]

```

#### Première migration

```python
$ python manage.py migrate
```

#### Création d'un super user

```python
$ python manage.py createsuperuser
```


## Utilisation de PostgreSQL

Il faut installer la librairie psycopg2.
```
$ pip install psycopg2
```

Si ça ne fonctionne pas, on peut installer le binaire avec:
```
$ pip3 install psycopg2-binary
```

Ensuite, il faut dire à Django quelle base de données utiliser dans le fichier *tutorial/settings.py*.

```python
# tutorial/settings.py

DATABASES = {  
	'default': {
		'ENGINE': 'django.db.backends.postgresql_psycopg2',
		'NAME': '<DATABASE_NAME>',
		'USER': '<USER>',
		'PASSWORD': '<PASSWORD>',
		'HOST': '<SERVER>',
		'PORT': '<PORT>'
	}  
} 

```
---
## Configuration Pycharm
Pour pouvoir lancer le serveur avec Pycharm: <br>
1) Edit configurations <br>
2) (+) Add new configuration <br>
3) Django server <br>
4) Dans environment variables: `PYTHONUNBUFFERED=1;DJANGO_SETTINGS_MODULE=<NOM_PROJET>.settings` 
5) Apply

Quand on va le rouler pour la première fois, ça va nosu demander si on veut "Enable Django Support". On coche oui et on spécifie le root du projet (le dossier contenant le */manage.py*) et on lui spécifie où trouver le fichier *<NOM_PROJET>/settings.py*. 

On peut aussi passer par les préférences pour modifier tout ça:

`Preferences \> Languages & Frameworks \> Django`


---
---
---

# Exemple d'API

<img src="https://i.pinimg.com/originals/69/37/cd/6937cd0be7744386e9c5ac84ce0e1d9a.jpg"  
alt="Markdown Monster icon"  
style="float: left; margin-right: 10px;" />

Nous avons au préalable créé un projet nommé *tutorial* et une app nommée *country*.

## URLs

### URL de notre endpoint

On ajoute notre endpoint à la liste d'urlpatterns dans le fichier *tutorial/urls.py*. Dans notre cas, nous prendrons simplement le root du endpoint.
```python
# tutorial/urls.py

from django.contrib import admin  
from django.urls import path, include  
  
urlpatterns = [  
    path('admin/', admin.site.urls),
    path('api/country', include('tutorial.country.urls'))  
]

```


### Controlleur de notre endpoint
On peut ensuite créer un nouveau fichier *tutorial/country/urls.py* pour y créer une nouvelle liste de urlpatterns. 
```python
# tutorial/country/urls.py

from django.urls import path, include  
  
urlpatterns = [  
  
]

```


## Models

> *"A model is the single, definitive source of information about your data. It contains the essential fields and behaviors of the data you’re storing. Generally, each model maps to a single database table."*
### Création

On crée le modèle de ce qu'on veut représenter dans la base de données.

```python
# tutorial/country/models.py

from django.db import models  
    
class Country(models.Model):  
    name = models.CharField(max_length=60)  
    continent = models.CharField(max_length=50)  
    population = models.IntegerField()
    
```

### Migrations

Une fois les modèles créés, on doit appliquer les migrations.
 
```
$ python manage.py makemigrations
$ python manage.py migrate
```

### Déploiement

On peut ensuite partir le serveur par Pycharm ou entrer la commande.
```python
$ python manage.py runserver
```

On peut maintenant aller dans le browser, entrer l'uri `/admin` et se connecter pour voir la page d'administration. Pour pouvoir voir le model qu'on vient de migrer, dans `tutorial/country/admin.py:

```python
from django.contrib import admin  
from .models import Country  

admin.site.register(Country)

```



## Serializers
> *"Serializers allow complex data such as querysets and model instances to be converted to native Python datatypes that can then be easily rendered into `JSON`, `XML` or other content types. Serializers also provide deserialization, allowing parsed data to be converted back into complex types, after first validating the incoming data."*

On peut maintenant créer le fichier *tutorial/country/serializers.py* pour pouvoir dire à DjangoRest quoi sérialiser/désérialiser. L'héritage de ModelSerializer est utile si l'on ne veut pas avoir à définir tout le serializer nous-même.
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

On crée notre view en lui précisant son queryset et son serializer_class.
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

On doit maintenant associer la vue des *Country* au router du endpoint.
```python
# tutorial/country/urls.py
  
from django.urls import path, include  
from tutorial.country import views  
from rest_framework import routers  
  
router = routers.DefaultRouter()  
router.register("country", views.CountryView)  
  
urlpatterns = [  
    path("", include(router.urls))  
]
```


---

## Références

La plupart des éléments de ce `README` proviennent de ce vidéo:


<a href="http://www.youtube.com/watch?feature=player_embedded&v=263xt_4mBNc" target="_blank">
<img src="https://files.realpython.com/media/Screenshot_2018-12-09_at_17.58.16.20be0c5d3f1e.png" 
alt="IMAGE ALT TEXT HERE" width="240" height="180" border="10" /></a></div>

