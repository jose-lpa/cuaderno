Title: PostgreSQL básico
Date: 2012-09-23 11:23
Category: SQL
Tags: sql, postgresql
Slug: postgresql-basico
Authors: José L. Patiño-Andrés
Summary: Comandos básicos de PostgreSQL.

### Crear nuevo usuario

    :::psql
    CREATE USER <user> WITH PASSWORD '<password>';

### Asignar privilegios al usuario

    :::psql
    GRANT ALL PRIVILEGES ON DATABASE <database> TO <user>;

Nótese que esta operación **no otorga permisos al usuario para crear nuevas
bases de datos**.

### Dar permiso al usuario para crear nuevas bases de datos

    :::psql
    ALTER USER <user> CREATEDB;

### Dar permisos de superusuario

Podemos dar todos los permisos de gestión a un usuario normal con el comando:

    :::psql
    ALTER ROLE <user> SUPERUSER;

Esto nos permite ejecutar diversos comandos, más allá de crear y utilizar
bases de datos, con un usuario "normal", como por ejemplo añadir extensiones
a una base de datos existente:

    :::psql
    CREATE EXTENSION postgis;

Comando que no podríamos ejecutar sin ser superusuario.

### Backup simple de una base de datos (en consola)

    :::bash
    pg_dump -U <user> -c <database> > database_dump.sql

Nótese que este comando vuelca la base de datos tal y como es, con sus
usuarios y sus permisos. Si se intenta cargar esta base de datos en otro
cluster PostgreSQL distinto al que la creó, habrá que crear primero dichos
usuarios. Para evitar esto, se pueden usar los siguientes parámetros al final
del comando `pg_dump`: `--no-owner --no-privileges`.

Para cargar la base de datos de nuevo, haremos:

    :::bash
    psql <database> < database_dump.sql

### Comprobar tamaño de una base de datos

    :::psql
    SELECT pg_size_pretty(pg_database_size('nombre_base_de_datos'));

### Comprobar tamaño de una de las tablas de la base de datos

    :::psql
    SELECT pg_size_pretty(pg_total_relation_size('nombre_tabla'));

### Activar esquema para un determinado usuario

En ocasiones existen tablas en nuestra base de datos que nuestro usuario actual
no es capaz de ver cuando ejecuta `\d`. Esto sucede porque esas tablas son de un
esquema que nuestro usuario no tiene asignado en el `search_path`. Podemos hacer
un cambio para que pueda ver todo lo relacionado con ese esquema, así:

    :::psql
    SET search_path = <nuevo_esquema>, public;
