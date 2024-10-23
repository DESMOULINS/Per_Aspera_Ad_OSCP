# SNIFFING:
El arte de escuchar un medio de comunicación para capturar la información que pase por el.

## Wiretapping:
Es la acción de escuchar llamadas por medio de un tercero sin autorización.

- Usualmente usado por gobiernos para que por medio de recursos legales las empresas de telefonia les entreguen la grabación de la llamadas telefonicas.

## Passive Sniffing:
Tecnicas donde no se envia paquetes para modificar el comportamiento de la red.

### HUB: 
Conectarse a un HUB hara que todo el trafico sea reenviado a todos los puertos, comunmente son dos tipos de acceso:
  - Compromiso de fisico de nodo
  - Malware
 
### SPAN PORT:
Puerto espejo o span port es un puerto dedicado de los switches para hacer que todo el trafico se redireccione a un puerto dedicado a escuchar todo este trafico, aqui usualmente van los equipos tipo NDR, pero claro acceder a uno ya directamente te da para sniffear todo el trafico.

## Active Sniffing:
Tecnicas donde se debe modificar el comportamiento normal del trafico de la red.

### Vulnerable protocols:
Basicamente todos los protocolos que vayan en texto plano son vulnerables:

- Telnet, rlogin
- http
- POP, IMAP, SMTP
- NNTP
- FTP

### Logica de compromiso:
Siempre que comprometamos una capa del modelo OSI, por ejemplo sí la capa comprometida fue la red, significa que todos los que esten encima seran tambien sniffeables.

### MAC Address Attacks:

#### Basic concepts:
- MAC (Medium Access Control) Address:
  - 6 bytes - 48 bits - 12 caracteres hexadecimal: D4-BE-D9-14-C8-29
    - 3 bytes - 24 bits - 6 caracteres hex: Organizationally unique identifier (OUI). D4-BE-D9
    - 3 bytes - 24 bits - 6 caracteres hex: Serial number. 14-C8-29
- CAM (Content Addressable Memory) Table:
  - VLAN - MAC ADD - Type - Learn - Age - Ports
  - **NOTA: Solo los switches manejan tablas CAM, los routers y pc solo manejan tablas ARP.**
  - **NOTA: Cuando un equipo manda un ARP Request, los switches solo sirven como reenviadores de brodcast, en cuanto un equipo responde que el tiene la MAC solicitada se reenvia la respuesta y el switch aprende la MAC Address**
 
#### ARP process:
- 0 - Initial CAM:

|-MAC-|-PORT-|
| AA  | 1    |
| CC  | 3    |

- 1 - IP 1.2 - ARP Request "who has 1.3?"
- 2 - SWITCH - Brodcast the request
- 3 - IP 1.3 - ARP Anwer "I'm 1.3 and my mac add is BB"
- 4 - SWITCH - Learn MAC ADD
  
|-MAC-|-PORT-|
| AA  | 1    |
| BB  | 2    | <- WRITE
| CC  | 3    |

- 5 - IP 1.2 - Send tcp request to 1.3 - BB (Con la mac en una trama "encapsulada")
- 6 - SWITCH - Read MAC ADD BB and his CAM
  
|-MAC-|-PORT-|
| AA  | 1    |
| BB  | 2    | <- HERE!
| CC  | 3    |

- 7 - SWITCH - Send packet to port 2

#### MAC Flooding 
En pocas palabras es llenar la tabla CAM para que el switch funcione como HUB,

- 1 - Enviando multiples "I'm ip 1.3 and my mac add is BB"... "I'm ip 1.3 and my mac add is CC"... 

|-MAC-|-PORT-|
| AA  | 1    |
| BB  | 2    |
| CC  | 2    |

- 2 - Se pone la NIC en modo promiscuo.

#### SWITCH Port steal:
En este caso lo que se hara sera spoofear la mac e ip de la victima enviando ARP Answers diciendo que nosotros somos la IP y MAC victima:

|-MAC-|-PORT-|
| AA  | 1    |
| BB  | 2    |
| BB  | 2    |

Al tener dos MAC iguales cuando llegue la trama el switch simplemente enviara el trafico a ambos puertos, o hara brodcast del trafico.

- Protección:
  - Port Security, permite asociar de forma fija una mac a un puerto, o limitar la cantidad de mac add permitidas dentro de un nodo.
```
cisco> switchport port-security
cisco> switchport port-security maximum 1 vlan access (cantidad maxima de mac add en el puerto)
cisco> switchport port-security violation restrict
cisco> switchport port-security aging time 2
cisco> switchport port-security aging type inactivity
cisco> snmp-server enable traps port-security trap-rate 5
```

### DHCP Attacks:
Los ataques de DHCP estan orientados a controlar la asignación de IP, para bypassear ACL por ejemplo, el ataque más comun es hacerle DOS a un DHCP para despues levantar uno propio.

