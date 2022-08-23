Title: Referencia de STAC API
Date: 2022-08-21 10:15
Category: GIS
Tags: gis,api,stac,servicios
Slug: referencia-stac-api
Authors: José L. Patiño-Andrés
Summary: Breve referencia de USO de STAC API

A continuación se muestran una serie de peticiones que se pueden hacer a una API
STAC para realizar determinadas acciones comunes.

Éste documente pretende ser una referencia rápida para realizar peticiones de
manera inmediata. Las especificaciones completas de STAC API están disponibles
[en GitHub](https://github.com/radiantearth/stac-spec).

## Ver Colecciones de datos

    :::bash
    curl -X GET "https://my-stac-api.com/collections" \
         -H "Authorization: Bearer <API_TOKEN>

## Crear nueva Colección de datos

    :::bash
    curl -X POST "https://my-stac-api.com/collections" \
         -H "Authorization: Bearer <API_TOKEN>" \
         -d '{"id": "<NOMBRE_COLECCIÓN>", \
              "type": "Collection", \
              "license": "proprietary", \
              "description": "<DESCRIPCIÓN_COLECCIÓN>", \
              "stac_version": "1.0.0", \
              "stac_extensions": [], \
              "extent": {"spatial": {"bbox": [[-180.0, -90.0, 180.0, 90.0]]}, "temporal": {"interval": [["1970-01-01T00:00:00Z", null]]}}}'

