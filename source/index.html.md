---
title: KeyClouding API

language_tabs: # must be one of https://git.io/vQNgJ
  - json
  # - ruby
  # - python
  # - javascript

# toc_footers:
#   - <a href='#'>Sign Up for a Developer Key</a>
#   - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

# includes:
#   - errors

search: true
---

# Introducción

En el siguiente documento se detallan los mecanismos y protocolos de comunicación a utilizar para el intercambio de información entre aplicaciones cliente y el servidor de datos de KeyClouding.

Para efectos del documento, la aplicación cliente realizará solicitudes al servidor y este enviará un set de resultados (o nulo) para la consulta indicada. A continuación, se presentan los estándares y tecnología a utilizar, luego las consultas y set de respuestas esperadas y finalmente una pequeña justificación de la implementación.

# Autenticación
```shell
"https://app.keyclouding.cl/api/v1/company/authentication?secret=secret_key"
```

> Si la respuesta es satisfactoria:

```json
[
  {
    "status": 200,
    "token": "AfTzE7BpcORyp6fN"
  }
]
```

> Si se produce un error:

```json
[
  {
    "status": 401,
    "token": null
  }
]
```
Para hacer uso de la API de KeyClouding, en toda consulta que se realiza al servidor se debe entregar un parámetro "token". Este se genera por medio de la autenticación y tiene una validez de 30 minutos.

Para generar el token, se debe hacer un request con método POST al URL
 `https://app.keyclouding.cl/api/v1/company/authentication`.

Se debe entregar a este request un parámetro "secret", que es entregado al usuario por KeyClouding. Según sea el resultado de la autenticación, se entregará el token correspondiente para realizar futuras consultas o un mensaje indicando que no se pudo ingresar con éxito.

A la derecha se muestra un ejemplo de autenticación donde el secret corresponde a "secret_key" y posibles respuestas según el resultado obtenido.




# Consultas y Funcionalidades

## Asignación de Cargo
```shell
"https://app.keyclouding.cl/api/v1/company/assign_ks?token=AfTzE7BpcORyp6fN&dni=13.433.615-3&ks_code=VET&country=CL&nombres=Juan&apellido_paterno=Perez&email=juan@perez.cl"
```
> Si la asignación es satisfactoria:

```json
[
  {
    "status": 200,
    "message”: “Applicant assigned.", 
    "password”: “0769019c",
    "ks_id": 32029
  }
]
```
> Si se produce un error de autenticación:

```json
  {
    "status": 401,
    "message": "Token inválido"
  }
]
```

> Si se produce un error de parámetros y sus respectivos mensajes:

```json
  {
    "status": 400,
    "message": "No se incluyó <parámetro>" ó
    "message": "Rut inválido" ó
    "message": "KS <parámetro> inexistente" ó
    "message": {
      "nombres":[
              "No puede estar en blanco"
      ],
      "apellido_paterno":[
              "No puede estar en blanco"
      ],
      "email": [
              "No es una dirección de correo electrónico válida."
      ]
    }
  }
]
```
Recibe los datos de un postulante y un cargo KS existente en el perfil de la empresa para que el sistema asigne los test respectivos.

### Request

`POST https://app.keyclouding.cl/api/v1/company/assign_ks`

### Parámetros de la consulta

