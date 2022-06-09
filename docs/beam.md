# APACHE BEAM

Maneja y transforma datos en masivo. Utiliza máquinas externas para ello.
Soporta batch (los datos tienen un fin) y streaming (los datos están entrando continuamente). 

[Tutorial básico](https://www.youtube.com/watch?v=xSgTsKWhU0Y)

## INICIAR
* Instalar SDK de Python Beam [https://beam.apache.org/get-started/quickstart-py/](con este link)
```
pip install apache-beam[gcp]
```
* Escribir código en un archivo python importando `apahe_beam`
* Se puede ejecutar en local si es pequeño, sino hay que usar alguna herramienta (nosotros usaremos dataflow)

## Conceptos básicos

### `Pipeline`
Un Pipeline es el proceso que queremos seguir. Esto inclue leer datos, transformarlos y escribir un output. Todos los programas de Beam tienen que crear un Pipeline. Además, habrá que especificar las opciones con las que queremos ejecutar el código y dónde queremos hacerlo (--runner).
```
--runner DirectRuner #ejecutarlo en local
--runner DataflowRunner #ejecutarlo en cloud
```

### `PCollection`
Representa un conjunto de datos distribuido (no tiene orden). Puede ser acotada y que esté fija en un archivo o infinita dado que entran datos continuamente.

Your pipeline typically creates an initial PCollection by reading data from an external data source, but you can also create a PCollection from in-memory data within your driver program. From there, PCollections are the inputs and outputs for each step in your pipeline.

### `PTransform`
A PTransform represents a data processing operation, or a step, in your pipeline. Every PTransform takes one or more PCollection objects as input, performs a processing function that you provide on the elements of that PCollection, and produces zero or more output PCollection objects.

### Proceso
Con el código lo q hacemos es escribir el pipeline que luego se ejecutará en una máquina (en nuestro caso Cloud Dataflow). Tembién podríamos ejecutarlo en local si la cantidad de datos es pequeña.

## Ejemplo de un pipeline

Tomará datos de una fuente (también podrían ser más), realizaremos transformaciones sobre esos datos, y los escribiremos. ETL (Extract, Transform, Load).

Tomaremos el Quijote lo leeremos, contaremos las palabras y mostraremos las 5 más repetidas.

`main.py:`

```
from typing import Tuple

import apache_beam as beam
import argparse

from apache_beam import PCollection
from apache_beam.options.pipeline_options import PipelineOptions

def main():
  parser = argparse.ArgumentParser(description="Nuestro primer pipeline")
  parser.add_argument("--entrada", help="Fichero de entrada")
  parser.add_argument("--salida", help="Fichero de salida")

def run_pipeline(custom_args, beam_args):
  entrada = custom_args.entrada
  salida = custom_args.salida

  opts = PipelineOptions(beam_args)

  with beam.Pipeline(options=opts) as p:
    lineas: PCollection[str] = p | "Leemos entrada" >> beam.io.ReadFromText(entrada)
                                                        # "En un lugar de La Mancha" --> ["En", "un", ...], [...], [...] --> "En", "un", "lugar", ....
                palabras = lineas | "Pasamos a palabras" >> beam.FlatMap(lambda l: l.split())
                contadas: PCollection[Tuple[str, int]] = limpiadas  | "Contamos" >> beam.combiners.Count.PerElement()
                                    #"En" -> ("En", 17)
                                    # "un" -> ("un", 28)
                palabras_top_lista = contadas | "Ranking" >> beam.combiners.Top.Of(n_palabras, key=lambda kv: kv[1])
                palabras_top = palabras_top_lista | "Desenvuelve lista" >> beam.FlatMap(lambda x: x)
                 formateado: PCollection[str] = palabras_top | "Formateamos" >> beam.Map(lambda kv: "%s,%d" % (kv[0], kv[1]))
                formateado | "Escribimos salida" >> beam.io.WriteToText(salida)


if __name__ == '__main__':
  main()
```

Ejecutar en local:
```
py main.py --entrada quijote.txt --salida salida.txt --runner DirectRunner 
```


## EJEMPLO PASADO POR ASIER

Ejemplo básico de un pipeline:

```
import apache_beam as beam
from apache_beam import Map
from apache_beam.io.textio import ReadFromText, WriteToText
from apache_beam.coders.coders import Coder
import argparse
class LatinCoder(Coder):
    """A coder used for reading and writing strings as Latin-1."""
    def encode(self, value):
        return value.encode('latin-1')
    def decode(self, value):
        return value.decode('latin-1')
    def is_deterministic(self):
        return True
def csv_to_dict(line):
    # TODO: realizar modificaciones a cada línea
    return line
if __name__ == '__main__':    
    parser = argparse.ArgumentParser()
    # Argumentos necesarios para ejecutar el Pipeline.
    parser.add_argument('--runner', required=True)
    parser.add_argument('--project', required=True)
    parser.add_argument('--region', required=True)
    parser.add_argument('--staging_location', required=True)
    parser.add_argument('--temp_location', required=True)
    parser.add_argument('--network', required=True)
    parser.add_argument('--subnetwork', required=True)
    parser.add_argument('--job_name', required=True)
    parser.add_argument('--input', required=True)
    parser.add_argument('--output', required=True)
    known_args, pipeline_args = parser.parse_known_args()
    p = beam.Pipeline(runner="DataflowRunner", argv=["--project", known_args.project,
                                                     "--staging_location", known_args.staging_location,
                                                     "--temp_location", known_args.temp_location,
                                                     "--save_main_session", "True",
                                                     "--region", known_args.region,
                                                     "--network", known_args.network,
                                                     "--subnetwork", known_args.subnetwork,
                                                     "--job_name", known_args.job_name
                                                     ])
    (p | 'Read' >> ReadFromText(file_pattern=known_args.input, skip_header_lines=1, coder=LatinCoder())
       | 'Make Dic' >> Map(csv_to_dict)
       | 'Write' >> # TODO: añadir código para escribir el resultado utilizando 'WriteToText'
     )
    p.run().wait_until_finish()
```

## Ejecutar en cloud

### Parámetros necesarios: 

[Explicación de los parámetros](https://cloud.google.com/dataflow/docs/quickstarts/quickstart-python#run-the-pipeline-on-the-dataflow-service)

```
parser = argparse.ArgumentParser()

# Argumentos necesarios para ejecutar el Pipeline.
parser.add_argument('--runner', required=True)
parser.add_argument('--project', required=True)
parser.add_argument('--region', required=True)
parser.add_argument('--staging_location', required=True)
parser.add_argument('--temp_location', required=True)
parser.add_argument('--network', required=True)
parser.add_argument('--subnetwork', required=True)
parser.add_argument('--job_name', required=True)
parser.add_argument('--input', required=True)
parser.add_argument('--output', required=True)

known_args, pipeline_args = parser.parse_known_args()

p = beam.Pipeline(runner="DataflowRunner", argv=["--project", known_args.project,
                                                 "--staging_location", known_args.staging_location,
                                                 "--temp_location", known_args.temp_location,
                                                 "--save_main_session", "True",
                                                 "--region", known_args.region,
                                                 "--network", known_args.network,
                                                 "--subnetwork", known_args.subnetwork,
                                                 "--job_name", known_args.job_name
                                                 ])
```

Algunos parámetros por defecto para probar:
```
STAGING_NAME = 'gs://apro-verlar/beam-test01/staging'
TEMP_NAME = 'gs://apro-verlar/beam-test01/temp'
REGION = 'us-central1'
NETWORK = 'default'
SUBNETWORK = 'regions/us-central1/subnetworks/default'
RUNNER = 'DataflowRunner'
DATAFLOW_PROJECT = 'liquid-alloy-178307'
```

Para apuntar a un fichero en storage el formato es:
```
gs://nombre del bucket/direccion donde esta el archivo en ese bucket
```
Ejemplo:
```
gs://apro-verlar-staging-temp/tirado_202202.csv
```

```
py beam4.py --input gs://apro-verlar-staging-temp/tirado_202202.csv --output gs://apro-verlar-staging-temp/tirado_202202_copia.csv --staging_location gs://apro-verlar/beam-test01/staging --temp_location gs://apro-verlar/beam-test01/temp --region us-central1 --network default --subnetwork regions/us-central1/subnetworks/default --runner DataflowRunner --project liquid-alloy-178307
```
