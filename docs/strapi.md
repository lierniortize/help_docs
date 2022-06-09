# STRAPI

```
npx create-strapi-app strapi-api
```

Es una herramienta para la creación de APIs que permitirá al usuario-administradores
cambiar las bases de datos desde una interfaz, sin tener que cambiar nada en el código.
Se contempla como opción de sustitución a Django, pero solo será posible en proyectos
simples sin muchos datos.

## Quickstart

```
npx create-strapi-app@latest my-project --quickstart
```


Una vez creado la app, nos ubicamos dentro de la carpeta que se nos ha creado con la 
creación del proyecto y ejecutamos el siguiente comando para poner el servidor en marcha:
```
npm run develop
```

Panel admin en: [http://localhost:1337/admin/](http://localhost:1337/admin/)



## Collection type

Acceder a [Content-type Builder](http://localhost:1337/admin/plugins/content-type-builder/content-types/plugin::users-permissions.user)
Desde ahí podremos crear nuevos "modelos" con sus atributos. Una vez creado el modelo, podemos
añadir los atributos deseados y crear tantos objetos de ese modelo como queramos.

## Acceder a los datos de la BD

Los datos estarán visibles en:
```
http://localhost:1337/api/modelo_name
```

Sin embargo strapi hace un filtro por defecto al obtener los datos. Si queremos acceder a 
todos tenemos que acceder por:
```
http://localhost:1337/api/modelo_name?populate=*
```

javascript:

```
function getModelo(){
    fetch('http://localhost:1337/api/modelo?populate=*')
      .then(response => response.json())
      .then(data =>
      data.data.forEach(readModelo))    
}

function readModelo(element){
        var api_url = 'http://localhost:1337'
        var modelo = element.attributes
        var modelo_atrb1 = modelo.atrb1
        var modelo_atrb2 = modelo.atrb2
        var modelo_img_url = modelo.img_atrb.data.attributes.url
        modelo_img_url = api_url  + img_url
```
