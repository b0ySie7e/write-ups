---
title: SQL injection UNION attack, determining the number of columns returned by the query - portswigger
date: 2024-08-04
categories: [Write up, portswigger]
tags: [sql injection]      # TAG names should always be lowercase
---

En este laboratorio debemos obtener el numero de columnas.

![20240802112228.png](20240802112228.png)

Luego de probar que el sitio web es vulnerable podemos enumerar el numero de columnas de la query anterior

```c
Gifts' ORDER BY 3--
```

Podemos hacer uso de `order by` de `union select null,null...`

```c
Gifts' union select null,null,null--
```

![20240802112559.png](20240802112559.png)

Observamos que tenemos 3 columnas.