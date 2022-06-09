# DJANGO

[Tutorial](https://docs.djangoproject.com/en/4.0/intro/tutorial01/)

```
cd Documents\Pruebak\django

django-admin startproject mysite
```

Esto creará un subdirectorio llamado `mysite` con la siguiente estructura:

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```

Después podemos hacer la migración, para que las aplicaciones que están en `settings.py` puedan hacer uso de la base de datos (por defecto SQLite3).

```
py manage.py migrate
```

Una vez realizado esto ya podemos ejecutar

```
py manage.py runserver {PORT}
```

Yendo a la dirección local dada veremos una pantalla de ¡Congratulations!

Primero hay que entender la diferencia entre proyecto y aplicación. Una aplicación siempre será parte de nuestro proyecto. El proyecto podrá tener varias aplicaciones. Por ejemplo el proyecto de una tienda online puede tener la aplicación de gestionar el panel de control, el del stock de almacén, el de los pagos... Y estas aplicaciones podrán ser reutilizadas en otros proyectos.


## La primera aplicación del proyecto

Cada aplicación en Django consiste en un apquete de Python que se estructura de una cierta manera por convención. Para crear la primera app, colocarse en la carpeta donde está el archivo ´manage.py´ y ejecutar:

Crearemos la aplicación llamada `polls`

```
py manage.py startapp polls
```
Se creará la carpeta `polls` de la aplicación con esta estructura:

```
polls/
     migrations/
        __init__.py

    __init__.py
    admin.py
    apps.py
    models.py
    tests.py
    views.py
```

## La primera view

En `polls/views.py`:

```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")

```
[Documentación de request response](https://docs.djangoproject.com/en/4.0/ref/request-response/)

En urls tenemos que hacer la conexión entre la página y nuestra función index.

En `polls/urls.py`:

```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

En `mysite/urls.py`:
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

Podemos crear un path diferente por cada vista que queramos. Es decir, podríamos crear una función nueva en `polls/views.py` y un nuevo path para ella en `polls/urls.py` y `mysite/urls.py`. 

## Modelos


La clase `model` creará tablas en nuestra base de datos y podremos modificarlos con los métodos de la clase `model`. No se puede crear un modelo que esté fuera de una aplicación.

Crearemos dos modelos: Question y Choice.

En `polls/models.py`:

```
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE) #Each Choice is related to a single Question
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

[Documentación](https://docs.djangoproject.com/en/4.0/topics/db/models/)

Cada clase será una tabla y cada field será un campo de esa tabla. 

Con este modelo Django es capaz de:

* Crear un 'database schema' para esta app
* Crear una API que deja a Python acceder a Question y Choice

### Activando modelos

Antes de empezar debemos decirle a nuestro proyecto que la app polls ha sido instalada. Para ello en `mysite/settings.py`

```
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

Y hacer la migración:

```
py manage.py makemigrations {NOMBRE_APLICACIÓN}
```

Con `makemigrations` le estamos diciendo a Django que hemos hecho algún cambio en nuestro modelo.

Pero después tenemos que ejecutar el SQL que corresponde con esa migración:

```
py manage.py sqlmigrate {NOMBRE_APLICACIÓN}{NUMERO QUE NOS HA DADO MAKEMIGRATIONS}
```

Por último tenemos que decirle que utilice ese código SQL para modificar las tablas en la base de datos. Creará un campo `id` en cada tabla, que la usará como clave primaria. 

```
py manage.py migrate
```

### Modificar bases de datos

```
py manage.py shell
```

Una vez en shell:

##### Insertar variable en base de datos

```
from django.utils import timezone


q = Question(question_text='¿Qué hay?', pub_date=timezone.now())
q.save()
```

##### Consultar campos de esa variable

```
q.id  # 1
q.question_text  #'Qué hay?'
q.pub_date  #datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

```

##### Cambiar valor de un campo

```
q.question_text = '¿Otra pregunta?'
q.save()
```

##### Ver toda la base de datos

```
Question.objects.all()  #<QuerySet [<Question: Question object (1)>]>
```

## Django admin

```
python manage.py createsuperuser
```

```
Username: admin
Email address: admin@example.com
Password: **********
Password (again): *********
Superuser created successfully.
```

En `polls.admin.py`

```
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

Ahora en [http://127.0.0.1:8000/admin](http://127.0.0.1:8000/admin) ya podremos acceder y modificar campos.

## Templates

Nuestros documentos `html` por convención deberían estar en el directorio `miAplicacion/templates/miAplicacion`

