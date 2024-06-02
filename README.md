### Welcome

This is a guide to run the python app with postgres database in docker-compose.

All the files needed are in this repository.

First let's start a project.

``` bash
sudo docker-compose run web django-admin startproject composeexample .
```

Now we edit the **composeexample/settings.py** file to make sure the following content is there.

``` python
import os

AllowedHosts = ["*"]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('POSTGRES_NAME'),
        'USER': os.environ.get('POSTGRES_USER'),
        'PASSWORD': os.environ.get('POSTGRES_PASSWORD'),
        'HOST': 'db',
        'PORT': 5432,
    }
}
```

The we can issue the following command to bring up the environment.

``` bash
docker-compose up
```

We might aswell also want to create a superuser in our webapp.

``` bash
sudo docker compose run web python manage.py createsuperuser
```

Let's connect to the DB create a table and add some data.

``` bash
su - postgres
psql

CREATE TABLE IF NOT EXISTS Ceg 
(ID INTEGER PRIMARY KEY NOT NULL,
Nev TEXT NOT NULL,
Cim CHAR(100));

INSERT INTO ceg VALUES (1,'milka','hungary');
INSERT INTO ceg VALUES (2,'kobanyai','hungary');
```

Now we can create the **composeexample/models.py** file with the following content.

``` python
from django.db import models

class ceg(models.Model):
	id = models.IntegerField(primary_key = True)
	nev = models.CharField(max_length = 100)
	cim = models.CharField(max_length = 100)
	class Meta:
		app_label  = ''
		db_table = 'ceg'
	def __str__(self):
		return self.nev
```

Let's also crete the **composeexxample/views.py** aswell.

``` python
from django.http import JsonResponse
from django.core import serializers
from .models import ceg

def basic(request): 
    cegek = serializers.serialize("json", ceg.objects.all())
    return JsonResponse(cegek, safe=False)
```

Let's adjust our **composeexample/urls.py** file aswell.

``` python
from . import views
urlpatterns = [
    path('',views.basic, name = "basic"),
    path('admin/', admin.site.urls),
]
```
