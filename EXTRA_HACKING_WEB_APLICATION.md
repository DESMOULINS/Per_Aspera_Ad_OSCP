# HACKING WEB APPLICATION:

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













