
# DOM XSS in jQuery anchor `href` attribute sink using `location.search` source

En este caso tenemos que explotar `href`. Nos dirigimos al apartado de `submit feedback`

![20240806151348.png](20240806151348.png)

En esta pestaña tenemos un formulario

![20240806135133.png](20240806135133.png)

Pero lo que nos interesa es la url `returnPath=`, si observamos el código podemos ver que tenemos la etiqueta `<a id="backLink" href="xss">back</a>` por lo que tenemos una vulnerabilidad de xss

![20240806151451.png](20240806151451.png)

Ahora ejecutamos código JavaScript

```c
javascript:alert(document.cookie)
```

Entonces enviamos la petición

![20240806151616.png](20240806151616.png)

