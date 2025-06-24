# Rutas estaticas

![20240711142743.png](20240711142743.png)

En esta practica configuraremos el Firewall de ambos sites para que tenga conectividad.

Los conectamos en el puerto numero 5. 

## SITE A
![20240711144600.png](20240711144600.png)

![20240711145338.png](20240711145338.png)

## SITE B

![20240711145819.png](20240711145819.png)

![20240711145554.png](20240711145554.png)

Luego de configurarlo y probamos aun no tendremos conectividad entre las PC.


![20240711150323.png](20240711150323.png)

Teniendo las direcciones IPs que se muestran en la figura, realizaremos ping de ambos lados.

### Pc SITE A

![20240711150715.png](20240711150715.png)

### PC SITE B

![20240711150708.png](20240711150708.png)


Podemos ver que no tenemos acceso, esto es debido a que tenemos que crear una política y también una ruta estática.

## Configuración de rutas estáticas

Para el caso de las rutas estaticas debemos recordar el grafico:

![20240711153959.png](20240711153959.png)

Entonces para el caso de configuración del SITE A tenemos que verlo de la siguiente manera: En la opción `Destination` colocaremos a donde queremos llevar que en este caso es la red `10.0.2.0/24` y colocar por que interface podemos llevar `Gateway` (IP `172.21.0.2`) que tiene conectividad con la red que en este caso es por la interface `port5`.

Para el caos de del SITE B es lo inverso, tenemos que verlo de la siguiente manera: En la opción `Destination` colocaremos a donde queremos llevar que en este caso es la red `10.0.1.0/24` y colocar por que interface podemos llevar `Gateway` (IP `172.21.0.1`) que tiene conectividad con la red que en este caso es por la interface `port5`.

### SITE A
Creamos una ruta estática.

![20240711153328.png](20240711153328.png)

Luego configuraremos, en este caso colocamos a donde queremos llegar y cual es el gateway por la que podremos acceder a este. Luego seleccionamos la interface por la que se tendrá acceso a dicha red, que en este caso será `172.21.0.2` 


![20240711153429.png](20240711153429.png)



### SITE B
Creamos una ruta estatica

![20240711151816.png](20240711151816.png)

Ahora ingresamos el destino (`Destination`) a donde queremos acceder o rango de IP, en `Gateway` ingresamos la dirección IP que tiene acceso la IP que ingresamos anteriormente y luego la interface por donde tiene conectividad al otro Firewall  

![20240711154538.png](20240711154538.png)

Pero a pesar de configurarlo no tendremos conectividad, por lo que debemos de configurar las políticas.

## Configuración de políticas
### SITE A
Nos dirigimos a  `Policy & Object > Firewall Policy` y creamos una nueva politica en ella ingresamos los siguiente:

- Le damos un nombre `TO PISO B`, la interface por donde ingresara el trafico `Incoming Interface`.
- Interface por donde saldrá el trafico `Outgoing Interface`.
- Rango o direcciones IP de las cuales se originaran el trafico `Source`.
- Rango o direcciones IP a las cuales ingresara el trafico `Destination`.
- En el apartado de `Schedule` se puede configurar el horario que puede permitir el trafico y en nuestro caso será siempre.
- En `Service` podemos indicarle el servicio o puerto al que queremos acceder en caso nuestro es `ALL`.  

![20240711154924.png](20240711154924.png)

Ahora hacemos lo mismo para recibir el trafico invirtiendo la interface:

![20240711161321.png](20240711161321.png)

### SITE B
Realizamos lo mismos cambiando las interfaces de entrada y salida como en el SITE A.

![20240711161234.png](20240711161234.png)

Lo mismo pero invirtiendo las interface para recibir el trafico.

![20240711161417.png](20240711161417.png)

Luego de guardar, probaremos a realizar ping:

![20240711163532.png](20240711163532.png)

![20240711163606.png](20240711163606.png)

Y tenemos conectividad de ambos lados.