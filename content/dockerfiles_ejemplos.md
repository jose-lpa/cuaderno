Title: Ejemplos de Dockerfiles
Date: 2021-05-19 14:42
Category: Contenedores
Tags: docker,dockerfile,ejemplos
Slug: ejemplos-dockerfiles
Authors: José L. Patiño-Andrés
Summary: Ejemplos útiles de Dockerfile


## Uso de SSH para acceder a datos privados en Dockerfile

En ocasiones tenemos que acceder a datos que no están accesibles al público
en nuestras _builds_ de Docker. Un ejemplo pueden ser repositorios privados
de Git que debemos descargar o instalar.

Para ello podemos hacer uso de la opción `--ssh` en `docker build`, que
reenvía las conexiones SSH al agente de la máquina anfitrión.

Ejemplo:

    :::dockerfile
    FROM python:3.8-slim-buster

    WORKDIR /app

    RUN pip3 install --no-cache-dir pip-tools && \
        apt-get -y update && \
        apt-get -y upgrade && \
        apt-get install -y git libspatialindex-dev && \
        apt-get clean && \
        rm -rf /var/lib/apt/lists/*


    RUN mkdir /root/.ssh/
    RUN mkdir -p -m 0600 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts

    COPY requirements.txt ./
    RUN --mount=type=ssh pip install -r requirements.txt

    COPY . .


    ENTRYPOINT ["python", "main.py"]

Teniendo este Dockerfile, podemos construir la imagen mediante el siguiente
comando, pasando el parámetro [`--ssh default`](https://docs.docker.com/develop/develop-images/build_enhancements/#using-ssh-to-access-private-data-in-builds):

    :::bash
    DOCKER_BUILDKIT=1 docker build --ssh default -t my_app .

## Multi-etapa

Podemos utilizar _builds_ [multi-etapa](https://docs.docker.com/develop/develop-images/multistage-build/)
para optimizar las imágenes de Docker o eliminar datos sensibles que no vayan a
usarse en la imagen final pero sean necesarios para llegar a ella, como por
ejemplo credenciales de acceso a algún repositorio, etcétera.

    :::dockerfile
    # syntax=docker/dockerfile:1
    FROM python:3.8-slim-buster AS base

    WORKDIR /app

    COPY requirements/*.txt ./requirements/

    # Install environment, multiple things needed here that are NOT needed in 
    # the final image: Git, development libraries, SSH artifacts...
    FROM base as environment_install

    RUN pip3 install --no-cache-dir pip-tools && \
        apt-get -y update && \
        apt-get -y upgrade && \
        apt-get install -y git libspatialindex-dev


    RUN mkdir /root/.ssh/
    RUN mkdir -p -m 0600 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts

    RUN --mount=type=ssh pip-sync requirements/main.txt

    # Finish the build taking the initial image as base, avoid adding all the
    # environment installation stuff.
    FROM base as application

    COPY --from=environment_install /usr/local/lib/python3.8/site-packages /usr/local/lib/python3.8/site-packages
    COPY --from=environment_install /usr/local/bin /usr/local/bin
    COPY ./ .


    ENTRYPOINT ["python", "main.py"]

