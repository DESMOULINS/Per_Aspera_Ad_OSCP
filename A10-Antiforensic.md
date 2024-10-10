# ANTIFORENSICS:
Son tecnicas para evadir la detección y escenarios donde se realicen auditorias a los logs de los sistemas donde ingresamos.

No es necesariamente estricto realizarlo porque el cliente nos va a pedir que expliquemos todo lo que realizamos, y esto puede ser un problema porque no solo borrariamos lo de nosotros sino que podriamos borrar otros logs de otros incidentes.

**NOTA: Lo que si es reglamentario es borrar TODOS los exploits, malware, registros maliciosos, usuarios, etc. que hayamos creado en las maquinas.**

## Windows:

### WIN Events:
Principales categorias.
- SECEVENT.EVT (security): failed logins, accessing files without privileges
- SYSEVENT.EVT (system): driver failure, things not operating correctly
- APPEVENT.EVT (applications events)

Otro modulo es el de "Registros de Aplicaciones y Servicios" aqui estan los eventos especificos de software instalado por ejemplo powershell, openssh, eventos del cliente VPN, antimalware, etc.

### Evitar que se registren los eventos.
Podemos aparte de borrar evitar que se guarden los logs por un periodo, esto puede ser menos invasivo que borrar.
```
> auditpol /get /category:*
> auditpol /clear /y
> auditpol /get /category:* /success:disable /failure:disable
> auditpol /get /category:"system","account logon" /success:disable /failure:disable
> auditpol /get /category:"sistema" /success:disable /failure:disable
```

Aún asi veremos que esto genere un evento de modificación de logs, el evento es el codigo 4719.

### Borrado de eventos:
```
> wevtutil el (listar categorias actuales)
> wevtutil cl <categoria a borrar> (como siempre la más importante es la de security)
```
```
powershell> Clear-EventLog –LogName Security,System
powershell> Clear-EventLog “Windows PowerShell”
powershell> Clear-EventLog -LogName ODiag, OSession -ComputerName localhost, Server02
```

### Scripts de borrado de logs:
Tambien se puede realizar mediante un powershell, pero hacerlo por este medio vale la pena personalizarlo para que no borre todo, porque ademas es más facil que el antimalware te lo detecte.
- https://gist.github.com/amnweb/0ddb3fdb7ec5da0f0c39f0e9aa9fdd56

### Meterpreter:
```
meterpreter> clearev
```

### Otros tipos de metodos para borrado de logs:
Incluso podemos bypassear el borrado con herramientas tipicas de limpieza de equipos, como con cclenear.

- DBAN (https://dban.org) 
- Privacy Eraser (https://www.cybertronsoft.com)
- Wipe (https://privacyroot.com)
- BleachBit (https://www.bleachbit.org

### Ocultar artefactos:
Sí vamos a dejar malware en carpetas que los usuarios vean tambien no es lo ideal que sea visibles.
```
attrib +h +s +r Test (ocultar la carpeta test)
attrib -h -s -r Test (mostrar la carpeta test)
```

### Modificar fecha de modificación:
Reducir horas en la modificación de archivos, util para evadir analisis de herramientas forenses, **claro** borra el evento de powershell...
```
powershell -Command "(Get-Item $File_name).LastWriteTime = $(Get-Date).AddHours(-10)"
```

### Ocultar usuarios:
Importantante paso tambien sí no queremos ser muy obvios cuando agregues usuarios para persistencia.
```
net user Test /add
net user Test /active:yes
net user Test /active:no (no lo mostrara en los tipicos lugares como al inciar sesión)
```

### Borrado seguro:
Un punto tambien importante es borrar de forma segura los artefactos que usemos, por ejemplo, en los pentest no forma parte del proyecto dejar artefactos o tareas programadas, es peligroso y poco etico dejarlas dentro de los equipos, no dejan de ser herramientas que en otro contexto un atacante real puede abusar de ellas.

```
cipher /w:C\temp\shells (borrado de seguro de archivos, primero da una pasada con 0x00 despues con 0xFF y al final con valores random)
```

### Windows Artifacts:
No solo los logs permiten dar seguimiento, windows tiene ficheros que se usan para optimización y otras tareas, y que tambien pueden ser usados para identificar que fue lo que se hizo en un equipo:

#### Timestamp:
Esto deshabilitara la función de windows de registrar el ultimo acceso o modificación de ficheros.
```
fsutil behavior set disablelastaccess 1
```

#### Hibernación:
El fichero hiberfil.sys contiene información de sistema para la hibernación, por lo que puede ser usado con fines forenses, para deshabilitarlo:
```
reg> Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power > HibernateEnabledDefault > SET DWORD (32-bit) = 0
cmd> powercfg.exe /hibernate off
```

#### Windows Virtual Memory: (Paging File)
Archivo de memoria virtual que se usa cuando la RAM fisica se llena.

Control Panel > System and Security > System > Advanced system settings > Settings > Performance > Advanced > Virtual Memory > Automatically manage paging file size for all drives (uncheck)

#### Puntos de restauración:
Control Panel > System and Security > System > System protection > Configure > Disable system protection

#### Thumbnail cache:
Windows guarda "miniaturas" de archivos como cache, dejando registro de su uso en el archivo thumbs.db

Windows + R > gpedit.msc > User Configuration > Administrative Templates > Windows Components > File Explorer > Turn off the caching of thumbnails in hidden thumbs.db files (Enabled)

#### Prefetch:
Windows para hacer más rapido la apertura de ejecutables guarda datos de aplicaciones, este es un muy buen lugar para encontrar ejecutables de los que se haya borrado WinEvents.

Windows + R > services.msc > SysMain (Superfetch) >  SysMain Properties (Local Computer) >  SysMain Properties (Local Computer) (Disabled)

## Linux
En este sistema operativo es un poco diferente porque los logs comunmente no esta centralizados como en windows con "win events", aqui usualmente no encontraremos con ficheros individuales.

### History:
La forma más rapida de saber el historial de comandos es con el comando history, el cual podemos evitar su registro con:

#### Deshabilitar:
```
> export HISTSIZE=0
```

#### Borrado:
```
> history -w (borrado del history de la sesión actual)
> history -c (borrado de todo el history)
> shred ~/.bash_history (rellena el archivo con caracteres random, haciendo un "borrado" seguro del archivo history) * no siempre es el mismo depende del  tipo de /bin/*sh que uses.
> shred ~/.bash_history && cat /dev/null > .bash_history && history -c && exit (combinación de varias tecnicas)
```

### Modificar fecha de modificación y acceso:
```
touch -a -d '<date> <time>' $File_name (acceso)
touch -m -d '<date> <time>' $File_name (modificación)
```

### /Var/log
Carpeta donde normalmente se guardan los logs de los servicios y software instalado.

Examples:
- /var/log/apache2/access.log
- /var/log/postgresql-#-main.log.#

### Ocultar archivos:
En linux es un poco más complicado porque es tipico los usuarios tengan más experiencia, pero para ocultarlos simplemente agrega un "." al inicio del nombre del fichero.
```
> touch .secret.txt
```













