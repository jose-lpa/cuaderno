Title: Funciones JSON en PostgreSQL
Date: 2023-06-07 14:45
Category: SQL
Tags: sql, postgresql
Slug: postgresql-json
Authors: José L. Patiño-Andrés
Summary: Funcionalidades de tipos JSON en PostgreSQL.

## Acceder a un campo en una columna JSONB

Supongamos que tenemos la tabla `items`, con la columna `properties` que es una columna `JSONB` con
una fila que contiene estos datos:

    :::json
    {
        "datetime": "2023-05-02T11:28:29.341176",
        "name": "Item 32",
        "type": "Petrol Engine",
        "price": 1043.25
    }

Si quisíéramos acceder al campo `name` del item con `id` 12345, la consulta sería:

    :::psql
    SELECT id, properties->>'name' FROM item WHERE id = 12345;


## Modificar valor de un campo en una columna JSONB

Si en la columna mencionada anteriormente quisiéramos ahora cambiar el campo `price` de la fila
de dicho item 12345, la consulta sería:

    :::psql
    UPDATE item 
        SET properties = jsonb_set(
            properties, '{price}', 
            to_jsonb(1043.25)
        ) 
    WHERE id = 12345;
