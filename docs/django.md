# DJANGO

[Django web](https://www.djangoproject.com/)

## INICIAR

[Tutorial](https://docs.djangoproject.com/en/4.0/intro/tutorial01/)

```
cd <directory where I want to create the project>

django-admin startproject myproject
```

Esto creará un subdirectorio llamado `myproject` con la siguiente estructura:

```
myproject/
    manage.py
    myproject/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```


### Crear aplicaciones en el proyecto
Primero hay que entender la diferencia entre proyecto y aplicación. Una aplicación siempre será parte de nuestro proyecto. El proyecto podrá tener varias aplicaciones. Por ejemplo el proyecto de una tienda online puede tener la aplicación de gestionar el panel de control, el del stock de almacén, el de los pagos... Y estas aplicaciones podrán ser reutilizadas en otros proyectos.

Cada aplicación en Django consiste en un paquete de Python que se estructura de una cierta manera por convención. Para crear la primera app, colocarse en la carpeta donde está el archivo ´manage.py´ y ejecutar:

Crearemos la aplicación llamada `core`

```
py manage.py startapp core
```
Se creará la carpeta `core` de la aplicación con esta estructura:

```
| myproject/
|     core/
|          migrations/
|             __init__.py
|         __init__.py
|         admin.py
|        apps.py
|         models.py
|         tests.py
|         views.py
|     myproject/
|         __init__.py
|         settings.py
|         urls.py
|         asgi.py
|         wsgi.py
|     manage.py
    
```

Para decirle a Django que tenga en cuenta la aplicación tenemos que cambiar el archivo `myproject/settings.py`:
```
INSTALED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'core'
    ]
```

### Modelos

[Documentación modelos](https://docs.djangoproject.com/en/4.0/topics/db/models/)

Tendremos que crear un modelo para cada tabla que queramos crear en la base de datos a la que
estemos conectados (por defecto SQLite3). Este modelo nos especificará los campos que tendrá la tabla.

```
from django.db import models
from django.utils.translation import gettext_lazy as _

class Vehiculo(models.Model):
    TIPO_CHOICES = [
        ('camion', 'Camión'),
        ('coche', 'Coche'),
        ('furgoneta', 'Furgoneta'),
    ]
    matricula = models.CharField(max_length=7, verbose_name=_('Matrícula'))
    description = models.CharField(max_length=100, verbose_name=_('Descripción'))
    precio = models.FloatField(verbose_name=_('Precio'))
    tipo = models.CharField(max_length=30, verbose_name=_('Eleccion'), choices=TIPO_CHOICES)
    class Meta:
        ordering = ['pk']
        verbose_name = 'Vehiculo'
        verbose_name_plural = 'Vehiculos'
```

#### Bases de datos relacionales

Si queremos relacionar tablas entre ellas utilizaremos `ForeignKey`.
[ForeignKey_docs](https://docs.djangoproject.com/en/4.0/topics/db/examples/many_to_one/)

```
class Conductor(models.Model):
    nombre = models.CharField(max_length=100, verbose_name=_('Nombre'))
    vehiculo = models.ForeignKey(Vehiculo, on_delete=models.PROTECT, verbose_name=_('Vehículo'), related_name=' ')
    class Meta:
        verbose_name = 'Conductor'
        verbose_name_plural = 'Conductores'
```

Cada conductor tendrá un vehícuo asociado. Para acceder a los atributos del vehículo
se utilizará `__` (doble barra baja). Ejemplo: `vehiculo__matricula`

Una vez creados los modelos tenedremos que hacer las migraciones para que la base de datos 
sepa cómo son estos modelos. Primero crearemos los archivos de migración:

```
py manage.py makemigrations
```

Después Django leerá esos archivos de migración creados y hará la migración:

```
py manage.py migrate
```

!!! warning "Warning." Cada vez que se modifica un modelo hay que hacer una migración. Si no están todos los archivos 
`migration` Django no podrá realizar la migración.



## Views and Urls

Cada `view` nos enseñará algo de la base de datos y será capaz de enviar estos datos como
respuesta a un request del front. Hay diferentes tipos de views para ello (se explican más adelante).
Para hacer la petición a cada una de las views se utilizan las url-s. 

### URLS
Para ello tenemos que configurar dos archivos:

`myproject/urls.py`
```
from django.conf.urls import include
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path('core/', include('core.urls')),
    path('admin/', admin.site.urls),
]
```
En este archivo se le especifica a Djago que mire en `core/urls.py` para ver qué urls tiene que utilizar
para cada una de las views.

`core/urls.py`
```
from django.urls import path
from core.views import *

app_name = 'core'

"""
Dentro de la aplicación ``core`` tendremos las siguientes URL-s. Algunas solamente sirven para mostrar la plantilla
HTML y las otras son para obtener datos con el filtro deseado y mostrarlos.
"""

urlpatterns = [
    path('conductores/', FlujosView.as_view(), name='conductores'),
    path('vehiculos/', AlertasView.as_view(), name='vehiculos')
]
```
Hace la relación entre las url-s de petición y la view encargada de dar respuesta a esta.

### VIEWS

Este archivo estará en `core/views.py`.

Hay diferentes tipos de views. Ejemplo:
```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")

```
[Documentación de request response](https://docs.djangoproject.com/en/4.0/ref/request-response/)

#### View para cargar una plantilla HTML

Podremos crear documentos HTML y cargarlos mediante un `TemplateView`. Para ello colocaremos 
los archivos necesarios para el HTML en estos directorios:

```
| myproject/
|     core/
|         migrations/
|            0001_initial.py
|            0002_auto_20220609_1028.py
|         static/
|            core/
|                css/
|                img/
|                js/
|         templates/
|            index.html
|            vehiculos_page.html
|         __init__.py
|         admin.py
|         apps.py
|         models.py
|         tests.py
|         views.py
|     myproject/
|         __init__.py
|         settings.py
|         urls.py
|         asgi.py
|         wsgi.py
|     manage.py 

```

Y ya podremos cargar las views:

```
class indexView(TemplateView):
    template_name = 'core/index.html'
    
class vehiculosView(TemplateView):
    template_name = 'core/vehiculos_page.html'
```

De esta forma cuando llamemos a la url `hostServer:port\core\vehiculos`, se nos cargará el 
`html` vehiculos_page. En cambio, no podremos acceder al `indexView` porque no lo hemos especificado
en el archivo `urls.py`.

#### Views para filtrar y agregar datos

Nos podemos apoyar en DjangoREST para esto. Ejemplo de filtro y agragación (el ejemplo 
no sigue el modelo de vehiculo y conductor de antes):

```
class cajasListAPIView(ListAPIView):
    """
    Filtrará las fechas y el tipo de caja 
    en cada caso y hará la agrupación para sumar cajas. 
    """
    queryset = Link.objects.all()
    filter_backends = [DjangoFilterBackend]
    filter_fields = {
        'dia': ["in", "exact"],
        'tipo_caja': ["in","exact"]
    }

    def get(self, request, **kwargs):
        queryset = self.get_queryset()
        filter_queryset = self.filter_queryset(queryset)
        values = filter_queryset.values('dia','tipo_caja')\
            .annotate(n_cajas=Sum('cajas'))
        return Response(values)
```

Podemos hacer este tipo de peticiones a esta view:
```
`hostServer:port\core\cajas?dia__in=20220628,20220629&tipo_caja=3`
```
Esta petición nos devolverá un JSON donde por nos sumará la cantidad de cajas de tipo 3 
del 28 de abril o 29 de abril.

### Carga de datos a heroku

Primero hay que especificarle a Django que queremos trabajar con una base de datos de Heroku.

...

La carga se hará solo una vez de forma manual.
```
py manage.py shell
```

Una vez en shell ejecutamos el script destinado a cargar datos. La forma más facil de 
subir datos es creando una lista con objetos del modelo que queramos cargar, y cargar 
toda la lista a la vez. Por ejemplo, 
si tenemos todos los conductores (modelo Conductor) en un excel, primero tendremos
que subir los vehículos (porque los conductores tienen un vehiculo como atributo) y después 
coger el atributo que queramos para el conductor:

```
import pandas as pd
from core.models import Conductor, Vehiculo

df_vehiculos = pd.read_excel("vehiculos.xlsx")
df_conductores = pd.read_excel("conductores.xlsx")

# Creamos una lista llena de objetos ``Vehiculo``
l_vehiculos = [Vehiculo(
    matricula=row['matricula'],
    description=row['description'],
    precio=row['precio'],
    tipo=row['tipo'],
    )
    for i, row in df_conductores.iterrows()]

# Subimos la lista a Heroku
Vehiculo.objects.bulk_create(l_vehiculos)

# Creamos una lista llena de objetos ``Conductor``
l_conductores = [Conductor(
    nombre=row['nombre'],
    matricula=Nodo.objects.get(matricula=row['matricula']),
    )
    for i, row in df_conductores.iterrows()]

# Subimos la lista a Heroku
Conductor.objects.bulk_create(l_conductores)
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

En `core.admin.py`

```
from django.contrib import admin

from .models import Vehiculo

admin.site.register(Vehiculo)
```

Ahora en [http://127.0.0.1:8000/admin](http://127.0.0.1:8000/admin) ya podremos acceder y modificar campos.

## Interacción con el front (javascript)

Para hacer peticiones a Django tenemos que utilizar URL-s. Ejemplo:

```
var url = "vehiculos/"
var response = await fetch(url)
response = await answer.json()
```
Esto nos devolverá la respuesta en formato JSON. Si quisieramos hacer algún cambio
en alguno de los atributos podemos iterar sobre ese JSON. Por ejemplo queremos rebajar todos
los vehículos un 20%:
```
var vehiculos = response.map(function(vehc){
    vehc.precio = vehc.precio * 0.8
});
```

También podremos hacer llamadas simultáneas a diferentes url-s:

```
var urls = ['vehiculos\', 'conductores\'];
var promises = urls.map(url => fetch(url).then(y => y.text()));

Promise.all(promises).then(response => {
    console.log(response)
});
```
Ese response nos enseñará las diferentes respuestas de Django.