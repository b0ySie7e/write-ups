
Nmap - trace the packets

```c
sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping

Starting Nmap 7.80 ( https://nmap.org )
SENT (0.0429s) TCP 10.10.14.2:63090 > 10.129.2.28:21 S ttl=56 id=57322 iplen=44  seq=1699105818 win=1024 <mss 1460>
RCVD (0.0573s) TCP 10.129.2.28:21 > 10.10.14.2:63090 RA ttl=64 id=0 iplen=40  seq=0 win=0
Nmap scan report for 10.11.1.28
Host is up (0.014s latency).

PORT   STATE  SERVICE
21/tcp closed ftp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
```

Puertos Filtered

Cuando un puerto aparece como filtrado, puede deberse a varias razones. En la mayoría de los casos, los cortafuegos tienen ciertas reglas establecidas para manejar conexiones específicas. Los paquetes pueden ser descartados o rechazados. Cuando se rechaza un paquete, Nmap no recibe respuesta de nuestro objetivo, y por omisión, la tasa de reintentos (--max-retries) se establece en 1. Esto significa que Nmap reenviará la petición al puerto objetivo para determinar si el paquete anterior no fue accidentalmente mal manejado.

```c
nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -Pn

Starting Nmap 7.80 ( https://nmap.org )
SENT (0.0381s) TCP 10.10.14.2:60277 > 10.129.2.28:139 S ttl=47 id=14523 iplen=44  seq=4175236769 win=1024 <mss 1460>
SENT (1.0411s) TCP 10.10.14.2:60278 > 10.129.2.28:139 S ttl=45 id=7372 iplen=44  seq=4175171232 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up.

PORT    STATE    SERVICE
139/tcp filtered netbios-ssn
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
```

Enumeración de puertos abiertos UDP


```c
sudo nmap 10.129.2.28 -F -sU

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:01 CEST
Nmap scan report for 10.129.2.28
Host is up (0.059s latency).
Not shown: 95 closed ports
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
137/udp  open          netbios-ns
138/udp  open|filtered netbios-dgm
631/udp  open|filtered ipp
5353/udp open          zeroconf
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 98.07 seconds
```

```c
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:15 CEST
SENT (0.0367s) UDP 10.10.14.2:55478 > 10.129.2.28:137 ttl=57 id=9122 iplen=78
RCVD (0.0398s) UDP 10.129.2.28:137 > 10.10.14.2:55478 ttl=64 id=13222 iplen=257
Nmap scan report for 10.129.2.28
Host is up, received user-set (0.0031s latency).

PORT    STATE SERVICE    REASON
137/udp open  netbios-ns udp-response ttl 64
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.04 seconds
```


Informe con nmap :

Mientras ejecutamos varios escaneos, siempre debemos guardar los resultados. Podemos utilizarlos más tarde para examinar las diferencias entre los distintos métodos de sondeo que hemos utilizado. Nmap puede guardar los resultados en 3 formatos diferentes.  
  
Salida normal (-oN) con la extensión de fichero .nmap  
Salida Grepable (-oG) con la extensión de archivo .gnmap  
Salida XML (-oX) con la extensión de archivo .xml

```c
xsltproc target.xml -o target.html
```

Enumeracion de servicios:

```c
sudo nmap <target> -sC
```

Categorías de los scripts

|**Category**|**Description**|
|---|---|
|`auth`|Determination of authentication credentials.|
|`broadcast`|Scripts, which are used for host discovery by broadcasting and the discovered hosts, can be automatically added to the remaining scans.|
|`brute`|Executes scripts that try to log in to the respective service by brute-forcing with credentials.|
|`default`|Default scripts executed by using the `-sC` option.|
|`discovery`|Evaluation of accessible services.|
|`dos`|These scripts are used to check services for denial of service vulnerabilities and are used less as it harms the services.|
|`exploit`|This category of scripts tries to exploit known vulnerabilities for the scanned port.|
|`external`|Scripts that use external services for further processing.|
|`fuzzer`|This uses scripts to identify vulnerabilities and unexpected packet handling by sending different fields, which can take much time.|
|`intrusive`|Intrusive scripts that could negatively affect the target system.|
|`malware`|Checks if some malware infects the target system.|
|`safe`|Defensive scripts that do not perform intrusive and destructive access.|
|`version`|Extension for service detection.|
|`vuln`|Identification of specific vulnerabilities.|


