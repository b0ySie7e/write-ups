# Configuración estática de Firewall

Antes debemos de obtener la dirección de IP de nuestro Firewall:

Para ello debemos ingresar:

```c
get system interfaces physcal
```

![20240710104702.png](20240710104702.png)

Luego vamos a la dirección IP con el navegador:

Para Cambiar en modo grafico hacemos lo siguiente:

Vamos a la opción de `Network`:

![20240710105339.png](20240710105339.png)

Luego seleccionamos con doble click el puerto o interface numero 1.

Luego de seleccionar la interface (port1) tendremos la siguiente figura o algo parecido:

![20240710111022.png](20240710111022.png)

Entonces seleccionamos la opción de manual y escribimos la dirección que deseamos:

![20240710105429.png](20240710105429.png)

Luego guardamos haciendo click en `OK` y habremos guardado.

Ahora hagámoslo desde consola:

Ingresamos los siguientes comandos

```c
config system interface
```

```c
edit port1
```

![20240710111439.png](20240710111439.png)

Actualmente tenemos la siguiente configuración:

![20240710111509.png](20240710111509.png)

Para cambiar en `static` debemos ingresar:

```c
set mode static
```

Luego obtendremos :

![20240710111637.png](20240710111637.png)

si queremos editar la dirección IP podemos realizarlo de la siguiente manera:

```c
set ip 192.168.0.105/24
```

y para guardar ingresamos el comando `end`

![20240710111806.png](20240710111806.png)

