# DDOS

## Categorias:
Volumetrico: ataca a -> Bandwidth (Directamente es acabarse el ancho de banda, medido por **bps** "bites por segundo")
  - Flood: Zombies inundan la conexión de un equipo
  - Amplification: Zombies o el atacante a traves de brodcast inunda la red.
Protocol: ataca a -> Cantidad de conexiones permitidas (Tablas ARP, tabla CAM, SYN, etc. Medido por **pps** "paquetes por segundo")
Application: ataca a -> Recursos que puede manejar (HTTP, SSH, Etc. Medido por **rps** "request por segundo")

## Basic attacks:
- Ping of death: Ping con el tamaño 65,538 bytes arriba del tamaño normado.
- Smurf attack: Ping al brodcast spoofing una ip, haciendo que la respuesta vaya a otro equipo.
- Pulse Wave DDoS Attack: Ataque de trafico por pulsos, dando espacio a "procesar" las respuesta al servidor para que abra las conexiones y les de tiempo de vida.
- Zero day: Abusar de vulnerabilidades de aplicaciones, lo más tipico es con funciones que con ciertos parametros hace que el procesamiento sea muy pesado, y al ser muchos termina crasheando.
- SYN: Inundar de SYN sin cerrar la conexión.
- Fragmentation: Al fragmentar se deben reempaquetar, esto puede causar perdida de performance.
- Slowloris: Envio de trafico sin generar mucho ruido, dejando abiertas las conexiones de HTTP.

## Contramedidas:


















