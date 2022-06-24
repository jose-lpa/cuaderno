Title: Comandos útiles de Git
Date: 2022-06-24 22:00
Category: Terminal
Tags: comandos,git,vcs
Slug: comandos-utiles-git
Authors: José L. Patiño-Andrés
Summary: Referencia de comandos útiles con el sistema de control de versiones Git

## Diff con estadísticas de directorios afectados por cambios

Se puede ver qué porcentaje del total de cambios entre versiones corresponde a
cada directorio mediante el uso de `git diff` con el siguiente comando:

    :::bash
    git diff --dirstat=files,0 <REVISIÓN>
    
Donde:
    - `files` ordena a Git analizar los cambios teniendo en cuenta los ficheros
      afectados. Ésta es la forma computacionalmente más barata de sacar las
      estadísticas. Otras formas son `cumulative` y `lines`, que tienen en
      cuenta las líneas cambiadas y los cambios acumulados en subdirectorios,
      respectivamente.
    - `0` ordena a Git tener cuenta cualquier porcentaje de cambios, por
      pequeño que éste sea. De lo contrario, Git sólo analizará aquellos 
      ficheros que tengan **al menos un 3% de cambios**.
    - `REVISIÓN` indica contra qué revisión del repositorio queremos comparar
      el estado actual.

Por ejemplo, si queremos ver qué directorios fueron afectados por cambios en
**el último commit**, haremos:

    :::bash
    git diff --dirstat=files,0 HEAD~1

     10.3% src/app/config
     80.0% src/app/core
      9.7% docs/

Éste sería el resultado para un commit que contiene cambios en el directorio
`src/` (hemos cambiado código) y en el directorio `docs/` (hemos actualizado
la documentación del proyecto).

Si por ejemplo queremos ver qué directorios de nuestra rama actual han sufrido
cambios con respecto a la rama `main`, el comando sería:

    :::bash
    git diff --dirstat=files,0 main
