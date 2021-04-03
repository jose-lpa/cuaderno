Title: Programación básica del Kernel Linux
Date: 2011-09-28 17:30
Category: Programación
Tags: linux,kernel,c,sistemas
Slug: programacion-basica-linux-kernel
Authors: José L. Patiño-Andrés
Summary: Conceptos básicos de programación del kernel Linux (2.6.x).


Breve resumen de conceptos básicos de programación del Kernel Linux, utilizando
Debian GNU/Linux como distribución. Todo hace referencia a la versión 2.6 del
Kernel.

## Instalación de paquetes necesarios

    :::bash
    apt-get install git libncurses5-dev

## Obtención del código fuente

Creamos un directorio y clonamos el repositorio del kerne:

    :::bash
    mkdir code
    git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux-2.6.git linux-X.X

    # O también...
    git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git linux-X.X

## Configuración inicial

Nos situamos dentro del directorio raíz y ejecutamos

    :::bash
    make menuconfig

Ahora podemos configurar manualmente las opciones de nuestro kernel. Sin embargo,
podemos también copiar la configuración de nuestro kernel actual, para asegurar
el funcionamiento:

    :::bash
    cp -v /boot/config-X.X.X .config
    make oldconfig  # En lugar del anterior `make menuconfig`.

Para evitar tener que pulsar Intro cada vez que la configuración nos pregunte,
podemos hacer lo siguiente:

    :::bash
    yes '' | make oldconfig

Podemos ponerle un nombre a nuestro kernel. Esto se hace cambiando la variable
`EXTRAVERSION` del fichero `Makefile` del directorio raíz.

## Compilación e instalación del kernel

Ahora podemos compilar el kernel:

    :::bash
    make

E instalarlo:

    :::bash
    make modules_install
    make install
    update-initramfs -c -k X.X.X  # `X.X.X` es la versión del kernel.
    update-grub

Si queremos ahorrar espacio en disco, también podríamos haber hecho en primer
lugar `make INSTALL_MOD_STRIP=1 modules_install`.

Si queremos que los ficheros generados después de la compilación estén en un
directorio diferente, dejando aparte el directorio de código fuente del kernel,
podemos hacerlo:

    :::bash
    mkdir ../build_dir
    make 0=../build_dir menuconfig
    make 0=../build_dir
    make 0=../build_dir modules_install

### Limpieza después de la compilación

Para limpiar el directorio raíz después de una compilación:

    :::bash
    make clean      # Borra archivos generados, pero mantiene la configuración.
    make mrproper   # Borra archivos generados, configuración y ficheros de respaldo.
    make distclean  # Borra todo como `mrproper` y además los ficheros de parches.

## Compilación de módulos

No es necesario tener un kernel compilado para poder compilar un módulo para ese
kernel. Simplemente hace falta tenerlo previamente configurado, bien con
`menuconfig` o bien con `oldconfig`. Entonces podemos hacer en el directorio
raíz:

    :::bash
    make modules_prepare

## Ejemplo de código

Podemos ver un ejemplo de código para un módulo del Kernel Linux en el
repositorio de GitHub [https://github.com/jose-lpa/kernel-tics](https://github.com/jose-lpa/kernel-tics)
