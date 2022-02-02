Title: Usar datos GeoJSON para filtrar consultas en PostGIS
Date: 2022-02-02 14:08
Category: SQL
Tags: sql,postgresql,postgis,gis
Slug: postgis-intersect-geojson
Authors: José L. Patiño-Andrés
Summary: Referencia de cómo usar un fichero GeoJSON para realizar consultas en una base de datos PostGIS

Suponiendo que tengamos un fichero, o simplemente un string, con GeoJSON, como por ejemplo:

    :::json
    {
        "type": "MultiPoint", 
        "coordinates": [
            [-1.6402482999999999, 37.411843399999995], 
            [-1.645345, 37.413398199999996], 
            [-1.6453499999999999, 37.4129673], 
            [-1.6453347, 37.41289], 
            [-1.6455814, 37.4127428], 
            [-1.6455834, 37.4136486],
            [-1.6402482999999999, 37.411843399999995]
        ]
    }

y una base de datos PostGIS donde tengamos una tabla llamada `field` con un campo `geometry`,
podríamos hacer una consulta para obtener todos los `field`s que intersecten con la geometría
definida en el GeoJSON:

    :::psql
    SELECT field.id, field.name 
    FROM field 
    WHERE ST_Intersects(field_zone.geometry, ST_GeomFromGeoJSON('
        {
            "type": "MultiPoint", 
            "coordinates": [
                [-1.6402482999999999, 37.411843399999995], 
                [-1.645345, 37.413398199999996], 
                [-1.6453499999999999, 37.4129673], 
                [-1.6453347, 37.41289], 
                [-1.6455814, 37.4127428], 
                [-1.6455834, 37.4136486],
                [-1.6402482999999999, 37.411843399999995]
            ]
        }'));

Los métodos de PostGIS utilizados son `ST_Intersects`, que comprueba intersección entre 2
objetos `geometry` o `geography`, y `ST_GeomFromGeoJSON`, que construye un objecto `geometry`
a partir de su representación en GeoJSON.

Se puede consultar la información detallada de ambas funciones en la documentación de PostGIS:

- [`ST_Intersects`](https://postgis.net/docs/ST_Intersects.html)
- [`ST_GeomFromGeoJSON`](https://postgis.net/docs/ST_GeomFromGeoJSON.html)
