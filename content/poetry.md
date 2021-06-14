Title: Referencia de uso de Poetry
Date: 2021-05-06 11:23
Category: Herramientas
Tags: poetry,python,packaging,dependencias
Slug: poetry
Authors: José L. Patiño-Andrés
Summary: Referencia de uso de la herramienta de gestión de dependencias Poetry.

[Poetry](https://python-poetry.org/docs/) es otra herramienta de CLI que sirve
para gestionar dependencias y paquetes de terceros en proyectos Python.

A continuación se muestra una pequeña referencia de uso básico de Poetry.

## Instalación

Poetry viene con una especie de instalador personalizado que instala `poetry`
de manera aislada. Ésta parece ser la forma recomendada de instalarlo. Para
ello podemos seguir las instrucciones en [la documentación de Poetry](https://python-poetry.org/docs/#installation).

De otro modo, simplemente podemos instalarlo con `pip`:

    :::bash
    pip install poetry

## Configuración

Poetry tiene multitud de [parámetros de configuración](https://python-poetry.org/docs/configuration/#configuration)
que pueden ser más o menos útiles. El que más útil parece de forma inmediata es
el que nos permite declarar qué directorio se usará para los entornos virtuales
[creados automáticamente por Poetry](https://python-poetry.org/docs/configuration/#virtualenvspath-string).

Podemos por tanto especificar qué directorio queremos que Poetry use para crear
esos entornos virtuales mediante el siguiente comando de configuración:

    :::bash
    poetry config virtualenvs.path $WORKON_HOME

Esto en el caso de que tengamos definida una variable `WORKON_HOME` que apunte
al directorio deseado. Si no, simplemente escribir el directorio.

## Uso básico

### `poetry new <NOMBRE_PROYECTO>`

Genera un proyecto Python vacío, con directorio raíz `<NOMBRE_PROYECTO>`.

El directorio contiene un fichero `pyproject.toml` que gestionará todas las
dependencias.

### `poetry init`

Inicializa Poetry en un proyecto ya existente. Similar a otros como `git init`.

### `poetry add <PAQUETE>`

Añade paquetes de terceros al proyecto, instalándolos en el entorno. Ejemplos:

- `poetry add Django`: añade Django al proyecto, usando la versión que considere
  más apropiada. 
- `poetry add Django==3.0`: añade Django al proyecto, usando la versión
  especificada.
- `poetry add pytest --dev`: añade Pytest al proyecto, **como dependencia de 
  desarrollo**.

Alternativamente, se pueden añadir las dependencias a mano editando el fichero
`pyproject.toml` en su sección `[tool.poetry.dependencies]`.

### `poetry remove <PAQUETE>`

Elimina la dependencia del proyecto.

### `poetry shell`

Activa el entorno virtual creado por Poetry. En caso de que no exista aún, crea
uno nuevo en el directorio especificado por el valor de configuración
`virtualenvs.path`.

### `poetry run <COMANDO>`

Ejecuta cualquier comando que se encuentre disponible en el entorno virtual.

### `poetry install`

Instala las dependencias del proyecto. Esto puede ser de dos maneras:

1. Si existe un archivo `poetry.lock`, instala las versions allí declaradas.
2. Si no existe dicho archivo, Poetry resuelve todas las dependencias declaradas
   en `pyproject.toml`, las instala y genera el fichero `poetry.lock`.
