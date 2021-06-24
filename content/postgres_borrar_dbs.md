Title: Borrar múltiples DBs en PostgreSQL
Date: 2021-06-24 10:06
Category: SQL
Tags: postgresql,trucos
Slug: borrar-multiples-dbs-postgres
Authors: José L. Patiño-Andrés
Summary: Cómo borrar múltiples bases de datos de nombres similares.

En el caso de que tengamos múltiples bases de datos llamadas de forma similar,
por ejemplo como resultado de tests automatizados que las han ido creando, se
puede facilitar el borrado de dichas bases de datos con la siguiente consulta:

    :::postgresql
    SELECT 'DROP DATABASE "'||datname||'";' 
    FROM pg_database 
    WHERE datname LIKE 'my_project_test_%';

En el caso de que tuviésemos muchas bases de datos con nombres del estilo
`my_project_test_1234`, esto nos devolvería un resultado tal que así:

    :::postgresql
                           ?column?
    ----------------------------------------------
     DROP DATABASE "my_project_test_101534";
     DROP DATABASE "my_project_test_101622";
     DROP DATABASE "my_project_test_101731";
     DROP DATABASE "my_project_test_101750";
     DROP DATABASE "my_project_test_101889";
     DROP DATABASE "my_project_test_102287";
     DROP DATABASE "my_project_test_102299";
     DROP DATABASE "my_project_test_102335";
     DROP DATABASE "my_project_test_102408";
     DROP DATABASE "my_project_test_102453";
     DROP DATABASE "my_project_test_103004";
     DROP DATABASE "my_project_test_103091";
     DROP DATABASE "my_project_test_104146";
     DROP DATABASE "my_project_test_108842";
     DROP DATABASE "my_project_test_108865";
     DROP DATABASE "my_project_test_109633";
    (16 rows)

Entonces borrarlas todas sería muy fácil: basta con copiar ese bloque
de consultas y pegarlo en la misma shell `psql`.
