# SQL

Lenguaje para base de datos relacionales. 

## Bases de datos relacionales

* Clave primaria: la que aparece solo una vez. Es el identificador. 
* Campo: Columnas
* Registro: Filas

Las tablas se pueden unir entre ellas utilizando campos que tienen en común. Se hace un esquema. 

* Clave foranea: Claves que son primarias en otras tablas, y que sirven para conectarse con ellas. 

## Consultas de SQL

### Básico

``` 
SELECT campo1, campo2 
FROM tabla 
WHERE condicion; /*El punto y coma no es necesari*/
```
ORDENADO:
```
SELECT * 
FROM tabla 
WHERE campo ASC/DESC;
```

### Tipos de dato

* Números
* Cadenas de caracteres
* Fechas

Cada tipo de dato tiene una forma de manipulación. 

### Cadenas de caracteres

Buscar cadenas de caracteres que contengan una palabra específica
```
SELECT campo
FROM tabla
WHERE descripcion LIKE '%palabra_especifica%'
```

CAMBIAR UN CAMPO
```
SELECT campo1, campo2, campo_nombre_viejo AS campo_nombre_nuevo
FROM tabla
```
CONCATENAR VALORES DE CAMPO
```
SELCT nombre, apellido
    CONCAT(nombre, ' ', apellido) AS nombre_y_apellido
FROM tabla

/* Se seleccionarán tres columnas: nombre, apellido y nombre_apellido
```

Aparte de CONCAT para cadenas de carácteres:

* `UPPER()`
* `LOWER()`
* `LENGHT()`

### Fechas

* `NOW()`
* `YEAR( NOW() )`
* `DATE()`
* `MONTH()`
* `DAY()`

### Números

* `MIN()` Elige el mínimo de un campo
* `MAX()` Elige un máximo de un campo
* `COUNT()` Cuenta la cantidad de registros de un campo
* `SUM()` Sumará los valores de un campo

### `DISTINCT`

```
SELECT DISTINCT campo
FROM tabla

/*Nos seleccionará los registros diferentes de un campo*/
```

```
SELECT COUNT (DISTINCT campo)
FROM tabla

/*Nos dirá cuántos registros deferentes tenemos en un campo*/
```
### `GROUP BY`
```
SELCT costo_renta, COUNT(*) as cantidad
FROM pelicula
GORUP BY costo_renta
```
```
---------------------------
| costo_renta | cantidad  |
---------------------------
| 0.99        |  341      |
| 2.99        |  323      |
| 4.99        |  336      |
---------------------------
```
Nos dirá que tenemos 341 películas de costo 0.99, 323 películas de costo 2.99 y 336 películas de costo 4.99.

OTRO EJEMPLO:

```
SELECT
    tienda_id,
    COUNT(DISTINCT pelicula_id) AS peliculas,
    COUNT(inventario_id) AS copias
FROM inventario
GROUP BY tienda_id
```

Nos dirá cuantas películas hay por cada tienda y cuantas copias diferentes también de cada tienda.

También podemos agrupar por más de un campo. Nos agrupará todos los registros que tengan los dos campos en común. 

### `AVG`

```
AVG(campo) /*Nos dirá el promedio de un campo*/
```

### `JOIN`
Para cruzar tablas. (Devuelve registro que tienen registros en ambas tablas)

```
SELECT tabla1.campo1, tabla1.campo2, tabla2.campo1
FROM tabla1
JOIN tabla2
    ON tabla2.campo_igual = tabla1.campo_igual
```

Podemos cambiar la notación:
```
SELECT A.campo1, A.campo2, B.campo1
FROM tabla1 A
JOIN tabla2 B
    ON B.campo_igual = A.campo_igual
```

También podemos hacer `JOIN` con más de dos tablas. Escribiremos dos `JOIN` para ello.

### `LEFT JOIN` y `RIGHT JOIN`

Si queremos todos los registros, aunque no estén en ambas tablas. 

Left nos devolverá todo lo que está en tabla1 aunque no esté en tabla2. Right hará lo contrario.

EJEMPLO: Nos dirá cuántas copias de cada pelicula (existente en la tienda1) hay en la tienda1:

```
SELECT
    p.pelicula_id
    p.titulo
    COUNT(i.inventario_id) AS copias
FROM pelicula p
LEFT JOIN inventario i
    ON p.pelicula_id=i.pelicula_id
WHERE i.tienda_id=1
GROUP BY p.pelicula_id, p.titulo
```
```
---------------------------------------
| pelicula_id | cantidad  |   copias  |
---------------------------------------
| 1           |  titulo1  |      8    |
| 3           |  titulo2  |      3    |
| 6           |  titulo3  |      4    |
|     ...     |  ...      |     ...   |
---------------------------------------
```
Sin utilizar `WHERE`:
Nos dirá cuántas copias de cada pelicula (existente en la tienda1) hay en la tienda1:

```
SELECT
    p.pelicula_id
    p.titulo
    COUNT(i.inventario_id) AS copias
FROM pelicula p
LEFT JOIN inventario i
    ON p.pelicula_id=i.pelicula_id
    AND i.tienda_id=1
GROUP BY p.pelicula_id, p.titulo
```
```
---------------------------------------
| pelicula_id | cantidad  |   copias  |
---------------------------------------
| 1           |  titulo1  |      8    |
| 2           |  titulo2  |      0    |
| 3           |  titulo3  |      3    |
|     ...     |  ...      |     ...   |
---------------------------------------
```
CUIDADO AL UTILIZAR `WHERE`!!

### CONDICIONALES
SELECT campo1, campo2
    CASE 
        WHEN campo2<3 THEN 'pequeño'
        WHEN campo2>3 AND campo2<5 THEN 'MEDIANO' /* (ELIF)Esta no se va a evaluar si la anterior ha sido correcta*/
        ELSE 'grande'
        END AS tamaño
FROM tabla

## BIG QUERY

[Documentación](https://cloud.google.com/bigquery/docs/reference/standard-sql/dml-syntax#merge_statement)





