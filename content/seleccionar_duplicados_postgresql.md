Title: Seleccionar duplicados en PostgreSQL
Date: 2018-10-12 16:32
Category: SQL
Tags: sql,postgresql,duplicados,consultas
Slug: seleccionar-duplicados-postgresql
Authors: José L. Patiño-Andrés
Summary: Seleccionar elementos duplicados en una tabla de datos en PostgreSQL

La siguiente query devuelve elementos duplicados en `table` cuyos valores para 
las columnas `column_1` y `column_2` son idénticos:

    :::sql
    SELECT column_1, column_2, COUNT(*) FROM my_table
    GROUP BY column_1, column_2
    HAVING count(*) > 1;
