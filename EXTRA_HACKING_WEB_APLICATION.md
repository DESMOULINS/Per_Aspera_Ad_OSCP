# HACKING WEB APPLICATION:

## Web Service Architecture:
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

## Tipos de request de API a BD:
CRUD Api y REST Api usan los tipos de acciones a realizar sobre la BD en base del tipo de Request por ejemplo:


| Operation	| HTTP Method	| Description |
-----------------------------------------
| Create | POST	| Adds the specified data to the database table |
| Read | GET	| Reads the specified entity from the database table |
| Update	| PUT	| Updates the data of the specified database table |
| Delete	| DELETE	| Removes the specified row from the database table |

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













