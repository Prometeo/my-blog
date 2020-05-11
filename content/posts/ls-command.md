+++
title =  "Ls Command"
date = 2020-05-09T18:48:14-05:00
tags = ["shell", "linux", "console", "commands"]
featured_image = ""
description = "Uso del comando ls"
+++

El Comando `ls` es uno de los primeros que debemos aprender a usar para poder interactuar con la consola de linux.
El comando `ls` no solo permite listar archivos y directorios, sino que también nos permite obtener infrormación importante sobre estos.

De acuerdo al artículo escrito por Eric Fischer acerca [Linux Document Project](http://www.tldp.org/LDP/LG/issue48/fischer.html), los orígenes de `ls`
se remontan al comando `listf` del *Compatible Time Sharing System* del MIT en 1961. Cuando CTSS fue remplazado por Multics el comando se convirtió
en `list` que es el equivalente a `list -all`, de acuerdo a [wikipedia](https://es.wikipedia.org/wiki/Compatible_Time-Sharing_System) `ls` apareció en
la versión original de AT&T Unix. El comando `ls` que usamos hoy viene incluido en el [GNU Core Utilities](https://www.gnu.org/software/coreutils/)

# Clasificación
La opción **-F**(--classify en GNU) nos muestra el tipo de archivo mediante un un caracter(indicador) después de cada entrada:

![example ls -F](/images/ls_command/ls_f.png)
* Un slash (/) indica que el archivo es un directorio.
* Un asterisco (*) indica que el archivo es un ejecutable.
* Un arroba (@) indica que el archivo es un enlace simbólico.

# Lista larga
Obtener una lista larga de `ls` es una operación tan común que muchas distribuciones tienen el comando `ll` el cual es un alias de `ls -l`.
La lista larga muestra muchos atributos importantes de los archivos, tales como permisos, usuario al que pertence, grupo al que pertence,
tamaño en bytes y la fecha de última modificación:
![example ls -l](/images/ls_command/ls_l.png)

Para ver el tamaño de los archivos de una forma mas legible, se debe agregar la opción **-h**:
![example ls -lh](/images/ls_command/ls_lh.png)

Si lo que se quiere es visualizar solo el propietario de los archivos se debe hacer uso de la opción **-o**,
para mostrar solo el grupo propietario del archivo usar la opción **-g**.

# Formato de fecha
El formato de lista obtenido con el comando `ls -l` es, generalmente, como el de la imagen siguiente:
![example ls -l](/images/ls_command/ls_l.png)

El formato de la fecha que se muestra en una lista larga se puede ser modificado con la opción `--time-style` mas el nombre del formato. por ejemplo:
![example ls -l format](/images/ls_command/ls_l_format_date.png)

Los posibles formatos de fecha son:
* full-iso (1970-01-01 21:12:00)
* long-iso (1970-01-01 21:12)
* iso (01-01 21:12)
* locale (uses your locale settings)
* posix-STYLE (replace STYLE with a locale definition)

# Otras opciones útiles
Generalmente el comando `ls` muestra una lista de archivos ordenados alfabeticamente, la opción `-t` nos permite ordenar la lista por
fecha de última modificación(el mas reciente primero).
![example ls -t](/images/ls_command/ls_t.png)

Para ordenar el resultado por extensión en vez del nombre de archivo se debe hacer uso de la opción `-X`:
![example ls -X](/images/ls_command/ls_X.png)

Si lo que se quiere es ignorar archivos de backup se debe hacer uso de la opción `-B` o `--ignore-backups`, esto funcionará con
archivos de backup con formatos comunes pero para otros formatos menos conocidos como los generados por
emacs ("#file#") se puede utilizar la opción `--hide`, por ejemplo: ls --hide="#*#"

Para listar directorios recursivamente se debe usar la opción `-R`:
![example ls -R](/images/ls_command/ls_R.png)

Para ordenar los archivos por tamaño usar la opcióm `-ls`

Para obtener mayor información sobre el comando `ls` y sus opciones ejecutar: `man ls`.
