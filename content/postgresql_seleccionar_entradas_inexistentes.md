Title: Seleccionar entradas inexistentes en PostgreSQL
Date: 2021-10-15 14:57
Category: SQL
Tags: sql, postgresql
Slug: seleccionar-entradas-inexistentes-postgresql
Authors: José L. Patiño-Andrés
Summary: Seleccionar filas inexistentes en PostgreSQL

En ocasiones podemos querer comprobar si el valor de una columna en una tabla
existe en otra tabla. **Sin que exista una relación de clave entre ellas**.

Por ejemplo, podríamos tener las tabla:

    :::postgresql
    CREATE TABLE usuario (
        id serial PRIMARY KEY,
        nombre VARCHAR (50) NOT NULL,
        email VARCHAR (255) NOT NULL
    );

    CREATE TABLE post (
        id serial PRIMARY KEY,
        id_usuario INT NOT NULL,
        titulo VARCHAR (100) NOT NULL,
        contenido TEXT NOT NULL
    );

La tabla `post` tiene una columna `id_usuario` donde se debería poner el `id`
del `usuario` que ha escrito el post. Sin embargo no es una Foreign Key, con
lo cual no hay manera de saber si todos los `post.id`s concuerdan realmente con
un `usuario.id` o si son números que no existen en la tabla `usuario` y por lo
tanto son datos erróneos.

Para ver cuántos `post` contienen números en `id_usuario` que no existen en
la tabla `usuario`, podemos hacer la siguiente consulta:

    :::postgresql
    SELECT 
        post.id_usuario, 
        post.id 
    FROM post 
    LEFT JOIN usuario ON usuario.id = posts.id_usuario 
    WHERE usuario.id IS NULL;