```c
sudo nmap <target> --script <category>
```

```c
❯ nmap -p22,80,110,139,143,445,31337 --script vuln 10.129.2.49 -vvv

PORT      STATE SERVICE      REASON
22/tcp    open  ssh          syn-ack ttl 63
80/tcp    open  http         syn-ack ttl 63
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-litespeed-sourcecode-download: Request with null byte did not work. This web server might not be vulnerable
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-wordpress-users: [Error] Wordpress installation was not found. We couldn't find wp-login.php
|_http-jsonp-detection: Couldn't find any JSONP endpoints.
| http-enum: 
|_  /robots.txt: Robots file
110/tcp   open  pop3         syn-ack ttl 63
139/tcp   open  netbios-ssn  syn-ack ttl 63
143/tcp   open  imap         syn-ack ttl 63
445/tcp   open  microsoft-ds syn-ack ttl 63
31337/tcp open  Elite        syn-ack ttl 63

Host script results:
|_smb-vuln-ms10-054: false
| smb-vuln-regsvc-dos: 
|   VULNERABLE:
|   Service regsvc in Microsoft Windows systems vulnerable to denial of service
|     State: VULNERABLE
|       The service regsvc in Microsoft Windows 2000 systems is vulnerable to denial of service caused by a null deference
|       pointer. This script will crash the service if it is vulnerable. This vulnerability was discovered by Ron Bowes
|       while working on smb-enum-sessions.
|_          
|_smb-vuln-ms10-061: false


```

Firewall and IDS/IPS Evasion

El método de sondeo TCP ACK (-sA) de Nmap es mucho más difícil de filtrar para cortafuegos y sistemas IDS/IPS que los sondeos regulares SYN (-sS) o Connect (sT) porque sólo envían un paquete TCP con sólo la bandera ACK. Cuando un puerto está cerrado o abierto, el host debe responder con una bandera RST. A diferencia de las conexiones salientes, todos los intentos de conexión (con la bandera SYN) procedentes de redes externas suelen ser bloqueados por los cortafuegos. Sin embargo, los paquetes con la bandera ACK a menudo son pasados por el cortafuegos porque éste no puede determinar si la conexión se estableció primero desde la red externa o desde la red interna.

## Decoys

Hay casos en los que, en principio, los administradores bloquean subredes específicas de diferentes regiones. Esto impide cualquier acceso a la red de destino. Otro ejemplo es cuando IPS debe bloquearnos. Por esta razón, el método de exploración Decoy (-D) es la elección correcta. Con este método, Nmap genera varias direcciones IP aleatorias insertadas en la cabecera IP para disfrazar el origen del paquete enviado. Con este método, podemos generar aleatoriamente (RND) un número específico (por ejemplo: 5) de direcciones IP separadas por dos puntos (:). A continuación, nuestra dirección IP real se coloca aleatoriamente entre las direcciones IP generadas. En el siguiente ejemplo, nuestra dirección IP real se coloca, por tanto, en la segunda posición. Otro punto crítico es que los señuelos deben estar vivos. De lo contrario, el servicio en el objetivo puede ser inalcanzable debido a los mecanismos de seguridad SYN-flooding.

#### Scan by Using Decoys

```c
sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 16:14 CEST
SENT (0.0378s) TCP 102.52.161.59:59289 > 10.129.2.28:80 S ttl=42 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0378s) TCP 10.10.14.2:59289 > 10.129.2.28:80 S ttl=59 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 210.120.38.29:59289 > 10.129.2.28:80 S ttl=37 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 191.6.64.171:59289 > 10.129.2.28:80 S ttl=38 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 184.178.194.209:59289 > 10.129.2.28:80 S ttl=39 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 43.21.121.33:59289 > 10.129.2.28:80 S ttl=55 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
RCVD (0.1370s) TCP 10.129.2.28:80 > 10.10.14.2:59289 SA ttl=64 id=0 iplen=44  seq=4056111701 win=64240 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.099s latency).

PORT   STATE SERVICE
80/tcp open  http
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds
```

