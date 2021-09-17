Title: Librería GeoPandas para análisis de datos GIS en Python
Date: 2021-08-18 15:04
Category: Programación
Tags: gis,python,geojson,pandas,librerías,geopandas
Slug: geopandas-libreria
Authors: José L. Patiño-Andrés
Summary: Descripción y ejemplos de uso de la librería GeoPandas para análisis de datos GIS en Python

A continuación se muestra una serie de ejemplos útiles de uso de la librería
Python [GeoPandas](https://geopandas.org/index.html).


## Activar soporte para ficheros KML

GeoPandas utiliza la librería [Fiona](http://fiona.readthedocs.io/en/latest/manual.html)
para cargar datos desde ficheros.

Por defecto, Fiona tiene el soporte para ficheros KML desactivado. Por lo tanto
si usamos la función `read_file()` de GeoPandas con un fichero KML nos devuelve
un error.

Para activar el soporte de estos ficheros KML, podemos ejecutar la siguiente
sentencia en nuestro programa Python:

    :::python
    gpd.io.file.fiona.drvsupport.supported_drivers["KML"] = "rw"


## Calcular áreas de polígonos en el dataframe

Suponiendo que tenemos un `GeoDataFrame` y queremos obtener las áreas de los
polígonos en el mismo:

    :::python
    import geopandas as gpd


    dataframe = gpd.read_file("my_data.geojson")

    for entry in dataframe.to_crs("EPSG:3587").area:
        print(entry)

Atención al cambio de sistema de coordenadas por `3587`, [sistema Mercator esférico](https://epsg.io/3857).
Esto cambia la proyección de las geometrías en la columna `geometry` y nos dará
las áreas en **metros cuadrados**.


## Añadir columna al dataframe

Tomando como ejemplo los cálculos anteriores para las áreas de los polígonos,
imaginemos ahora que queremos añadir dichas áreas a las filas del dataframe:

    :::python
    from typing import Generator

    import geopandas as gpd
    

    def get_areas(dataframe: gpd.GeoDataFrame) -> Generator[float, None, None]:
        """Generador que arroja las áreas de los polígonos en el dataframe."""
        assert dataframe.empty is False

        for area in dataframe.to_crs("EPSG:3587").area:
            yield area / 10000

    
    dataframe = gpd.read_file("my_data.geojson")

    dataframe["area"] = [area for area in get_areas(dataframe)]


## Selección de datos filtrados

Podemos seleccionar datos de las filas de un dataframe que cumplan una condición
determinada, usando el método `.loc`.

Por ejemplo, para seleccionar únicamente aquellas filas que tengan una geometría
**inválida** haríamos:

    :::python
    for invalid in dataframe.loc[dataframe["geometry"].is_valid == False].name:
        # ...operaciones con los datos de la columna `name`...
