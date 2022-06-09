# ALGORITMO GENÉTICO

## PSEUDOCÓDIGO

* INICIACIÓN: Generación aleatoria de población inicial, estará constituida por un conjunto de cromosomas, los cuales representan las posibles soluciones del problema. Si no se hace aleatoriamente es importante tener diversidad. Sino se puede llegar a una convergencia prematura (un óptimo local).

* EVALUACIÓN: A cada cromosoa se le aplicará una función de aptitud para saber cómo de buena es esta solución. 

* CONDICIÓN DE TÉRMINO: Dos opciones:

    - Establecer un máximo de iteraciones
    - La población no cambia de iteración a iteración

    Pasos: 

    - Selección. después de saber la óptima aptitud de cada cromosoma, se eligen los que cruzaremos con la siguiente generación.
    - Cruzamiento. Se recombina.
    - Mutación. Modificar al azar parte del cromosoma de los individuos para explora nuevas opciones. 
    - Reemplazo. Seleccionar los mejores individuos para que conformen la siguiente generación. 

## EJEMPLO CON TSP (PYTHON)

[Link al ejemplo](https://towardsdatascience.com/evolution-of-a-salesman-a-complete-genetic-algorithm-tutorial-for-python-6fe5d2b3ca35)

Links de interés:
[Link2](http://www.theprojectspot.com/tutorial-post/applying-a-genetic-algorithm-to-the-travelling-salesman-problem/5)
[Link3](https://gist.github.com/turbofart/3428880)
[Link4](https://gist.github.com/NicolleLouis/d4f88d5bd566298d4279bcb69934f51d)
[Link5](https://en.wikipedia.org/wiki/Travelling_salesman_problem)

Se quiere buscar la ruta con menos distancia que pase por diferentes ciudades. Condiciones:
    
* Hay que pasar por todas las ciudades
* Hay que acabar en la ciudad donde se ha iniciado la ruta

### Conceptos

* GEN: Una ciudad
* CROMOSOMA (individuo): Una ruta que cuple las condiciones
* POBLACIÓN: Un conjunto de cromosomas (generación)
* PADRES: Dos rutas que se combinan para crear una nueva
* CONJUNTO DE CRUZADO: Conjunto de padres que será usado para crear la siguiente población
* FUNCIÓN DE APTITUD: Función que nos dice cuánto de buena es una ruta (en nuestro caso cuánto de corta)
* MUTACIÓN: Una froma de introducir variaciones en nuestra población cambiando dos ciudades de forma aleatoria
* ELITISMO: Una forma de llevar los mejores cromosomas a la siguiente generación

### Algoritmo

1) Crear población
2) Determinar aptitud
3) Seleccionar conjunto de cruzado
4) Cruzar
5) Mutación
6) Repetir

### Python

#### Paquetes

```
import numpy as np, random, operator, pandas as pd, matplotlib.pyplot as plt

```

#### Dos clases: City y Fitness

`City`:

```
class City:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def distance(self, city):
        xDis = abs(self.x - city.x)
        yDis = abs(self.y - city.y)
        distance = np.sqrt((xDis ** 2) + (yDis ** 2))
        return distance
    
    def __repr__(self):
        return "(" + str(self.x) + "," + str(self.y) + ")" #es solamente notación para ver más fácil las coordenadas de cada ciudad
```
`Fitness`:
```
class Fitness:
    def __init__(self, route):
        self.route = route
        self.distance = 0
        self.fitness= 0.0
    
    def routeDistance(self):
        if self.distance == 0:
            pathDistance = 0
            for i in range(0, len(self.route)):
                fromCity = self.route[i]
                toCity = None
                if i + 1 < len(self.route): #Poruqe la ruta empieza y acaba en el mismo sitio, esto cuenta el último tramo
                    toCity = self.route[i + 1]
                else:
                    toCity = self.route[0]
                pathDistance += fromCity.distance(toCity)
            self.distance = pathDistance
        return self.distance
    
    def routeFitness(self):
        if self.fitness == 0:
            self.fitness = 1 / float(self.routeDistance()) #Cuanto menor es la distancia mejor es la aptitud (mejor fitness)
        return self.fitness
```
#### Creamos población

Creamos una ruta de forma aleatoria:

```
def createRoute(cityList):
    route = random.sample(cityList, len(cityList))
    return route
```

Como queremos una población creamos muchas rutas:

```
def initialPopulation(popSize, cityList):
    population = []

    for i in range(0, popSize):
        population.append(createRoute(cityList))
    return population
```

ESTO SÓLO SE REALIZARÁ PARA LA POBLACIÓN INICIAL. A PARTIR DE AQUÍ CREAREMOS NUEVAS GENERACIONES MEDIANTE CRUZADO Y MUTACIÓN

#### Determinamos aptitud

De aquí sacaremos una lista ordenada con el ID de ruta y la aptitud asociada a esta.

```
def rankRoutes(population):
    fitnessResults = {}
    for i in range(0,len(population)):
        fitnessResults[i] = Fitness(population[i]).routeFitness()
    return sorted(fitnessResults.items(), key = operator.itemgetter(1), reverse = True)

```

#### Selecionamos el conjunto de cruzado

Tenemos dos formas diferentes para seleccionar a los padres que usaremos para crear las siguientes generaciones:

* Proporcional a la aptitud: La aptitud de cada individuo relativo a la población es usado para determinar la probabilidad de selección. Podemos llamarlo probabilidad por peso de aptitud (en el ejemplo haremos esto).
* Selección por campeonato: Se selecciona aleatoriamente un conjunto de individuos. Y el que mayor aptitud tenga se selecciona como el primero de los padres. Repetimos el proceso para elegir el segundo padre. 

