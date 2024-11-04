# Session Hijacking:
El proceso de ingreso o conexión es.

Inicio de conexión ->

<- Acepta conexión

Envia credenciales ->

<- Acepta credenciales

Configuración de la sesión ->

<- Configuración de la sesión (Asigna Sessión ID)

Pide información ->

<- Transmite comunicación

<- Transmite comunicación

<- Transmite comunicación

## Principales vulnerabilidades:
- Ausencia de bloqueo de cuentas por uso de sessions id invalidos
- Tiempo indefinido de vida
- Algoritmos debiles de creación de session ID
- Manejo inseguro de sessions id
- Las conexiones TCP/IP son las más vulnerables
- Falta de encriptación de la conexión

## Proceso normal de robo de sesión:
- Sniff: Ataques MITM 
- Monitor: Analizar el trafico
- Session Desynchronization: Romper la conectividad del cliente al servidor.
- Session ID Prediction: Toma control de la sesión.
- Command Injection: Comienza la inyección en base a la sesión.

## Application Level Session Hijacking:

- Session sniffing: Wireshark, SmartSniffer
- Predictable session token: El atacante consigue un alto numero de sessions ID, para poder predecir siguientes ID.
  - Ejemplo, sí tenemos una session que esta basado en la fecha, hora y numero de usuario, podemos predecir cual es el ID asignado a un usuario.
- Man-in-the-middle attack: Similar a sniffing pero un MITMA es cuando directamente nosotros podemos somos quien literal esta en medio, no un oyente sino que la información pasa por nosotros.
- Man-in-the-browser attack: Inyección de codigo en navegadores donde se colecta las peticiones y con ello los tokens enviados.
- Cross-site script attack: Robo de credenciales sin flags de httponly
- Cross-site request forgery attack: Sucede cuando una web permite request provenientes de una redirección enviando peticiones peligrosas como eliminar datos.
- Session replay attack: 
- Session fixation: Inyección del SID, puede ser por varios medios por ejemplo enviar un correo con liga de login y SID en el GET bank.com?ID=1111 haciendo que al ingresar el usuario se logee y el sistema mantenga el SID antes de ser autenticado.
- CRIME Attack: Es un ataque de tipo False/True porque basado en la compresión de protocolos como HTTPS podemos enviar multiples request a una pagina web buscando que responda con un cambio en el tamaño de la compresión cuando un caracter de la cookie coincida.
- Donation Attack: Mismo proceso que Session fixation pero con un SID legitimo, para que un usuario haga peticiones sobre la sesión de otro usuario a la que el atacante ya tiene acceso.
- PetitPotam Hijacking: En este caso a traves del servicio EFSRPC que corre en equipos Windows puede hacersele la petición de autenticación a otro equipo, en este caso haria que sautentique contra nosotros, eso provocaria que sí usa NTLM podamos capturar el hash, tambien existe otro metodo que es por medio de autenticaciones por medio de certificados.

## Network Level:

- Blind Hijacking: Robo de la sesión pero no podemos leer una respuesta al inyectar comandos.
- UDP Hijacking: 
- TCP/IP Hijacking: El robo se da al inteceptar el "numero de la secuencia", y usandolo para suplantar el flujo de la conexión.
- RST Hijacking: Enviar paquete de RST a la maquina cliente, para nosotros asumir la conexión por medio de una IP spoofeada.
- Man-in-the-Middle: Packet Sniffer
- IP Spoofing: Source Routed Packets, se modifica la cabecera del destino del paquete colocando otra.

## Tools:
- Hetty: HTTP Proxy
- Bettercap: Suitcompleta de MITM
- Droidsniff: Android

## Detection:
- Manual method: 
- Automatic method: IDS / IPS

## Medidas de seguridad:
- Protocolos cifrados
- Cookies sobre canales cifrados
- IPSEC: Cifrado usado por VPN para cifrar todo la comunicación a excepto del paquete de origen y destino IP
  - Caracteristicas:
    - port 500 negociación
  - Modos:
    - Transport Mode: No cifra el encabezado donde viene la IP.
    - Tunnel Mode: Cifra hasta el encabezado, reemplazandolo con una IP nueva, solo puede pasar por ruteadores que permitan IPSEC.
  - Arquitectura:
    - Protocolo AH (Encabezado de autenticación): sin cifrado
    - Protocolo ESP (Carga útil de seguridad de encapsulación): cifrado
  - Debilidades:
    - Antes de que la información viaje por la red, no es cifrado por lo que ataques de trojanos que lean el trafico antes de ser enviada por el nodo.


































