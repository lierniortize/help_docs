# LIBRERÍA PANDAS DE PYTHON

## INICIO
```
 pip install wheel
 pip install pandas
 pip install openpyxl
```

## CÓDIGO BÁSICO
```
import pandas as pd

df = pd.read_excel('nombre_de_archivo.xlsx', sheet_name=nombre de hoja)
```

## Cambiar valor de una columna dependiendo de otra.

```
A     B
-------
S     1
N     2
N     7
```

Queremos multiplicar el valor de la columna `B` por 10 si la columna `A` es 'S' y dejarla igual si es 'N'

```
df['B'] = df.apply(lambda x: x['B']*10 if x['A'] == 'S' else x['B'], axis=1)

```
