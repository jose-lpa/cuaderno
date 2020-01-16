# Cuaderno de notas

Anotaciones sobre comandos, trucos y utilidades que he ido recopilando a lo
largo del tiempo. Antiguamente en una libreta, ahora en un sitio web estático
creado con [Pelican](https://blog.getpelican.com/).

## Uso de Pelican

Las instrucciones completas para usar el generador se pueden encontrar en
[la documentación de Pelican](https://docs.getpelican.com/en/stable/index.html).

### Añadir nueva entrada

Las nuevas entradas se añaden en ficheros Markdown `.md` en el directorio
`content`. El fichero debe tener la siguiente cabecera:

```
Title: <título de la entrada>
Date: <fecha de publicación en formato YYYY-mm-dd HH:MM>
Category: <categoría a la que corresponde la entrada>
Tags: <lista de tags, separados por coma>
Slug: <texto para la URL de la entrada>
Authors: José L. Patiño-Andrés
Summary: <frase de resumen de la entrada>
```

Tras dicha cabecera, se escribe la entrada en Markdown.

### Generar contenido

Al acabar de escribir la entrada, generamos el contenido HTML/CSS con el
siguiente comando:

```
pelican content
```

Esto creará nuevos ficheros en el directorio `output` (ignorado por Git).

### Comprobación

Se puede navegar por el sitio para comprobar las entradas creadas con el
siguiente comando:

```
pelican --listen
```