-D RND: Genera cinco direcciones IP aleatorias que indican la IP de origen de la que proviene la conexión

Los ISP y los enrutadores suelen filtrar los paquetes falsificados, aunque provengan del mismo rango de red. Por lo tanto, también podemos especificar las direcciones IP de nuestros servidores VPS y usarlas en combinación con " `IP ID`" manipulación en los encabezados IP para escanear el objetivo.

Otro escenario sería que sólo las subredes individuales no tuvieran acceso a los servicios específicos del servidor. Entonces también podemos especificar manualmente la dirección IP de origen ( `-S`) para probar si obtenemos mejores resultados con esta. Los señuelos se pueden utilizar para escaneos SYN, ACK, ICMP y escaneos de detección de sistema operativo. Entonces, veamos un ejemplo de este tipo y determinemos qué sistema operativo es más probable que sea.
#### Testing Firewall Rule

```c
nmap 10.129.2.28 -n -Pn -p445 -O

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-22 01:23 CEST
Nmap scan report for 10.129.2.28
Host is up (0.032s latency).

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Too many fingerprints match this host to give specific OS details
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.14 seconds
```

#### Scan by Using Different Source IP


```c
nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-22 01:16 CEST
Nmap scan report for 10.129.2.28
Host is up (0.010s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), Linux 2.6.32 - 3.10 (96%), Linux 3.4 - 3.10 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Synology DiskStation Manager 5.2-5644 (94%), Linux 2.6.32 - 2.6.35 (94%), Linux 2.6.32 - 3.5 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 4.11 seconds
```
## Proxy DNS

Por defecto, `Nmap`realiza una resolución DNS inversa a menos que se especifique lo contrario para encontrar información más importante sobre nuestro objetivo. Estas consultas de DNS también se pasan en la mayoría de los casos porque se supone que el servidor web determinado debe encontrarse y visitarse. Las consultas de DNS se realizan a través del archivo `UDP port 53`. Hasta ahora sólo se `TCP port 53`utilizaba para el llamado " `Zone transfers`" entre servidores DNS o para transferencias de datos de más de 512 bytes. Esto está cambiando cada vez más debido a las expansiones de IPv6 y DNSSEC. Estos cambios hacen que muchas solicitudes de DNS se realicen a través del puerto TCP 53.

Sin embargo, `Nmap`todavía nos brinda una manera de especificar los servidores DNS nosotros mismos ( `--dns-server <ns>,<ns>`). Este método podría ser fundamental para nosotros si estamos en una zona desmilitarizada ( `DMZ`). Los servidores DNS de la empresa suelen ser más confiables que los de Internet. Así, por ejemplo, podríamos utilizarlos para interactuar con los hosts de la red interna. Como otro ejemplo, podemos utilizar `TCP port 53`como puerto de origen ( `--source-port`) para nuestros escaneos. Si el administrador usa el firewall para controlar este puerto y no filtra IDS/IPS correctamente, nuestros paquetes TCP serán confiables y se transmitirán.

#### SYN-Scan of a Filtered Port


```c
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 22:50 CEST
SENT (0.0417s) TCP 10.10.14.2:33436 > 10.129.2.28:50000 S ttl=41 id=21939 iplen=44  seq=736533153 win=1024 <mss 1460>
SENT (1.0481s) TCP 10.10.14.2:33437 > 10.129.2.28:50000 S ttl=46 id=6446 iplen=44  seq=736598688 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up.

PORT      STATE    SERVICE
50000/tcp filtered ibm-db2

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
```

#### SYN-Scan From DNS Port

```c
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53

SENT (0.0482s) TCP 10.10.14.2:53 > 10.129.2.28:50000 S ttl=58 id=27470 iplen=44  seq=4003923435 win=1024 <mss 1460>
RCVD (0.0608s) TCP 10.129.2.28:50000 > 10.10.14.2:53 SA ttl=64 id=0 iplen=44  seq=540635485 win=64240 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.013s latency).

PORT      STATE SERVICE
50000/tcp open  ibm-db2
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds
```

```c
ncat -nv --source-port 53 10.129.2.28 50000

Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Connected to 10.129.2.28:50000.
220 ProFTPd
```

