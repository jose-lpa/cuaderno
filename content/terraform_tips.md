Title: Resolución de problemas en Terraform
Date: 2022-06-01 09:50
Category: Cloud
Tags: terraform,iac
Slug: terraform-troubleshooting
Authors: José L. Patiño-Andrés
Summary: Referencia de problemas encontrados en Terraform y pasos para su resolución

## Modo depuración

Podemos ejecutar Terraform con diferentes modos de _logging_ de manera que 
podamos ver con más detalle las operaciones que está realizando a bajo nivel:

    :::bash
    TF_LOG=DEBUG terraform plan

Los distintos modos disponibles son:

- `TRACE`
- `DEBUG`
- `INFO`
- `WARN`
- `ERROR`

## Desbloquear estado remoto de Terraform

Terraform guarda el estado de la infraestructura remotamente, de manera que 
múltiples usuarios puedan ver el estado actual. Cuando un usuario inicia una
operación, Terraform bloquea el estado remoto, así nadie más puede operar al
mismo tiempo y provocar errores o conflictos.

A veces una operación falla y el estado remoto puede quedar bloqueado. Ésto se
puede solucionar con el comando de desbloqueo:

    :::bash
    terraform force-unlock <LOCK_ID>

Donde `LOCK_ID` es el ID del estado. Éste ID lo podremos ver en el mensaje de
error que nos notifique que el estado ha sido bloqueado, por ejemplo:

    :::bash
    ╷
    │ Error: Error acquiring the state lock
    │ 
    │ Error message: writing "gs://terraform-backend/my_project/staging/default.tflock" failed: googleapi: Error 412: At least one of the pre-conditions you specified did not hold., conditionNotMet
    │ Lock Info:
    │   ID:        1677955805365913
    │   Path:      gs://terraform-backend/my_project/staging/default.tflock
    │   Operation: OperationTypePlan
    │   Who:       jose@planck
    │   Version:   1.0.0
    │   Created:   2022-06-01 18:50:05.296016717 +0000 UTC
    │   Info:      
    │ 
    │ 
    │ Terraform acquires a state lock to protect the state from being written
    │ by multiple users at the same time. Please resolve the issue above and try
    │ again. For most commands, you can disable locking with the "-lock=false"
    │ flag, but this is not recommended.
    ╵

## Error: Resource instance managed by newer provider version

Éste error puede saltar cuando el estado remoto de Terraform tiene asignada una
versión del proveedor para determinado recurso que es diferente (más nueva) a
la que tenemos en el estado local.

Podemos comprobar qué diferencias de versión hay mediante el siguiente comando:

    :::bash
    terraform state pull

El comando nos mostrará un JSON (podemos redirigir la salida y guardarlo en un
fichero para leerlo más cómodamente) en el que tenemos que buscar el nombre de
nuestro recurso conflictivo y comparar el campo `schema_version`. Normalmente,
es posible que el campo muestre un valor de `1`, mientras que en nuestro local
ese mismo valor será de `0`.

La solución más efectiva para este problema es borrar directamente el recurso
en remoto. **"Borrar", no destruir**. Ésto se hace borrando el registro para
dicho recurso. Por ejemplo, veamos cómo borraríamos un recurso de tipo
`random_password`:

    :::bash
    terraform state rm \
        module.my_cloud_run_app.module.postgres_database.random_password.password[0]

Una vez borrado, podemos volver a importar el recurso, que todavía existe
porque recordemos que **no ha sido destruído**, con el siguiente comando:

    :::bash
    terraform import \
        --var-file=my_environment.tfvars \
        module.user_management_api.module.postgres_database.random_password.password[0] \
        <VALOR_DEL_PASSWORD>

**Nota:** `<VALOR_DEL_PASSWORD>` es el ID de recurso de Terraform para el
recurso `random_password`, de acuerdo con la [documentación de Terraform](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password#import).

Una vez hecho esto, ya podríamos volver a usar los comandos de `plan` y `apply`.
