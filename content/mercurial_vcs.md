Title: Uso básico de Mercurial
Date: 2010-11-21 15:30
Category: Herramientas
Tags: scm,vcs,mercurial
Slug: mercurial-vcs
Authors: José L. Patiño-Andrés
Summary: Configuración y uso básico del gestor de código Mercurial.

## Preparar Mercurial

Creamos/editamos el fichero `~/.hgrc` en sistemas \*nix, o `mercurial.ini` en 
sistemas Windows:

    :::ini
    [ui]
    username = José L. Patiño <jose@sharklasers.com>
    editor = vim

    [extensions]
    hgext.graphlog =

## Inicializar un proyecto

- `hg init <PROYECTO>` (si no especificamos `<PROYECTO>`, el presente directorio
  será considerado la raíz del proyecto.
- `cd project`

## Añadir/eliminar ficheros

- `hd add` (`hg addremove`)
- `hg commit`

## Guardar cambios

- `hg status` muestra el estado actual.
- `hg diff` muestra detalle de los cambios realizados.
- `hg commit`
- `hg push` sube los cambios al repositorio. Añadir `--new-branch` si la rama no 
  existe.

## Ver historial

- `hg log` presenta una lista de cambios ordenados en el tiempo.
- `hg log -p -r 3`
    - `-p` o `--patch` muestra un diff en las revisiones.
    - `-r` o `--revision` muestra una revisión específica.

## Trabajar con ramas de desarrollo

- `hg branch feature/nueva_feature` crea una nueva rama llamada 
  `feature/nueva_feature`. Si no añadimos el nombre de la rama, Mercurial nos
  mostrará el nombre de la rama actual.
- `hg update default` nos devuelve a la rama `default`, que es la rama inicial
  con la que se crea el repositorio. Cambiando `default` por el nombre de la
  rama que queramos, podemos movernos por las distintas ramas.
- `hg merge feature` unifica los cambios de la rama llamada `feature` en la
  rama actual en la que nos encontremos.

## Descargar un repositorio ya creado

- `hg clone ssh://user@server:port//project/directory`

Si queremos descargar sólo una rama, podemos añadir al comando anterior la
opción `u feature`, donde `feature` es el nombre de la rama que queremos.

## Empaquetar código

- `hg archive [-t/--type] destino`
    - `-t` o `--type` puede ser `-files` (por defecto), `-tar`, `-tbz2`, `-tgz`,
      `-uzip` o `-zip`.

## Configuración

La configuración de Mercurial se gestiona en el fichero `~/.hgrc` en sistemas 
Unix y en el archivo `mercurial.ini` en sistemas Windows.

A continuación una muestra de algunas [opciones de configuración](https://www.mercurial-scm.org/doc/hgrc.5.html):

    :::ini
    [ui]
    username = John Doe <john.doe@sharklasers.com>
    editor = vim

    [extensions]
    hgext.extdiff =
    hgext.graphlog = 
    color = 
    shelve =
    pager =

    [extdiff]
    cmd.cdiff = colordiff  # Needs 'colordiff' installed in the system.
    opts.cdiff

    [color]
    status.unknown = yellow bold
    diff.file_a = red bold underline
    diff.file_b = green bold underline
    diff.hunk = blue bold
    diff.deleted = red bold
    diff.inserted = green bold
