# ESCANEO DE REDES:
Sección primordial para conocer el área de ataque, y sobre todo limitarlo para no tocar sistemas que no se buscara auditar.

## Conceptos basicos y tips:
Antes de revisar servicios siempre es primordial ver que host estan vivos, siempre ten en mente que cada herramienta y tecnica tiene un uso en particular, usar todas las tecnicas nunca es una opción.

Por ejemplo para esta parte dependera el tipo de caja:
- Cajas negras y sin alcance definido:
  - Aqui si es valido el descubrimiento pasivo, encontrar otros host que tengas en tu mismo segmento, encontrar segmentos alcanzables, etc.

- Cajas negras, gris o blancas con alcance definido:
  - Aquí lo del punto anterior es meramente opciónal, incluso en la gran mayoria innecesario, no tiene valor porque tu objetivo es solo conocer la seguridad del alcance bien definido (ips o dominios), por eso aquí es solo valido hacer un barrido de icmp para validar que el objetivo que te dieron son alcanzables, no tiene valor hacerle pentest a equipos que no son alcanzables, ese objetivo es el de una prueba de segmentación no de pentesting.
  - Incluso hasta no intentes hacer evasión de firewall en estos casos, el cliente debe darte equipos alcanzables (claro excepto solo este bloqueado icmp y los demas protocolos esten permitidos).
  - Claro puede haber matices, por ejemplo, si puedes descubrir otros dominios o subdominios asociados a la IP del alcances.

## Tecnicas pasivas
Tecnicas basadas en tocar lo menos posible a los equipos en la red.

ARP:

NMAP SCAN:
Nmap tiene varias formas de descubrimiento:
| Tipo:     |      Descripción       | Flag: |
|-----------|------------------------|-------|
| * ICMP ECHO | Request tipo echo por icmp | -PE | 
| ICMP TIMESTAMP | Request tipo timestamp por icmp | -PP |
| ICMP ADDRESS MASK | Request tipo address mask por icmp  | -PM |
| TCP SYN PING SCAN | Request tipo SYN a top ports esperando respuesta ACK para marcar up host | -PS |
| TCP ACK PING SCAN | Request tipo ACK a top ports esperando respuesta RST para marcar up host | -PA |
| UDP PING SCAN | Request por UDP a top ports esperando respuesta ICMP Error para marcar up host | -PU |
| IP PING SCAN | Request usando varios protocolos sino se especifica uno, por ejemplo ICMP, IGMP, TCP, UDP o SCTP esperando respuesta de uno para marcar up host | -PO |

* Cuando se tiene un alcance definido con este esta bien, los demas son más para cuando no se tiene alcances definidos.
















