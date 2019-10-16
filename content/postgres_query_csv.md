Title: Generar CSV con PostgreSQL
Date: 2018-09-04 15:44
Category: SQL
Tags: sql, postgresql, csv
Slug: postgresql-consulta-csv
Authors: José L. Patiño-Andrés
Summary: Generación de ficheros CSV con resultados de consultas PostgreSQL.

Utilizando el siguiente comando podemos volcar los resultados de una consulta en
PostgreSQL a un fichero CSV sin salir de la consola (Bash, Zsh...):

```
psql -h [URL] -u [USUARIO] -d [NOMBRE] -c "COPY (SELECT * FROM tabla_bd) TO STDOUT WITH CSV HEADER DELIMITER ',';" > resultado.csv
```
