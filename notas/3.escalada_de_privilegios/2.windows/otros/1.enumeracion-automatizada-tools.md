# Enumeración automatizada - Tools

Existen varios scripts para realizar la enumeración del sistema de manera similar a las vistas en la tarea anterior. Estas herramientas pueden acortar el tiempo del proceso de enumeración y descubrir diferentes vectores potenciales de escalada de privilegios. Sin embargo, recuerde que las herramientas automatizadas a veces pueden pasar por alto la escalada de privilegios.

A continuación se muestran algunas herramientas que se utilizan comúnmente para identificar vectores de escalada de privilegios. Siéntase libre de ejecutarlos contra cualquiera de las máquinas en esta sala y ver si los resultados coinciden con los vectores de ataque discutidos.

#### WinPEAS

WinPEAS es un script desarrollado para enumerar el sistema de destino y descubrir rutas de escalada de privilegios. Puede encontrar más información sobre winPEAS y descargar el ejecutable precompilado o un script .bat. WinPEAS ejecutará comandos similares a los enumerados en la tarea anterior e imprimirá su resultado. El resultado de winPEAS puede ser extenso y, en ocasiones, difícil de leer. Por eso sería una buena práctica redirigir siempre la salida a un archivo, como se muestra a continuación:

```shell-session
C:\> winpeas.exe > outputfile.txt
```

WinPEAS se puede descargar [aquí](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) .

#### PrivescCheck

PrivescCheck es un script de PowerShell que busca la escalada de privilegios comunes en el sistema de destino. Proporciona una alternativa a WinPEAS sin requerir la ejecución de un archivo binario.

#### PrivescCheck se puede descargar [aquí](https://github.com/itm4n/PrivescCheck) .

**Recordatorio** : para ejecutar PrivescCheck en el sistema de destino, es posible que deba omitir las restricciones de la política de ejecución. Para lograr esto, puede utilizar el `Set-ExecutionPolicy`cmdlet como se muestra a continuación.

```shell-session
PS C:\> Set-ExecutionPolicy Bypass -Scope process -Force
PS C:\> . .\PrivescCheck.ps1
PS C:\> Invoke-PrivescCheck
```

#### WES-NG: Windows Exploit Suggester - Next Generation

Algunos scripts que sugieren exploits (por ejemplo, winPEAS) requerirán que los cargue en el sistema de destino y los ejecute allí. Esto puede hacer que el software antivirus los detecte y los elimine. Para evitar hacer ruido innecesario que pueda llamar la atención, es posible que prefieras usar WES-NG, que se ejecutará en tu máquina atacante.

WES-NG es un script de Python que se puede encontrar y descargar [aquí](https://github.com/bitsadmin/wesng) .

Una vez instalado, y antes de usarlo, escriba el `wes.py --update`comando para actualizar la base de datos. El script hará referencia a la base de datos que crea para comprobar si faltan parches que puedan provocar una vulnerabilidad que pueda utilizar para elevar sus privilegios en el sistema de destino.

Para utilizar el script, deberá ejecutar el `systeminfo` comando en el sistema de destino. No olvide dirigir la salida a un archivo .txt que deberá mover a su máquina atacante.

Una vez hecho esto, wes.py se puede ejecutar de la siguiente manera;

```shell-session
user@kali$ wes.py systeminfo.txt
```
