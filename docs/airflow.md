# APACHE AIRFLOW


Es una herramienta para crear, planificar y monitorizar fluos de trabajo (DAGs)

Las secuencias de tareas pueden ser ejecutadas por planificación (el desarrollador especificado cuándo se ejecutara, puntual o periodicamente) o evento (ha encontrado nuevos archivo o nuevos datos, ha acabado otro proceso...)

Un DAG (Gráfico Aciclico Dirigido) es un esquema que nos dice que se tiene que ejecutar cada vez. 

Apache airflow nos deja programar en Python, nos deja ejecutar, panificar y distribuir tareas. Además podemos monitorear, hacer loggings y lanzar alertas. Se pueden hacer pruebas unitarias. 

Se pueden crear plugins según tus necesidades. 

No tiene soporte nativo para Windows.

## INICIAR

Haremos el host de airflow en Google Cloud composer.

Instalar airflow (cuidado con las versiones!):
```
pip install "apache-airflow==2.2.3" --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.2.3/constraints-3.9.txt"

```


## APLICACIONES
* Depósito de datos: limpieza, organización, verificación de calidad de datos...

* Machine Learning: Flujos de dato automaticos

* Growth analytics

* Search

* Data infrastructure maintenance

## OPERATORS
Para llevar a cabo cada actividad tendremos un operador diferente. Si no existe la actividad que
queramos siempre podremos crear nuestra propia función. Los operators son los encargados de llevar a 
cabo estas actividades. A continuación se explica para qué son algunos de ellos.

[Link to all the operators](https://airflow.apache.org/docs/apache-airflow-providers-google/stable/_api/airflow/providers/google/cloud/operators/bigquery/index.html)


* `GoogleCloudStorageObjectSensor`

    Sensor para ver si un fichero existe.
    [Documentación](https://airflow.apache.org/docs/apache-airflow/1.10.3/_api/airflow/contrib/sensors/gcs_sensor/index.html)


* `GCSToBigQueryOperator`
  
  Para pasar a BigQuery. [Documentación](https://airflow.apache.org/docs/apache-airflow-providers-google/stable/_api/airflow/providers/google/cloud/transfers/gcs_to_bigquery/index.html)

    * ```write_disposition``` [Doc](https://googleapis.dev/python/bigquery/latest/generated/google.cloud.bigquery.job.WriteDisposition.html)

    * ```time_partitioning``` [Doc](https://googleapis.dev/python/bigquery/latest/generated/google.cloud.bigquery.table.TimePartitioning.html)


* `BigQueryInsertJobOperator`

    [Doc](https://registry.astronomer.io/providers/google/modules/bigqueryinsertjoboperator)


* `PythonVirtualenvOperator`

    Para enviar emails.
    [Doc](https://registry.astronomer.io/providers/apache-airflow/modules/pythonvirtualenvoperator)


* `BashOperator`

  Enviar comando


* `PythonOperator`

  Para escribir código python y usar sus módulos (p.e. NumPy, Pandas)


* `EmailOperator`

    Enviar emails


* `SimpleHttpOperator`

    Post, Get... 


* `MySqlOperator`


* `DockerOperator`


* `Sensores`
Si tengo un DAG que no se tiene que ejecutar por tiempo sino por evento, el sensor estará escuchando, y entonces es cuando se ejecutará el DAG.

## EJEMPLO
Se programará el DAG que lleve a cabo lo siguiente:

Escuchar a que nos llegue el archivo (evento) > query > cargar un archivo propio > computar (aplicar modelos...) > subir data > enviar correo de confirmación

Descargar data > Enviar esos datos a otro sitio para ser procesados > Monitorear cuando el proceso se complete o durante > Obtener resultado y generar un informe > Enviar el informe por email

### EJEMPLO EN PSEUDOCÓDIGO

```
imports

options

start_date

default_args

with DAG(...) as dag:
    tasks

En qué orden queremos que se ejecuten los tasks.
    Para ejecutar a la vez [task1, task2]
    Para ejecutar una detrás de otra: task1 >> task2
```
Options for schedule interval:
```
None, @once, @hourly, @daily, @weekly, @monthly, @yearly
```
### EJEMPLO EN GITHUB/VERLAR

El repositorio [airflow_code](https://github.com/VERLAR/airflow-code/tree/master/batu-composer-py3) está organizado de la siguiente manera:
```
| apache_beam_code/
| credentials/
| python_functions/
| queries/
| schemas/
| send_error_mail/
| __init__.py
| dag1.py
| dag2.py
| ...
```
Los archivos con los DAG están en el root más alto, después tenemos las carpetas con los 
códigos que utilizarán dichos DAGs. 

Ejemplo de un DAG con los eventos explicados arriba:
[github/VERLAR](https://github.com/VERLAR/airflow-code/blob/master/batu-composer-py3/dag_apro_cuma_afectacion_margen.py)

