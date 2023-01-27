Title: Configuración de repositorios en Ubuntu
Date: 2022-06-10 10:23
Category: Sistema
Tags: ubuntu,ppa,debian,apt,repositorios,dpkg
Slug: ubuntu-repositorios
Authors: José L. Patiño-Andrés
Summary: Breve guía de configuración de repositorios en distribuciones Ubuntu.

## Listar repositorios configurados

El siguiente comando nos mostrará una lista de los repositorios que hay
configurados actualmente en nuestro sistema:

```bash
apt policy
```

La respuesta será más o menos:

```bash
Package files:
 100 /var/lib/dpkg/status
     release a=now
 500 https://packages.microsoft.com/repos/ms-teams stable/main amd64 Packages
     release o=ms-teams stable,a=stable,n=stable,l=ms-teams stable,c=main,b=amd64
     origin packages.microsoft.com
 500 http://apt.postgresql.org/pub/repos/apt focal-pgdg/main amd64 Packages
     release o=apt.postgresql.org,a=focal-pgdg,n=focal-pgdg,l=PostgreSQL for Debian/Ubuntu repository,c=main,b=amd64
     origin apt.postgresql.org
 500 https://nvidia.github.io/nvidia-docker/ubuntu20.04/amd64  Packages
     release v=1.0,o=https://nvidia.github.io/nvidia-docker,a=bionic,n=bionic,l=NVIDIA CORPORATION <cudatools@nvidia.com>,c=
     origin nvidia.github.io
 500 https://nvidia.github.io/nvidia-container-runtime/stable/ubuntu20.04/amd64  Packages
     release v=1.0,o=https://nvidia.github.io/nvidia-container-runtime,a=bionic,n=bionic,l=NVIDIA CORPORATION <cudatools@nvidia.com>,c=
     origin nvidia.github.io
 500 https://nvidia.github.io/libnvidia-container/stable/ubuntu20.04/amd64  Packages
     release v=1.0,o=https://nvidia.github.io/libnvidia-container,a=bionic,n=bionic,l=NVIDIA CORPORATION <cudatools@nvidia.com>,c=
     origin nvidia.github.io
 500 https://deb.nodesource.com/node_14.x focal/main amd64 Packages
     release o=Node Source,n=focal,l=Node Source,c=main,b=amd64
     origin deb.nodesource.com
 500 https://packages.microsoft.com/ubuntu/20.04/prod focal/main amd64 Packages
     release o=microsoft-ubuntu-focal-prod focal,a=focal,n=focal,l=microsoft-ubuntu-focal-prod focal,c=main,b=amd64

 ...
```

## Borrar repositorio del sistema

Se pueden eliminar repositorios que no necesitamos mediante el comando:

```bash
sudo add-apt-repository --remove ppa:<NOMBRE_REPOSITORIO>/<PPA>
```

Donde `<NOMBRE_REPOSITORIO` es el nombre del repositorio que queremos borrar,
por ejemplo en nuestra lista de arriba podríamos querer borrar `nvidia-docker`,
y `PPA` es "Personal Package Archive".

Ejemplo:

```bash
sudo add-apt-repository --remove nvidia.github.io/nvidia-docker
```

Note that we deleted the `ppa:` at the beginning, because this one was not a
repository from Ubuntu.

## Eliminar mensajes de error debido a la arquitectura

En ocasiones podemos tener un repositorio configurado que añade arquitecturas
no compatibles con nuestra máquina. En esos casos veremos un mensaje de error
como:

```bash
N: Skipping acquire of configured file 'main/binary-i386/Packages' as repository 'http://apt.postgresql.org/pub/repos/apt focal-pgdg InRelease' doesn't support architecture 'i386'
```

Para eliminar este mensaje, hay que buscar cuál es el repositorio que tenemos
mal configurado en `/etc/apt/sources.list.d`. En nuestro caso era el repositorio
oficial de PostgreSQL para la arquitectura `i386`, luego podemos buscar con el
comando:

```bash
grep postgres * | grep -v i386
```

Una vez tenemos los ficheros, los editamos añadiendo el filtro `[arch=amd64]`.
En nuestro ejemplo:

```
deb [arch=amd64] http://apt.postgresql.org/pub/repos/apt/ focal-pgdg main
```

## Añadir claves GPG de repositorios

Los repositorios proveen de una clave GPG que hay que instalar y mantener actualizada, ya que de lo
contrario podemos encontrarnos con errores al hacer `apt update`, como por ejemplo éste:

    :::bash
    W: An error occurred during the signature verification. 
    The repository is not updated and the previous index files will be used. 
    GPG error: https://packages.cloud.google.com/apt cloud-sdk InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY B53DC80D13EDEF05
    W: Failed to fetch https://packages.cloud.google.com/apt/dists/cloud-sdk/InRelease  
    The following signatures couldn't be verified because the public key is not available: NO_PUBKEY B53DC80D13EDEF05
    W: Some index files failed to download. They have been ignored, or old ones used instead.

Para instalar las claves GPG, por ejemplo valiéndonos de éste mismo ejemplo de repositorio para
Google Cloud SDK, haremos lo siguiente:

    :::bash
    wget -O- https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/cloud.google.gpg

Es decir, `wget -O-` a la clave GPG del repositorio, que habitualmente se ofrece en el mismo, un
`gpg --dearmor` a la clave y el resultado lo guardaremos en `/usr/share/keyrings/clave.gpg`.

Después de hacer esto, nos aseguraremos de que el fichero con la clave se encuentra señalado en la
configuración del repositorio, en nuestro ejemplo en `/etc/apt/sources.list.d/google-cloud-sdk.list`:

    :::debsources
    deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main
