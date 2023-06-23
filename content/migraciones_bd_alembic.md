Title: Migraciones de base de datos con Alembic
Date: 2021-04-03 13:21
Category: SQL
Tags: migraciones,python,sql,rdbms,sqlalchemy
Slug: migraciones-bd-alembic
Authors: José L. Patiño-Andrés
Summary: Pequeño tutorial de la herramienta Python de migraciones Alembic.

Manual de uso de la herramienta [Alembic](https://alembic.sqlalchemy.org/en/latest/index.html).

## Inicio del entorno Alembic

### Instalación

Instalar mediante pip:

    :::bash
    pip install alembic

### Inicialización y puesta a punto

Inicializar el entorno de trabajo de Alembic:

    :::bash
    cd directorio/del/proyecto
    alembic init alembic

O también, **si queremos usar Alembic con un controlador de BD asíncrono, el comando es:

    :::bash
    alembic init -t async alembic

Esto crea un nuevo directorio llamado `alembic/` y un fichero `alembic.ini`,
ambos en el `directorio/del/proyecto`. El directorio `alembic tiene este
contenido:

    :::bash
    directorio/del/proyecto/
    ├── alembic
    │   ├── env.py
    │   ├── README
    │   ├── script.py.mako
    │   └── versions
    │       ├── 15d42b4f39ae_migración_1.py
    │       ├── 802aaa051d6b_migración_2.py
    │       └── ...
    ├── alembic.ini
    ...


El directorio `versions` inicialmente está vacío y es ahí donde se crearán los
ficheros Python con las migraciones.

El fichero [`alembic.ini` ](https://alembic.sqlalchemy.org/en/latest/tutorial.html#editing-the-ini-file)
es un fichero de autoconfiguración inicial que es leído por Alembic cuando se le
invoca desde la línea de comandos. Podemos configurar ahí el logging de Python
o la conexión a la base de datos, por ejemplo, aunque esto último es mucho mejor
hacerlo en el `env.py` de forma dinámica.

En el fichero `alembic.ini` configuraremos el directorio donde están los scripts
en caso de que no esté configurado ya:

    :::ini
    [alembic]

    script_location = directorio/del/proyecto/alembic


El fichero `env.py` es donde haremos las modificaciones de configuración
necesarias.

- Añadir el directorio base del proyecto al `PYTHONPATH`:

        :::python
        BASE_DIR = os.path.abspath(os.path.join(__file__, os.path.pardir, os.path.pardir, os.path.pardir))
        sys.path.append(BASE_DIR)

- Cargar la URI de conexión a la base de datos desde una variable de entorno:

        :::python
        config.set_main_option('sqlalchemy.url', os.getenv("SQLALCHEMY_DATABASE_URI"))

- Añadir los metadatos de los models SQLAlchemy del proyecto para soporte de
  autogeneración de migraciones:

        :::python
        from proyecto.database import Base
        from proyecto import models
        # Importar los modelos.
        target_metadata = Base.metadata

Con esto ya deberíamos estar listos para usar Alembic y hacer migraciones con 
nuestra base de datos.


## Comandos de uso

### Actualizar base de datos a su estado más reciente

    :::bash
    alembic upgrade head

### Crear nueva migración

    :::bash
    alembic revision -m "Descripción de la migración"

O dejar que Alembic genere el fichero Python de migración automáticamente:

    :::bash
    alembic revision --autogenerate -m "Descripción de la migración"

### Actualizar base de datos a una revisión específica

    :::bash
    alembic upgrade <ID_REVISIÓN>

El `<ID_REVISIÓN>` se muestra en los ficheros de revisión, en el docstring
inicial, bajo el parámetro `Revision ID:`.

### Actualizar a una revisión relativa

Podemos migrar por pasos, hacia adelante o hacia atrás. Algunos ejemplos:

    :::bash
    alembic upgrade +2
    alembic upgrade <REVISION_NUMBER>+2
    alembic downgrade -1
    ...

### Ver en qué revisión se encuentra actualmente la base de datos

    :::bash
    alembic current

### Ver histórico de revisiones.

    :::bash
    alembic history --verbose
