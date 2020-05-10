+++
title =  "Ls Command"
date = 2020-05-09T18:48:14-05:00
tags = []
featured_image = "images/banner.jpg"
description = "Uso del comando ls"
+++

El comando `ls` es uno de los mas importantes porque no solo lista archivos y directorios, sino también muestra información importante sobre estos.

De acuerdo al artículo escrito por Eric Fischer acerca [Linux Document Project](http://www.tldp.org/LDP/LG/issue48/fischer.html), los orígenes de `ls`
se remontan al comando `listf` en el Compatible Time Sharing System del MIT en 1961. Cuando CTSS fue remplazado por Multics el comando se convirtió
en `list`.

# Classify
El argumento **-F**(--classify en GNU)  sirve para mostrar que tipo de archivo es
![example ls -F](/images/ls_command/ls_f.png)
* Un slash (/) indica que el archivo es un directorio.
* Un asterisco (*) indica que el archivo es un ejecutable.
* Un arroba (@) indica que el archivo es un enlace simbólico.

# Long list
Obtener una lista larga de `ls` es tan común que muchas distribuciones tienen un alias para esto: `ll`.
La lista larga muestra muchos atributos importantes de los archivos, como permisos, a que usuario pertenece el archivo,
a que grupo pertence el archivo, el tamaño del archivo en bytes y la fecha de última modificación:
![example ls -l](/images/ls_command/ls_l.png)

Para ver el tamaño de los archivos de una forma mas amigable se debe agregar el argumento **-h**:
![example ls -l](/images/ls_command/ls_lh.png)

Si se quiere visualizar solo el propietario de los archivos usar el argumento **-o**, para solo el grupo propietario del archivo usar **-g**
