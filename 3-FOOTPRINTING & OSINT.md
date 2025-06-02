# Introducción:
Aqui estara todo lo relacionado a reconocimiento footprinting & osint.

## OSINT WEB:
Esto es sumamente importante, por ejemplo, lo más clasico es encontrar la pagina web de desarrollo que no tiene protección o versiones antiguas de las paginas que nos dara información como directorios o paginas ocultas.

### GOOGLE DORKS
El mejor repositorio general por excelencia es:
- https://www.exploit-db.com/google-hacking-database

Pero sí google te empieza a bloquear o solo quieres los basicos:
- https://pentest-tools.com/information-gathering/google-hacking

O por ejemplo estos son ejemplos de eccouncil:

- caché: Este operador permite ver la versión en caché de la página web. 
  - [cache:www.eccouncil.org]- La consulta devuelve la versión en caché de la página web www.eccouncil.org
- allinurl: Este operador restringe los resultados a las páginas que contienen todos los términos de consulta especificados en la URL. 
  - [allinurl: EC-Council career]- La consulta sólo devuelve páginas que contengan las palabras «EC-Council» y «career» en la URL.
- inurl: Este operador restringe los resultados a las páginas que contengan la palabra especificada en la URL 
  - [inurl: copy site:www.eccouncil.org]-La consulta sólo devuelve las páginas del sitio de EC-Council en cuya URL aparezca la palabra «copy».
- allintitle: Este operador restringe los resultados a las páginas que contienen todos los términos de la consulta especificados en el título. 
  - [allintitle: detectar malware]-La consulta sólo devuelve páginas que contengan las palabras «detectar» y «malware» en el título
- inanchor: Este operador restringe los resultados a las páginas que contienen los términos de la consulta especificados en el texto ancla de los enlaces a la página. 
  - [Anti-virus inanchor:Norton]-La consulta sólo devuelve páginas con texto de anclaje en los enlaces a las páginas que contienen la palabra «Norton» y la página que contiene la palabra «Anti-virus».
- allinanchor: Este operador restringe los resultados a las páginas que contienen todos los términos de la consulta especificados en el texto de anclaje de los enlaces a la página. 
  - [allinanchor: mejor proveedor de servicios en la nube]-La consulta devuelve sólo las páginas en las que el texto de anclaje de los enlaces a las páginas contiene las palabras «mejor», «nube», «servicio» y «proveedor».
- enlace: Este operador busca sitios web o páginas que contengan enlaces al sitio web o página especificados. 
  - [link:www.eccouncil.org]-Encuentra páginas que apuntan a la página principal de EC-Council
- relacionados: Este operador muestra sitios web similares o relacionados con la URL especificada. 
  - [related:www.eccouncil.org]-La consulta proporciona la página de resultados del motor de búsqueda de Google con sitios web similares a eccouncil.org
- info: Este operador encuentra información para la página web especificada. 
  - [info:eccouncil.org]-La consulta proporciona información sobre la página de inicio www.eccouncil.org
- ubicación: Este operador encuentra información para una ubicación específica. 
  - [ubicación: EC-Council]-La consulta proporciona resultados basados en el término EC-Council.

### Videos:
Metadata de videos de youtube:
- https://mattw.io/youtube-metadata/

### Descubrimiento de FTP publicos relacionados a la empresa auditada:
- https://www.searchftps.net/

### Buscador de vulnerabilidades de equipos publicos:
- https://www.shodan.io/

### Buscador de información de ip o dominios:
Este es el más importante aqui si o si en la mayoria de las ocasiones vas a encontrar la información más valiosa para los pentest:
- https://search.censys.io/
- https://www.netcraft.com/tools/
- https://centralops.net/co/
- https://www.robtex.com/dns-lookup/google.com

### Información sobre herramientas:
OsintFramework: buscador general de herramientas aunque muchas ya no funcionan o tienen costo.
- https://osintframework.com/

### EMAIL Tracking:

### WHOIS:
Esta información basica dificilmente te dara información interesante, pero... hay unas que te correlacionar donde te muestra paginas similares en base a su información de whois, obteniendo sitios similares o de desarrollo publicados por empresas.

Domaintools: El basico whois
- https://whois.domaintools.com/google.com 

Builtwith: Uno de mis favoritos para encontrar dominios asociados al del alcance.
- https://builtwith.com/relationships/google.com

Whoisology: Otro de los escenciales es buscar el historial y ver que otros dominios pertenecen al mismo propietario.
- https://whoisology.com/amiprecio.com
- https://whoisfreaks.com/tools/dns/history/lookup
- https://virustotal.com/

### DNS:
Este tambien es otro de los puntos más importantes y que es vital porque amplia nuestro punto de vista de el alcance:

- nslookup: el más basico hace un request del dominio, aunque puedes modificarlo especificando el tipo:
```
> set type=a (especificar registros A)
> server 8.8.8.8 (usar dns en especifico)
> set type=cname (especificar registro cname)
```
- dig: comando más completo pero requieres instalarlo de la https://www.isc.org/download/
```
> dig Hostname ANY
> dig DomaiNameHere
> dig @DNS-server-name Hostname
> dig @DNS-server-name IPAddress
> dig @DNS-server-name Hostname|IPAddress type
> dig +trace domain.com
> dig any victim.com @<DNS_IP>
> dig version.bind CHAOS TXT @DNS (Banner grabbing)
> dig -x 192.168.0.2 @<DNS_IP> (Busqueda inversa muy util para encontrar virtual hosting)
```
- dnsrecon: las anteriores son de TI pero esta es ya más de pentesting, dado que permite fuzzear, recuerda cambiar el dns destino siempre, a veces en los pentest podrias apuntar a google por defecto.
```
> dnsrecon -r 127.0.0.0/24 -n <IP_DNS> (Busqueda inversa de ips en el DNS)
> dnsrecon -d <dominio> -t axfr -n <IP_DNS>  (DNS Zone transfer)
> dnsrecon -d <dominio> -t zonewalk -n <IP_DNS>  (DNS Zone walking)
> dnsrecon -d <DOMAIN> -t brt -D subdomains-1000.txt -n <IP_DNS> (Fuzzear subdomains)
> dnsrecon -d <domain> -t std -n <IP_DNS> (DNS enumeración estandar, Muy util sí quieres encontrar puertos de ldap y por ende al DNS sin escanear)
```
- security trails: buscador web de historial dns, muy util para ver pasadas IP, esto sera vital por ejemplo para encontrar la ip detras de wafs.
- virustotal: en la seccion "relations" hay mucha información sobre el historial de ips y dominios, hasta historial de certificados

