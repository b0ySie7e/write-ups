# Harvesting Passwords from Usual Spots

La forma más sencilla de obtener acceso a otro usuario es recopilar credenciales de una máquina comprometida. Estas credenciales podrían existir por muchas razones, incluido un usuario descuidado que las deja en archivos de texto sin formato; o incluso almacenado por algún software como navegadores o clientes de correo electrónico.

Esta tarea presentará algunos lugares conocidos para buscar contraseñas en un sistema Windows.

### Unattended Windows Installations

Al instalar Windows en una gran cantidad de hosts, los administradores pueden usar los Servicios de implementación de Windows, que permiten implementar una única imagen del sistema operativo en varios hosts a través de la red. Este tipo de instalaciones se denominan instalaciones desatendidas porque no requieren la interacción del usuario. Dichas instalaciones requieren el uso de una cuenta de administrador para realizar la configuración inicial, que podría terminar almacenándose en la máquina en las siguientes ubicaciones:

```
- C:\Unattend.xml
- C:\Windows\Panther\Unattend.xml
- C:\Windows\Panther\Unattend\Unattend.xml
- C:\Windows\system32\sysprep.inf
- C:\Windows\system32\sysprep\sysprep.xml
```

Como parte de estos archivos, es posible que encuentre credenciales:

```shell-session
<Credentials>
    <Username>Administrator</Username>
    <Domain>thm.local</Domain>
    <Password>MyPassword123</Password>
</Credentials>
```

### Powershell History

Cada vez que un usuario ejecuta un comando usando Powershell, se almacena en un archivo que guarda una memoria de los comandos anteriores. Esto es útil para repetir rápidamente comandos que ha utilizado antes. Si un usuario ejecuta un comando que incluye una contraseña directamente como parte de la línea de comando de Powershell, luego podrá recuperarla usando el siguiente comando desde un `cmd.exe`mensaje:

```shell-session
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

**Nota:** El comando anterior solo funcionará desde cmd.exe, ya que Powershell no lo reconocerá `%userprofile%`como una variable de entorno. Para leer el archivo desde Powershell, deberá reemplazarlo `%userprofile%`con `$Env:userprofile`.

### Saved Windows Credentials

Windows nos permite utilizar las credenciales de otros usuarios. Esta función también brinda la opción de guardar estas credenciales en el sistema. El siguiente comando enumerará las credenciales guardadas:

```shell-session
cmdkey /list
```

Si bien no puede ver las contraseñas reales, si nota alguna credencial que valga la pena probar, puede usarla con el  `runas` comando y la  `/savecred` opción, como se ve a continuación

```shell-session
runas /savecred /user:admin cmd.exe
```

### IIS Configuration

Internet Information Services (IIS) es el servidor web predeterminado en las instalaciones de Windows. La configuración de sitios web en IIS se almacena en un archivo llamado `web.config`y puede almacenar contraseñas para bases de datos o mecanismos de autenticación configurados. Dependiendo de la versión instalada de IIS, podemos encontrar web.config en una de las siguientes ubicaciones:

```
- C:\inetpub\wwwroot\web.config
- C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
```

A continuación se muestra una forma rápida de encontrar cadenas de conexión de bases de datos en el archivo:

```shell-session
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString
```

### Retrieve Credentials from Software: PuTTY

PuTTY es un cliente SSH que se encuentra comúnmente en sistemas Windows. En lugar de tener que especificar los parámetros de una conexión cada vez, los usuarios pueden almacenar sesiones donde la IP, el usuario y otras configuraciones se pueden almacenar para su uso posterior. Si bien PuTTY no permitirá a los usuarios almacenar su contraseña SSH, almacenará configuraciones de proxy que incluyen credenciales de autenticación de texto sin cifrar.

Para recuperar las credenciales de proxy almacenadas, puede buscar ProxyPassword en la siguiente clave de registro con el siguiente comando:

```shell-session
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s
```

**Nota:** Simon Tatham es el creador de PuTTY (y su nombre es parte de la ruta), no el nombre de usuario cuya contraseña recuperamos. El nombre de usuario del proxy almacenado también debería ser visible después de ejecutar el comando anterior.

Así como PuTTY almacena credenciales, cualquier software que almacene contraseñas, incluidos navegadores, clientes de correo electrónico, clientes FTP , clientes SSH, software VNC y otros, tendrá métodos para recuperar cualquier contraseña que el usuario haya guardado.
