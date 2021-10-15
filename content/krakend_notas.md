Title: Ejemplo de configuración de KrakenD API Gateway
Date: 2021-06-11 16:23
Category: Microservicios
Tags: krakend,gateway,api,rest,graphql,golang
Slug: krakend-notas
Authors: José L. Patiño-Andrés
Summary: Ejemplo de configuración de KrakenD API Gateway y anotaciones.

[KrakenD](https://www.krakend.io/) es un producto software que implementa un
sistema de pasarela para APIs.

Puede ser útil en un entorno de microservicios si queremos implementar un API
Gateway que represente un punto de acceso principal para los clientes a nuestra
plataforma.

## Configuración

Configurar KrakenD puede hacerse de forma muy sencilla mediante una herramienta
online disponible para ello: [designer.krakend.io](https://designer.krakend.io).

Siguiendo los menús y formularios podemos crear una configuración para nuestra
pasarela, que después descargaremos en un fichero JSON.

A continuación se muestra un ejemplo de dicho fichero:

    :::json
    {
      "version": 2,
      "extra_config": {},
      "timeout": "10000ms",
      "cache_ttl": "300s",
      "output_encoding": "json",
      "name": "HB-API",
      "endpoints": [
        {
          "endpoint": "/products_list",
          "method": "GET",
          "output_encoding": "json",
          "extra_config": {},
          "backend": [
            {
              "url_pattern": "/v1.0/products",
              "encoding": "json",
              "sd": "static",
              "method": "GET",
              "extra_config": {
                "github.com/devopsfaith/krakend/http": {
                    "return_error_details": "api.example.com"
                }
              },
              "host": [
                "https://api.example.com"
              ],
              "disable_host_sanitize": false
            }
          ],
          "headers_to_pass": [
              "Authorization"
          ]
        },
        {
          "endpoint": "/zone_details",
          "method": "POST",
          "output_encoding": "json",
          "extra_config": {},
          "backend": [
            {
              "url_pattern": "/graphql",
              "encoding": "json",
              "sd": "static",
              "method": "POST",
              "extra_config": {
                "github.com/devopsfaith/krakend/http": {
                    "return_error_details": "graphql-api.example.com"
                }
              },
              "host": [
                "https://graphql-api.example.com"
              ],
              "disable_host_sanitize": false
            }
          ],
          "headers_to_pass": [
              "Authorization",
              "Content-Type"
          ]
        },
        {
          "endpoint": "/aggregated_data",
          "method": "GET",
          "output_encoding": "json",
          "extra_config": {},
          "backend": [
            {
              "url_pattern": "/api/v1/product_types",
              "encoding": "json",
              "sd": "static",
              "method": "GET",
              "extra_config": {
                "github.com/devopsfaith/krakend/http": {
                    "return_error_details": "europe-west1-example-staging.cloudfunctions.net"
                }
              },
              "host": [
                "http://europe-west1-example-staging.cloudfunctions.net"
              ],
              "disable_host_sanitize": false
            },
            {
              "url_pattern": "/survey-service/api/v1/vendors",
              "encoding": "json",
              "sd": "static",
              "method": "GET",
              "extra_config": {
                "github.com/devopsfaith/krakend/http": {
                    "return_error_details": "europe-west1-example-staging.cloudfunctions.net"
                }
              },
              "host": [
                "http://europe-west1-example-staging.cloudfunctions.net"
              ],
              "disable_host_sanitize": false
            }
          ],
          "headers_to_pass": [
              "Authorization"
          ]
        }
      ]
    }

El JSON de ejemplo muestra una pasarela KrakenD configurada para atender 3 
_endpoints_:

- `/products_list`
- `/zone_details`
- `/aggregated_data`

### `/products_list`

Enruta la petición del usuario hacia `https://api.example.com/v1.0/products`,
haciendo un `GET`. Devuelve la respuesta de este _backend endpoint_ al usuario.

### `/zone_details`

Enruta la petición del usuario a `https://graphql-api.example.com/graphql`,
haciendo un `POST` con la consulta graphql del usuario.

### `/aggregated_data`

Enruta la petición del usuario hacia dos _endpoints_ diferentes:
`http://europe-west1-example-staging.cloudfunctions.net/api/v1/product_types`
y
`http://europe-west1-example-staging.cloudfunctions.net/api/v1/vendors`.

Una vez ha obtenido las respuestas de estos dos puntos, fusiona los resultados
en una única respuesta que devuelve al usuario.

Hay que tener en cuenta que si una de estas dos peticiones a esos servicios
toma más del tiempo definido en el campo `timeout`, la respuesta entera fallará.

### Configuración extra

Existe configuración extra, especificada en los campos `"extra_config"`, que se
puede consultar en la [documentación de KrakenD](https://www.krakend.io/docs).

En nuestro caso, hemos configurado los _backends_ para que sus errores se 
muestren en los logs de KrakenD, en lugar de simples errores `500`.

También se pueden pasar las cabeceras HTTP a los _backends_, como se muestra en
los campos `"headers_to_pass"`. Esto es especialmente útil para pasar cabeceras
de autenticación.

## Ejecución en Docker

Una vez tenemos este fichero JSON, podemos simplemente levantar un contenedor 
con la pasarela KrakenD hacia nuestros backends, que estará disponible en
http://localhost:8080, mediante el comando:

    :::bash
    docker run -p 8080:8080 -v $PWD:/etc/krakend/ devopsfaith/krakend run --config /etc/krakend/krakend.json
