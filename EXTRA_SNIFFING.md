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






















