---
title: HostPortDiscovery - Script
date: 2023-03-13
categories: [Scripts, Linux]
tags: []     # TAG names should always be lowercase
---

# HostPortDiscovery

- Al tener una maquina que no podemos enumerar los hosts desde nuestra maquina atacante, esto puede llegar a ser un problema en la intrusión y el reconocimiento de las redes internas, por lo que este script nos puede ayudar en este inconveniente. Se usaron dos conceptos para realizar el script.

- La primera es usada para la enumeración de hosts y la segunda para la enumeración de puertos

## Escaneo de Host 

- Se uso de `ping` para obtener los hosts activos. Lo normal es usar `ping -c 1 10.10.10.10`. pero en este caso la `ip` lo almacenamos en una variable que luego lo incluimos, para luego redirigir el output al `/dev/null` y no tener outputs durante la ejecución de ping, por ultimo se concadena con un `&&` esto hace referencia a un `and` en un lenguaje de programación. Entonces si la ejecución del primero no es exitosa(dando referencia a la ejecución de ping) no se imprimirá el mensaje de la segunda parte en la consola. 

```bash
timeout 1 bash -c "ping -c 1 $ip &>/dev/null" && printf "[✓] $ip Host active \n" #& #threads
```

## Escaneo de Puertos

- Para la enumeracion de puertos, vamos uso del recurso de /dev/tcp que aun que en la consola nos dirá que no exista, este si existe (o almenos entiendandolo asi, que esto tiene otra explicación pero ese no es la finalidad de este post)  

Si nosotros enviamos un comentario vacio a la ruta `/dev/tcp/192.168.98.1/53` con la dirección ip y el puerto, no tendremos ningun ouput si el puerto esta activo, pero podremos ver el codigo de estado. El codigo de estado indica si lo que se ha ejecutado se llevó acabo correctamente, esto nos servira para saber si el puerto esta o no abierto.

- Ejemplo de cuando el puerto `53` esta activo 

```bash
❯ echo '' > /dev/tcp/192.168.98.1/53
❯ echo $?
❯ 0
```

- Ejemplo de cuando el puerto `577` no esta activo

```bash
❯ echo '' > /dev/tcp/192.168.98.1/577
bash: connect: Conexión rehusada
bash: /dev/tcp/192.168.98.1/53444: Conexión rehusada
❯ echo $?
❯ 
```

Una vez entendido el concepto, podemos agregar `2>/dev/null` para redirigir el output del stderr(Los mesajes de los errores).
Entonces tendremos las dos formas:
1. Cuando el puerto no esta activo
2. El puerto esta activo

### Puerto Activo

```bash
❯ echo "" 2>/dev/null >/dev/tcp/192.168.98.1/5344 && echo " [+] Activo"
❯
```

### Puerto no Activo

```bash
❯ echo "" 2>/dev/null >/dev/tcp/192.168.98.1/53 && echo " [+] Activo"
❯ [+] Activo
```

## Otras Funciones

### Funcion Ctrl C

- La función Ctrl C realiza una salida forzada del programa cuando el usuario que este ejecutando presione la tecla `ctrl c`

```bash
function ctrl_c(){
    printf "\n\n ${PRed}[*] Going out ...\n"
    tput cnorm; exit 1
}

trap ctrl_c INT

```

### Función para la validación de la IP

- Esta función ayuda con la validación de la IP para no tener inconvenientes en la ejecucion del programa y tener el control del input

```bash
function def_validIp () {
    local  ip=$1
    local  stat=1
    if [[ $ip =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]];then
        OIFS=$IFS
        IFS='.'
        ip=($ip)
        IFS=$OIFS
        [[ ${ip[0]} -le 255 && ${ip[1]} -le 255  && ${ip[2]} -le 255 && ${ip[3]} -le 255 ]]
        stat=$?
    fi
    return $stat
}
``` 

## Script


