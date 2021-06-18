Title: Seleccionar duplicados en PostgreSQL
Date: 2018-10-12 16:32
Category: SQL
Tags: sql,postgresql,duplicados,consultas
Slug: seleccionar-duplicados-postgresql
Authors: José L. Patiño-Andrés
Summary: Seleccionar elementos duplicados en una tabla de datos en PostgreSQL

La siguiente query devuelve elementos duplicados en `table` cuyos valores para 
las columnas `column_1` y `column_2` son idénticos:

    :::postgresql
    SELECT column_1, column_2, COUNT(*) FROM my_table
    GROUP BY column_1, column_2
    HAVING count(*) > 1;

Usando esta consulta, podríamos borrar **todos** los elementos duplicados de
forma automática mediante esta otra consulta:

    :::postgresql
    DELETE FROM my_table AS main 
        USING (
            SELECT column_1, column_2, COUNT(*) FROM my_table 
            GROUP BY (column_1, column_2) 
            HAVING COUNT(*) > 1
        ) AS duplicates 
    WHERE main.column_1 = duplicates.column_1 
        AND main.column_2 = duplicates.column_2;
