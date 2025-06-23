# ACTIVE DIRECTORY STRUCTURE:
```
FOREST:
|INLANEFREIGHT.LOCAL/ (TREE)
|├── ADMIN.INLANEFREIGHT.LOCAL
|│   ├── GPOs
|│   └── OU (Unidades organizacionales)
|│       └── EMPLOYEES
|│           ├── COMPUTERS
|│           │   └── FILE01
|│           ├── GROUPS
|│           │   └── HQ Staff
|│           └── USERS
|│               └── barbara.jones
|├── CORP.INLANEFREIGHT.LOCAL
|└── DEV.INLANEFREIGHT.LOCAL
|
|INLANAMER.LOCAL/ (TREE)
 └── ADMIN.INLANAMER.LOCAL 
```
## Protocolos antiguios relacionados:
- MBROWSE:
 - Servicio antiguo de win 7 y XP, que se usaba cuando no habia AD, para descubrir equipos en la red, su seleccion se daba que entre las computadoras de la red, donde seleccionaban quien iba a ser su MBROWSE simplemente entre la que tuviera más actualizaciones y configuraciones.

## Termonology:
- Schema:
 - Es la estructura de los objetos dentro del AD, user, nombre, etc. esta información no se modifica a menos que sea necesario, como que una app de terceros requiera un campo adicional, Crear atributos personalizados.
- GUID: 
  - Valor de 128-bit que sirve como identificador unico para los objetos creados en el AD, como los users, groups, computers, domain, domain controller.
- Security Identifier (SID): 
  - Valor tipo GUID unico que se asocian al usuario con una lista de permisos, este ID tiene una relación uno a uno, no se puede asignar a varios objetos o viceversa, es unico.
  - Más sin embargo existen los Well-known SIDs que son "genericos" y se usan para temas generales, como:
   - S-1-5-18	    SYSTEM
   - S-1-5-32-544	Administrators (grupo local)
   - S-1-5-11	    Authenticated Users
  - Ejemplo:
   - S-1-5-21-3623811015-3361044348-30300820-1013
    - S: Indica que es un SID.
    - 1: Versión del SID.
    - 5: Autoridad de identificación (5 = NT Authority).
    - 21-...: Dominio o identificador único del emisor.
    - 1013: RID (Relative Identifier), identifica al objeto dentro del dominio.
- DN:
 - Describe la estructura a la pertene un objeto.
 - cn=bjones, ou=IT, ou=Employees, dc=inlanefreight, dc=local
- sAMAccountName
 - Username of the account
- servicePrincipalName:
 - SPN es la cadena de conexión creada por el AD para servicios, permitiendo relacionar un servicio a una cuenta de usuario o de maquina.
 - Por ejemplo, sí yo creo el usuario mysql, a este usuario le puedo asignar un SPN:
  - MSSQLSvc/sql01.corp.local:1433
 - Y al ser un servicio cualquier otro usuario en la red autenticado puede mandar a pedirle un TGS de ese servicio, el cual contiene el hash embebido en el TGS.
 - NOTA: Podrias no tener acceso a ese SPN por que no tienes permisos, por lo que el TGS seria inservible para realizar cambios, pero ya traes contigo el hash del usuario "mysql" porque el TGS lo trae embebido, por eso lo esperado es que la contraseña sea muy compleja para evitar reversing.
- Replication:
 - Proceso de replicacion de los cambios en el AD encargado por el servicio Knowledge Consistency Checker (KCC)
- ACL y ACE:
 - Cada objeto (usuario, equipo, etc.) tiene su propia lista ACL, que puede ser DACL (permisos sobre el objeto) y SACL (auditoria sobre el objeto)
 - Y cada ACL está compuesto por ACE, y las ACE son como tal cada renglon de la lista:
  - UserAccont1
   - DACL:
    - Administrators: Puede eliminar, leer y modificar el objeto. - ACE1
    - Soporte: Puede leer y modificar el objeto. - ACE2
   - SACL:
    - Todos: Auditar el acceso exitoso al objeto - ACE1
    - JPerez: Auditar el acceso no exitoso al objeto. - ACE2
 - FQDN:
  - Nombre del host en relación a un dominio o subdominio.
 - Tombstone y AD Recible Bin:
  - Ambos son contenedores de los objetos eliminados, pero el tombstone viene activado por defecto y solo es lo basico por ejemplo para que no existe SID iguales y su restauración es complicada, el Recicle Bin hay que activarlo y si contiene toda la información para realizar una restauración directa.

## Componentes de un AD:
- NTDS.DIT:
 - Es en si la base de datos que almacena la información de autenticación del AD, sí tiene una configuración de "Store password with reversible encryption" está base de datos tendra tambien las contraseñas en texto plano almacenadas. 
- SysVol:
 - Carpeta publica del AD donde vienen las GPO, scripts de inicio, etc. necesarios para la configuración de clientes, usualmente accesible para cualquier usuario autenticado.
