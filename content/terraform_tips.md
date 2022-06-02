Title: Resolución de problemas en Terraform
Date: 2022-06-01 09:50
Category: Cloud
Tags: terraform,iac
Slug: terraform-troubleshooting
Authors: José L. Patiño-Andrés
Summary: Referencia de problemas encontrados en Terraform y pasos para su resolución

## Error: Resource instance managed by newer provider version

Éste error puede saltar cuando el estado remoto de Terraform tiene asignada una
versión del proveedor para determinado recurso que es diferente (más nueva) a
la que tenemos en el estado local.

Podemos comprobar qué diferencias de versión hay mediante el siguiente comando:

```bash
terraform state pull
```

El comando nos mostrará un JSON (podemos redirigir la salida y guardarlo en un
fichero para leerlo más cómodamente) en el que tenemos que buscar el nombre de
nuestro recurso conflictivo y comparar el campo `schema_version`. Normalmente,
es posible que el campo muestre un valor de `1`, mientras que en nuestro local
ese mismo valor será de `0`.

La solución más efectiva para este problema es borrar directamente el recurso
en remoto. **"Borrar", no destruir**. Ésto se hace borrando el registro para
dicho recurso. Por ejemplo, veamos cómo borraríamos un recurso de tipo
`random_password`:

```bash
terraform state rm \
    module.my_cloud_run_app.module.postgres_database.random_password.password[0]
```

Una vez borrado, podemos volver a importar el recurso, que todavía existe
porque recordemos que **no ha sido destruído**, con el siguiente comando:

```bash
terraform import \
    --var-file=my_environment.tfvars \
    module.user_management_api.module.postgres_database.random_password.password[0] \
    <VALOR_DEL_PASSWORD>
```

**Nota:** `<VALOR_DEL_PASSWORD>` es el ID de recurso de Terraform para el
recurso `random_password`, de acuerdo con la [documentación de Terraform](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password#import).

Una vez hecho esto, ya podríamos volver a usar los comandos de `plan` y `apply`.