#### DHCP Process:
El proceso se puede aprender por el acronimo de DORA.
```
DHCP CLIENT -> Discovery       (Brodcast - 255.255.255.255:port 67)              -> DHCP SERVER
  -> QUIEN ME ASIGNA UNA IP?
DHCP CLIENT <- Offers          (Unicast or brodcast - 255.255.255.255:port 68)   <- DHCP SERVER
  <- TE INTERESA LA IP X.X.X.X CON GATEWEY ... DNS... ETC?
DHCP CLIENT -> Request         (Brodcast - 255.255.255.255:port 67)              -> DHCP SERVER
  -> SI ME INTERESA, PUEDO TOMARLA?
DHCP CLIENT <- Acknowledgment  (Unicast or brodcast - 255.255.255.255:port 68)   <- DHCP SERVER
  -> SI
```

#### DHCP Starvation:
Basicamente es realizar un DOS de request de IP con diferentes MAC Address para dejar al servidor DHCP sin IP disponibles para repartir.

Pero previo a esto debemos llenar la tabla CAM con multiples MAC en uno o varios nodos del switch, para que justamente podamos mandar a pedir más IP para las multiples MAC que tenemos.

```
DHCP CLIENT -> Discovery  (Brodcast - 255.255.255.255:port 67)  -> DHCP SERVER
  -> QUIEN ME ASIGNA UNA IP?
DHCP CLIENT -> Discovery  (Brodcast - 255.255.255.255:port 67)  -> DHCP SERVER
  -> QUIEN ME ASIGNA UNA IP?
DHCP CLIENT -> Discovery  (Brodcast - 255.255.255.255:port 67)  -> DHCP SERVER
  -> QUIEN ME ASIGNA UNA IP?
DHCP CLIENT -> Discovery  (Brodcast - 255.255.255.255:port 67)  -> DHCP SERVER
  -> QUIEN ME ASIGNA UNA IP?
```

#### DHCP Rogue:
Aqui es levantar un Servidor DHCP gemelo malicioso, donde las respuestas que debemos vamos a asignar que nosotros somos el gateway, DNS, etc.

- Tools: mitm6, DHCPwn, DHCPig.

#### Mitigaciones:
- port security: limitara que no podamos tener varias mac address y por defecto no podemos pedir más ips.
- DHCP Snooping: Permite asociar VLANs exclusivas de donde deben provenir y enviarse el trafico de DHCP.
```
cisco> ip dhcp snooping → this turns on DHCP snooping
cisco> ip dhcp snooping vlan 4,104 → this configures VLANs to snoop
cisco> ip dhcp snooping trust → this configures interface as trusted
```
  - Otra caracteristica que tiene es que crea una tabla donde relaciona:
    - | IP | MAC ADD | PORT | DURACIÓN DE ARRENDAMIENTO |
    - Esta tabla es usada por otras medidas de seguridad como DAI (Dynamic ARP Inspection), explicado más adelante.

### ARP Spoofing:
Es algo similar a "switch port stealing" pero aqui solo spoofeamos la IP, y la mac address la mantenemos porque no intentamos modificar la tabla CAM, nuestro objetivo es modificar la tabla ARP cache de los dispositivos de la red.

- Esto ocurre aunque no se hayan pedido un "who has?".
- Y se basa en mandar una mayor cantidad de "I'm x.x.x.x" que la ip autentica.

#### Mitigaciones:
El principal metodo es DAI (Dynamic ARP Inspection), por lo que usando la tabla creada con DHCP Snopping vemos sí es una mac address usando la IP asignada por el DHCP.

- Nota: Cuando uses ip estaticas, no es viable usar DAI, su uso esta enfocado para cuando existe un DHCP en la red.
```
cisco> ip arp inspection vlan 10
```

#### Detección:
Veremos una inundación de "I'am" visibles en herramientas de monitoreo de red como:

- Capsa Portable Network Analyzer
- Wireshark
- ARP Antispoofer

### IRDP Spoofing:
Aquí lo que se busca es que por medio del protocolo IRDP (ICMP Route Discovery Protocol) añadir default routs a otros equipos. Estos tipo de request (asignados con prioridad y vida alta) son recibidos y aceptados por windows cuando el equipo atacante esta dentro de la misma subnet, incluso ignorando los emitidos por el DHCP.

- Aquí lo comun es que se use para ataques MITM para que el atacante reciba el trafico y actue como ruteador leyendo el trafico que pase.

#### Mitigación:
Apagar IRDP dado que es un protocolo que no soporta autenticación.


### VLAN Hopping:
Tecnicas para lograr ingresar a una VLAN a la que no tenemos permisos y realizar movimiento lateral, la tecnica se divide en dos:

#### Switch Spoofing:
Aquí lo que se hace es conectar un "rouge switch" que creara una conexión troncal, para que el switch envie el trafico de todas las VLAN.
- Esto solo funcionara sí el puerto tiene está configurado como:
  - dynamic auto
  - dynamic desirable
  - trunk mode

