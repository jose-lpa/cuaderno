Title: Usar Font Awesome en proyectos React JS
Date: 2021-04-28 08:58
Category: Frontend
Tags: frontend,styling,reactjs,fontawesome
Slug: reactjs-fontawesome
Authors: José L. Patiño-Andrés
Summary: Instalación y configuración de Font Awesome en proyectos ReactJS.

En proyectos frontend podemos usar [Font Awesome](https://fontawesome.com/),
una librería de iconos vectoriales, para mostrar iconos y símbolos en nuestra
pantalla.

A continuación se describe cómo podemos instalar y configurar un proyecto en
[ReactJS](https://reactjs.org/) para que utilice Font Awesome.

## Instalación

En nuestro proyecto ReactJS, instalaremos la versión libre de la librería
Font Awesome usando `npm`:

    :::bash
    npm install --save @fortawesome/fontawesome-free

## Configuración:

A continuación, podemos importar esta librería en nuestro `index.js`, por
ejemplo:

    :::javascript
    import '@fortawesome/fontawesome-free/css/all.min.css';

## Uso:

Ahora podemos usar la librería de iconos, usándolos como clases en nuestro
HTML. Por ejemplo:

    :::html
    <i class="fab fa-js" aria-hidden="true"></i>

mostrará un pequeño icono con el emblema de JavaScript.
