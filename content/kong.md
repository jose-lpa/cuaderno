Title: Kong API Gateway
Date: 2021-09-25 22:56
Category: Microservicios
Tags: kong,gateway,api,rest
Slug: kong-api-gateway
Authors: José L. Patiño-Andrés
Summary: Ejemplo de configuración de Kong API Gateway y anotaciones.


## Iniciar Kong API Gateway con Docker

    :::shell
    docker run -d --name kong \
               -e "KONG_DATABASE=off" \
               -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
               -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
               -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
               -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
               -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
               -p 8000:8000 \
               -p 8443:8443 \
               -p 8001:8001 \
               -p 8444:8444 \
               kong


## Configuración 'DB-less'

Kong puede ejecutarse en modo 'DB-less, es decir sin necesidad de una base de 
datos, a través de ficheros YAML con instrucciones declarativas. 

### Generación de fichero YAML inicial

Para generar un fichero inicial desde una instancia de Kong ejecutándose a 
través de Docker hacemos:

    :::shell
    docker exec -it kong kong config init /home/kong/kong.yml

Ahora tenemos en el contenedor un fichero `kong.yml` inicial con ejemplos e
instrucciones de configuración. Para pasarlo a la máquina anfitrión desde la que
ejecutamos Docker hacemos:

    :::shell
    docker exec -it kong cat /home/kong/kong.yml >> kong.yml

### Cargar configuración desde fichero YAML

Una vez tengamos la configuración en nuestro fichero `kong.yml` podemos pasar a
cargarla en Kong a través de la API de administración:

    :::shell
    curl -X POST http://localhost:8001/config -F config=@kong.yml
