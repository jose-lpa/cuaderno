Title: Grabación de imágenes ISO en pendrive USB
Date: 2022-08-21 10:15
Category: Terminal
Tags: iso,linux,consola,usb,bootloader,comandos
Slug: grabar-imagen-iso
Authors: José L. Patiño-Andrés
Summary: Ejemplo de cómo grabar una imagen ISO en un pendrive USB

## Desmontar dispositivo USB

Primero hay que asegurarse de que el dispositivo USB en el que vamos a grabar la
imagen ISO está desmontado:

    :::bash
    udisksctl unmount --block-device /dev/sd<X>


## Grabar imagen ISO

    :::bash
    sudo dd if=/path/de/la/imagen.iso of=/dev/sd<X>

El proceso puede durar unos minutos. Después de eso, el pendrive tendrá la imagen
ISO grabada y podremos usarlo como disco de arranque, por ejemplo.
