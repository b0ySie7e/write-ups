---
title: Ninja Skills -  Tryhackme
date: 2023-11-5
categories: [Write up,Challenge, Linux, Tryhackme]
tags: [bash, script, easy]     # TAG names should always be lowercase
---

Este challenge tiene la finalidad de poner en practica un poco de bash encontrando los archivos y las especificaciones que nos da cada uno de las tareas. Vamos usar el comando `find` para encontrar los dicersos archivos.

![20231106005341.png](20231106005341.png)

Link [Ninja Skills](https://tryhackme.com/room/ninjaskills)
- Created by [tryhackme](https://tryhackme.com/p/tryhackme) and  [Mokmokmok](https://tryhackme.com/p/Mokmokmok)

Empezamos, vamos a conectarnos por `ssh`

```
❯ ssh new-user@10.10.169.250
new-user
```

Podemos guardar todos los archivos en una archivo `txt` para luego hcaer una busqueda con el comando `find`

```java
[new-user@ip-10-10-169-250 tmp.ChQraGcF54]$ cat files.txt | while read files;  do find / -name $files 2>/dev/null; done
/etc/8V2L
/mnt/c4ZX
/mnt/D8B3
/var/FHl1
/opt/oiMO
/opt/PFbD
/media/rmfX
/etc/ssh/SRSq
/var/log/uqyw
/home/v2Vb
/X1Uy
```
Podemos enumerar cada uno de los archivos y la ruta donde se encuentran

## Task 1

```java
find / -type f -group best-group 2>/dev/null
```

`find`: Este es el comando principal utilizado para buscar archivos y directorios en el sistema de archivos.

`/`: Indica que la búsqueda debe comenzar desde el directorio raíz del sistema de archivos.

`-type f`: Esta opción especifica que solo se deben buscar archivos regulares (no directorios ni enlaces simbólicos). En otras palabras, se limita la búsqueda a archivos normales.

`-group best-group`: Aquí se especifica el nombre del grupo de usuarios al que deben pertenecer los archivos buscados. En este caso, se está buscando archivos que pertenezcan al grupo llamado "best-group". Puedes reemplazar "best-group" con el nombre real del grupo que estás buscando.

`2>/dev/null`: Esta parte del comando redirige cualquier mensaje de error (stderr) a /dev/null, lo que significa que los errores no se mostrarán en la salida estándar. Esto se hace para evitar mensajes de error que puedan aparecer si el grupo "best-group" no existe o si no tienes permiso para acceder a ciertos directorios.



## Task 2

```java
find / -type f \( -name "8V2L" -o -name "bny0" -o -name "c4ZX" -o -name "D8B3" -o -name "FHl1" -o -name "oiMO" -o -name "PFbD" -o -name "rmfX" -o -name "SRSq" -o -name "uqyw" -o -name "v2Vb" -o -name "X1Uy" \) -exec grep -EH '[0-9{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' {} \; 2>/dev/null
```

`find /`: Esto indica que la búsqueda debe comenzar desde el directorio raíz del sistema de archivos.

`-type f`: Especifica que se deben buscar archivos regulares (no directorios ni enlaces simbólicos).

`\( ... \)`: Encierra un conjunto de condiciones entre paréntesis. En este caso, se están definiendo múltiples condiciones para los nombres de archivo.

`-name "8V2L" -o -name "bny0" -o ... -o -name "X1Uy"`: Cada una de estas condiciones (-name) busca archivos con un nombre que coincida con una de las cadenas proporcionadas (en este caso, una lista de nombres de archivo).

`-exec grep -EH '[0-9{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' {} \;`: Después de encontrar los archivos que coinciden con los nombres especificados, este comando ejecuta grep en cada archivo encontrado. grep busca patrones en el contenido de los archivos.

`-EH`: grep se utiliza con las opciones -E para habilitar expresiones regulares extendidas y -H para mostrar el nombre del archivo en la salida.

`'[0-9{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}'`: Este es el patrón de búsqueda utilizado por grep. Está buscando una cadena que coincida con una dirección IP válida (por ejemplo, "192.168.1.1"). La expresión regular busca cuatro grupos de números del 0 al 255, separados por puntos.

`{}`: Representa el archivo actual que find pasa a grep para su procesamiento.

`2>/dev/null`: Redirige los mensajes de error (stderr) a /dev/null, lo que significa que los errores se descartarán y no se mostrarán en la salida estándar.

## Task 3

```java
find / -type f -exec sha1sum {} \; 2>/dev/null | grep 9d54da7584015647ba052173b84d45e8007eba94
```

`find /`: Esto indica que la búsqueda comienza desde el directorio raíz del sistema de archivos.

`-type f`: Esta parte del comando especifica que solo se buscarán archivos regulares (no directorios ni enlaces simbólicos).

`-exec sha1sum {} \;`: Después de encontrar los archivos regulares, find ejecutará el comando sha1sum en cada uno de ellos. {} representa el archivo actual que find pasa al comando sha1sum. El ; indica el final del comando -exec para cada archivo encontrado.

`2>/dev/null`: Esto redirige los mensajes de error (stderr) a /dev/null, lo que significa que los errores se descartarán y no se mostrarán en la salida estándar.

`|`: Esto es una tubería (pipe) que redirige la salida estándar del comando anterior hacia el siguiente comando.

`grep 9d54da7584015647ba052173b84d45e8007eba94`: grep se utiliza para buscar patrones en la entrada que recibe a través de la tubería. En este caso, busca el patrón 9d54da7584015647ba052173b84d45e8007eba94 en la salida generada por el comando sha1sum.

## Task 4

```java
find / -type f \( -name "8V2L" -o -name "bny0" -o -name "c4ZX" -o -name "D8B3" -o -name "FHl1" -o -name "oiMO" -o -name "PFbD" -o -name "rmfX" -o -name "SRSq" -o -name "uqyw" -o -name "v2Vb" -o -name "X1Uy" \) -exec wc -l {} \; 2>/dev/null
```

`find /`: Esto indica que la búsqueda comienza desde el directorio raíz del sistema de archivos.

`-type f`: Esta parte del comando especifica que solo se buscarán archivos regulares (no directorios ni enlaces simbólicos).

`\( -name "8V2L" ... -o -name "X1Uy" \)`: Esto define un conjunto de nombres de archivo que find buscará. Utiliza la opción -name para especificar nombres de archivo individuales y la opción -o para indicar "o" (es decir, buscar archivos que coincidan con cualquiera de estos nombres).

`-exec wc -l {} \;`: Después de encontrar los archivos que coinciden con los nombres especificados, find ejecutará el comando wc -l en cada uno de ellos. {} representa el archivo actual que find pasa al comando wc -l, y el ; indica el final del comando -exec para cada archivo encontrado.

`2>/dev/null`: Esto redirige los mensajes de error (stderr) a /dev/null, lo que significa que los errores se descartarán y no se mostrarán en la salida estándar.


## Task 5

```java
find / -type f \( -name "8V2L" -o -name "bny0" -o -name "c4ZX" -o -name "D8B3" -o -name "FHl1" -o -name "oiMO" -o -name "PFbD" -o -name "rmfX" -o -name "SRSq" -o -name "uqyw" -o -name "v2Vb" -o -name "X1Uy" \) -exec ls -ln {} \; 2>/dev/null
```

`find /`: Esto indica que la búsqueda comienza desde el directorio raíz del sistema de archivos.

`-type f`: Esta parte del comando especifica que solo se buscarán archivos regulares (no directorios ni enlaces simbólicos).

`\( -name "8V2L" -o -name "bny0" ... -o -name "X1Uy" \)`: Esto define un conjunto de nombres de archivo que find buscará. Utiliza la opción -name para especificar nombres de archivo individuales y la opción -o para indicar "o" (es decir, buscar archivos que coincidan con cualquiera de estos nombres).

`-exec ls -ln {} \;`: Después de encontrar los archivos que coinciden con los nombres especificados, find ejecutará el comando `ls -ln` en cada uno de ellos. `{}` representa el archivo actual que find pasa al comando `ls -ln`, y el `;` indica el final del comando `-exec` para cada archivo encontrado.

`2>/dev/null`: Esto redirige los mensajes de error (stderr) a `/dev/null`, lo que significa que los errores se descartarán y no se mostrarán en la salida estándar.

## Task 6

```java
find / -type f \( -name "8V2L" -o -name "bny0" -o -name "c4ZX" -o -name "D8B3" -o -name "FHl1" -o -name "oiMO" -o -name "PFbD" -o -name "rmfX" -o -name "SRSq" -o -name "uqyw" -o -name "v2Vb" -o -name "X1Uy" \) -perm +001 2>/dev/null
```

`find /`: Esto indica que la búsqueda comienza desde el directorio raíz del sistema de archivos.

`-type f`: Esta parte del comando especifica que solo se buscarán archivos regulares (no directorios ni enlaces simbólicos).

`\( -name "8V2L" ... -o -name "X1Uy" \)`: Esto define un conjunto de nombres de archivo que find buscará. Utiliza la opción -name para especificar nombres de archivo individuales y la opción -o para indicar "o" (es decir, buscar archivos que coincidan con cualquiera de estos nombres).

`-perm +001`: Esto establece un filtro en función de los permisos del archivo. En este caso, busca archivos con permisos que incluyan al menos un bit de ejecución (permisos de ejecución para cualquier tipo de usuario: propietario, grupo u otros).

2>/dev/null: Esto redirige los mensajes de error (stderr) a /dev/null, lo que significa que los errores se descartarán y no se mostrarán en la salida estándar.

Happy hacking :)
