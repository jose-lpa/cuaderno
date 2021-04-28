Title: Usar Bulma CSS en proyectos React JS
Date: 2021-04-28 09:23
Category: Frontend
Tags: frontend,styling,reactjs,bulma,css
Slug: reactjs-bulma
Authors: José L. Patiño-Andrés
Summary: Instalación y configuración de Bulma CSS en proyectos ReactJS.

[Bulma](https://bulma.io/) es un framework CSS que destaca por ser muy sencillo
de entender y usar y tener menos carga que otros más famosos como Bootstrap.


A continuación se describe cómo podemos instalar y configurar un proyecto en
[ReactJS](https://reactjs.org/) para que utilice Bulma CSS.

## Instalación

En nuestro proyecto ReactJS, instalamos Bulma CSS mediante `npm`:

    :::bash
    npm install bulma

## Configuración

A continuación, podemos importar Bulma CSS en nuestro `index.js`, por
ejemplo:

    :::javascript
    ...
    import 'bulma/css/bulma.min.css';
    import './index.css';

**Atención:** siempre hay que importar Bulma antes que cualquier otro CSS
personalizado. Así evitamos que alguna clase CSS de Bulma sobreescriba un
estilo nuestro propio, cuando lo que querríamos es hacer justo lo contrario.
