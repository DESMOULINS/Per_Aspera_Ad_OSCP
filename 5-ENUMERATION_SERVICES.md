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
> onesixtyone -c /usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt <IP>
```

#### SNMP CHECK
Ya que tengas la contraseñas puedes validar que puedes conectarte y hacer una revisión rapida de que información trae.
```
> snmp-check [DIR_IP] -p [PORT] -c [COMM_STRING]
```

#### SNMPBULK:
Tambien puedes dumpear toda la información sin que lo parsee en automatico la herramienta.
```
> snmpbulkwalk -c [COMM_STRING] -v [VERSION] [IP] .
> snmpwalk -v [VERSION_SNMP] -c [COMM_STRING] [DIR_IP] .1
```

### LDAP:
En pocas palabras es el servicio de directorio que guarda usuarios, equipos y accesos en formato similar a una base de datos dentro de los directorios activos u otros sistemas alternos por ejemplo redhat.

#### LDAPSEARCH:
Uno de los primeros pasos es que sino tenemos usuarios validos podemos probar con un usuario anonimo, en sistemas antiguos y linux es comun nos regrese información.
```
> ldapsearch -H ldaps://addomain.home:636/ -x -s base -b '' "(objectClass=*)" "*" +
```

#### ADexplorer:
Herramienta de sysinternals que provee entorno grafico para visualizar información de un servicio de ldap.

#### NMAP:
Despues de obtener una lista de usuarios ya sea por un acceso anonimo o haberlo obtenido por otro medio podemos realizar un pequeño ataque de fuerza bruta.
```
> nmap -p 389 --script ldap-brute --script-args ldap.base='"cn=users.dc=DOMAIN,dc=com"' <ip>
```

#### Python:
Tambien a traves del modulo de ldap podemos interactuar con el servicio de ldap.
```
> python3
>>> import ldap3
>>> server=ldap3.Server('x.x.x.x',get_info=ldap3.ALL,port=389)
>>> connection=ldap3.Connection(server)
>>> connection.bind()
TRUE
>>> server.info
-----
>>> connection.search(search_base='DC=CEH,DC=com', search_filter='(&(objectclass=*))', search_scope='SUBTREE', attributes='*')
-----
>>> connection.entries
-----
>>> connection.search(search_base='DC=CEH,DC=com', search_filter='(&(objectclass=person))', search_scope='SUBTREE', attributes='userpassword')
-----
```

### NFS:
Tambien es uno de los servicios que podemos encontrar sin autenticación, NFS es en pocas palabras un servicio donde puedes compartir carpetas para que un usuario las pueda montar en su propio sistema

Como segundo dato NFS debera estar asociado a un servicio y puerto de RPC (remote procedure call), que a modo de sobre simplificar, RPC actua como la api donde se reciben las peticiones de sincronización, y el puerto de NFS es unicamente por el que salen y entran archivos.

#### RPCScan:
El primer paso es escanear el puerto de rpc, para detectar los puertos de nfs.
```
> rpc-scan.py x.x.x.x --rpc
```
Despues hay que mapear las carpetas que tiene listas para montar.
```
> nfs-showmount x.x.x.x
> showmount -e x.x.x.x
```

### DNS:
Servicio que sirve como directorio donde correlaciona ips, nombres de dominio, ips de email servers, mapeo de puertos, entre otros.

#### DIG:
Herramienta similar a nslookup pero con más opciones que viene por defecto en linux, para windows debemos instalarla.
```
> dig [type] @[NameServer] [Target Domain] 
> dig ns @dns.com www.domain.com
> dig mx @dns.com www.domain.com
> dig a @dns.com www.domain.com
> dig -x 192.168.0.2 @dns.com (PTR)
> dig version.bind CHAOS TXT @dns.com (versión de dns)
> dig any @dns.com www.domain.com (regresa todos los registros)
> dig axfr @dns.com www.domain.com (transferencia de zona)
```

#### nslookup:
Herramienta que viene por defecto en windows.
```
> nslookup
> set querytype=[type]
> server [NameServer]
> [Target Domain]
> ls -d [Target Domain] (transferencia de zona)
```

#### dnsrecon:
Herramienta más especifica para pentesting especializada en hacer fuzzing de subdominios, dominios, transferencia de zona, etc.
```
> dnsrecon -d [Target Domain] -z (DNSSSEC Zone walking, abusando de dnssec se busca encontrar registros internos)
> dnsrecon -r <SEGMENTO>/24 -n <IP_DNS>
> dnsrecon -t std -d [Target Domain] -n <IP_DNS> (busca toda la información basica del dominio similar a any de dig)
> dnsrecon -t brt -D subdomains-1000.txt -d <DOMAIN> -n <IP_DNS> (fuzzea subdominios)
```

### SMTP:
Servicio de envio de mensajes de e-mail, recordemos este solo es de salida el de entrada es POP3 o IMAP, SMTP puede trabajar sobre envio de datos en claro y cifrado, esto dependera del puerto usado por ejemplo:

- Puerto 25:
  - Este es el puerto tradicional utilizado por SMTP. Se usa principalmente para la comunicación entre servidores de correo. Sin embargo, debido a problemas de spam y restricciones en algunas redes, su uso para clientes de correo salientes ha disminuido.
- Puerto 587:
  - Este puerto es el estándar recomendado para el envío de correo desde un cliente de correo a un servidor de correo. Soporta autenticación y es más seguro que el puerto 25.
- Puerto 465:
  - Este puerto se utilizaba antiguamente para SMTP sobre SSL (SMTPS), pero fue reemplazado por el puerto 587. Sin embargo, todavía es usado por algunos servicios para conexiones seguras.
 
#### NMAP:
Nmap tiene una serie de scripts, lo recomendable es decirle que valide todas por medio de:
```
> nmap -p 25 --script=smtp* x.x.x.x
```
Lo más importante dentro de este servicio es la enumeración de "commands" por que algunos nos permitiran enumerar usuarios, hacer relay (reenvio de correo), etc.

### SAMBA/SMB:
Servicio más comun para compartir archivos a traves de windows, que tambien podemos usarlo para sacar mucha información de sistemas, ademas de ser uno de los más vulnerados.

#### ENUM4LINUX:
```
> enum4linux (OPTION TO SCAN) -u '' -p '' x.x.x.x 
```
- OPTIONS TO SCAN:
  - -U        get userlist
  - -M        get machine list
  - -S        get sharelist
  - -n        Do an nmblookup (similar to nbtstat)
  - -P        get password policy information
  - -G        get group and member list
  - -d        be detailed, applies to -U and -S
  - -o        Get OS information
  - -i        Get printer information
  - -A        All options (-U -S -G -P -r -o -n -i)
  - -u user   specify username to use (default "")  
  - -p pass   specify password to use (default "")
 































