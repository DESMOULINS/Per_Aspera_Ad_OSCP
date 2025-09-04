# Introducción:
Aqui estara todo lo relacionado a reconocimiento footprinting & osint.

### EMAIL Tracking:

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







