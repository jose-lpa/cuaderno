Title: Uso básico de GPG
Date: 2021-06-5 15:34
Category: Sistema
Tags: gpg,seguridad,cifrado
Slug: gpg-basico
Authors: José L. Patiño-Andrés
Summary: Uso básico de claves GPG

## Generar claves

    :::bash
    gpg --gen-key

Seguir las instrucciones para generar las claves GPG.

Se puede también ejecutar con selección manual de opciones con el comando:

    :::bash
    gpg --full-generate-key

## Listar claves existentes

### Claves públicas

    :::bash
    gpg --list-keys

Se mostrará una lista similar a:

    :::bash
    gpg: checking the trustdb
    gpg: marginals needed: 3  completes needed: 1  trust model: pgp
    gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
    gpg: next trustdb check due at 2023-06-05
    /home/jose/.gnupg/pubring.kbx
    -----------------------------
    pub   rsa4096 2020-02-04 [SC] [expires: 2023-06-05]
          635BCDD1CFC5981624523F4174FF027519B21EDF
    uid           [ultimate] José Luis Patiño Andrés <jose.patino-andres@protonmail.com>
    sub   rsa4096 2020-02-04 [E] [expires: 2023-06-05]

### Claves privadas

    :::bash
    gpg --list-secret-keys

## Mostrar fingerprints

    :::bash
    gpg --fingerprint your_email@address.com

Mostrará una salida como:

    :::bash
    pub   4096R/311B1F84 2020-02-04
          Key fingerprint = CB9E C70F 2421 AF06 7D72  F980 8287 6A15 311B 1F84
    uid                  José Luis Patiño Andrés <jose.patino-andres@protonmail.com>
    sub   4096R/8822A56A 2023-06-05

## Cifrar ficheros

### Cifrado

Para cifrar un fichero con nuestra clave GPG ejecutamos el comando:

    :::bash
    gpg --encrypt --local-user <USUARIO_QUE_CIFRA> --recipient <USUARIO_QUE_RECIBE_FICHERO> <FICHERO>

Así ciframos con nuestra clave un fichero que podrá abrir sólo el usuario que
hayamos especificado (podemos ser nosotros mismos).

Esto generará un fichero igual al anterior, con la extensión `.gpg`.

El mismo comando, más corto:

    :::bash
    gpg -e -u <USUARIO_QUE_CIFRA> -r <USUARIO_QUE_RECIBE_FICHERO> <FICHERO>

### Descifrado

Para descifrar, simplemente:

    :::bash
    gpg <FICHERO>

## Exportación de claves

### Exportar claves privadas

    :::bash
    gpg --export-secret-keys > private.key

### Exportar claves públicas

    :::bash
    gpg --export > public.key

## Importación de claves

### Importar clave privada

    :::bash
    gpg --allow-secret-key-import --import private.key

### Importar clave pública

    :::bash
    gpg --import public.key

## Borrar claves

Primero hay que borrar las claves privadas:

    :::bash
    gpg --delete-secret-keys <USUARIO>

Y luego las públicas:

    :::bash
    gpg --delete-key <USUARIO>