### SEGMENTOS PUBLICOS:
Esta sección no tenia sentido para mi cuando inicie no la veia util, pero... es util en cajas negras porque usualmente las empresas compran ips publicas por segmentos, por ejemplo compran la ip publica 8.8.8.8 pero la 8.8.8.9 y 8.8.8.10 ir viendo ips alternas puede ir a darnos con ambientes de desarrollo.
- arin.net: puedes encontrar cual es el segmento publico al que corresponde la ip.
  - https://search.arin.net/rdap/?query=18.18.18.18

### Buscador de información sobre personas:

Esto te ayudara principalmente en temas de campañas de phishing, no siempre se usa en pentest.

- https://www.peekyou.com/
- https://www.spokeo.com/
- https://www.beenverified.com/

## OSINT EN DEEP WEB:
No siempre es comun realizarlo pero puedes probar haciendo buscas rapidas de contraseñas filtradas o información que te sea util para ingreso rapido.

The Hidden Wiki: Sitio ONION que funciona como un servicio Wikipedia de sitios web ocultos. 
- http://zqktlwiuavvvqqt4ybvgvi7tyo4hjl5xgfuvpdf6otjiycgwqbym2qad.onion/wiki

Exonerator: Historial de existencia de paginas ONION
- https://metrics.torproject.org/exonerator.html

Ventas de tarjetas:
- http://ymvhtqya23wqpez63gyc3ke4svju3mqsby2awnhd3bk2e65izt7baqad.onion
- http://s57divisqlcjtsyutxjz2ww77vlbwpxgodtijcsrgsuts4js5hnxkhqd.onion

Buscadores:
https://onionlandsearchengine.net/


## OSINT POR TERMINAL:
Esto es diferente al web porque ocuparemos instalar herramienatas especificas.

### Herramientas generales:
TheHarvester: En lo personal siento ya se quedo atras, hoy en día facilmente Google o cualquier buscador detecta tu alto consumo de peticiones y te bloquea, es importante bajar el rate.
```
theHarvester -d eccouncil -l 200 -b linkedin
```
Recon-ng:
```
> recon-ng
> marketplace install all (instalar todos los paquetes extra)
> workspaces create testing
> workspaces list
>> db insert domains
>> show domains
>> modules load brute
>> modules load recon/domains-hosts/brute-hosts
>> goptions set verbosity false
>> info (info del modulo)
>> run
>> show hosts (resumen de host encontrados)
> modules load reporting/html (crea un reporte no lo veo necesario pero puede ayudarte)
>> options set filename result.html
>> run
```
Maltego: Herramienta grafica similar a recon-ng pero con mas poder pero... claro hay que pagar.

BillCipher: Tiene varias herramientas para website o ip.
- https://github.com/bahatiphill/BillCipher

### Información de usuarios:
sherlock: Buscador de usuarios
- https://github.com/sherlock-project/sherlock
```
python3 sherlock satya nadella 
```
UserRecon: Buscador de usuarios
https://github.com/wishihab/userrecon
Shearchfy: aunque en lo personal no me funciono mucho.

### OSINT de paginas web:
Photon:
```
python3 photon.py -u http://www.certifiedhacker.com -l 3 -t 200 --wayback
```
GRecon:
```
python3 grecon.py
```
Domainfy: buscador de información relacionado a palabras clave por ejemplo dominios
```
domainfy -n domain -t all (buscara dominios .com .mx etc asociados a la palabra)
domainfy -n domain.com -t all (buscara dominios .com.com .com.mx etc asociados a la palabra)
```

### Extraer datos de paginas (metadata):
Esta parte puede ser util aunque no siempre regresa información que vayamos a usar, tenemos dos bandos; con costo y siendo sincero en la mayoria de las empresas va a ser muy raro que te autoricen presupuestos para nuevas herramientas así, y hay otros sin costo que pueden ser más utiles aunque con funciones más puntuales.

Web Data Extractor wdepro.exe: Extrae información interna de paginas sobre documentos, imagenes, etc. **Tiene costo**
- http://www.webextractor.com/

Cewl: Hara una wordlist de las palabras más clave de una pagina web, puede ser util para hacer ataques de contraseña, directorios, etc. **Eh notado que sitios con WAF o protección no regresa info**
```
cewl -d 2 -m 5 -w docswords.txt https://www.dominio.com
```

### Clonador de paginas:
Es importante para temas de phishing en pentest solo en temas especificos tal vez para pentest de redes wifi.

HTTrack Web Site Copier:
- https://www.httrack.com/

Cyotek:
https://www.cyotek.com

### Routing:
Encontrar en que segmento te encuentras asi como información general de tu salida a internet es vital para mapear como esta la red a la que te dieron acceso.
- tracert: ayudara principalmente a mapear cual es el recorrido hasta la salida de internet así como conocer que otros gateway y subnets tienes acceso.







