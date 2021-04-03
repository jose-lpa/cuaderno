Title: Concurrencia en Python con gevent
Date: 2012-02-05 11:20
Category: Programación
Tags: python,concurrencia,greenlets,gevent
Slug: concurrencia-python-gevent
Authors: José L. Patiño-Andrés
Summary: Uso de la librería gevent en Python para programación concurrente.

[gevent](http://www.gevent.org/) es una librería de networking de Python basada
en co-rutinas que usa "greenlets" para proporcionar una API de alto nivel para
sincronizar tareas.

## Ejemplo de uso

    :::python
    import gevent


    def do_something_async(*args, **kwargs):
        # Hacer lo que quieras aquí...


    jobs = [
        gevent.spawn(
            do_something_async,
            n,
            arg_1, arg_2, arg_3, ...
            kwarg_1, kwarg_2, kwarg_3, ...
        )
        for n in whatever
    ]

    gevent.joinall(jobs, raise_error=True)