También usaremos el elitismo. Es decir, los mejores individuos de la población pasarán automáticamente a la siguiente generación. 

Es decir, crearemos el conjunto de cruzado en dos pasos: 

* Usaremos el output de `rankRoute` para saber cuales usamos en la siguiente función

```
def selection(popRanked, eliteSize):
    selectionResults = []
    df = pd.DataFrame(np.array(popRanked), columns=["Index","Fitness"]) #Calculamos el peso relativo de la aptitud
    df['cum_sum'] = df.Fitness.cumsum()
    df['cum_perc'] = 100*df.cum_sum/df.Fitness.sum()
    
    for i in range(0, eliteSize): #Mantenemos en la selección las rutas élite
        selectionResults.append(popRanked[i][0])
    for i in range(0, len(popRanked) - eliteSize): #Comparamos pesos relativos de aptitud de rutas aleatorias para seleccionar los mejores
        pick = 100*random.random()
        for i in range(0, len(popRanked)):
            if pick <= df.iat[i,3]:
                selectionResults.append(popRanked[i][0])
                break
    return selectionResults #Una lista con IDs de rutas seleccionadas para crear el conjunto de cruzado
```
Creamos el conjunto de cruzado: 

```
def matingPool(population, selectionResults):
    matingpool = []
    for i in range(0, len(selectionResults)):
        index = selectionResults[i]
        matingpool.append(population[index])
    return matingpool
```

#### Cruce

Como todas las ciudades tienen que aparecer exactamente una vez, utilizaremos una función que llamaremos `ordered crossover`. Seleccionaremos una subsecuecia de uno de los padres, y rellenaremos los huecos con el segundo en el mismo orden de aparición pero teniendo en cuenta de que no se pueden repetir.

Ejemplo de `ordered crossover`

```
PADRE1: 1|2|3|4|5|6|7|8|9
PADRE2: 9|8|7|6|5|4|3|2|1

Subsecuencia: ·|·|·|·|·|6|7|8|·

Cruzado: 9|5|4|3|2|6|7|8|1
                  -------
```

La función de cruzado:

```
def breed(parent1, parent2):
    child = []
    childP1 = []
    childP2 = []
    
    geneA = int(random.random() * len(parent1))
    geneB = int(random.random() * len(parent1))
    
    startGene = min(geneA, geneB)
    endGene = max(geneA, geneB)

    for i in range(startGene, endGene): #subsecuencia
        childP1.append(parent1[i])
        
    childP2 = [item for item in parent2 if item not in childP1] #rellenar

    child = childP1 + childP2
    return child
```
#### Creación de nueva población


```
def breedPopulation(matingpool, eliteSize):
    children = []
    length = len(matingpool) - eliteSize
    pool = random.sample(matingpool, len(matingpool))

    for i in range(0,eliteSize): #seguimos queriendo mantener las rutas élite
        children.append(matingpool[i])
    
    for i in range(0, length): #usamos la función de cruzar para rellenar la generación
        child = breed(pool[i], pool[len(matingpool)-i-1])
        children.append(child)
    return children
```

#### Mutación

La mutación cumple una función importante en AG, ya que ayuda a evitar la convergencia local mediante la introducción de rutas novedosas que nos permitirán explorar otras partes del espacio de solución.

Utilizaremos `mutacion por intercambio`. Lo haremos de la siguiente forma: en un individuo de poca probabilidad intercambiaremos dos ciudades.

```
def mutate(individual, mutationRate):
    for swapped in range(len(individual)):
        if(random.random() < mutationRate):
            swapWith = int(random.random() * len(individual))
            
            city1 = individual[swapped]
            city2 = individual[swapWith]
            
            individual[swapped] = city2
            individual[swapWith] = city1
    return individual
```

Añadimos esas rutas mutadas a la nueva generación:

```
def mutatePopulation(population, mutationRate):
    mutatedPop = []
    
    for ind in range(0, len(population)):
        mutatedInd = mutate(population[ind], mutationRate)
        mutatedPop.append(mutatedInd)
    return mutatedPop
```

#### Repetición

Unimos todo en recursividad:

```
def nextGeneration(currentGen, eliteSize, mutationRate):
    popRanked = rankRoutes(currentGen)
    selectionResults = selection(popRanked, eliteSize)
    matingpool = matingPool(currentGen, selectionResults)
    children = breedPopulation(matingpool, eliteSize)
    nextGeneration = mutatePopulation(children, mutationRate)
    return nextGeneration
```

#### Puesta en marcha

```
def geneticAlgorithm(population, popSize, eliteSize, mutationRate, generations):
    pop = initialPopulation(popSize, population)
    print("Initial distance: " + str(1 / rankRoutes(pop)[0][1]))
    
    for i in range(0, generations):
        pop = nextGeneration(pop, eliteSize, mutationRate)
    
    print("Final distance: " + str(1 / rankRoutes(pop)[0][1]))
    bestRouteIndex = rankRoutes(pop)[0][0]
    bestRoute = pop[bestRouteIndex]
    return bestRoute

```

Creamos una lista de ciudades y ejecutamos. Hay que ver qué suposiciones funcionan mejor. En este ejemplo, tenemos 100 individuos en cada generación, mantenemos 20 individuos de élite, usamos una tasa de mutación del 1 % para un gen determinado y recorremos 500 generaciones:
```
cityList = []

for i in range(0,25):
    cityList.append(City(x=int(random.random() * 200), y=int(random.random() * 200)))

geneticAlgorithm(population=cityList, popSize=100, eliteSize=20, mutationRate=0.01, generations=500)

```

