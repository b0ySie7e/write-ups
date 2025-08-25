# Cron Jobs

Los trabajos cron se utilizan para ejecutar scripts o archivos binarios en momentos específicos. De forma predeterminada, se ejecutan con el privilegio de sus propietarios y no del usuario actual. Si bien los trabajos cron configurados correctamente no son inherentemente vulnerables, pueden proporcionar un vector de escalada de privilegios en algunas condiciones.\
La idea es muy simple; Si hay una tarea programada que se ejecuta con privilegios de root y podemos cambiar el script que se ejecutará, entonces nuestro script se ejecutará con privilegios de root.

Las configuraciones de trabajos cron se almacenan como crontabs (tablas cron) para ver la próxima hora y fecha en que se ejecutará la tarea.

Cada usuario del sistema tiene su archivo crontab y puede ejecutar tareas específicas ya sea que hayan iniciado sesión o no. Como es de esperar, nuestro objetivo será encontrar un trabajo cron establecido por root y hacer que ejecute nuestro script, idealmente un shell.

Cualquier usuario puede leer el archivo manteniendo los trabajos cron de todo el sistema en `/etc/crontab`

```shell
karen@ip-10-10-132-184:~$ cat /etc/crontab
```

<figure><img src="../../.gitbook/assets/20231008170907.png" alt=""><figcaption></figcaption></figure>

```shell
* * * * *  root /antivirus.sh
* * * * *  root antivirus.sh
* * * * *  root /home/karen/backup.sh
* * * * *  root /tmp/test.py
```

```
* * * * *  root /home/karen/backup.sh
```

<figure><img src="../../.gitbook/assets/20231008172944.png" alt=""><figcaption></figcaption></figure>

```shell
karen@ip-10-10-132-184:~$ echo 'chmod +s /bin/bash' >> backup.sh 
```

* Luego de un minuto la `bash` seria SUID y solo `bash -p`

<figure><img src="../../.gitbook/assets/20231008173400.png" alt=""><figcaption></figcaption></figure>
