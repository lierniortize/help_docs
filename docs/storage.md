# GOOGLE STORAGE

## AUTORIZACIÓN
[https://cloud.google.com/sdk/gcloud/reference/auth](https://cloud.google.com/sdk/gcloud/reference/auth)

Cada cuenta tendrá acceso a diversos proyectos. Habrá cuentas con diferentes permisos. Para ver las cuentas con que están disponibles en local:
```
gcloud auth list
```
Para añadir una cuenta a esta lista:
```
gcloud auth login
```
Para cambiar la cuenta con la que queremos que se conecte por defecto:
```
gcloud auth application-default login
```


## Operar con archivos
```
gsutil cp
```
```
gsutil help cp
```

### Subir documentos
En local:
```
gsutil cp *.txt gs://my-bucket
```
### Descargar documentos
En cloud:
```
gsutil cp gs://my-bucket/*.txt
```
### Copiar documentos en la misma dirección de storage(?Proau bazpare)
En la consola de cloud:
```
gsutil cp -D gs://my-bucket/*.txt gsutil cp gs://my-bucket/*.txt
```

### Copiar repositorio
En la consola de cloud:
```
gsutil cp -r gs://my-bucket/*.txt gsutil cp gs://my-bucket/*.txt
```

