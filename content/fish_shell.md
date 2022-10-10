Title: Configuración básica de shell `fish`
Date: 2022-10-10 14:52
Category: Sistema
Tags: sistema,fish,shell,terminal
Slug: fish-shell
Authors: José L. Patiño-Andrés
Summary: Breve referencia de configuración y consejos para Fish shell

## Instalación (Ubuntu/Debian)

Para instalar [fish shell](https://fishshell.com/) en una máquina con SO Ubuntu
o Debian, basta con el siguiente comando:

```bash
sudo apt install fish fish-common
```

Una vez finalizado el breve proceso de instalación, podemos configurar `fish`
para que sea nuestra shell por defecto con el comando `chsh`, el cual nos
presentará un diálogo como el siguiente:

```bash
Changing the login shell for jose
Enter the new value, or press ENTER for the default
	Login Shell [/bin/bash]:
```

Introduciremos el valor para que nuestra shell sea `fish`:

```
/usr/bin/fish
```

Y con esto ya tendremos `fish` instalada como nuestra shell por defecto. Hay que
tener en cuenta sin embargo que es probable que nuestra aplicación de terminal,
por ejemplo Konsole (KDE) o Gnome Terminal, tenga configurado por defecto Bash
como shell. En ese caso, será preciso configurar dicha aplicación gráfica para
que use Fish en lugar de Bash.

## Python & Virtualenvwrapper

Si queremos utilizar [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/)
en nuestro entorno, Fish nos provee de un plugin desarrollado por Justin Mayer
llamado [VirtualFish](https://github.com/justinmayer/virtualfish).

Su instalación es muy sencilla:

```bash
pip install virtualfish
```

```bash
vf install compat_aliases projects environment
```

Y ya estaríamos listos para usar comandos típicos de Virtualenvwrapper tales
como `mkvirtualenv`, `workon`, etcétera.

Existe sin embargo una pequeña pega: el entorno virtual NO es mostrado en el
prompt cuando está activado. Esto tiene una muy fácil solución que podemos leer
en la propia [documentación de VirtualFish](compat_aliases projects environment):
hay que añadir a la función de configuración `fish_prompt` el siguiente
fragmento de código:

```shell
if set -q VIRTUAL_ENV
    echo -n -s (set_color -b blue white) "(" (basename "$VIRTUAL_ENV") ")" (set_color normal) " "
end
```

Tras ello, simplemente ejecutamos el siguiente comando para que los cambios
queden grabados en la función `fish_prompt`:

```bash
funsave fish_prompt
```
