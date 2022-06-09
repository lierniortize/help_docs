# SIMPY

## EJEMPLO 1

```
import simpy

def alarm(env):
    yield env.timeout(10)
    print("time to wake up!)
```

Ese es un proceso. 

`env` es el `enviroment` de simpy. Controla en qué punto del tiempo está la simulacion. Los procesos pertenecen a un `env`.

`yield` puede ser leído en este contexto como "wait for", (es parecido a un `await`). Este código, solamente imprimirá "time to wake up!". Los 10 que decimos que le espere son abstractos. Un dato que se guardará en el `enviroment`, pero no se relacionan con tiempo real.

## EJEMPLO 2

```
import simpy

def alarm(env):
    yield env.timeout(10)
    print("time to wake up!")

def alarm_2(env):
    yield env.timeout(200)
    print("wake up again!")

env = simpy.Environment()
env.process(alarm(env))
env.process(alarm_2(env))

env.run()
```

Da igual en que orden llamemos a la alarma 1 o 2. Siempre se imprimirá primero la de la 1 y luego la de la 2. Él va contando por dentro y sabe que la alarma uno tiene que ejecutarse a las 10 unidades y la 2 a las 200 unidades.

## EJEMPLO DE UN BAR

```
import simpy

def barista(env):
    TIME_TO_MAKE_COFEE = 5
    while True:
        yield wait_for_customer()
        yield env.tiemout(TIME_TO_MAKE_COFFE)
```

El barista espera indefinidamente para un cliente, cuando le llega hace el café en 5 unidades de tiempo. 

#### RESOURCES



##### `get()`

Coger de un recurso

##### `put()`

Añadir a un recurso

##### Tipos de recurso

* Semáforos
* Suministro homogéneo
* Bolsa de objetos

####

```
import simpy

def barista(env):
    TIME_TO_MAKE_COFEE = 5
    while True:
        # Request 1 customer (to make the coffee for)
        yield customer_resource.get(1)
        # Make the coffee
        yield env.tiemout(TIME_TO_MAKE_COFFE)

def arrivals(env, customer_resource, arrival_times):
    for arrival_time in arrival_times:
        # Wait until the next arrival time
        yield env.timeout(arrival_time - env.now)
        # Add a customer
        yield customer_resource.put(1)

env = simpy.Enviroment()
# I've created a LogginWrapper to log when the resource level changes
customer_resource = LogginWrapper(simpy.container(env))

env.process(arrivals(env, cursomer_resource, arrival_times))
env.process(arista(env, customer_resource))

env.run(until=8 * 68)


```