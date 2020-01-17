Title: Inspección de ficheros XML grandes con `less`
Date: 2019-02-12 15:02
Category: Terminal
Tags: sistema, terminal, consola, Linux,
Slug: less-ficheros-grandes
Authors: José L. Patiño-Andrés
Summary: Cómo inspeccionar ficheros XML de gran tamaño con `less`.

En ocasiones podemos necesitar inspeccionar ficheros XML de gran tamaño (del 
orden de cientos de MB) para por ejemplo comprobar su esquema o incluso
asegurarnos de que un determinado contenido está presente en el fichero.

Abrir grandes ficheros XML en un navegador web es lento y se corre el riesgo
de hacer inusable o incluso tumbar el navegador, por muy moderno o potente que 
sea el equipo. Editores de texto como Vim o Emacs también pueden acabar de la
misma forma si cargamos ficheros de cientos de megas, o incluso de GB.

Las herramientas comerciales que existen para trabajar con ficheros XML cuestan
dinero, o pueden no estar disponibles para nuestro SO.

Estos problemas pueden ser solucionados desde la terminal de Linux de una forma
muy sencilla, usando la combinación de tres utilidades:

- La librería `libxml`
- [`highlight`](https://linux.die.net/man/1/highlight)
- [`less`](https://linux.die.net/man/1/less)

Teniendo en cuenta que tenemos esas tres utilidades disponibles en nuestro
sistema, simplemente hay que hacer uso del siguiente comando:

    :::bash
    xmllint <fichero.xml> --format | highlight --stdout -O xterm256 --syntax xml | less -RN

Con eso podremos tener en la pantalla del terminal el fichero XML abierto, con
coloreado de sintaxis y líneas numeradas, para inspeccionarlo cómodamente.
