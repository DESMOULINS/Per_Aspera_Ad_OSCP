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

## Honeypot:
### Tipos:
- Low interaction: Servicios limitados y simulados.
- Medium Interaction: Se simula el sistema operativo y servicios.
- High Interaction: Se simula por completo el sistema y servicios.
- Pure: Se simula por completo el ambiente de producto.

### Herramientas:
- KFSensor:
- HoneyBOT:

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
















