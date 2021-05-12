Title: Referencia de uso de Pipenv
Date: 2021-05-05 16:23
Category: Herramientas
Tags: pipenv,python,packaging,dependencias
Slug: pipenv
Authors: José L. Patiño-Andrés
Summary: Referencia de uso de la herramienta de gestión de dependencias Pipenv.

[Pipenv](https://pypi.org/project/pipenv/) es una herramienta de CLI que
soluciona algunos problemas derivados de la gestión de dependencias (paquetes
externos) en proyectos Python.

A continuación se añade una pequeña referencia de cómo usar Pipenv.

## Instalación

Basta con instalar Pipenv mediante `pip`:

    :::bash
    pip install pipenv

## Configuración

Realmente no es necesario hacer ningún cambio de configuración para Pipenv tal
como viene por defecto pero, si se quiere, se pueden usar las variables de
entorno que se describen en [la documentación de Pipenv](https://pipenv.pypa.io/en/latest/advanced/#configuration-with-environment-variables).

Hay que recordar que Pipenv utiliza [pew](https://pypi.org/project/pew/), con
lo cual si tenemos definida una variable `WORKON_HOME` donde guardamos todos
nuestros virtualenvs, Pipenv utilizará ésta variable para guardar los suyos.
Esto es útil por ejemplo cuando estamos usando Virtualenvwrapper.

## Uso básico

### `pipenv shell`

Crea un entorno virtual, si todavía no existe, para el proyecto actual y lo
inicializa. Pipenv instalará en él todos los paquetes.

### `pipenv install <PAQUETE>`

Instala `<PAQUETE>` y lo añade a los ficheros `Pipfile` y `Pipfile.lock`.

Ejemplos:

- `pipenv install flask==1.1.2`: instala Flask en su versión 1.1.2.
- `pipenv install flask`: instala Flask sin especificar una versión en concreto
  (usará la última versión disponible).
- `pipenv install -e git+https://github.com/requests/requests.git#egg=requests`: 
  instala la librería `requests` directamente desde su VCS (GitHub).
- `pipenv install pytest --dev`: instala pytest en el entorno de desarrollo.
- `pipenv install --ignore-pipfile`: instala lo que haya definido en el fichero
  `Pipfile.lock` (debe existir este fichero previamente, ver siguiente comando
  `pipenv lock`.
- `pipenv install --dev`: instala el entorno de desarrollo.
- `pipenv install --deploy`: útil para despliegues o creación de imágenes de
  contenedores. Aborta la ejecución si `Pipfile.lock` no está actualizada o la
  versión de Python es incorrecta.

Un comando útil puede ser:

    :::bash
    pipenv install --deploy --system --ignore-pipfile

Este comando instalaría todos los paquetes necesarios para el proyecto siguiendo
las instrucciones en el fichero `Pipfile.lock`, abortando si hay problemas con
`Pipfile.lock` o si la versión de Python es incorrecta, y además lo instalaría
todo en el Python del sistema, sin usar entornos virtuales. Esto puede ser muy
útil en la creación de imágenes de contenedores.

### `pipenv lock`

Crea o actualiza el fichero `Pipfile.lock` que contiene el registro bloqueado de
las versiones de paquetes a usar en el proyecto. Este fichero no debe ser 
editado manualmente, sino siempre creado/actualizado con el comando 
`pipenv lock`.

### `pipenv graph`

Imprime en pantalla un árbol de dependencias mostrando los paquetes del 
proyecto.

Se puede usar `pipenv graph --reverse` a la hora de resolver conflictos de
dependencias, si es el caso.

Para comprobar si tenemos instalado un determinado paquete, podemos usar el
comando

    :::bash
    pipenv graph | grep <PAQUETE>

### `pipenv run <COMANDO>`

Ejecuta comandos que existan en el entorno creado por Pipenv sin necesidad de
entrar en el entorno virtual.

### `pipenv sync`

Deja el entorno Python como está especificado en `Pipfile.lock`. Se puede usar
`pipenv sync --system` para no usar entornos virtuales e instalar todo en el
Python del sistema. Esto es útil para crear imágenes de contenedores.
