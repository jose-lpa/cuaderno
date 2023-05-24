Title: Aplicación en Docker con paquetes privados de GAR
Date: 2023-01-27 16:17
Category: Contenedores
Tags: docker,python,gcp,gar
Slug: docker-gcp-artifact-registry
Authors: José L. Patiño-Andrés
Summary: Referencia de configuración de un Dockerfile para instalar paquetes desde un repositorio privado en Google Artifact Registry.

En ocasiones, podemos necesitar acceder a entornos privados cuando estamos
construyendo una imagen de Docker. Por ejemplo, instalar un paquete Python
que está en un repositorio privado de Google Cloud Artifact Registry.

Ésta referencia explica cómo hacerlo.

## Credenciales de Google Cloud

Lo primero que vamos a necesitar son unas credenciales de GCP de una service
account que tenga acceso a ese repositorio GAR privado.

Una vez tengamos esas credenciales, que vienen en forma de un fichero JSON,
podemos pasar al siguiente punto.

## Dockerfile

En el Dockerfile vamos a añadir las siguientes líneas:

    :::dockerfile
    ARG GOOGLE_APPLICATION_CREDENTIALS=/run/secrets/gsa_key

    RUN pip install keyring keyrings.google-artifactregistry.auth

    # AHORA podemos hacer `pip install ...` a nuestro paquete privado.
    RUN --mount=type=secret,id=gsa_key pip install -r requirements.txt

Ésto se hace porque los secretos de Docker se guardan en el volumen `/run/secrets`
bajo el mismo nombre que se le ha pasado al secreto. Entonces lo que hacemos es
crear la variable de entorno necesitada por `keyrings.google-artifactregistry.auth`
para autenticarse con `pip` contra el repositorio privado.

## Ejecutando Docker...

Ahora podemos construir la imagen así:

    :::bash
    DOCKER_BUILDKIT=1 docker build \
      --secret id=gsa_key,src=/ruta/al/fichero/credenciales.json \
      -t my-application .

Y ésto nos meterá el fichero de credenciales en el secreto de Docker `gsa_key`
que ya nos hemos encargado de mencionar en el Dockerfile en nuestra línea `ARG`.
