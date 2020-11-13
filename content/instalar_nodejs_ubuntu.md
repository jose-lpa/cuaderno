Title: Instalación de NodeJS & NPM en Ubuntu/Debian
Date: 2020-07-01 16:10
Category: Sistema
Tags: nodejs, npm, javascript, librerías, instalación
Slug: instalar-nodejs-ubuntu
Authors: José L. Patiño-Andrés
Summary: Instrucciones de instalación de NodeJS y gestor de paquetes NPM en distribuciones Ubuntu/Debian.

### Añadir repositorio oficial

Primero hay que añadir el repositorio oficial de NodeJS para distribuciones
Debian. Esto lo hacemos fácilmente con un script que nos proporcionan y que se
puede descargar de aquí: `https://deb.nodesource.com/setup_12.x`

`curl -sL https://deb.nodesource.com/setup_14.x > setup_14.x`

Podemos elegir otras versiones si fuera necesario y las hubiera.

Cambiamos los permisos del script: `chmod +x setup_14.x`

Ejecutamos el script y nos añade la clave de firmado del repositorio y el
repositorio a nuestra lista de APT: `sudo ./setup_14.x`

## Instalar NodeJS

Ahora ya sólo queda hacer uso de APT para instalar los paquetes necesarios:

`sudo apt install nodejs`
