# Stored XSS into HTML context with nothing encoded

En este Lab tenemos una base de datos que almacena unos comentarios, si ingresamos código html y le damos a guardar para luego ver que se nos ejecuta como código html.

![20240806113020.png](20240806113020.png)

Ahora probamos código JavaScript

```c
<script>alert('seven')</script>
```

![20240806113223.png](20240806113223.png)

Si recargamos podemos ver que efectivamente se nos ejecuta el código JavaScript.

![20240806113334.png](20240806113334.png)

