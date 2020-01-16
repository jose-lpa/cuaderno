Title: Uso básico de Tmux
Date: 2011-01-12 10:32
Category: Sistema
Tags: tmux, terminal, consola, sistema
Slug: tmux-uso
Authors: José L. Patiño-Andrés
Summary: Pequeña referencia de comandos para la aplicación Tmux.

### `Ctrl + b + ...`

- **`c`:** Crear nueva instancia de terminal (nueva "ventana").
- **`n`:** Siguiente ventana.
- **`p`:** Anterior ventana.
- **`0...9`:** Ir directamente a ventana 0...9.
- **`&`:** Cerrar ventana actual.
- **`%`:** Dividir panel verticalmente.
- **`"`:** Dividir panel horizontalmente.
- **`s`:** Mostrar todas las sesiones activas.
- **`$`:** Renombrar sesión actual.
- **`(`:** Ir a sesión anterior.
- **`)`:** Ir a siguiente sesión.

### Compartir sesiones

1. Un usuario arranca la sesión: `tmux new -s <nombre_sesión>`.
2. Otro usuario se le une: `tmux attach -t <nombre_sesión>`.

### Modo "copia"

`Ctrl + b + [` entra en modo copia. Una vez en modo copia, podemos usar estos
comandos:

- **`/`:** Búsqueda ascendente.
- **`?`:** Búsqueda descendente.
- **`n`:** Siguiente ocurrencia en búsqueda.
- **`N`:** Anterior ocurrencia en búsqueda.
- **`<Espacio>`:** Iniciar selección de copia.
- **`<Esc>`:** Cancelar selección de copia.
- **`<Intro>`:** Copiar selección.
- **`q`:** Salir del modo copia.

Una vez salimos del modo copia, si hemos copiado algo, con `Ctrl + b + ]` 
pegaremos los contenidos del buffer.