- Objetos: Es lo que permite a algo tener una identidad.
* Leaf object: no puede contener otro objeto.
 - Users: Se le asigna SID y GUID, porque es el objeto que puede llegar a tener más atributos. (leaf object)
 - Contacts: No se le asigna SID, solo GUID e información muy basica para hacer referencias. (leaf object)
 - Printers: No se le asigna SID, solo GUID e información basica como puerto, nombre, driver, etc. (leaf object)
 - Computers: Se le asigna SID y GUID, basado atributos de lo que puede hacer un equipo (leaf object)
 - Shared folders: No se le asigna SID, solo GUID y contiene atributos como nombre, ruta de acceso y permisos.
 - Groups: Contenedor de objetos, incluso in grupo puede contener a otro grupo, tiene un SID y GUID.
 - OU: Es usalmente usado como los principales "sub-contenedores" de un dominio por asi decirlo y puede contener grupos o cualquier otro objeto directamente.
 - Domain: es la raiz de los contenedores, pero tambien almacena los sub-contenedores por defecto, por ejemplo los usarios que no son asignados a un OU o grupo aquí caen.
 - Domain controllers: Es un objeto tipo OU, que contiene a todos los domain controllers como objetos tipo "computer", y aquí se aplican atributos especiales como los FSOM, KDC, etc.
 - Sites: No contienen objetos normales, solo Subnets, controladores de dominio y replicación, se usan para controlar la forma en la que un AD tiene mejor latencia en ciertas redes.
 - Built-in: Contiene los grupos por defecto al momento de la creación de AD.
 - Foreign Security Principals: Objeto especial que almacena las relaciones de confianza con otros forerts.

## Roles y funciones dentro del AD:
- FSMO:
 - Describre la forma y manejo de la información para que no todos tengan el mismo rol dentro de un forest o un tree/dominio, la clave es que solo se permite uno por forest o tree:
  - Forest:
   - Schema Master: Maneja la estructura de los objetos dentro del forest.
   - Domain Name Master: Control la adición o eliminación de dominios dentro de cada dominio.
  - Tree:
   - RID Master: Asigna bloques de RID a los DC del dominio.
   - PDC Emulator: Emula un controlador de dominio principal de NT4, priorizando cambio de contraseñas, autenticación fallida, critico en multiples sitios.
   - Infraestructura Master: Encargado de realizar cambios a objetos de otros dominios.  

- Otros roles o funciones: (Si pueden existir más de uno)
 - Read only DC: usado para localizaciones donde la seguridad no es garantizada, solo teniendo una copia limitada del DC que no permite cambios.
 - KDC: 
 - Global catalog: es un catalogo limitado de todo el bosque, permite que cuando un usuario de otro dominio se autentica permita ejecutar la autenticación.
 - AdminSDHolder: Es un servicio que se ejecuta por defecto cada 30 minutos dentro del PDC emulator, lo que hace es replicar una copia de todas la ACL y sus ACE de usuarios privilegiados a las ACL y ACE actuales del PDC, luego son distribuidas a todos los DC, para prevenir que un DC tenga una configuración anomala o maliciosa.
  - adminCount: Es el atributo de 0 (not protected) o 1 (protected) que define sí un objeto es tomado en cuenta por el servicio de AdminSDHolder. 
 - dsHeuristics: Es una configuración avanzada del Directory Services (servicio general del AD) que enciende o apaga configuraciones especificas de sus funciones como AdminSDHolder, ademas no tiene GUI se configura por command line, por ejemplo:
  - Seguridad:	Permitir herencia de ACLs en cuentas protegidas (adminCount)
  - Auditoría:	Cambiar cómo se almacenan eventos o logs
  - Replicación:	Controlar si ciertos atributos se replican entre DCs
  - Autenticación:	Ajustes en la forma de manejar tokens o búsquedas
  - Compatibilidad:	Ajustes para trabajar con versiones antiguas de AD o LDAP
 - sIDHistory:
  - Es un historial de SID de un usuario, un uso es cuando se migra de dominio para evitar tener que crearle todos los permisos desde 0 en el nuevo SID, por ejemplo, JuanP tiene el SID 1 en google.com, pero se cambio a alto.google.com, se le asigna el SID 2, pero el SID 1 se le asigna al sIDHistory para que mantenga los permisos.
  - Tambien maliciosamente se puede usar, sí somos JuanP y nos colocamos el SID del domainAdmin en el sIDHistory, en teoria ya somos tambien un domainAdmin, y podriamos evitar levantar sospechas por convertirnos directamente en el domain admin.

## Trust:

### Niveles de confianza:
- Parent-child: Confianza clasica entre el dominio y sus subdominios.
- Cross-link:	Relación directa entre subdominios de diferentes dominios.
- External: Relación directa de un subdominio a und dominio de otro arbol.
- Tree-root: Confianza entre dominios del mismo forest.
- Forest: Confianza entre dominios de diferente forest.

### Herencia:
Transitive: La confianza se pasa a los child domain de confianza.
Non-transitive: Solamente el child-domain seleccionado es de confianza

### Dirección:
Bi-direccional: Ambos obtienen los permisos de acceso entre ellos
One-way: Solo uno puede acceder al otro.








