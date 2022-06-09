# APAHCE AIRFLOW


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



## EJEMPLO
Escuchar a que nos llegue el archivo (evento) > query > cargar un archivo propio > computar (aplicar modelos...) > subir data > enviar correo de confirmación

Descargar data > Enviar esos datos a otro sitio para ser procesados > Monitorear cuando el proceso se complete o durante > Obtener resultado y generar un informe > Enviar el informe por email

EJEMPLO EN CÓDIGO:

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
EJEMPLO:
```
# -*- coding: utf-8 -*-

from airflow import models
from airflow.providers.google.cloud.sensors.gcs import GCSObjectExistenceSensor
from airflow.operators.python import PythonVirtualenvOperator
from airflow.providers.google.cloud.operators.dataflow import DataflowCreatePythonJobOperator
from airflow.providers.google.cloud.operators.bigquery import BigQueryExecuteQueryOperator
from datetime import datetime, timedelta
from python_functions.generic_functions.ms_teams_webhook.main import failure_ms_teams
import pendulum
import os

from python_functions.dag_afect_mrg import afect_mrg_redis

# GCS OPTIONS
GCS_INPUT = 'gs://apro-verlar/incoming/'
GCS_FILE_NAME = 'CONSULTA_GBI_AFECTACION_MARGEN_{date}.csv.gz'
GCS_BUCKET = 'apro-verlar'
GCS_FILE_DIR = 'incoming/CONSULTA_GBI_AFECTACION_MARGEN_{date}.csv.gz'

# BIGQUERY OPTIONS
TMP_PROJECT_ID = 'liquid-alloy-178307'
TMP_TABLE_ID = 'TMP.afectacionMargen'
MAES_LOC = 'batu-203805.GBI.maestroLocalizaciones'
FINAL_TABLE = 'batu-203805.GBI.afectacionMargen'
# TODO
TABLE_VENTAS = 'batu-203805.GBI.ventasDesglosadasLocalizacionSeccion'

# DATAFLOW OPTIONS
APACHE_BEAM_PY_FILE = '/home/airflow/gcs/dags/apache_beam_code/dag_afect_mrg/gcs_to_bq.py'
DATAFLOW_PROJECT = 'liquid-alloy-178307'
JOB_NAME = 'afectacion-margen-{date}'
STAGING_NAME = 'gs://apro-verlar/beam-test01/staging'
TEMP_NAME = 'gs://apro-verlar/beam-test01/temp'
REGION = 'us-central1'
NETWORK = 'default'
SUBNETWORK = 'regions/us-central1/subnetworks/default'
RUNNER = 'DataflowRunner'

START_DATE = datetime(year=2021, month=5, day=20, hour=6, minute=0, tzinfo=pendulum.timezone('Europe/Madrid'))

default_args = {
    'owner': 'Versia Procesos Operativos',
    'start_date': START_DATE,
    'retries': 2,
    'retry_delay': timedelta(minutes=15),
    'email': eval(os.environ['SEND_EMAIL_TO']),
    'email_on_failure': True,
    'email_on_retry': False,
    'on_failure_callback': failure_ms_teams(author='Asier')
}


with models.DAG("dag_afect_mrg",
                default_args=default_args,
                schedule_interval='0 7 * * *') as dag:

    # Load requirements for PythonVirtualenvOperator
    with open('/home/airflow/gcs/dags/python_functions/dag_afect_mrg/requirements.txt', 'r') as fd:
        requirements = fd.read()
        requirements = requirements.split('\n')

    sensor_task = GCSObjectExistenceSensor(
        task_id="sensor_task",
        bucket=GCS_BUCKET,
        google_cloud_conn_id='google_cloud_default',
        object=GCS_FILE_DIR.format(date=(datetime.now() - timedelta(days=1)).strftime('%Y%m%d')),
    )

    gcs_bq = DataflowCreatePythonJobOperator(
        task_id="GCS_to_BQ",
        py_file=APACHE_BEAM_PY_FILE,
        options={
            'project': TMP_PROJECT_ID,
            'staging_location': STAGING_NAME,
            'temp_location': TEMP_NAME,
            'region': REGION,
            'network': NETWORK,
            'subnetwork': SUBNETWORK,
            'job_name': JOB_NAME.format(date=datetime.now().strftime('%Y%m%d')),
            'input': GCS_INPUT + GCS_FILE_NAME,
            'output': TMP_TABLE_ID,
            'runner': RUNNER
        },
        job_name=JOB_NAME.format(date=datetime.now().strftime('%Y%m%d')),
    )

    update_table = BigQueryExecuteQueryOperator(
        task_id='Update_data',
        sql='queries/dag_afect_mrg/update_data.sql',
        allow_large_results=True,
        use_legacy_sql=False,
    )

    ventas_loaded = GCSObjectExistenceSensor(
        task_id="ventas_loaded",
        bucket='ticket-test',
        google_cloud_conn_id='google_cloud_default',
        object=f'airflow_flags/tmp_airflow_dag_ventas_{datetime.now().strftime("%Y%m%d")}.txt',
        timeout=60 * 60 * 12,
    )

    afect_mrg_redis = PythonVirtualenvOperator(
        task_id="Afect_Mrg_Redis",
        python_callable=afect_mrg_redis.run,
        requirements=requirements,
        op_kwargs={
            'final_table': FINAL_TABLE,
            'maes_loc': MAES_LOC,
            'table_ventas': TABLE_VENTAS
        },
        system_site_packages=False,
    )

    delete_tmp = BigQueryExecuteQueryOperator(
        task_id='Delete_tmp_data',
        sql='queries/dag_afect_mrg/delete_tmp.sql',
        allow_large_results=True,
        use_legacy_sql=False,
    )

    sensor_task >> gcs_bq >> update_table >> [delete_tmp, afect_mrg_redis]
    ventas_loaded >> afect_mrg_redis

```

