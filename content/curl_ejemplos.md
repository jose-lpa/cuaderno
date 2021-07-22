Title: Ejemplos de uso de curl
Date: 2021-07-21 10:40
Category: Herramientas
Tags: curl,networking,herramientas,consola,shell
Slug: curl-ejemplos
Authors: José L. Patiño-Andrés
Summary: Ejemplos de uso de la herramienta de consola curl

## Configuración básica

Podemos añadir estos argumentos de entrada del comando `curl` a un fichero
`.curlrc` en nuestro directorio de usuario y entonces se ejecutarán siempre.

### Silenciar salida de metadatos de curl

De normal, al hacer `curl http://example.com` curl imprime una cabecera con
metadatos del estado de la comunicación.

    :::bash
    curl http://example.com

    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                   Dload   Upload  Total   Spent    Left  Speed
    100 12051  100 12051    0     0  16110      0 --:--:-- --:--:-- --:--:-- 16089

Si esto nos molesta, lo podemos eliminar con el argumento `--silent`.

### Mostrar cabeceras HTTP

Normalmente curl no va a mostrar las cabeceras HTTP en su salida. Si queremos
verlas, podemos usar el argumento `--dump-header /dev/stderr`

    :::bash
    curl --dump-header /dev/stderr http://example.com

    HTTP/2 200
    date: Wed, 21 Jul 2021 09:37:08 GMT
    content-type: application/json
    content-length: 12051
    x-request-id: 9ff84e15fc2b8258ab994008c5c2ad60
    strict-transport-security: max-age=15724800; includeSubDomains
    access-control-allow-origin: *
    access-control-allow-credentials: true
    ...

### Acabar con salto de línea

Si los datos mostrados por curl en su salida no contenían un salto de línea al
final el prompt de nuestro terminal se puede ver encajado al final de la 
respuesta de curl, lo cual no es muy conveniente. Para arreglar esto podemos
usar el argumento `--write-out "\n"`


## Mostrar respuestas JSON formateadas

Podemos usar las herramientas [json_pp](https://github.com/deftek/json_pp) o
[jq](https://github.com/stedolan/jq) en nuestros comandos, pasándoles la
respuesta del comando `curl` mediante una tubería.

    :::bash
    curl http://example.com | json_pp

    :::bash
    curl http://example.com | jq

La más conveniente a mi parecer es `jq`, ya que colorea el JSON resaltando la
sintaxis.

### Contar número de elementos en la respuesta JSON

Podemos contar el número de elementos devueltos en una respuesta JSON con el
comando `jq` también:

    :::bash
    curl http://example.com | jq length

    34

Si el campo que queremos contar está anidado, por ejemplo si la respuesta fuera así:

    :::json
    {
        "results": {
            "items": [
                1,
                2,
                3,
                4
            ]
        }
    }

podríamos hacer lo siguiente:

    :::bash
    curl http://example.com | jq -r '.results.items | length'

    4


## Hacer `POST` a una API REST JSON

    :::bash
    curl -X POST "http://example.com/items" 
         -H "Content-Type: application/json"
         --data '{"name": "New Item", "quantity": 25, price: 12.5}'

## Autenticación mediante cabecera HTTP (API token)

    :::bash
    curl http://example.com
         -H "Auhtorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXUyJ9"