```bash
#!/bin/bash
#192.168.98.5

# Colors
PBlack='\033[30m'		# 1
PRed='\033[31m'		    # 2
PGreen='\033[32m'		# 3
POrange='\033[33m'		# 4
PBlue='\033[34m'		# 5
PPurple='\033[35m'		# 6
PCyan='\033[36m'		# 7
PGray='\033[37m'		# 8
PWhite='\e[37m'		    # 9
NC='\033[39m'


function ctrl_c(){
    printf "\n\n ${PRed}[*] Going out ...\n"
    tput cnorm; exit 1
}

trap ctrl_c INT


function def_help(){
    printf "${PGreen}[*] The necesary arguments are:\n"
    printf "\t-H  : Host scanning\n"
    printf "\t-P  : Port scanning\n"
    printf "\t Example: \n"
    printf "\t$0 -H ${PCyan}10.10.10.1-254 \t ${PGreen}Active Host \n" 
    printf "\t$0 -P ${PCyan}10.10.10.10 \t ${PGreen}Active Ports \n"
}

function def_validIp () {
    local  ip=$1
    local  stat=1
    if [[ $ip =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]];then
        OIFS=$IFS
        IFS='.'
        ip=($ip)
        IFS=$OIFS
        [[ ${ip[0]} -le 255 && ${ip[1]} -le 255  && ${ip[2]} -le 255 && ${ip[3]} -le 255 ]]
        stat=$?
    fi
    return $stat
}

#Function hosting scanning
function scanHost (){
    local ipAddress=$1
    local range=$2
    local ipStart=$(echo "$ipAddress"| cut -d "-" -f 1| awk '{print $4}' FS=".")
    local length=${#ipAddress}
    local ipAddress=(${ipAddress::l-2})  
    printf "${PPurple}[*] ${PGreen}Host Enumeration, ${POrange}Waiting\n"  
    for ((i =$ipStart; i<$range+1; i++)); do
        local ip="$ipAddress.$i"
        timeout 1 bash -c "ping -c 1 $ip &>/dev/null" && printf "${PPurple} [✓] ${PBlue} $ip ${POrange} Host active \n" #& #threads
        #wait
    done
}

#Function Port scanning
function scanPort (){
    local ipAddress=$1
    if def_validIp $ipAddress; then
        printf "${PPurple}[*] ${PGreen}Port Enumeration    ${PBlue}$host:\n"
        for ((port=0; port<=65535; port++)); do
            if echo "s7v7n" 2>/dev/null > /dev/tcp/"$ipAddress"/"$port"
            then
                
                printf "${PPurple} [✓] ${PBlue} $port ${POrange} Port Active\n"
            fi
        done
    else
        printf "${PRed}[X] Invalid IP address\n"
        def_help
        exit 1

    fi

}

#Function main
function def_main(){
    if [[ $# -ne 2 ]]; then
        printf "${PRed}[X] Error, wrong arguments\n";
        def_help
        exit 1;
    else 
        if [[ $1 = "-H" ]]; then
            local ipAddress=$(echo $2| cut -d "-" -f 1)
            local ip=$(echo $2| cut -d "-" -f 2)

            if def_validIp $ipAddress; then
                scanHost $ipAddress $ip
            else 
                printf "${PRed}[X] Invalid IP address\n"
                def_help
                exit 1
            fi
        fi
        ##Scan Ports
        if [[ $1 = "-P" ]]; then
            local host=$2
            scanPort $host
        fi
    fi
}

def_main $@
```

## Uso

### Ayuda

```shell
[X] Invalid IP address
[*] The necesary arguments are:
        -H  : Host scanning
        -P  : Port scanning
         Example: 
        ./hostPortDiscovery.sh -H 10.10.10.1-254         Active Host 
        ./hostPortDiscovery.sh -P 10.10.10.10    Active Ports 
```
### Enumeración de Hosts

```shell
❯ ./hostPortDiscovery.sh -H 192.168.98.1-255
[*] Host Enumeration, Waiting
 [✓]  192.168.98.1  Host active 
 [✓]  192.168.98.2  Host active 
 [✓]  192.168.98.3  Host active 
 [✓]  192.168.98.6  Host active 
 [✓]  192.168.98.8  Host active
```
### Enumeración de Puertos

```shell
❯ ./hostPortDiscovery.sh -P 192.168.98.1
[*] Port Enumeration    192.168.98.1:
 [✓]  53  Port Active

```
