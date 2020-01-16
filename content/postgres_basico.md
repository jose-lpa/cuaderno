Title: PostgreSQL básico
Date: 2012-09-23 11:23
Category: SQL
Tags: sql, postgresql
Slug: postgresql-basico
Authors: José L. Patiño-Andrés
Summary: Comandos básicos de PostgreSQL.

### Crear nuevo usuario

```
CREATE USER <user> WITH PASSWORD '<password>';
```

### Asignar privilegios al usuario

```
GRANT ALL PRIVILEGES ON DATABASE <database> TO <user>;
```

Nótese que esta operación **no otorga permisos al usuario para crear nuevas
bases de datos**.

### Dar permiso al usuario para crear nuevas bases de datos

```
ALTER USER <user> CREATEDB;
```

### Backup simple de una base de datos (en consola)

```
pg_dump -U <user> -c <database> > database_dump.sql
```

Nótese que este comando vuelca la base de datos tal y como es, con sus
usuarios y sus permisos. Si se intenta cargar esta base de datos en otro
cluster PostgreSQL distinto al que la creó, habrá que crear primero dichos
usuarios. Para evitar esto, se pueden usar los siguientes parámetros al final
del comando `pg_dump`: `--no-owner --no-privileges`.

Para cargar la base de datos de nuevo, haremos:

```
psql <database> < database_dump.sql
```