##### Mitigación:
```
cisco> switchport mode access
cisco> switchport mode nonegotiate
```

#### Double Tagging:
- Ataque dirigido a nodos que esten en una vlan nativa, es decir la VLAN 1, estas VLAN son las que se asigna por defecto a nodos que no han sido configurados y aún esta por defecto, sí el nodo ya está en "access mode" y tiene una vlan por ejemplo la 20, el ataque no funcionara.
- Este ataque solo funciona en la VLAN 1 por que elimina el primer tag que indica donde indica que es vlan 1, quedandose solo el segundo paquete, es decir la VLAN que especifiquemos.
 
##### Mitigación:
No usar vlans nativas en los switch, asignando en todos los nodos una.
```
cisco> switchport access vlan 2
cisco> switchport trunk native vlan 999 (modificar la vlan nativa a una random que no se usara)
cisco> vlan dot1q tag native (no eliminar el tag incluso de las vlan nativas)
```

### STP Attack:
Ataques muy especifico y que sino se tiene cuidado hara que toda la red se caiga, por que puede crear bucles. El spanning tree protocol permite que cuando existen dos caminos en los switches se escoja el más optimo basado buscar el camino más corto.

Sí tenemos acceso acceso a un nodo podemos enviar BPDU falsos indicando que ahora nuestro switch por donde debe pasar el trafico, provocando que ahora todo el trafico pase por el switch donde estamos.

##### Mitigación:
1- Limitar que puertos que no son de switches no puedan recibir BPDU
```
cisco> configure terminal
cisco> interface gigabiteethernet-slot/port 
cisco> spanning-tree portfast bpduguard
```
2- No permitir que otro switch se convierta en root:
```
configure terminal
interface gigabiteethernet slot/port 
spanning-tree guard root
```
3- Loop guard:
```
configure terminal
interface gigabiteethernet slot/port 
spanning-tree guard loop
```
4- UDLD (Unidirectional Link Detection):
```
configure terminal
interface gigabiteethernet slot/port
udld { enable | disable | aggressive }
```

### DNS Poisoning:

#### Intranet DNS Spoofing
Por medio de un ataque de suplantación se duplica la IP del servidor DNS original, por ejemplo por medio de ARP Poisoning, haciendo que el atacante levante un servidor DNS Falso para responder con resoluciones maliciosas.

#### Internet DNS Spoofing
Aquí el ataque es modificando el DNS por defecto del equipo, ya sea por malware o un ataque de DHCP rouge.

#### Proxy Server DNS Poisoning
Lo que se hace es justamente abusar de caracteristicas de proxy, inyectando un proxy malicioso por ejemplo en edge por medio de malware.

#### DNS Cache Poisoning
Los DNS locales tienen una tabla de DNS Cache, esta sirve para almacenar temporalmente los nombres de dominio y su resolución, para no tener que estar consultando a un dns autoritativo a cada rato, pero como DNS va sobre UDP, significa que no establece la conexión sino que simplemente espera respuesta.

- Esto quiere decir que sí logramos ponernos en medio del DNS local y del DNS autoritativo, podremos enviarle resoluciones maliciosas al DNS local.
  - Para estos hay 3 formas:
    - Suplantar la IP del dns autoritativo.
    - Inundar con respuestas al DNS local, con multiples Transactions ID para buscar atinarle y que el DNS local acepte la respuesta.
   
Este ataque es un poco complejo porque solo dominios muy comunes es facil saber que se pueden estar pidiendo de forma constante.

##### Herramientas:
DerpNSpoof
DNS Spoof (https://github.com)
DNS-poison (https://github.com)
Ettercap (https://www.ettercap-project.org)
Evilgrade (https://github.com)
DNS Poisoning Tool (https://github.com)

### Detección de sniffing:
- Buscar dispositivos en modo promiscuo, se pueden dejar delatar porque:
  - ARP Method:
    - A todas las preguntas de "Who is" por ARP siempre responderan, no importa sí es una ip random.
    - Hablando de SO especificos, al usar ff:00:00:00:00:00 de MAC ADD en el request de ARP y ser recibida por el atacante la procesara y respondera el PING, pero cualquier otra NIC en modo normal buscara saber a quien pertenece la MAC ADD enviando un ARP Request.
  - Ping Method:
    - Sí formamos un paquete con la IP correcta del dispositivo pero ponemos mal la MAC ADD siempre responderan, esto no es normal en tarjetas normales.
la descartara.
  - DNS Method:
    - La mayoria de las nic en modo promiscuo hacen reverse lookup por cada petición que reciben.
 
- Implementar IDS:
  - Un IDS podria detectar que un equipo esta cambiando su MAC constantemente.

#### Herramientas de detección:
- nmap --script=sniffer-detect ...
- NetScan Tools Pro
















