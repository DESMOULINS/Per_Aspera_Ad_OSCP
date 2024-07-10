# ENUMERACIÓN DE SERVICIOS
Esto es crucial a menos de que te pidan solo revisar puertos en especifico, pero fuera de ello lo comun es que tienes que analizar que puertos, tipo de servicio, versiones, etc.

## Enumeración general:
Primero hay que ver que numero de puerto es el que esta abierto, pero antes de eso un poco de contexto.

### Tipo de conexión de puertos:
- Entrada: Son puertos que se abren y se ponen a escuchar, que podemos contactar directamente sino hay ACL.
- Salida: Son puertos que se abren al momento para poder hablar con un puerto de entrada, por ejemplo sí vamos a hablar con servidor web por el 80 el servidor cliente abre uno dinamico 49155 para que 80 sepa a donde va a hablar.

### Tipos de puertos:
- Puertos Estandar:
  - Rango: 0-1023
  - Uso: Estos puertos son reservados y están asignados a servicios y protocolos bien conocidos.
  - Ej: http(80), https(443), ssh(22), ftp(21).

- Puertos Registrados:
  - Rango: 1024-49151
  - Uso: Estos puertos son asignados por la IANA (Internet Assigned Numbers Authority) a aplicaciones específicas que no requieren un puerto en el rango estándar. Aunque no son tan conocidos como los puertos estándar, muchos servicios y aplicaciones utilizan puertos registrados.
  - Ej: Microsoft SQL Server(1433), RDP(3389), MySQL(3306), PostgreSQL(5432).

- Puertos Dinámicos o Privados:
  - Rango: 49152-65535
  - Uso: Estos puertos son utilizados para conexiones temporales o aplicaciones privadas. No están asignados a ningún servicio específico y son comúnmente utilizados para conexiones de cliente a servidor.
  - Ej:
    - Conexiones dinámicas y temporales para aplicaciones de red.
    - Aplicaciones personalizadas que no requieren un puerto registrado.
   
### Identificar estado de puertos:
Los estados de los puertos son varios y dependera mucho de los controles que esten en medio de la comunicación:

- Puerto abierto:
  - Puerto listo para escuchar y abrir la comunicación.
- Puerto filtrado:
  - Es más un dato informacional que nos da nmap para puertos que posiblemente esten abiertos pero detecta que la comunicación no se puede completar porque hay un control.
- Puerto cerrado:
  - Puerto que por un control no permite comunicación, el puerto no esta escuchando o directamente no existe servicio en ese puerto.
 
En la primera etapa lo más facil es revisar los puertos abiertos, los cerrados o filtrados no es un dato primordial.

### Escaneo de estado de puertos:
- Pendiente colocar cada bandera de estado de puerto.

### Identificar puertos no estandar o no reconocibles:
- Pendiente

## Reconocimiento de servicios estandar:
Cada servicio estandar tiene peculiaridades muy comunes de las cuales podemos abusar para obtener información interna.

### NETBIOS:
Conjunto de servicios y una API que permite la comunicación entre aplicaciones en diferentes computadoras dentro de una red local (LAN). Facilita el nombrado, descubrimiento de recursos y administración de sesiones. Aunque ha sido reemplazado en gran medida por protocolos más modernos, sigue siendo utilizado para compartir archivos e impresoras en redes Windows.

- nbstat: permite mapear archivos compartidos.
  - Mapear recursos de una IP.
```
windows> nbstat -a <IP>
```
  - Mapear recursos guardados en cache.
```
windows> nbstat -c
```
  - Mapear recursos del dispositivo actual-
```
windows> net use
windows> net share
```

#### NetBIOS Enumerator:
Herramienta grafica para mapeo de netbios.

#### NMAP:
NMAP tiene scripts tambien.

```
> nmap -sV -v -p 137 --script nbstat.nse <ip>
> nmap -sU -v -p 137 --script nbstat.nse <ip>
```

### SNMP:
Protocolo estándar utilizado para gestionar y monitorizar dispositivos de red como routers, switches, servidores e impresoras. Facilita la recolección de información y la configuración remota de estos dispositivos. Es ampliamente utilizado para mantener el rendimiento, detectar problemas y asegurar el buen funcionamiento de las redes.
Lo interesante de este protocolo es que usualmente tienen contraseñas por defecto, y nos dan información interesante sobre usuarios, puertos, otras ips, etc. Sí tenemos permisos de escritura podemos incluso realizar un RCE.

#### NMAP:
El primer paso es mediante un escaneo por udp, donde descubras que el puerto 161/162 esta abierto.

#### ONESIXTYONE:
El segundo paso es la fuerza bruta para encontrar communitys debiles, nota: usa listas de community más largas o personalizadas sino funciona la comun.
```
onesixtyone -c /usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt <IP>
```

#### SNMP CHECK
Ya que tengas la contraseñas puedes validar que puedes conectarte y hacer una revisión rapida de que información trae.
```
snmp-check [DIR_IP] -p [PORT] -c [COMM_STRING]
```

#### SNMPBULK:
Tambien puedes dumpear toda la información sin que lo parsee en automatico la herramienta.
```
snmpbulkwalk -c [COMM_STRING] -v [VERSION] [IP] .
snmpwalk -v [VERSION_SNMP] -c [COMM_STRING] [DIR_IP] .1
```

### LDAP:
En pocas palabras es el servicio de directorio que guarda usuarios, equipos y accesos en formato similar a una base de datos dentro de los directorios activos u otros sistemas alternos por ejemplo redhat.

#### LDAPSEARCH Y NMAP:
Uno de los primeros pasos es que sino tenemos usuarios validos podemos probar con un usuario anonimo, en sistemas antiguos y linux es comun nos regrese información.
```
ldapsearch -H ldaps://addomain.home:636/ -x -s base -b '' "(objectClass=*)" "*" +
```

#### ADexplorer:
Herramienta de sysinternals que provee entorno grafico para visualizar información de un servicio de ldap.

#### 



























