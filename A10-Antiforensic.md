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
```

### Scripts de borrado de logs:
Tambien se puede realizar mediante un powershell, pero hacerlo por este medio vale la pena personalizarlo para que no borre todo, porque ademas es más facil que el antimalware te lo detecte.
- https://gist.github.com/amnweb/0ddb3fdb7ec5da0f0c39f0e9aa9fdd56

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

### Ocultar usuarios:
Importantante paso tambien sí no queremos ser muy obvios cuando agregues usuarios para persistencia.
```
net user Test /add
net user Test /active:yes
net user Test /active:no (no lo mostrara en los tipicos lugares como al inciar sesión)
```

### Borrar artefactos:
Un punto tambien importante es borrar de forma segura los artefactos que usemos, por ejemplo, en los pentest no forma parte del proyecto dejar artefactos o tareas programadas, es peligroso y poco etico dejarlas dentro de los equipos, no dejan de ser herramientas que en otro contexto un atacante real puede abusar de ellas.

```
cipher /w:c/Users/shells (borrado de seguro de archivos, primero da una pasada con 0x00 despues con 0xFF y al final con valores random)
```

## Linux
En este sistema operativo es un poco diferente porque los logs comunmente no esta centralizados como en windows con "win events", aqui usualmente no encontraremos con ficheros individuales.

### History:
La forma más rapida de saber el historial de comandos es con el comando history, el cual podemos evitar su registro con:

```
history -w (borrado del history)
shred ~/.bash_history (rellena el archivo con caracteres random, haciendo un "borrado" seguro del archivo history) * no siempre es el mismo depende del tipo de /bin/*sh que uses.
shred ~/.bash_history && cat /dev/null > .bash_history && history -c && exit (combinación de varias tecnicas)
```

### Ocultar archivos:
En linux es un poco más complicado porque es tipico los usuarios tengan más experiencia, pero para ocultarlos simplemente agrega un "." al inicio del nombre del fichero.
```
> touch .secret.txt
```













