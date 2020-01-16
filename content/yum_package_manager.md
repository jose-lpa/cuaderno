Title: Uso del gestor de paquetes Yum.
Date: 2016-09-20 14:06
Category: Sistema
Tags: CentOS, paquetes, Linux, terminal, consola, sistema
Slug: yum-package-manager
Authors: José L. Patiño-Andrés
Summary: Pequeña referencia de comandos para el uso del gestor de paquetes Yum.

- Mostrar todas las versiones disponibles del paquete `<paquete>`: 

    ```
    yum --showduplicates list <paquete> | expand
    ```

- Instalar paquete:

    ```
    yum update <paquete>
    ```

- Actualizar paquete:

    ```
    yum install <paquete>
    ```

- Desactualizar paquete (anterior versión):

    ```
    yum downgrade <paquete>
    ```

- Limpiar caché de repositorios (refrescar datos del repo):

    ```
    yum clean expire-cache
    ```
