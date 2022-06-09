# REDIS
[redis-py](https://redis-py.readthedocs.io/en/stable/)

[https://try.redis.io/](Pruebak iteko)



## BASIC DATA STRUCTURES

### Keys and Expiration

[Documentation](https://redis.io/commands#generic)

Keys are the primary way to access data values. Hay 16 databases, por defecto siempre se utilizará database0. 

EJEMPLO (por convención, pero se puede hacer de otra forma):

```
"user:1000:followers"
```
* user: nombre del objeto
* 1000: identificador único de la instancia
* followers: objeto compuesto

### `SET`

Para asignar un valor a una llave. 

```
SET customer:1000 "fred"
```
En nuestra llave (cliente de ID 1000) `customer:1000`estará guardado el valor ` "fred" `

Si hacemos `SET` a una llave que no existe la crearemos. si no queremos que esto suceda podemos utilizar `NX`

```
SET provider:100 "freddy" NX
```
Nos retornará `OK` si no existe y nos devolverá `NIL` si existe.

### `GET`
Para obtener el valor que tiene guardada la llave `customer:1000`

```
GET customer:1000 => fred
```

En cambio, si queremos que la llave exista antes de hacer `SET` utilizaremos `XX` de la misma manera.

### `KEYS` and `SCAN`

`KEYS` itera sobre todas las llaves, para ver si existe la que le hemos pedido. Si la base de datos es grande podría tardar demasiado. 
`SCAN` itera usando un cursor y devuelve un espacio de referencia. Está bien para usarlo en producción.

EJEMPLOS:

KEYS
Todos los clientes cuyo ID empiezan con 1:
```
keys customer:1* => customer:1500
                    customer:1000

```

SCAN
```
SCAN slot [MATCH pattern][COUNT count]
```
```
scan 0 MATCH customer: 1* => 1) 14336
                             2) empty list or set

scan 14336 MATCH customer:1* => 1) 14848
                                2) empty list or set

scan 14848 MATCH customer:1* COUNT 10000 => 1) 1229
                                            2) 1)customer:1500
                                               2)customer:1000
scan 1229 MATCH customer:1* => 1) 0
                               2) empty list or set

```
Cuando aparece 0 significa que no hay nada más sobre lo que iterar.
...

### Borrar llaves

### `DEL`
Para borrar una llave y el valor asociado a ella o desvincular su llave con el valor.
```
DEL key[key...]
UNLINK key[key]
```

### `EXISTS`
```
EXISTS server:name => 1 (existe)
EXISTS server:blabla => 0 (no existe)
```


Es una base de datos que funciona por llave-valor. Un valor siempre se guardará dentro de una llave y la única forma de obtener dicho valor es tener la llave.

Se explicarán los comandos más básicos a continuación, pero la lista completa esta [aquí](https://redis.io/commands)


### `INCR` y `INCRBY`
Incrementan el valor de una llave dada. `INCR` lo incrementa por 1 y `INCRBY` lo incrementa por el valor que nosotros le proporcionemos.

Son operaciones atómicas. 

```
SET connections 10
INCR connections
GET connections => 11
```

```
SET connections 10
INCRBY connections 100
GET connections => 110
```
### `DECR` y `DECRBY`
Decrementan el valor de una llave dada. `INCR` lo incrementa por 1 y `INCRBY` lo incrementa por el valor que nosotros le proporcionemos.

### `EXPIRE` y `TTL`
Para decir que una llave tiene que existir solo para un tiempo especificado en segundos.
```
SET resource:lock "Redis Demo"
EXPIRE resource:lock 120
```

Con `TTL` podremos mirar cuento le queda a una llave:
```
TTL resource:lock => 113
```
Si TTL es -1 significa que esa llave es para siempre. 

### LISTAS



## PYTHON

redis-py library

```
pip install redis
```
### Connect to redis:

```
import os

import pytest
import redis

USERNAME = os.environ.get('REDISOLAR_REDIS_USERNAME')
PASSWORD = os.environ.get('REDISOLAR_REDIS_PASSWORD')


@pytest.fixture
def redis_connection(app):
    client_kwargs = {
        "host": app.config['REDIS_HOST'],
        "port": app.config['REDIS_PORT'],
        "decode_responses": True
    }

    if USERNAME:
        client_kwargs["username"] = USERNAME
    if PASSWORD:
        client_kwargs["password"] = PASSWORD

    yield redis.Redis(**client_kwargs)


def test_say_hello(redis_connection):
    result = redis_connection.set("hello", "world")
    value = redis_connection.get("hello")
    assert result is True
    assert value == "world"
```

### Redis Clients

Gestiona conexiones, implementa el protocolo redis (RESP) y nos deja usar un lenguaje sencillo (GET, SET, INCR...).

[Walrus](https://walrus.readthedocs.io/en/latest/)

[redis-py](https://redis-py.readthedocs.io/en/stable/)

Utilizaremos `redis-py` como nuestra librería cliente.

#### Cómo conectarse a redis utilizando `redis-py`

```
import redis
from redis.sentinel import Sentinel
from rediscluster import RedisCluster


def connection_examples():
    # Connect to a standard Redis deployment.
    client = redis.Redis("localhost", port=6379, decode_responses=True,
                         max_connections=20)

    # Read and write through the client.
    client.set("foo", "bar")
    client.get("foo")
```

#### Basic operations

Redys type > Python type:

* String > bytes or string
* List > list of bytes or strings
* Set > set of bytes or strings
* Hash > dictionary

```
PLANETS = [
    "Mercury", "Mercury", "Venus", "Earth", "Earth", "Mars", "Jupiter", "Saturn",
    "Uranus", "Neptune", "Pluto"
]

EARTH_KEY = "earth"

## LISTS

def test_redis_list(redis, key_schema):
    key = key_schema.planets_list_key()

    assert len(PLANETS) == 11

    # Add all test planets to a Redis list
    result = redis.rpush(key, *PLANETS)

    # Check that the length of the list in Redis is the same
    assert result == len(PLANETS)

    # Get the planets from the list
    # Note: LRANGE is an O(n) command. Be careful running this command
    # with large lists.
    planets = redis.lrange(key, 0, -1)
    assert planets == PLANETS

    # Remove the elements that we know are duplicates
    # Note: O(n) operation.
    redis.lrem(key, 1, "Mercury")
    redis.lrem(key, 1, "Earth")

    planet = redis.rpop(key)
    assert planet == "Pluto"

    assert redis.llen(key) == 8

## SETS

def test_redis_set(redis, key_schema):
    key = key_schema.planets_set_key()

    # Add planets to a Redis set
    redis.sadd(key, *PLANETS)

    # Return the cardinality of the set
    assert redis.scard(key) == 9

    # Fetch all values from the set
    # Note: SMEMBERS is an O(n) command. Be careful running this command
    # with high-cardinality sets. Consider SSCAN as an alternative.
    assert redis.smembers(key) == set(PLANETS)

    # Pluto is, of course, no longer a first-class planet. Remove it.
    response = redis.srem(key, "Pluto")
    assert response == 1

    # Now we have 8 planets, as expected.
    assert redis.scard(key) == 8

## HASHES

def test_redis_hash(redis):
    earth_properties = {
        "diameter_km": "12756",
        "day_length_hrs": "24",
        "mean_temp_c": "15",
        "moon_count": "1"
    }

    # Set the fields of the hash.
    redis.hset(EARTH_KEY, mapping=earth_properties)

    # Get the hash we just created back from Redis.
    stored_properties = redis.hgetall(EARTH_KEY)
    assert stored_properties == earth_properties

    # Test that we can get a single property.
    assert redis.hget(EARTH_KEY, "diameter_km") == earth_properties["diameter_km"]
```
CUIDADO!! Algunos métodos pueden ser lentos. Para muchos datos, hay que utilizar los métodos de la familia SCAN.



### DAOs



Get started with redis cloud:

Create a subscription and a database. To connect with the database we need:

* `public endpoint` for the host and port
* `password` for the authorization

First code:

```
import redis

redis = redis.Redis(
     host='redis-16018.c228.us-central1-1.gce.cloud.redislabs.com',
     port='16018',
     password='AAa6DTbT7c0kYoZuVDM3mLTrEuGMinDt')

redis.set('mykey', 'Hello from Python!')
value = redis.get('mykey')
print(value)

redis.zadd('vehicles', {'car': 0})
redis.zadd('vehicles', {'bike': 0})
vehicles = redis.zrange('vehicles', 0, -1)
print(vehicles)
```
Output:
```
b'Hello from Python!'
[b'bike', b'car']
```

## REDISEARCH

[Tutorial](https://developer.redis.com/howtos/redisearch/?s=redise)

[Aggregations tutorial](https://github.com/RediSearch/redisearch-py#aggregations)

[RediSearch en python](https://github.com/RediSearch/redisearch-py)

Redisearch es una herramienta de búsqueda. 

```
pip install redisearch
```

Crear una instancia cliente. [tutorial](https://faun.pub/redisearch-using-python-client-3581309e3475)

```
from redisearch import Client

client = Client("my-index")
```

[Client Python API](https://d128ysc22mu7qe.cloudfront.net/python_client/)

```
from redis import ResponseError
from redisearch import Client, IndexDefinition, TextField

SCHEMA = (
    TextField("title", weight=5.0),
    TextField("body")
)

client = Client("my-index")

definition = IndexDefinition(prefix=['blog:'])

try:
    client.info()
except ResponseError:
    # Index does not exist. We need to create it!
    client.create_index(SCHEMA, definition=definition)
```


```
from redisearch import Client, TextField, NumericField, Query

# Creating a client with a given index name
client = Client('myIndex')

# Creating the index definition and schema
client.create_index([TextField('title', weight=5.0), TextField('body')])

# Indexing a document
client.add_document('doc1', title = 'RediSearch', body = 'Redisearch implements a search engine on top of redis')

# Simple search
res = client.search("search engine")

# the result has the total number of results, and a list of documents
print res.total # "1"
print res.docs[0].title

# Searching with snippets
res = client.search("search engine", snippet_sizes = {'body': 50})

# Searching with complex parameters:
q = Query("search engine").verbatim().no_content().paging(0,5)
res = client.search(q)
```

### QUERYING

#### Finding exact string matches

## REDIS OM

[GitHub](https://github.com/redis/redis-om-python)

[Yotube tutorial](https://www.youtube.com/watch?v=DFNKmbGKa5w)

[Youtube tutorial](https://www.youtube.com/watch?v=UhnEyMDWuyI)

[Curso de redis](https://university.redis.com/courses/ru203/)

EJEMPLO:
Vamos a montar un refugio de animales y tenemos un csv con los siguientes campos: `name, species, age, weight, sex, fee, children, other_animals, description`.

#### Creamos un Redis Model en OM:

```
from typing import Text
from redis_om import (Field, HashModel)

class Adoptable (HashModel):
    name: str = Field(index=True)
    species: str = Field(index=True)
    age: str = Field(index=True)
    weight: float = Field(index=True)
    sex: str = Field(index=True)
    fee: str = Field(index=True)
    children: str = Field(index=True)
    other_animals: str = Field(index=True)
    description: str = Field(index=True, full_text_search=True) #Le estamos diciendo a redis cómo queremos que lo indexe, ya que queremos ser capaces de hacer un 'full text search' (utilizará un query de redisearch)
```

Lo que estamos modelando se guardará como un hash en redis. Le decimos los campos (Field) que queremos que guarde y que queremos que los indexe. 

Lo indexará con redisearch y creará un índice para mantenimiento y búsqueda. De esta forma, cuando creemos instancias, `client` gestionará esto volviendolo a guardar en redis.

Cada animal se guardará asociado a un valor (tal y como funciona redis, con clave-valor), pero nosostros queremos ser capaces de hacer búsquedas dependiendo de algún campo (ej. queremos un perro que pese menos de 20kg). Reisearch nos ayudará en esto.

Si quisieramos modelar algo con campos anidados (que no sea clave-valor como en este caso) podríamos utilizar el modelado JSON de redis. Ambos son modelos `pedantic` (una librería de validación de python) por lo que podemos añadir criterios adicionales (que un campo sea un email, que un int esté entre valores que especifiquemos...). 

#### Cargar datos en redis

Creamos una base de datos en redis a partir de un CSV:

```
import csv
from adoptable import Adoptable
from redis_om impor Migrator

with open['animal_data.csv'] as csv_file:
    animail_reader = csv.DictReader(csv_file)

    for animal in animal_reader:
        adoptable = Adoptable(**animal)
        print(f"{animal[name]} has pk = {adoptable.pk}") # pk lo crea redis OM como clave primaria
        adoptable.save()

# Hacer la migración de datos
Migrator().run()
```

Con `pk` estamos creando claves que identifiquen a cada animal como único localmente antes de guardarlo en redis. 

Con `Migrator.run()` activamos algo parecido a "un detector de cambios". Así todo lo que hagamos estará registrado siempre. 

Podemos descargarnos [RedisInsight](https://redis.com/es/redis-enterprise/redisinsight/#insight-form) para interactuar con nuestra base de datos. Es una interfaz donde podemos ver y gestionar nuestros datos fácilmente. 

#### Querys

```
from adoptable import Adoptable

def show_result(results):
    for adoptable in results:
        print(adoptable)
        print("")

def find_by_name():
    print("find by name: ")
    return Adoptable.find(Adoptable.name == "Luna").all()

def find_male_dogs():
    print("find male dogs: ")
    return Adoptable.find((Adoptable.species == "dog") &
                         (Adoptable.sex == "m")
                         )

def find_dogs_in_age_range():
    print("find dogs in ages range: ")
    return Adoptable.find((Adoptable.species == "dog") &
                         (Adoptable.age < 8) &
                         (Adoptable.age < 11)
                         ).sort_by("age")


def find_cats_good_with_children():
    print("find cats good with children: ")
    return Adoptable.find((Adoptable.species == "cat") &
                            (Adoptable.children == "y") &
                            (Adoptable.description % "play") &
                            ~(Adoptable.description % "anxious") &
                            ~(Adoptable.description % "nervous")
                            )

show_results(find_by_name())
```

En la última función nos basamos en redisearch para las búsquedas. Si queremos buscar más allá de valores concretos, nos podemos fijar en la descripción. Al buscar con `%` lo que hacemos es buscar una palabra parecida a la que estamos buscando, incluyendo también esa palabra. En cambio, con el símbolo `~` lo que hacemos es excluir esa palabra y parecidas de nuestra búsqueda. 

## REDIS_OM y REDISEARCH

[Tutorial redisearch](https://developer.redis.com/howtos/redisearch/?s=redise)

[Aggregations tutorial redisearch](https://github.com/RediSearch/redisearch-py#aggregations)

[RediSearch en python](https://github.com/RediSearch/redisearch-py)

[GitHub Redis_OM](https://github.com/redis/redis-om-python)

[Yotube tutorial](https://www.youtube.com/watch?v=DFNKmbGKa5w)

[Youtube tutorial](https://www.youtube.com/watch?v=UhnEyMDWuyI)

[Curso de redis oficial](https://university.redis.com/courses/ru203/)



Vamos a manejar una base de datos con los siguientes campos: `firstName,lastName,salary,department,isAdmin`

### Iniciar Redis en Windows

Tres opciones:

1) WSL DE LINUX

2) CON DOCKER

* Instalar docker y preparamos un puerto localhost libre.
* Preparar un documento 'docker-compose.yaml'

```
    version: "3.9"
    services:
    redis:
        container_name: redis_om_python_demo
        image: "redislabs/redisearch:edge"
        ports:
        - 6379:6379
        deploy:
        replicas: 1
        restart_policy:
            condition: on-failure
```

* Instalamos los paquetes necesarios (algunos de ellos están en `requirements.txt`)
```
    pip install -r requirements.txt
```

* En otra terminal ejecutamos `docker-compose up` y habremos añadido el container que queríamos en docker.

3) REDIS CLOUD

* `set REDIS_OM_URL = redis://username:password@public-endpoint/dataset_name`
* [Blog explicado](https://www.mortensi.com/2021/12/connect-to-redis-with-python/)


#### RedisInsight

[Link: RedisInsight](https://redis.com/es/redis-enterprise/redisinsight/#insight-form)

Es una interfaz para ver qué está pasando. Traer la base de datos especificando `host` y `port`. Si estamos en cloud también tendremos que meter la contraseña. 

### Creamos un Redis Model en OM:

```
from typing import Text
from redis_om import (Field, HashModel)
from redisearch.client import TagField

class Employee(HashModel):
    firstName: str = Field(index=True)
    lastName: str = Field(index=True)
    salary: int = Field(index=True)
    department: str = Field(index=True)
    isAdmin: int = Field(index=True)
```

Lo que estamos modelando se guardará como un hash en redis. Le decimos los campos (Field) que queremos que guarde y que queremos que los indexe. 

Lo indexará con redisearch y creará un índice para mantenimiento y búsqueda. 

Cada persona se guardará asociado a un valor (tal y como funciona redis, con clave-valor), pero nosostros queremos ser capaces de hacer búsquedas dependiendo de algún campo. Reisearch nos ayudará en esto.

Si quisieramos modelar algo con campos anidados (que no sea clave-valor como en este caso) podríamos utilizar el modelado JSON de redis. Ambos son modelos `pedantic` (una librería de validación de python) por lo que podemos añadir criterios adicionales (que un campo sea un email, que un int esté entre valores que especifiquemos...). 

### Cargar datos en redis

Creamos una base de datos en redis a partir de un CSV. Es algo que ejecutaremos solamente una vez.

```
import csv

from employee import Employee
from redis_om import Migrator

with open('employee.csv') as csv_file:
    employees = csv.DictReader(csv_file)

    for employee in employees:
        emp = Employee(**employee)
        
        # print(f"{employee['firstName']} -> {emp.pk}") 
        emp.save()

# Create a RediSearch index
Migrator().run()
```

Con `pk` estamos creando claves que identifiquen a cada persona como único localmente, antes de guardarlo en redis. 

Con `Migrator().run()` activamos algo parecido a "un detector de cambios". Así todo lo que hagamos estará registrado siempre. 

Lo ejecutamos en la terminal:
```
py upload_employee.py
```

Podemos verificar que de verdad se ha subido en RedisInsight

#### Si queremos conectarnos con Redis Cloud

Ejecutamos en el cmd:
```
set REDIS_OM_URL = redis://username:password@public-endpoint/dataset_name
```
Y realizamos los pasos de arriba para cargar los datos.

Me ha dado error y lo hemos forzado en el código. Definimos la conexión a nuestra base de datos antes de importar lo demás.

```
import csv

import os

os.environ["REDIS_OM_URL"]="redis://default:CgdemOw59KKOuX4ZugQVaThJv738hBAf@redis-10742.c124.us-central1-1.gce.cloud.redislabs.com:10742/books"

from employee import Employee
from redis_om import Migrator

with open('employe.csv') as csv_file:
    employees = csv.DictReader(csv_file)

    for employee in employees:
        emp = Employee(**employee)

        print(f"{employee['firstName']} -> {emp.pk}")
        emp.save()

# Create a RediSearch index
Migrator().run()

```

### Queries

Podemos hacerlo con redisOM o redisearch.

#### Redisearch

Para Redisearch necesitamos definir un `client`.

```
from redisearch import Client, IndexDefinition, TextField, NumericFieldTagField, GeoField

def initializeClient():
    SCHEMA = (
        TextField("firstName"),
        TextField("lastName"),
        NumericField("salary"),
        TextField("department"),
        NumericField("isAdmin"),
        TagField("tag"),
        GeoField("location")
    )
    client = Client("myIndex")
    definition = IndexDefinition(prefix=[':employee.Employee:']) # lo que tienen en común antes del hash
    try: # No podemos cargar un cliente con el mismo index
        client.info()
    except ResponseError:
        # Index does not exist. We need to create it!
        client.create_index(SCHEMA, definition=definition)
    return client

client = initializeClient()
```

#### EJEMPLOS

##### Imports
```
from employee import Employee
from redis import ResponseError
from redisearch import Client, IndexDefinition, TextField, NumericField, TagField, GeoField
from redisearch import reducers
from redisearch.aggregation import AggregateRequest, Asc, Desc
```

##### Task 1 : find by first name
OM: 
```
def find_by_first_name():
    return Employee.find(Employee.firstName == 'ahmad').all() # search based on first name (could be done like this)
```
Redisearch: 
```
def find_by_first_name_redisearch(client):
    res =  client.search("@firstName:ahmad")
    return res
```

##### Task 2 : find by first name (autocompletando con redisearch)
``` 
def find_by_name_wildcard_redisearch(client):
    res =  client.search("@firstName:br*")
    for result in res.docs:
        print(result)
```

##### Task 3: Find by first and last name con OM
```
def find_by_first_and_last_name():
    return Employee.find((Employee.firstName == 'ahmad') & (Employee.lastName == 'bazzi')).all()
```

##### Task 4: Sort in ascending
OM: 
```
def sort_by_salary():
    return Employee.find(Employee.salary>0).sort_by("salary")
```
##### Task 5: Sort in descending with redisearch:
```
def sort_by_salary_redisearch_descending():
    request = AggregateRequest('*').group_by(['@salary','@firstName'], reducers.count().alias('count')).sort_by(Desc('@salary'))
    result = client.aggregate(request)
    for r in result.rows:   
        print(r)
```






