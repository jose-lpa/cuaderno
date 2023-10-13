Title: Tipo `ENUM` en PostgreSQL
Date: 2023-10-13 16:30
Category: SQL
Tags: sql, postgresql
Slug: postgresql-enums
Authors: José L. Patiño-Andrés
Summary: Consultas relacionadas con el tipo ENUM en PostgreSQL

### Ver todos los `ENUM`s registrados en la base de datos

    :::psql
    SELECT * FROM pg_type WHERE typcategory = 'E';

### Ver todos los posibles valores para un `ENUM` determinado

Suponiendo que nuestro `ENUM` se llame "status"...

    :::psql
    SELECT unnest(enum_range(NULL::status));
