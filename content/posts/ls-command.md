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

# Clasificación
El argumento **-F** sirve para mostrar que tipo de archivo es
![example ls -F](/images/ls_command/ls_f.png)
* Un slash (/) indica que el archivo es un directorio.
* Un asterisco (*) indica que el archivo es un ejecutable.
* Un arroba (@) indica que el archivo es un enlace simbólico.
