# ESCANEO DE REDES:
Sección primordial para conocer el área de ataque, y sobre todo limitarlo para no tocar sistemas que no se buscara auditar.

## Conceptos basicos y tips:
Antes de revisar servicios siempre es primordial ver que host estan vivos, siempre ten en mente que cada herramienta y tecnica tiene un uso en particular, usar todas las tecnicas nunca es una opción.

Por ejemplo para esta parte dependera el tipo de caja:
- Cajas negras y sin alcance definido:
  - Aqui si es valido el descubrimiento pasivo, encontrar otros host que tengas en tu mismo segmento, encontrar segmentos alcanzables, etc.
- Cajas negras, gris o blancas con alcance definido:
  - Aquí lo del punto anterior es meramente opcional, incluso en la gran mayoria innecesario, no tiene valor porque tu objetivo es solo conocer la seguridad del alcance bien definido (ips o dominios), por eso aquí es solo valido hacer un barrido de icmp para validar que el objetivo que te dieron son alcanzables, no tiene valor hacerle pentest a equipos que no son alcanzables, ese objetivo es el de una prueba de segmentación no de pentesting.
  - Incluso hasta no intentes hacer evasión de firewall en estos casos, el cliente debe darte equipos alcanzables (claro excepto solo este bloqueado icmp y los demas protocolos esten permitidos).
  - Claro puede haber matices, por ejemplo, si puedes descubrir otros dominios o subdominios asociados a la IP del alcances, pero esto va más hacia descubrimiento de servicios a nivel servicios como DNS.

## Tecnicas pasivas
Tecnicas basadas en tocar lo menos posible a los equipos en la red, principalmente util sí no hubo especificación de alcance.
ARP:
Algo tan sencillo como revisar la tabla de ARP de tu maquina puede ser util.
```
linux> arp -e / arp -a
```
```
windows> arp -a
```
Escaneo pasivo:
Netdiscover:
Herramienta muy util para escaneo pasivo, basicamente se pone a escuchar y guarda los origenes de las peticiones arp.
```
linux> sudo netdiscover -i etho -p
```
Wireshark:
En la sección de estadisticas > puntos finales, podras ver tanto las peticiones que se hayan escuchado de protocolos como TPC, UDP, IPV4, IPV6 Y Ethernet.

## Tecnicas activas:

### Host Discovery:
- NMAP SCAN:
Nmap tiene varias formas de descubrimiento:

| Tipo:     |      Descripción           | Flag: |
|-----------|----------------------------|-------|
| ICMP ECHO | Request tipo echo por icmp | -PE   | 
| ICMP TIMESTAMP | Request tipo timestamp por icmp | -PP |
| ICMP ADDRESS MASK | Request tipo address mask por icmp  | -PM |
| TCP SYN PING SCAN | Request tipo SYN a top ports esperando respuesta ACK para marcar up host | -PS |
| TCP ACK PING SCAN | Request tipo ACK a top ports esperando respuesta RST para marcar up host | -PA |
| UDP PING SCAN | Request por UDP a top ports esperando respuesta ICMP Error para marcar up host | -PU |
| IP PING SCAN | Request usando varios protocolos sino se especifica uno, por ejemplo ICMP, IGMP, TCP, UDP o SCTP esperando respuesta de uno para marcar up host | -PO |
| ARP SCAN | Request WHO Has tipo ARP para ver quien responde, recuerda esto solo funciona en tu mismo segmento | -PR |

(*) Cuando se tiene un alcance definido con ICMP ECHO esta bien, los demas son más para cuando no se tiene alcances definidos y hay host ocultos.
(-sn) Podemos colocarla sí queremos asegurarnos que solo se haga escaneo de host y no ningun tipo de escaneo a puertos por ejemplo -PR si realiza escaneo por defecto, y al reves cuando hagamos escaneo de puertos usamos -Pn para evitar hacer validaciones que el host esta activo.

ALTERNATIVAS A NMAP:
En algunas ocasiones tendremos que usar un windows al incio porque no habran terminado de montar la vm con kali que se le pidio al cliente, o detalles puntuales como saber el ttl del icmp hacia varios host, para ello se pueden usar alternativas como:

- Hping3: Similar a nmap
  - https://www.kali.org/tools/hping3/ 
- SX: Similar a nmap 
  - https://github.com/v-byte-cpu/sx 
- Zenmap:
  - Versión de nmap para windows
- Metasploit:
  - Tambien util pero para primeras certs, en oscp y otras certs no te dejaran usarlo.

El CEH te dice varios pero siendo sincero no los vamos a usar porque simplemente son de paga y no es normal se gaste presupuesto en esto:
MegaPing.
NetScanTools Pro.

### OS Discovery:
Cada equipo responde diferente dependiendo del ttl, por ejemplo icmp tiene uno por defecto que claro puede ser modificado pero casi nadie lo hace, tambien los puertos pueden tener ttl, por ejemplo sí vemos diferentes ttls posiblemente sea porque son diferentes equipos quien responde como lo pueden ser en caso de firewalls, balanceadores, contenedores, etc. o incluso que el servicio haya sido modificado, recuerda que esto solo es como referencia.

Por ejemplo, los más comunes son:
- Linux/MAC OS – 64
- Windows – 128
- Cisco Routers – 255
- Solaris/AIX – 265

Pero claro esto puede variar una tabla con una completa referencia es:
- https://gbhackers.com/operating-systems-can-be-detected-using-ping-command/

La otra opción es claro referirnos por los tipicos puertos de cada OS por ejemplo un linux tipicamente tendra servicios de ssh, rpc, etc, un windows tendra servicios de smb, netbios, etc.

No te claves tanto en este punto, solo sirve como contexto.

- Colasoft Packet Builder
En casos muy especificos puedes crear paquetes desde 0, esto puede servir para evasión de firewalls, curiosar y conocer a profundir como se forman los paquetes.

### Evasión de firewalls:
Ahora siendo sincero esto no es primordial, podrias evadir el firewall para saber sí un host o puerto esta abierto, pero... y luego? puedes enviar una conexión completa? lo dudo... o seran muy especificos y dificl, lo que en realidad sera util es por ejemplo en ataques especificos que provengan desde otras ips comprometidas o cuando encuentres una forma de ejecutar comandos desde dentro del servidor por ejemplo un SSRF (Server Side Request Forgery), ahi si podria valer la pena hacer evasión para encontrar sí el servicio existe pero no desde de tu IP, y buscar otro punto donde no se tenga restricciones como los que mencione.
