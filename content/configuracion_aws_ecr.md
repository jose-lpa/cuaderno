Title: Configuración de AWS ECR
Date: 2017-09-15 15:20
Category: Cloud
Tags: aws,cloud,ecr,configuración
Slug: configuracion-aws-ecr
Authors: José L. Patiño-Andrés
Summary: 3 pasos para configurar AWS ECR en máquina local.

## Pasos

1. Ejecutar:

        :::bash
        aws ecr get-login --no-include-email --region <REGIÓN_AWS>

2. Copiar y ejecutar la salida del comando anterior.
3. Ejecutar:

        :::bash
        docker run --rm --entrypoint=bash -it 1234567890.dkr.ecr.amazonaws.com/<IMAGEN>:<VERSIÓN>
