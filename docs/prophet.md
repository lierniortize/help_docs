# PROPHET

[Documentación python](https://facebook.github.io/prophet/docs/quick_start.html#python-api)

## INSTALACIÓN

Documentación:
```
pip install pystan==2.19.1.1
pip install prophet
```

Si diera error:
```
pip install localpip 
localpip install fbprophet
```

Para importar:
```
from fbprophet install Prophet
```

## ANÁLISIS DEL CÓDIGO "NÚCLEO" 

[Código](https://github.com/VERLAR/airflow-code/blob/master/batu-composer-py3/python_functions/dag_ventas_semanales_prophet/main.py)

Predicción de ventas o necesidad de artículos indistintivamente, ya que una variable está relacionada con la otra. 

El objetivo será crear una tabla `prediccion centro`.

1) Se cargan los datos de ventas: `df`

2) Cargamos información de calendario y centros: `calendario` y `Maes_cen`

3) `create_everyday_data()`

4) `filter_predict_data()`
Se filtra:

* Cogemos los datos a partir de 2017
* Cogemos los centros que hayan tenido ventas algún día de los últimos 15
* Cogemos los centros que tengan un mínimo histórico de 7 días

5) Se cargan los datos del 2017-2018 manualmente (`manual_2017_2018_calendar`) y `add_calendar_info`:

* Añadimos la información del calendario al DataFrame principal.
* Generamos una nueva columna `historico_anual` para indicar si tiene más de un año de histórico o no

6) `add_factor_seasonalities`
Añadimos extepciones de julio, agosto, y septiembre (hasta el 15) y COVID. En nuevas columnas que se llaman: `season_7`, `season_8`,  `season_9`, `alerta_covid19`, `confinamiento_covid19`, `toque_de_queda_covid19`

7) `add_regressor`
Variables regresoras:

* Bajada gasolineras
* Recuperación gasolineras
* Ciere Bilbao

8) `delete_outliers`

* Periodo COVID
* Huelga 8-M
* Nevada 2018/02/28 y 2018/03/01

Outliers por perímetro:

* Euskadi
* Vegalsa
* Cataluña

Outliers extremas. Nuestra variable x tendrá que cumplir:
mediana·0,05<=x<=mediana·20

Las demás se descartan.

A PARTIR DE AHORA EMPEZAREMOS A APLICAR LAS FUNCIONES POR CADA CENTRO:

1) `correct_calendar_center()`, `delete_center_outliers()`

2) Se tienen en cuenta los siguientes días según de dónde es el centro y siempre que tengamos más de cuatro meses de histórico. Sino se eliminan los días de apertura de la media.

Ahora mismo lo máximo que está afinado es por tipo de centro (hiper, super, gasolinera) y perímetro (norte,sur,caprabo,vegalsa,baleares)

* Día sin IVA
* Vales
* Día sin IVA en alimentación y fresco
* Día sin IVA electro black friday (sólo en hiper)
* Promoción doble
* Navidad
* Nochevieja
* Jueves Santo
* 24 y 31 de diciembre
* Fiestas nacionales
* Festivos
* Festivos nacionales com
* Reyes (en panadería)
* San José (en charcutería)

3) De todo lo anterior se crea una tabla `center` con fechas y ventas correspondientes a éstas.
```
m.fit(center)
future = m.make_future_dataframe(periods=DIAS_PREDICCION)
add_future_regressor()
add_future_factor_seasonalities()
forecast = m.predict(future)
```

Al final, se crea una tabla `predicciones_centro` uniendo todas las predicciones que hemos hecho para cada centro.

### EJECUCIÓN

```
prophet.py [-h] --yhat_name YHAT_NAME --load_data_query LOAD_DATA_QUERY --table_name TABLE_NAME
```