Parámetro | Carácter | Descripción
--------- | -------- | -----------
token (string) | obligatorio | Token generado por la autenticación.
dni (string) | obligatorio | Documento de identidad del postulante.
ks_code (string) | obligatorio | Sigla que representa el KS a asignar. Debe ser la misma que la creada en el perfil de KeyClouding.
country (string) | obligatorio | País emisor del documento de identidad del postulante (sigla internacional). La lista de códigos de paises disponibles se encuentra [aquí](https://es.wikipedia.org/wiki/ISO_3166-1) (tenga cuidado en elegir el código correspondiente de 2 letras).
nombres (string) | obligatorio | Obligatorio sólo si dni del postulante no había sido ingresado en el sistema. En caso de que el postulante ya exista en el sistema y este parámetro venga en la consulta, se sobreescriben los nombres del postulante.
apellido_paterno (string) | obligatorio | Obligatorio sólo si dni del postulante no había sido ingresado en el sistema. En caso de que el postulante ya exista en el sistema y este parámetro venga en la consulta, se sobreescribe el apellido paterno del postulante.
email (string) | obligatorio | Obligatorio sólo si dni del postulante no había sido ingresado en el sistema. En caso de que el postulante ya exista en el sistema y este parámetro venga en la consulta, se sobreescribe el email del postulante.
proceso (string) | obligatorio | Nombre del proceso del cual está participando el postulante.
user_id (integer) | opcional | ID del usuario de la plataforma bajo el cual estarán a cargo las asignaciones. Si no se especifica queda por defecto bajo alguno de los administradores de la empresa.
phone (string) | opcional | Teléfono del postulante en formato internacional (+569xxxxxxxx) para envío de SMS. En caso de que el formato no coincida no arrojará error, pero no se guardará el teléfono ni se enviará el SMS. Para que el formato del teléfono sea válido el código de país debe coincidir con el campo Country. Este se encuentra en la misma tabla de código de países.
apellido_materno (string) | opcional | En caso de que el postulante ya exista en el sistema y este parámetro venga en la consulta, se sobreescribe el apellido paterno del postulante.
genero (string) | opcional | "Masculino" o "Femenino". En caso de que el campo no cumpla con el formato, el valor no será asignado al campo, pero se realizará de todas maneras la asignación del KS.
fecha_nacimiento (string) | opcional | Formato dd-mm-yyyy. En caso de que el campo no cumpla con el formato, el valor no será asignado al campo, pero se realizará de todas maneras la asignación del KS.
direccion_residencia (string) | opcional | Dirección de residencia del postulante.


La respuesta obtenida está en el siguiente formato:
### Respuesta JSON

Parámetro | Descripción
--------- | -----------
status (integer) | Resultado de la publicación de un nuevo postulante. Los valores admitidos son 200 (OK), 401 (Unauthorized), 403 (Forbidden) ó 400 (Bad Request).
ks_id (integer) | Identificador único de la asignación del KS.
password (string) | Contraseña del postulante.
message (string) | Mensaje indicando el estado de la solicitud.


<aside class="success">
Recuerda: una asignación completa es sinónimo de una asignación exitosa!
</aside>

## Consulta de resultados
```shell
"https://app.keyclouding.cl/api/v1/company/results_ks?token=AfTzE7BpcORyp6fN&ks_id=32029"
```
> Si la respuesta es satisfactoria:

```json
[
  {
    "status": 200,
    "estado_ks": "Rendido",
    "nota": "3.5",
    "rango": "Poco satisfactorio",
    "informe_resumen": "https://www.amazons3.com/kc/235324234.pdf",
    "informe_otros": [
		        {
			              "codigo_test": "ARP",
                    "informe_url": "https://www.amazons3.com/kc/12341.pdf" 
		        },
		        {
			              "codigo_test": "DCT",
                    "informe_url": "https://www.amazons3.com/kc/12341.pdf" 
		        }
    ] 
  }
]
```

> Si la respuesta es satisfactoria, pero estado_ks es distinto de "Rendido":

```json
[
  {
    "status": 200,
    "estado_ks": "Inválido",
    "nota": null,
    "rango": null,
    "informe_resumen": null,
    "informe_otros": [
            {
			              "codigo_test": "ARP",
                    "informe_url": "null"
		        },
            {
			              "codigo_test": "DCT",
                    "informe_url": "https://www.amazons3.com/kc/12341.pdf"
		        }
    ] 
  }
]
```

> Si se produce un error de autenticación:

```json
[
  {
    "status": 401,
    "message": "Token inválido"
  }
]
```

> Si se produce un error de parámetros:

```json
  {
    "status": 400,
    "message": "Asignación inválida"
  }
]
```
Recibe ks_id (identificador único del KS) y el sistema retorna los resultados de la rendición.

### Request

`POST https://app.keyclouding.cl/api/v1/company/results_ks`

### Parámetros de la consulta

Parámetro | Carácter | Descripción
--------- | -------- | -----------
token (string) | obligatorio | Token generado por la autenticación.
ks_id (integer) | obligatorio | Identificador único de asignación, el mismo que fue retornado en la Asignación de Cargo.

La respuesta obtenida está en el siguiente formato:
### Respuesta JSON

Parámetro | Descripción
--------- | -----------
status (integer) | Resultado de la publicación de un nuevo postulante. Los valores admitidos son 200 (OK), 401 (Unauthorized), 403 (Forbidden) ó 400 (Bad Request).
estado_ks (string) | Estado de la rendición del KS, el cual puede tomar los valores “Rendido”, ”Inválido”, “Inconsistente”, “En Proceso” ó “Eliminado”.
nota (string) | Nota obtenida en el KS ó null si el estado_ks es distinto de “Rendido”.
rango (string) | Texto que indica si la persona es muy adecuada, adecuada, aceptable o poco satisfactoria para el cargo ó null si el estado_ks es distinto de “Rendido”.
informe_ks_url (string) | URL donde está alojado el informe PDF del resultado del KS ó null si el estado_ks es distinto de “Rendido”.
informes_parciales (string) | JSON Array con los códigos de los test y sus respectivas URL donde está alojado del informe PDF del resultado del test parcial, ó null si el estado_ks es distinto de “Rendido”.

## Lista de Key Scorings (cargos) activos
```shell
"https://app.keyclouding.cl/api/v1/company/list_ks?token=AfTzE7BpcORyp6fN"
```
> Si la respuesta es satisfactoria:

```json
[
  {
    "status": 200,
    "ks_list": [
      {
        "id": 165,
        "ks_code": "TELEM",
        "nombre": "TELEMARKETING",
        "tests": [
          {
            "id": 7,
            "codigo": "CIE",
            "nombre": "Comprensión de Instrucciones Escritas",
            "tiempo": 8,
            "estado": "Activo",
            "contra_reloj": true
          },
          {
            "id": 4,
            "codigo": "ARP",
            "nombre": "Aprendizaje y Resolución de Problemas",
            "tiempo": 12,
            "estado": "Activo",
            "contra_reloj": true 
          }
        ]
      },
      {
        "id": 158,
        "ks_code": "AUX",
        "nombre": "AUXILIAR",
        "tests": [
          {
            "id": 13,
            "codigo": "CET",
            "nombre": "Comportamiento de Equipos de Trabajo",
            "tiempo": 30,
            "estado": "Activo",
            "contra_reloj": false
          },
          {
            "id": 4,
            "codigo": "ARP",
            "nombre": "Aprendizaje y Resolución de Problemas",
            "tiempo": 12,
            "estado": "Activo",
            "contra_reloj": true 
          }
        ]
      }
    ]
  }
]
```

> Si se produce un error de autenticación:

```json
[
  {
    "status": 401,
    "message": "Token inválido"
  }
]
```
Recibe el token de autentificación de la empresa y retorna todos los KS (cargos) activos.

### Request

`POST https://app.keyclouding.cl/api/v1/company/list_ks`

### Parámetros de la consulta

Parámetro | Carácter | Descripción
--------- | -------- | -----------
token (string) | obligatorio | Token generado por la autenticación.

La respuesta obtenida está en el siguiente formato:
### Respuesta JSON

Parámetro | Descripción
--------- | -----------
status (integer) | Resultado de la consulta. Los valores admitidos son 200 (OK), 401 (Unauthorized), 403 (Forbidden) ó 400 (Bad Request).
ks_list | Lista con todos los KS activos y algunos parametros.

Los parámetros anteriormente señalados se encuentran a continuación:

Parámetro | Descripción
--------- | -----------
id (integer) | ID del KS.
ks_code (string) | Código del KS.
nombre (string) | Nombre del KS.
tests | Lista de tests correspondientes al ks y algunos parámetros de cada test tales como id, código, nombre, tiempo, estado y contra_reloj.

## Creación de Webhook

> Ejemplo de Respuesta JSON del POST que se hace al callback_url:

```json
[
  {
    "messages": "Entity_model CpcAssignment have changed",
    "is_success": true,
    "data": {
      "ks_id": 32029
    }
  }
]
```

> Si la respuesta es satisfactoria:

```json
[
  {
    "messages": "Webhook created successfully",
    "is_success": true,
    "data": {
      "params":{
        "id": 12,
        "callback_url": "http://localhost:3000/webhooks",
        "entity_model": "CpcAssignment",
        "company_id": 79,
        "created_at": "2018-10-23T13:37:52.174-03:00",
        "updated_at": "2018-10-23T13:37:52.174-03:00"
      }
    }
  }
]
```

> Si la respuesta es satisfactoria pero ya existe webhook para la entidad seleccionada:

```json
[
  {
    "messages": "Cannot create webhook with the same entity",
    "is_success": false,
    "data": {
      "params":{
        "id": 12,
        "callback_url": "http://localhost:3000/webhooks",
        "entity_model": "CpcAssignment",
        "company_id": 79,
        "created_at": "2018-10-23T13:37:52.174-03:00",
        "updated_at": "2018-10-23T13:37:52.174-03:00"
      }
    }
  }
]
```

> Si se produce un error de parámetros:

```json
[
  {
    "messages": "Cannot create webhook",
    "is_success": false,
    "data": {
      "params": {
        "entity_model": "CpcAssignment"
      },
      "errors":{
        "callback_url": [
          "No puede estar en blanco",
          "No es válido"
        ]
      }
    }
  }
]
```
Permite a un cliente suscribirse a un sistema de actualizaciones sobre un modelo de KeyClouding. Al crear un webhook se debe indicar una URL callback, a la que se le enviará una solicitud POST al momento de existir actualizaciones sobre el modelo suscrito. Es importante destacar que en la URL callback entregada por el cliente se podrán agregar parametros de autenticación correspondientes a las credenciales especificas de cada sistema.
### Request

`POST https://app.keyclouding.cl/api/v1/webhook/endpoints`

### Parámetros de la consulta

Los parámetros deben ser enviados a través del body de la request.

Parámetro | Carácter | Descripción
--------- | -------- | -----------
token (string) | obligatorio | Token generado por la autenticación.
callback_url (string) | obligatorio | URL a la cual se efectuará la consulta POST ante una modificación de alguna instancia del modelo suscrito.
entity_model (string) | obligatorio | Nombre del modelo al cual se quiere suscribir (e.g "CpcAssignment")

La respuesta obtenida está en el siguiente formato:
### Respuesta JSON

Parámetro | Descripción
--------- | -----------
messages (string) | Mensaje indicando el estado de la solicitud.
is_success (boolean) | Indica si la creación del webhook fue o no exitosa.
data | JSON array con distintos parámetros.

Los parámetros anteriormente señalados se encuentran a continuación:

Parámetro | Descripción
--------- | -----------
id (integer) | ID del webhook creado en el sistema.
callback_url (string) | URL proporcionada para realizar las llamadas de callback.
entity_model (string) | Modelo suscrito por el webhook.
created_at (string) | Fecha de creación del webhook.
updated_at (string) | Fecha de actualización de webhook.

<aside class="success">
Recuerda: actualmente sólo se pueden realizar seguimiento de las actualizaciones de la entity_model 'CpcAssignment'
</aside>

## Listar Webhooks existentes
```shell
"https://app.keyclouding.cl/api/v1/webhook/list_webhooks?token=AfTzE7BpcORyp6fN"
```
> Si la respuesta es satisfactoria:

```json
[
  {
    "messages": "List of webhooks",
    "is_success": true,
    "data": {
        "webhook": [
            {
                "id": 12,
                "callback_url": "http://localhost:3000/webhooks",
                "entity_model": "CpcAssignment",
                "company_id": 79,
                "created_at": "2019-01-03T13:16:18.732-03:00",
                "updated_at": "2019-01-03T13:16:18.732-03:00"
            }
        ]
    }
  }
]
```

> Si se produce un error de parámetros:

```json
[
  {
    "messages": "Cannot find webhook",
    "is_success": false,
    "data": {
      "params": ""
    }
  }
]
```
Permite obtener una lista con los webhooks existentes de la entidad o un mensaje informando que no posee ninguno en caso contrario.

### Request

`GET https://app.keyclouding.cl/api/v1/webhook/list_webhooks`

### Parámetros de la consulta

Parámetro | Carácter | Descripción
--------- | -------- | -----------
token (string) | obligatorio | Token generado por la autenticación.

La respuesta obtenida está en el siguiente formato:
### Respuesta JSON

Parámetro | Descripción
--------- | -----------
messages (string) | Mensaje indicando el estado de la solicitud.
is_success (boolean) | Indica si la creación del webhook fue o no exitosa.
data | JSON array con distintos parámetros.

Los parámetros anteriormente señalados se encuentran a continuación:

Parámetro | Descripción
--------- | -----------
id (boolean) | ID del webhook creado en el sistema.
callback_url (string) | URL proporcionada para realizar las llamadas de callback.
entity_model (string) | Modelo suscrito por el webhook.
created_at (string) | Fecha de creación del webhook.
updated_at (string) | Fecha de actualización de webhook.

## Eliminación de Webhook
```shell
"https://app.keyclouding.cl/api/v1/webhook/endpoints?token=AfTzE7BpcORyp6fN&id=43"
```
> Si la respuesta es satisfactoria:

```json
[
  {
    "messages": "webhook deleted successfully",
    "is_success": true,
    "data": { }
  }
]
```

> Si se produce un error de parámetros:

```json
[
  {
    "messages": "Cannot find webhook",
    "is_success": false,
    "data": {
      "params": ""
    }
  }
]
```
Permite eliminar un webhook asociado a un modelo del sistema, para dejar de recibir solicitudes al momento de actualizaciones de instancias del modelo suscrito.

### Request

`DELETE https://app.keyclouding.cl/api/v1/webhook/endpoints`

### Parámetros de la consulta

Parámetro | Carácter | Descripción
--------- | -------- | -----------
token (string) | obligatorio | Token generado por la autenticación.
id (integer) | obligatorio | Identificador único del webhook, el mismo que fue retornado en la Creación de Webhook.

La respuesta obtenida está en el siguiente formato:
### Respuesta JSON

Parámetro | Descripción
--------- | -----------
messages (string) | Mensaje indicando el estado de la solicitud.
is_success (boolean) | Indica si la eliminación del webhook fue o no exitosa.
data | {}


# Estándares y Tecnología

## Comunicación

La comunicación entre el sistema cliente y el servidor se realizará de la siguiente manera:

    1. La app cliente envía mediante POST una solicitud HTTPS a una URL específica determinada por el tipo de recurso requerido. La estructura de la URL será establecida por KeyClouding y la app cliente se adaptará a las rutas indicadas. 

    2. El servidor procesa los parámetros POST y retorna una respuesta en JSON (JavaScript Object Notation); o error en caso de no poder responder a la solicitud.

    3. El cliente recibe la respuesta. Si es un error notifica al usuario (si corresponde). Si es una respuesta JSON, la deserializa y procesa los objetos de la respuesta.
  
## Tecnología

Toda la comunicación entre plataformas utilizará estándares REST y protocolo de intercambio de mensajes mediante JSON, por lo que la comunicación es transparente independiente de la plataforma cliente empleada. 

Por otro lado, el servidor posee servicios HTTPS y conectividad a internet, de tal manera de permitir la solicitud de recursos mediante URL.

# Justificación de la Tecnología

Toda la comunicación entre el cliente y el servidor (C-S) se realiza mediante llamadas HTTP al servidor entregando los parámetros necesarios mediante POST. Esta forma de comunicación es utilizada ya que, por un lado, es transparente y multi-plataforma (cualquier aplicación con soporte para comunicación HTTP, independiente de la plataforma, puede comunicarse con la API del servidor), además de ser ampliamente soportada y reconocida. Por otro lado, al seguir el paradigma REST, el servidor procesa sólo cuando recibe una solicitud y no tiene que estar pendiente de conexiones entrantes, ya que el listening lo realiza el servidor web.

La respuesta del servidor se serializa mediante JSON. Esto debido a que JSON es un mecanismo de codificación para el traspaso de mensajes entre sistemas que optimiza el proceso de transferencia y de codificación/decodificación. La razón de esto es que JSON serializa los objetos utilizando la menor cantidad de caracteres sin comprimir la información simplificando, a su vez, el proceso de codificación. Esto es muy importante, ya que los dispositivos móviles tienen conectividad y CPU limitadas.

Finalmente, la combinación HTTP/JSON es ampliamente utilizada en aplicaciones de alto rendimiento y carga, como Facebook, Twitter, Instagram, etc; debido tanto a lo anteriormente planteado como al gran soporte que existe en la actualidad a estas tecnologías.


# Ambiente de prueba

Durante el proceso de integración, se habilitará un ambiente de prueba para que el cliente pueda realizar solicitudes a la API libremente. El ambiente de pruebas no posee envío de emails para evitar potenciales errores en la comunicación de cara al cliente o al usuario. 

Es por esta razón, que se facilitará un usuario de prueba ficticio, junto con sus credenciales. Adicionalmente, se habilitará una API_KEY que funcionará exclusivamente en este ambiente. Una vez que las pruebas hayan concluido, se otorgará la API_KEY correspondiente al ambiente de producción. 

La URL del ambiente de pruebas mantiene la misma forma, pero cambia el host:
`https://staging.keyclouding.cl/api/v1/`