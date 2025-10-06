
# Reflected XSS into HTML context with nothing encoded

Para es te laboratorio se tiene un input de búsqueda, en el que te representa lo que ingresas

![20240806112006.png](20240806112006.png)

En la url podemos ver que hace uso de `/?search=` para filtrar, por lo que haremos uso de esto para explotar la vulnerabilidad.

```c
/?search=<marquee>hello+my+friend<%2Fmarquee>
```

vemos que nos representa como código html y ahora probaremos con código JavaScript.

```c
<script>alert('seven')</script>
```

