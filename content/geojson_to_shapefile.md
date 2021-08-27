Title: Librería GDAL para desarrollo GIS
Date: 2021-07-29 17:23
Category: Programación
Tags: gis,python,geojson
Slug: gdal-libreria-gis
Authors: José L. Patiño-Andrés
Summary: Instalación y ejemplos de uso de la librería GDAL para desarrollo de aplicaciones GIS

A continuación se muestra una pequeña referencia sobre la instalación y algunos
ejemplos de uso de la librería [GDAL](https://gdal.org).

## Instalación

Necesitaremos la librería base del sistema, escrita en C++, la cual es usada
por GDAL-Python durante su compilación. La podemos instalar mediante `apt` en 
sistemas Debian:

    :::bash
    sudo apt-get install libgdal1-dev

Una vez instalada la librería del sistema ya podemos instalar GDAL-Python. Es
mucho mejor utilizar `pygdal`, que es una versión actualizada más amigable con
el sistema de instalación de `pip`:

    :::bash
    pip install pygdal=="`gdal-config --version`.*"

Con eso ya debería ser posible usar GDAL en nuestros programas Python.

## Ejemplos de uso

### Conversión de un fichero GeoJSON a ESRI Shapefile

Podemos convertir un fichero de datos en formato [GeoJSON](https://geojson.org)
a [ESRI Shapefile](https://gdal.org/drivers/vector/shapefile.html) con GDAL de
la siguiente manera:

    :::python
    from osgeo import gdal 

    source = gdal.OpenEx("/ruta/al/fichero.json")
    result = gdal.VectorTranslate("result.shp", source, format="ESRI Shapefile")

Esto generará por supuesto una serie de ficheros, ya que Shapefile se compone de
varios ficheros aparte del `.shp`:

    :::bash
    result.dbf
    result.prj
    result.shp
    result.shx