## OPERATORS
[Link to all the operators](https://airflow.apache.org/docs/apache-airflow-providers-google/stable/_api/airflow/providers/google/cloud/operators/bigquery/index.html)

### GoogleCloudStorageObjectSensor

Fitxero bat ba ote dagoen ikusteko sensorea.
```
GoogleCloudStorageObjectSensor
```
[Documentación](https://airflow.apache.org/docs/apache-airflow/1.10.3/_api/airflow/contrib/sensors/gcs_sensor/index.html)

### GCSToBigQueryOperator
BigQueryra pasatzeko

```
GCSToBigQueryOperator
```
[Documentación](https://airflow.apache.org/docs/apache-airflow-providers-google/stable/_api/airflow/providers/google/cloud/transfers/gcs_to_bigquery/index.html)

```write_disposition``` [Doc](https://googleapis.dev/python/bigquery/latest/generated/google.cloud.bigquery.job.WriteDisposition.html)

```time_partitioning``` [Doc](https://googleapis.dev/python/bigquery/latest/generated/google.cloud.bigquery.table.TimePartitioning.html)

### BigQueryInsertJobOperator

[Doc](https://registry.astronomer.io/providers/google/modules/bigqueryinsertjoboperator)

### PythonVirtualenvOperator

Email bidaltzeko.

[Doc](https://registry.astronomer.io/providers/apache-airflow/modules/pythonvirtualenvoperator)


### BashOperator

Enviar comando

### PythonOperator

Para escribir código python y usar sus módulos (p.e. NumPy, Pandas)

### EmailOperator
Enviar emails

### SimpleHttpOperator
Post, Get... 

### MySqlOperator

### DockerOperator

### Sensores
Si tengo un DAG que no se tiene que ejecutar por tiempo sino por evento, el sensor estará escuchando, y entonces es cuando se ejecutará el DAG.