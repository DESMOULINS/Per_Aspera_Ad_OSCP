# HACK WEB SERVERS:

## Web server components:
- Document Root: Es el nombre de la carpeta base por ejemplo domain.com estara comunmente en /var/www/html/domain entonces "domain" sera el Document root.
- Server Root: Carpetas y archivos que soportan al servicio, los basicos son -conf (archivos de configuración), -logs (registros), -cgi-bin (carpeta de ejecutables / hoy en dia ya casi no se usa), y -bin la carpeta de el codigo como tal del servicio.
- Virtal document tree: Permite asignar "alias" a el Document Tree para que cuando sea llamado se haga referencia a otra ubicación local o remota.
- Virtual Hosting: Permite tener varias paginas web en un solo servidor web, dependiendo de cual sea el dominio que busquemos en el web server, sera como nos respondera.
- Web proxy: Permite filtrar lo que pase del cliente al server.

## Modelo OSI:
Logica de quien entra en el manejo de los datos en base al modelo OSI.

- Capa 7: La aplicación como tal desarrollada en codigo php, js. etc.
- Capa 5: Web server
- Capa 4: Bases de datos.
- Capa 3: Sistema operativo.

## Ataques basicos:

- DNS Server hijacking: Suplantar el DNS para reenviar el trafico a web site malicioso.
- DNS Amplification: Suplantar la IP, y enviar muchos request para que el DNS los envie a la IP real, haciendo un DDOS. (El DNS debe permitir )
- Directory Transveral: Lectura de archivos confidenciales fuera de los de la aplicación.
- Web site dafecement: Cambiar el contenido de la pagina para mostrar mensajes.
- Web server misconguration: Configuración de web server mal aplicada:
  - Verbose debug/error messages: Mostrar mensajes de error en productivo puede revelar información sensible.
  - Anonymous or default users/passwords.
  - Sample configuration and script files: Lectura de archivos sensibles o leer el codigo fuente de scripts.
  - Remote administration functions: Funciones de adminstración expuestas.
  - Unnecessary services enabled.
  - Misconfigured/default SSL certificates: Certificados ssl mal configurados pueden relevar por ejemplos los subdomains o virtual hostings que responde el server.
  - HTTP Response-Splitting Attack: Insertar saltos de linea en los location o set-cookie para que el web server responda dos veces, la primera se enviara al atacante la siguiente podria enviarse al siguiente en enviar un request la pagina.
  - Web Cache Poisoning Attack: Basicamente es envenenar la cache del web server, para que cuando otro usuario lo visite se envie contenido malicioso.
  - Web Server Password Cracking: Comunmente los web servers tambien publican servicios complementarios, que facilitan su adminsitración, que pueden tambien se atacados.
    - SMTP and FTP servers
    - Web shares
    - SSH tunnels
    - Web form authentication
  - Application:
    - El más complejo y que tendra un capitulo propio.
   
## Metodologia basica:

- Information Gathering
- Webserver Footprinting
- Mirroring Website
- Vulnerability Scanning
- Session Hijacking
- Hacking Webserver Passwords













