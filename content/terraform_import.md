Title: Importar un proyecto existente en Terraform
Date: 2024-02-07 10:15
Category: Cloud
Tags: terraform,iac
Slug: terraform-import
Authors: José L. Patiño-Andrés
Summary: Descripción de los pasos necesarios para importar un proyecto GCP existente en Terraform

En ocasiones podemos encontrarnos con proyectos Google Cloud ya existentes que no han sido nunca
gestionados mediante Terraform, con lo cual si queremos empezar a usar ésta herramienta necesitamos
importar el proyecto (y sus recursos, o al menos todos los posibles) a nuestra nueva configuración
de Terraform.

Para llevar a cabo esta operación hay que seguir los siguientes pasos:

## Crear nueva configuración de Terraform

Lo primero que debemos hacer es crear una nueva configuración en Terraform para el proyecto
existente. Podemos hacerlo creando un nuevo directorio:

```bash
mkdir infrastructure
cd infrastructure
```

Y creando los ficheros `.tf` de configuración:

### `main.tf`

```terraform
resource "google_project" "project" {
  name            = var.gcp_project.name
  project_id      = var.gcp_project.id
  billing_account = var.gcp_project.billing_account
  folder_id       = var.gcp_project.folder_id
}
```

### `providers.tf`

```terraform
provider "google" {
  project = var.gcp_project.id
  region  = var.gcp_project.region
}
```
### `variables.tf`

```terraform
variable "gcp_project" {
  type = object({
    id   = string
    name = string
    region          = string
    billing_account = string
    folder_id       = string
  })

  default = {
    id = "my-project"
    name = "Infrastructure"
    region          = "europe-west2" # London
    billing_account = "012345-678901-ABCDEF"
    folder_id       = "123456789012"
  }
}
```

## Inicializar Terraform

En este momento, una vez creado el proyecto base, podemos inicializar Terraform con el comando:

```bash
terraform init
```

## Importar proyecto GCP

Si ahora hacemos un `terraform plan`, Terraform intentará **crear** el proyecto desde cero:

```diff
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_project.project will be created
  + resource "google_project" "project" {
      + auto_create_network = true
      + billing_account     = "012345-678901-ABCDEF"
      + effective_labels    = (known after apply)
      + folder_id           = "123456789012"
      + id                  = (known after apply)
      + name                = "Infrastructure"
      + number              = (known after apply)
      + project_id          = "my-project"
      + skip_delete         = (known after apply)
      + terraform_labels    = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

Esto es normal porque, para el estado actual de Terraform, el proyecto no existe. **Ahora es cuando
tenemos que importarlo**:

```bash
terraform import google_project.project my-project
```

## Importar los recursos del proyecto GCP

Una vez tenemos el proyecto GCP funcionando con Terraform, podemos ir añadiendo individualmente los
recursos existentes. Para ello tenemos que hacer de nuevo `terraform import ...` para cada recurso
que exista en el proyecto.

Afortunadamente, la herramienta CLI de Google Cloud `gcloud` tiene un comando especial que hace
exactamente eso: devolvernos el código Terraform y el comando `terraform import ...` **de cada uno
de los recursos existentes en un proyecto determinado**. En nuestro caso, éste comando es:

```bash
gcloud beta resource-config bulk-export --project=my-project --resource-format=terraform
```

## Deshacer un `import` inválido

En caso de que nos equivoquemos e importemos algo que no queríamos, podemos deshacer dicho `import`
con el comando `terraform state rm ...`. Por ejemplo, si hemos importado un GCP Storage bucket
llamado `my-data` con el comando

```bash
terraform import google_storage_bucket.my_data my-data
```

y vemos que es un error y no queríamos importarlo, el comando para deshacerlo sería:

```bash
terraform state rm google_storage_bucket.my_data
```
