Title: Generación de claves para cifrado de datos
Date: 2021-09-17 14:10
Category: Seguridad
Tags: openssl,ssh,cifrado,claves
Slug: generacion-claves
Authors: José L. Patiño-Andrés
Summary: Breve reseña de herramientas para generación de claves de cifrado

## Generación de un par de claves OpenSSL RSA

### Generar clave RSA privada

    :::shell
    openssl genrsa -<CIFRADO> -out <NOMBRE_FICHERO> <NUM_BITS>

Donde:

* `CIFRADO` es el algoritmo de cifrado usado, por ejemplo `-des3`.
* `NOMBRE_FICHERO` es el nombre del fichero en el que se guardará la clave
  privada, por ejemplo simplemente `private.pem`.
* `NUM_BITS` es el número de bits que tendrá la clave, por ejemplo `2048`.

### Generar clave RSA pública

Una vez tenemos nuestra clave privada en un fichero, `private.pem` por ejemplo,
podemos exportar la clave pública que será entregada a otros usuarios o 
sistemas que necesiten cifrar datos para nosotros:

    :::shell
    openssl rsa -in <FICHERO_CLAVE_PRIVADA> -outform <FORMATO> -pubout -out <FICHERO_CLAVE_PÚBLICA>

Donde:

* `FICHERO_CLAVE_PRIVADA`: el fichero donde tenemos la clave privada, en nuestro
  ejemplo era `private.pem`.
* `FORMATO`: el formato de salida, en nuestro caso el valor sería `PEM`.
* `FICHERO_CLAVE_PÚBLICA`: el fichero en que queremos guardar la clave pública,
  por ejemplo `public.pem`.
